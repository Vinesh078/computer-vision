

### Real-Time Autonomous Driving Computer Vision System

An Advanced Driver Assistance System (ADAS) perception pipeline built with Python and computer vision. The system unifies multi-object tracking, monocular depth estimation, road/sky segmentation, lane tracking, and predictive collision risk assessment into a single, high-performance execution loop optimized for GPU acceleration.

---

## 🚀 Core Features

* **Multi-Object Tracking (YOLOv8 + ByteTrack):** Detects and tracks pedestrians, cyclists, cars, motorcycles, buses, and trucks. Implements exponential smoothing on bounding boxes and tracks motion trails over historical frames.
* **Monocular Depth Estimation (MiDaS):** Uses `MiDaS_small` (or `DPT_Large`) to compute real-time depth maps, translating relative disparities into metric distance approximations.
* **Predictive Collision Risk Assessor:** Monitors approaching speeds using distance history. It dynamically calculates **Time-To-Collision (TTC)** and generates situational alerts (`SAFE`, `CAUTION`, `WARNING`, `CRITICAL`).
* **Hough-Polynomial Lane Tracking:** Fits second-order polynomial curves to detected lane markers, overlays a safe driving corridor, and performs lane departure warning analysis (`LEFT DEPARTURE`, `RIGHT DEPARTURE`, `CENTERED`).
* **Heuristic Road & Sky Segmentation:** Fast HSV color-space processing mixed with depth priors isolates the driveable asphalt layer and the sky boundaries without heavy semantic model overhead.
* **Heads-Up Display (HUD) Rendering:** Composes a futuristic dashboard complete with telemetry data bars, bounding box info panels, flashing emergency banners, a mini-depth panel, a top-down **Bird's-Eye View (BEV)** radar grid, and a faux-LiDAR **3D Point Cloud Scatter**.

---

## 🛠️ Architecture & Data Flow

1. **Frame Capture:** The framework loads images from video streams, webcams, or RTSP streams and resizes them to a target display grid ($1280 \times 720$).
2. **Parallel Multi-Task Execution:**
* *YOLOv8* predicts object categories and location bounding boxes.
* *ByteTrack* assigns unique tracking IDs and measures trajectory paths.
* *MiDaS* builds structural disparity maps (optimized with inference-mode FP16 frame skipping).
* *Canny/Hough Edge Arrays* calculate road line trajectories.


3. **Telemetry Calculation:** The `CollisionRiskAssessor` checks distance deltas to extract exact approach rates and triggers safety metrics.
4. **UI Canvas Composition:** The system merges transparency profiles, side layouts, and metric indicators to drop into an output frame and video file stream.

---

## 📦 Tech Stack & Dependencies

The system features an **automated dependency installer** built right into the script. On launch, it will silently check for and install missing packages. The primary stack includes:

* **Core:** `torch`, `torchvision`, `numpy`, `scipy`
* **Vision & Math:** `opencv-python`, `pillow`, `matplotlib`
* **Deep Learning Tracking:** `ultralytics`, `supervision`
* **Utilities:** `tqdm`

---

## 🔧 Installation & Setup

1. **Clone the Repository:**
```bash
git clone https://github.com/tubakhxn/adas-perception-pipeline.git
cd adas-perception-pipeline

```


2. **Set up a Virtual Environment (Recommended):**
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

```


3. **Install Dependencies:**
You can let the script auto-install packages on its first run, or install them manually:
```bash
pip install opencv-python numpy torch torchvision pillow scipy matplotlib supervision ultralytics tqdm

```



---

## 🚗 Usage

Run the primary script via terminal arguments. By default, it looks for an active webcam (`0`).

```bash
# Run using default webcam (Index 0)
python driving.py

# Run on a local video file
python driving.py path/to/dashcam_video.mp4

# Run on an RTSP Network Stream
python driving.py rtsp://username:password@ip_address:port/stream

# Run without depth estimation (Optimized performance fallback for CPU-only systems)
python driving.py --source path/to/video.mp4 --no-depth

# Specify a custom output file destination
python driving.py path/to/video.mp4 --output local_render.mp4

```

### 🎮 Runtime Controls

When the visualization pipeline window is focused, you can control execution dynamically:

* `Q` : Quit execution loop safely and save records.
* `P` : Pause / Resume the video feed stream.
* `S` : Take a high-resolution HUD snapshot frame (`screenshot_XXXXX.jpg`).

---

## ⚙️ Performance Tuning & Configurations

You can modify performance characteristics globally by adjusting the `CFG` dictionary directly inside `driving.py`:

```python
CFG = {
    "yolo_model":        "yolov8n.pt",          # Nano variant for speed; switch to yolov8s/m for accuracy
    "yolo_conf":         0.35,                  # Confidence threshold
    "depth_model":       "MiDaS_small",          # Use "DPT_Large" or "DPT_Hybrid" for richer structures
    "depth_interval":    2,                      # Run depth prediction every N frames to optimize GPU throughput
    "frame_skip":        1,                      # Process every Nth frame
    "fp16":              True,                   # Toggle half-precision computation on CUDA GPUs
}

```

