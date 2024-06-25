# splash
/**
 * Frame Differencing 
 * by Golan Levin. 
 *
 * Quantify the amount of movement in the video frame using frame-differencing.
 */


import processing.video.*;

int numPixels;
int[] previousFrame;
Capture video;
PImage colorbuf;

ParticleSystem ps;
boolean firstFrameSkipped = false;

void setup() {
  size(1920, 1080);
 //フレームレートをで遅延を改善できるか
 frameRate(240);
  ps = new ParticleSystem(10000);

  // This the default video input, see the GettingStartedCapture 
  // example if it creates an error
  video = new Capture(this, 640, 480, "pipeline:ksvideosrc device-index=1 ! image/jpeg,width=640,height=360 ! jpegdec ! videoconvert");

  // Start capturing the images from the camera
  video.start(); 
  numPixels = video.width * video.height;
  // Create an array to store the previously captured frame
  previousFrame = new int[numPixels];
  
  colorbuf = createImage(video.width, video.height, RGB);
  colorbuf.loadPixels();
  smooth(0);
}

void draw() {
  if (video.available()) {
    // When using video to manipulate the screen, use video.available() and
    // video.read() inside the draw() method so that it's safe to draw to the screen
    video.read(); // Read the new frame from the camera
    video.loadPixels(); // Make its pixels[] array available
    if (!firstFrameSkipped) {
      for (int i = 0; i < numPixels; i++) {
        previousFrame[i] = video.pixels[i];
      }
      firstFrameSkipped = true;
    } else {
      for (int i = 0; i < numPixels; i++) { // For each pixel in the video frame...
        color currColor = video.pixels[i];
        color prevColor = previousFrame[i];
        // Extract the red, green, and blue components from current pixel
        int currR = (currColor >> 16) & 0xFF; // Like red(), but faster
        int currG = (currColor >> 8) & 0xFF;
        int currB = currColor & 0xFF;
        // Extract red, green, and blue components from previous pixel
        int prevR = (prevColor >> 16) & 0xFF;
        int prevG = (prevColor >> 8) & 0xFF;
        int prevB = prevColor & 0xFF;
        // Compute the difference of the red, green, and blue values
        int diffR = abs(currR - prevR);
        int diffG = abs(currG - prevG);
        int diffB = abs(currB - prevB);
        // Add these differences to the running tally
        int diffValue = diffR + diffG + diffB;
        if (diffValue > random(300, 1000)) {
          int x = i % video.width;
          int y = i / video.width;
          ps.addParticle(x * width / video.width, y * height / video.height);
        }
        // Render the difference image to the screen
        colorbuf.pixels[i] = color(diffR, diffG, diffB);
        // The following line is much faster, but more confusing to read
        //pixels[i] = 0xff000000 | (diffR << 16) | (diffG << 8) | diffB;
        // Save the current color into the 'previous' buffer
        previousFrame[i] = currColor;
      }
      colorbuf.updatePixels();
    }
    // To prevent flicker from frames that are all black (no movement),
    // only update the screen if the image has changed.
    image(colorbuf, 0, 0, width, height);
  }
  ps.run();
}
