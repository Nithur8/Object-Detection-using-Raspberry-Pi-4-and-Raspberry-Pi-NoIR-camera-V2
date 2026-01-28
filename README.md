# Object-Detection-using-Raspberry-Pi-4-and-Raspberry-Pi-NoIR-camera-V2 and YOLO11n
--- 
## Documentation References

The following official documentation and guides were referred to while learning and working with Raspberry Pi fundamentals:

* **Getting Started**

  * Raspberry Pi OS installation and initial setup
    [https://www.raspberrypi.com/documentation/computers/getting-started.html](https://www.raspberrypi.com/documentation/computers/getting-started.html)

* **Remote Access**

  * Accessing Raspberry Pi headlessly using SSH
    [https://www.raspberrypi.com/documentation/computers/remote-access.html](https://www.raspberrypi.com/documentation/computers/remote-access.html)

* **Network Configuration**

  * Managing Wi-Fi connections and network settings
    [https://www.raspberrypi.com/documentation/computers/configuration.html](https://www.raspberrypi.com/documentation/computers/configuration.html)

These resources were used to understand system setup, remote operation, and network management before implementing object detection.

## Raspberry Pi Fundamentals (Prerequisites)

Before implementing object detection using the **Raspberry Pi 4** and **NoIR Camera V2**, the following Raspberry Pi concepts and techniques were practiced:

* Setting up and operating **Raspberry Pi OS in headless mode**
* Accessing and controlling the Raspberry Pi remotely using **SSH**
* Managing Wi-Fi connections and network configuration via terminal
* Working with the **libcamera** stack and **rpicam-apps**
* Capturing images using **rpicam-still** and verifying camera functionality
* Understanding the build and usage of **rpicam-detect (TensorFlow Lite)**

These fundamentals form the base for implementing and experimenting with real-time object detection on Raspberry Pi.

## Process

Follow these steps on the Raspberry Pi 4 to run the YOLO11n model.

1. Update system packages
```bash
sudo apt update && sudo apt upgrade -y
```

2. Create project folder and a Python virtual environment, then activate it
```bash
mkdir -p ~/YOLO_Detect
cd ~/YOLO_Detect

python3 -m venv venv
source venv/bin/activate

pip install --upgrade pip setuptools wheel
```

3. Install Ultralytics and common Python dependencies
```bash
pip install ultralytics opencv-python-headless numpy onnx onnxruntime
```

4. Plug in the NoIR Camera V2 and verify it works
```bash
# quick preview
libcamera-hello -t 2000

# capture a still image to confirm
libcamera-still -o test.jpg
ls -l test.jpg
```

5. Download the YOLO11n PyTorch model (.pt)
```bash
mkdir -p models
`yolo detect predict model=yolo11n.pt`
```

6. Export the Ultralytics .pt model to ONNX (required before converting to ncnn)
```bash
python - <<'PY'
from ultralytics import YOLO
YOLO("models/yolo11n.pt").export(format="onnx")
PY
# resulting file: models/yolo11n.onnx
```

7. Build ncnn tools (onnx2ncnn / ncnnoptimize) and convert ONNX → ncnn
```bash
# install build tools
sudo apt install -y git build-essential cmake

# clone and build ncnn (can be slow on Pi)
git clone --depth=1 https://github.com/Tencent/ncnn.git
cd ncnn
mkdir build && cd build
cmake .. -DNCNN_VULKAN=OFF
make -j4

# convert ONNX to ncnn format (run from ncnn/build/tools/onnx or adjust path)
./onnx2ncnn ../../YOLO_Detect/models/yolo11n.onnx ../../YOLO_Detect/models/yolo11n.param ../../YOLO_Detect/models/yolo11n.bin

# optional optimize step
./ncnnoptimize ../../YOLO_Detect/models/yolo11n.param ../../YOLO_Detect/models/yolo11n.bin ../../YOLO_Detect/models/yolo11n-opt.param ../../YOLO_Detect/models/yolo11n-opt.bin 65536
```

8. Create a simple Python detect script (Ultralytics .pt runtime)
```bash
cat > detect.py <<'PY'
from ultralytics import YOLO

model = YOLO("models/yolo11n.pt")
# use source=0 for the first camera; adjust conf and other args as needed
model.predict(source=0, conf=0.25, show=True, save=False)
PY
```

9. Run the model (Ultralytics .pt)
```bash
source venv/bin/activate
python detect.py
```

10. (If using ncnn runtime) Use the ncnn examples or your ncnn Python/C++ wrapper to load `models/yolo11n.param` + `models/yolo11n.bin` (or the `-opt` files) and run inference on camera frames. The PyTorch/.pt script above runs immediately; converting to ncnn gives a more optimized runtime on ARM.

End of Process.
