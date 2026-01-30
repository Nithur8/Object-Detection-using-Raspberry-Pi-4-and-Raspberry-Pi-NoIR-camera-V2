# Object detection process

I performed object detection on the Raspberry Pi using the official libcamera-based rpicam-detect workflow from the Raspberry Pi documentation. I used rpicam-detect for camera capture and real-time post-processing with TensorFlow Lite so that the Pi handles both acquiring frames from the NoIR camera and running the detection model.

What I prepared

- Enabled the camera interface and made sure the system is up to date.
- Installed the libcamera/rpicam apps (or removed pre-installed rpicam-apps when necessary) so I could run rpicam-detect.
- Installed the TensorFlow Lite runtime and dependencies required to run a .tflite model on the Pi.
- Placed the TensorFlow Lite model (.tflite) and labels file in an accessible location (I used /usr/share/rpicam/models/ for convenience, or you can keep them in the project directory).

How I ran detection

I used the rpicam-detect application with the TensorFlow Lite post-processing stage (object_detect_tf). rpicam-detect handles camera capture and passes frames to the chosen post-processing stage which performs the neural network inference and returns detection results.

An example command I used (adjust paths and thresholds to match your model and preferences):

rpicam-detect --post-process object_detect_tf \
  --model /usr/share/rpicam/models/ssd_mobilenet_v2_coco.tflite \
  --labels /usr/share/rpicam/models/coco_labels.txt \
  --threshold 0.5 \
  --preview

- --post-process object_detect_tf: selects the TensorFlow Lite object detection post-processing stage described in the Raspberry Pi docs.
- --model and --labels: point rpicam-detect to the .tflite model and the labels file for interpreting model outputs.
- --threshold: minimum confidence for reporting detections (adjust to suit your use case).
- --preview: enable the camera preview window while detections run (omit if running headless).

Notes and tips from my setup

- If your system included pre-installed rpicam-apps and you prefer the packaged versions from the Raspberry Pi docs, follow the documentation's guidance to remove or replace pre-installed rpicam-apps before installing/updating the official packages.
- Use TensorFlow Lite models optimized for the Pi (quantized models are faster and often viable for real-time detection on the Pi 4).
- Put frequently used models under /usr/share/rpicam/models/ so they are easy to reference from rpicam-detect.
- If running headless or on startup, you can run rpicam-detect without --preview and route outputs (bounding boxes, JSON results, or other outputs supported by the post-processing stage) to files or services as needed.

References

- rpicam-detect: https://www.raspberrypi.com/documentation/computers/camera_software.html#rpicam-detect
- rpicam-apps (remove pre-installed rpicam-apps): https://www.raspberrypi.com/documentation/computers/camera_software.html#remove-pre-installed-rpicam-apps
- TensorFlow Lite install (post-processing with TensorFlow Lite): https://www.raspberrypi.com/documentation/computers/camera_software.html#post-processing-with-tensorflow-lite
- TensorFlow Lite object detection stage (object_detect_tf): https://www.raspberrypi.com/documentation/computers/camera_software.html#object_detect_tf