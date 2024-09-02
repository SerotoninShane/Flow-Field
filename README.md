# p5.js Interactive Visuals
This project contains two interactive p5.js sketches:

[Flow Field Sketch](#flow-field-sketch)
[Transparent Sphere Sketch](#transparent-sphere-sketch)

## Flow Field Sketch

```javascript
window.onresize = () => {
  location.reload();
};

const flowFieldSketch = (sketch) => {
  let points = [];
  let allPoints = [];
  let mult = 0.005;
  let radius = 400;
  let density = 200;
  let space;

  sketch.setup = () => {
    let canvas = sketch.createCanvas(sketch.windowWidth, sketch.windowHeight);
    canvas.parent('flow-field');
    sketch.background(30);
    sketch.angleMode(sketch.DEGREES);
    sketch.noiseDetail(1);

    space = sketch.width / density;

    for (let x = 0; x < sketch.width; x += space) {
      for (let y = 0; y < sketch.height; y += space) {
        let p = sketch.createVector(x + sketch.random(-20, 20), y + sketch.random(-20, 20));
        allPoints.push(p);
      }
    }

    sketch.shuffle(allPoints, true);
  };

  sketch.draw = () => {
    sketch.noStroke();

    // Determine how many points to process this frame
    let max = sketch.min(sketch.frameCount * 20, allPoints.length);

    // Add new points to the 'points' array up to the 'max' limit
    for (let i = points.length; i < max && i < 5000; i++) {
      points.push(allPoints[i]);
    }

    // Process and filter the points that are still inside the circle
    points = points.filter((p, index) => {
      if (index >= max) return true; // Skip processing if beyond the current max limit

      let r = sketch.map(p.x, 0, sketch.width, 255, 255);
      let g = sketch.map(p.y, 0, sketch.height, 10, 100);
      let b = sketch.map(p.x, 0, sketch.width, 0, 50);

      let alpha = sketch.map(sketch.dist(sketch.width / 2, sketch.height / 2, p.x, p.y), 0, 300, 255, 100);

      sketch.fill(r, g, b, alpha);

      let angle = sketch.map(sketch.noise(p.x * mult, p.y * mult), 0, 1, 0, 90);
      p.add(sketch.createVector(sketch.cos(angle), sketch.sin(angle)));

      if (sketch.dist(sketch.width / 2, sketch.height / 2, p.x, p.y) < radius) {
        sketch.ellipse(p.x, p.y, 1);
        return true; // Keep the point in the array
      } else {
        return false; // Remove the point from the array
      }
    });

    // Stop the sketch when no points are left inside the circle and all points have been added
    if (points.length === 0 && max >= allPoints.length) {
      sketch.noLoop();
    }
  };
};
new p5(flowFieldSketch);
```

**Descriptio**n: This sketch creates a flow field animation with particles that move and change color based on Perlin noise. It dynamically updates the particles and visual effects based on the canvas size.

## Transparent Sphere Sketch

```javascript
const transparentSphereSketch = (sketch) => {
  let name;
  let txt;

  sketch.setup = () => {
    let canvas = sketch.createCanvas(sketch.windowWidth, sketch.windowHeight, sketch.WEBGL);
    canvas.parent('transparent-sphere');

    // Create a larger graphics buffer for higher resolution
    let bufferWidth = 2000;  // Increase the width
    let bufferHeight = 1000; // Increase the height
    name = sketch.createGraphics(bufferWidth, bufferHeight);
    name.textAlign(sketch.CENTER, sketch.CENTER);
    name.textSize(110);  // Increase the text size for higher resolution

    let customText = "Shane Bogue";  // Replace this with any text you want

    let outlineWeight = 6;
    name.fill(30,30,30,1);  // Outline color (black)
    for (let x = -outlineWeight; x <= outlineWeight; x++) {
      for (let y = -outlineWeight; y <= outlineWeight; y++) {
        name.text(customText, name.width / 2 + x, name.height / 2 + y);
      }
    }
    
    let shadowOffset = 3;
    name.fill(0,0,0,100);  // Shadow color (black)
    name.text(customText, name.width / 2 + shadowOffset, name.height / 2 + shadowOffset);
    
    name.fill(220,220,220);
    name.text(customText, name.width / 2, name.height / 2);
    txt = name.get();
    name.remove()
  };

  sketch.draw = () => {
    sketch.clear()
    sketch.stroke(255,255,255,10);

    sketch.texture(txt);
    sketch.orbitControl()
    sketch.rotateY(sketch.millis() * 0.0005)

    sketch.push()
    sketch.fill(255,255,255,10)
    sketch.drawingContext.enable(sketch.drawingContext.CULL_FACE)
    sketch.drawingContext.cullFace(sketch.drawingContext.BACK)
    sketch.sphere(360)
    sketch.pop()
    
    // Front face
    sketch.push()
    sketch.drawingContext.enable(sketch.drawingContext.CULL_FACE)
    sketch.drawingContext.cullFace(sketch.drawingContext.FRONT)
    sketch.sphere(360)
    sketch.pop()

    sketch.drawingContext.disable(sketch.drawingContext.CULL_FACE)
  };
};

new p5(transparentSphereSketch);
```

**Description**: This sketch creates a rotating transparent sphere with a custom text texture. The sphere's appearance is managed using WEBGL and includes text with shadow and outline effects for added visual depth.
