# MediaPipe.js-Face-Mesh-and-3D-rendering
This project implements a sophisticated real-time facial analysis tool using MediaPipe Face Landmarker. It features 2D face masking (extracting the face from the video), 3D head pose estimation (visualized via a 3D cube), and a liveness detection system (tracking blinks and nods).

This code demonstrates a real-time web application that utilizes **MediaPipe Face Landmarker** to track facial features, perform liveness detection, and render both a 2D masked face and a 3D head-orientation cube.




https://github.com/user-attachments/assets/ba629879-0dce-4e3d-b6d8-c2829516a2d0



---
## Key Features

* **Real-time Face Mesh:** Tracks 468+ facial landmarks using the GPU-accelerated MediaPipe model.
* **Dynamic Face Masking:** Uses the `Face Oval` landmarks to create a clipping path, extracting the user's face from the background in real-time.
* **3D Pose Estimation:** Calculates **Yaw, Pitch, and Roll** to project a 3D "nose cube" and orientation vector onto a 2D canvas.
* **Liveness Verification:** Integrated `LivenessDetector` to prevent spoofing by requiring user interaction (blinks/nods) within a session timeout.
* **Three.js Integration:** Includes a `FaceMeshSkinRenderer` for mapping video textures onto 3D face geometries.

## Project Structure

* `index.html`: The main entry point containing the UI layout and core detection logic.
* `LivenessDetector.js`: Logic for validating user movement and blink patterns.
* `FaceMeshSkinRenderer1.js`: Handles the 3D rendering of the face mesh using Three.js.
* `three.module.min.js`: The 3D engine used for the orientation visualization.

## Technical Implementation

### 1. Face Masking (The "Clipping" Technique)

The `drawMaskedFace` function isolates the face by:

1. Mapping specific landmark indices (the "Face Oval").
2. Creating a 2D Canvas path based on those coordinates.
3. Using `ctx.clip()` to ensure only the video pixels within the face boundary are drawn.

### 2. 3D Orientation Logic

The script calculates a local coordinate system ( axes) relative to the face:

* **X-Axis:** Vector between the left and right eye.
* **Y-Axis:** Vector from the forehead to the chin.
* **Z-Axis (Forward):** Calculated via the **Cross Product** of the X and Y axes.

This allows the "Nose Cube" to rotate perfectly with the user's head movements.

### 3. Liveness State Machine

The system tracks the session state:

* **Pending:** Waiting for movement.
* **Verified:** Green "✅ Verified" status appearing after successful blinks/nods.
* **Rejected:** Red overlay appearing if the timeout is reached or movement is invalid.

## Setup & Installation

1. **Dependencies:** Ensure you have the MediaPipe libraries in your `/js` folder or update the imports to use a CDN:
```javascript
import { FaceLandmarker, FilesetResolver, DrawingUtils } from 'https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@latest';

```


2. **Web Server:** Due to security restrictions with `getUserMedia` and ES Modules, you **must** run this via a local web server (e.g., Live Server in VS Code or `python -m http.server`).
3. **Hardware:** A webcam and a browser supporting WebGL/WebAssembly are required.

## Usage

1. Grant camera permissions when prompted.
2. Center your face in the frame.
3. **To Verify:** Follow the on-screen prompt (Blink and Nod).
4. The top canvas displays the mesh and 3D cube, while the bottom canvas displays the isolated "Masked" face.

---


## The  `LivenessDetector` Class Main functions:

### The main Constant
```
  constructor({
      blinkThreshold = 0.18,
      nodThreshold = 15,
      timeout = 8000,
      cooldown = 1000,
    } = {}) {
    this.blinkThreshold = blinkThreshold;
    this.nodThreshold = nodThreshold;
    this.cooldown = cooldown;
    this.timeout = timeout;

    this.lastBlinkTime = 0;
    this.lastNodTime = 0;
    this.livenessConfirmed = false;
    this.noddedDown = false;
    this.sessionStart = null;
    this.rejected = false;
  }

```

### The headpose extraction

```
  computeHeadPose(landmarks) {
    const xAxis = this._normalizeVector(this._vectorBetween(landmarks[263], landmarks[33]));
    const yAxis = this._normalizeVector(this._vectorBetween(landmarks[10], landmarks[152]));
    const zAxis = this._normalizeVector(this._crossProduct(xAxis, yAxis));

    const R = [
      [xAxis.x, yAxis.x, zAxis.x],
      [xAxis.y, yAxis.y, zAxis.y],
      [xAxis.z, yAxis.z, zAxis.z]
    ];

    const yaw = Math.atan2(R[1][0], R[0][0]) * 180 / Math.PI;
    const pitch = Math.atan2(-R[2][0], Math.sqrt(R[2][1] ** 2 + R[2][2] ** 2)) * 180 / Math.PI;
    const roll = Math.atan2(R[2][1], R[2][2]) * 180 / Math.PI;

    return { yaw, pitch, roll };
  }


//Get the Eye landmarks values:

  getEAR(eye) {
    const dy = (a, b) => Math.hypot(eye[a].x - eye[b].x, eye[a].y - eye[b].y);
    const dx = (a, b) => Math.hypot(eye[a].x - eye[b].x, eye[a].y - eye[b].y);

    const vertical = (dy(1, 5) + dy(2, 4)) / 2;
    const horizontal = dx(0, 3);

    return vertical / horizontal;
  }

// ====== Blink Detection ======

  const leftEye = [landmarks[33], landmarks[159], landmarks[158], landmarks[133], landmarks[153], landmarks[145]];
  const rightEye = [landmarks[362], landmarks[386], landmarks[385], landmarks[263], landmarks[380], landmarks[374]];
  const leftEAR = this.getEAR(leftEye);
  const rightEAR = this.getEAR(rightEye);
  const avgEAR = (leftEAR + rightEAR) / 2;

  const blink = avgEAR < this.blinkThreshold && now - this.lastBlinkTime > this.cooldown;
  if (blink) {
    this.lastBlinkTime = now;
    console.log("👁️ Blink detected");
  }

// ====== Nod Detection ======

  let nod = false;
  if (pitch < -this.nodThreshold && !this.noddedDown) {
    this.noddedDown = true;
  }

  if (pitch > 5 && this.noddedDown && now - this.lastNodTime > this.cooldown) {
    nod = true;
    this.noddedDown = false;
    this.lastNodTime = now;
    console.log("🙆 Nod detected");
  }

```
### The Mediapipe 468 mesh Indexes. Just zoom in the image to get the needed index:

 ![MP mesh indexes](https://github.com/sachadee/MediaPipe.js-Face-Mesh-and-3D-rendering/tree/SachaDee/canonical_face_model_uv_visualization.png) 
