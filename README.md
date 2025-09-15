## DUM-E: Robotic Arm + Computer Vision + LLM

DUM-E is a small robotic arm system that combines computer vision (homography + object detection), a control server for the arm, and LLM-based reasoning for planning actions from images and user input.

### Repo layout
- **`final/`**: End-to-end task runner with Groq LLM and a speech interface.
- **`backend/`**: WebSocket server that captures a frame, runs CV pipeline, and returns a processed image + object centers.
- **`arm_control_server.py`**: Flask server exposing endpoints to control the robotic arm via inverse kinematics and serial.
- **`arm/`, `Camera/`, `ObjectDetection/`, `Joint/`**: Supporting modules and experiments (CV, detection, joint control, utilities).
- **`ServoControl/`, `ServoTest/`**: Arduino sketches.

See also module docs: [`backend/README.md`](backend/README.md), [`final/README.md`](final/README.md).

### Requirements
- **Python**: 3.10+ recommended
- **OS**: macOS or Linux
- **Hardware**: USB-connected Arduino/arm on a serial port, and a webcam

If using macOS and `PyAudio`, install PortAudio first:
```bash
brew install portaudio
```

### Installation
Create a virtual environment and install dependencies from all modules:
```bash
cd /Users/dganjali/GitHub/DUM-E
python3 -m venv .venv
source .venv/bin/activate
pip install -U pip
pip install -r requirements.txt \
            -r backend/requirements.txt \
            -r arm_requirements.txt \
            -r Joint/requirements.txt
```

### Environment variables
Set your Groq API key (note both casings are used in code; setting both avoids surprises):
```bash
export GROQ_API_KEY="YOUR_KEY_HERE"
export groq_api_key="$GROQ_API_KEY"
```

### How to run
- **Start the Arm Control Server (Flask)**
  ```bash
  python arm_control_server.py
  ```
  - Default host/port: `http://0.0.0.0:5000`
  - Endpoints: `POST /arm_control`, `GET /arm_status`, `GET /health`
  - The serial device path is set in the file. Update it if needed (e.g., `/dev/serial/by-id/...`).
  - Quick test:
    ```bash
    curl -X POST http://localhost:5000/arm_control \
      -H 'Content-Type: application/json' \
      -d '{"action":"move_to_idle"}'
    ```

- **Start the CV WebSocket backend**
  ```bash
  python backend/app.py
  ```
  - WebSocket URL: `ws://localhost:5000`
  - Returns base64 JPEG with boxes and detected object centers. See [`backend/README.md`](backend/README.md) for a minimal client.

- **Run the end-to-end system (speech interface)**
  ```bash
  python final/main.py
  ```
  - The script records speech, transcribes via Groq Whisper, captures a scene with detection, asks the Groq model for the next action, and sends it to the arm server.
  - Ensure the Arm Control Server is running and reachable at the URL configured near the top of `final/main.py`.

### Troubleshooting
- **Camera not found**: Try different camera IDs (0, 1, 2). Close other apps using the camera.
- **Cannot connect to arm**: Verify the serial device path in `arm_control_server.py` and that your user has permission to access the serial port.
- **Groq errors or no response**: Confirm `GROQ_API_KEY`/`groq_api_key` are exported in the same shell as your Python process.
- **PyAudio installation issues (macOS)**: `brew install portaudio`, then reinstall `PyAudio` inside the venv.
- **Module import errors**: Run from the repo root with the virtualenv activated so relative imports resolve.

### Notes
- The `final/` and `backend/` directories each contain additional documentation and may represent slightly different flows (speech vs. simple CLI or WebSocket). Prefer the code in your branch as the source of truth if documentation differs.
- This project requires a real arm connected over USB serial to perform motions. If you do not have hardware, you can still run the CV and LLM planning loops without sending actions.
