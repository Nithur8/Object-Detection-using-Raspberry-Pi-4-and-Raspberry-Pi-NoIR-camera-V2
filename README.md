# Object Detection on Raspberry Pi 4 with Raspberry Pi NoIR Camera V2 (YOLO11n)

This repository demonstrates object detection on a Raspberry Pi 4 using the Raspberry Pi NoIR Camera V2. The detection method has been updated from the previous `rpmi detect` method to use the YOLO11n model via the Ultralytics YOLO Python API.

> Note: YOLO11n is expected to be provided as a .pt weight file. This repo does not store the weight file; instead a script will download it at install/run time.

## What changed
- Replaced instructions for the previous detection method with a new YOLO11n-based workflow.
- Added a detection script using the Ultralytics YOLO API that works with a Pi Camera (libcamera / Picamera2) and falls back to a USB webcam (cv2.VideoCapture).
- Added a model download helper script to fetch the YOLO11n weights automatically.
- Added a systemd unit example to run detection on boot.

## Repository contents (added)
- `detect_yolo11n.py` — main detection script using Ultralytics YOLO
- `requirements.txt` — Python dependencies
- `scripts/download_model.sh` — helper to download YOLO11n weights into `models/`
- `run.sh` — convenience wrapper to run detection
- `systemd/yolo11n.service` — example systemd unit to auto-start detection

## Requirements (on Raspberry Pi)
- Raspberry Pi 4 (or equivalent)
- Raspberry Pi NoIR Camera V2 connected and enabled (libcamera / Picamera2 recommended)
- Raspberry Pi OS (Bullseye or later recommended)
- Python 3.8+ with pip
- Optional: GPU/accelerator if you have one (but script runs on CPU)

## Install (recommended)
1. Update system packages:
   sudo apt update && sudo apt upgrade -y

2. Install system packages for camera and OpenCV support:
   sudo apt install -y libatlas-base-dev libjpeg-dev libtiff5-dev libjasper-dev libqtgui4 libqt4-test

3. Install Python dependencies (virtualenv recommended):
   python3 -m venv venv
   source venv/bin/activate
   pip install --upgrade pip
   pip install -r requirements.txt

   Notes:
   - On Raspberry Pi, installing `torch` may be non-trivial. The `ultralytics` package may pull dependencies; if installation of `torch` fails, refer to PyTorch for Raspberry Pi instructions for CPU builds.
   - If using Picamera2, install it via the Raspberry Pi package manager (libcamera-based). Example:
     sudo apt install -y python3-picamera2

4. Download YOLO11n weights:
   - Provide a direct download URL in the environment variable `YOLO11N_URL`, for example:
     export YOLO11N_URL="https://example.com/path/to/yolo11n.pt"
   - Then run:
     bash scripts/download_model.sh

   If you do not set `YOLO11N_URL`, the download script will prompt and fallback to a placeholder sample model (you should replace that with your YOLO11n weight URL).

## Run detection (local)
- Quick run:
  bash run.sh

- Or run directly:
  python3 detect_yolo11n.py --weights models/yolo11n.pt --conf 0.25 --source 0

Options:
- `--weights` Path to the .pt weight file (default: models/yolo11n.pt)
- `--conf` Confidence threshold (default: 0.25)
- `--source` Camera source: `0` for default webcam, or `picam` to use Picamera2 (default: autodetect)

Press `q` in the display window to quit.

## Autostart (systemd)
To run the detector on boot:
1. Copy `systemd/yolo11n.service` to `/etc/systemd/system/yolo11n.service`.
2. Edit `ExecStart` to point to your virtualenv/python and the repo path.
3. Enable and start:
   sudo systemctl enable yolo11n
   sudo systemctl start yolo11n

## Performance & tips
- Use the smallest weight (YOLO11n) for faster inference on CPU.
- Reduce input resolution to increase FPS (the script downsizes for performance).
- Consider converting to a tflite or ONNX variant if you need further speedups or want to run on accelerators.

## Notes
- Do not commit large weight files to the repository. The provided `scripts/download_model.sh` downloads the model at runtime.
- If you want me to include a sample YOLO11n weight, provide a downloadable URL and confirm you want it referenced in the download script.