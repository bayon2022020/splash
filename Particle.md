// A simple Particle class

class Particle {
  PVector position;
  PVector velocity;
  PVector acceleration;
  int lifespan;

  Particle() {
    acceleration = new PVector();
    velocity = new PVector();
    position = new PVector();
  }

  void activate(int x, int y) {
    acceleration.set(0, 0.05);
    velocity.set(random(-1, 1), random(-2, 0));
    position.set(x, y);
    lifespan = 255;
  }

  void run() {
    update();
    display();
  }

  // Method to update position
  void update() {
    velocity.add(acceleration);
    position.add(velocity);
    lifespan -= 1;
  }

  // Method to display
  void display() {
    stroke(255);
    point(position.x, position.y);
  }

  // Is the particle still useful?
  boolean isDead() {
    return lifespan < 0;
  }
}
