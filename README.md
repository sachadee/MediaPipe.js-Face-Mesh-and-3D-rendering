# MediaPipe.js-Face-Mesh-and-3D-rendering
This project implements a sophisticated real-time facial analysis tool using MediaPipe Face Landmarker. It features 2D face masking (extracting the face from the video), 3D head pose estimation (visualized via a 3D cube), and a liveness detection system (tracking blinks and nods).

This code demonstrates a real-time web application that utilizes **MediaPipe Face Landmarker** to track facial features, perform liveness detection, and render both a 2D masked face and a 3D head-orientation cube.

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
