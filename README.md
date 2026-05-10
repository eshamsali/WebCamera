#  Remote Control Car — WebRTC

A browser-based remote control car system using **WebRTC** for live video streaming and **Firebase Realtime Database** for signaling and directional commands.

---

##  Project Files

| File | Description |
|------|-------------|
| `camera.html` | Runs on the car — opens the camera and streams video |
| `control.html` | Runs on your phone/laptop — shows the live feed and sends commands |
| `style.css` | Shared styles (legacy, mostly replaced by inline styles) |

---

## How It Works

```
[Camera Device]                    [Controller Device]
  camera.html          Firebase        control.html
      │                   │                │
      │── WebRTC Offer ──▶│──────────────▶│
      │◀─ WebRTC Answer ──│◀──────────────│
      │                   │                │
      │◀══════ Live Video Stream (P2P) ════│
      │                   │                │
      │              car/direction         │
      │◀──────────────────│◀── press() ───│
```

1. **camera.html** opens the device camera and creates a WebRTC offer, uploading it to Firebase.
2. **control.html** reads the offer from Firebase, creates an answer, and completes the WebRTC handshake.
3. Video streams **peer-to-peer** directly between devices (low latency).
4. Direction commands (`forward`, `backward`, `left`, `right`, `stop`) are written to Firebase under `car/direction` — your car firmware reads from there.

---

## Setup & Usage

### 1. Open the Camera Page (on the car device)
Open `camera.html` in a browser on the device attached to the car.  
Click **"Start Stream"** — grant camera permission when prompted.

### 2. Open the Control Page (on your phone/laptop)
Open `control.html` in a browser on any other device.  
Click **"Connect to the car"** — the live video feed will appear once connected.

### 3. Control the Car
Use the **D-pad buttons** on screen, or use your keyboard:

| Key | Action |
|-----|--------|
| `W` / `↑` | Forward |
| `S` / `↓` | Backward |
| `A` / `←` | Left |
| `D` / `→` | Right |
| Release key | Stop |

---

##  Firebase Configuration

Both files share the same Firebase project. The config is already embedded in the HTML — no extra setup needed unless you want to use your own Firebase project.

```js
const firebaseConfig = {
  apiKey: "...",
  databaseURL: "https://webcam-8fc00-default-rtdb.europe-west1.firebasedatabase.app",
  projectId: "webcam-8fc00",
  // ...
};
```

### Firebase Database Structure
```
webrtc/
  offer             ← SDP offer from camera
  answer            ← SDP answer from controller
  offerCandidate    ← ICE candidate from camera
  answerCandidate   ← ICE candidate from controller

car/
  direction         ← "forward" | "backward" | "left" | "right" | "stop"
```

---

##  Connecting Car Hardware

Your car's microcontroller (e.g. ESP8266, ESP32, Arduino) should listen to the Firebase path `car/direction` and drive the motors accordingly.

**Example logic:**
```
if direction == "forward"  → move forward
if direction == "backward" → move backward
if direction == "left"     → turn left
if direction == "right"    → turn right
if direction == "stop"     → stop all motors
```

---

##  Requirements

- Both devices must be on a network that allows WebRTC (most home/mobile networks work fine)
- Camera device must allow camera access in the browser
- HTTPS required for `getUserMedia` — use a local server or hosting service (GitHub Pages, Netlify, etc.)

---

##  Tech Stack

- **WebRTC** — peer-to-peer video streaming
- **Firebase Realtime Database** — signaling + car commands
- **Vanilla JS** — no frameworks needed
