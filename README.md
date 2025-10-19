# Car Speed Detection — YOLOv8 + Kalman Filter

A lightweight Python project to detect and estimate vehicle speeds in video using YOLOv8 for detection and a Kalman filter for robust multi-object tracking. This repository provides a ready-to-run pipeline for detection → tracking → speed estimation, with guidance for calibration and tuning so you can adapt it to different cameras and scenes.

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [How it works (high level)](#how-it-works-high-level)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
  - [Command-line example](#command-line-example)
  - [Python API example](#python-api-example)
- [Calibration & Speed Conversion](#calibration--speed-conversion)
- [Configuration & Tuning](#configuration--tuning)
- [Dependencies](#dependencies)
- [Troubleshooting & Tips](#troubleshooting--tips)
- [Contributing](#contributing)
- [License](#license)

## Overview
This project combines:
- YOLOv8 (Ultralytics) for accurate, real-time vehicle detection,
- A Kalman filter-based tracker (plus a matching algorithm) to maintain consistent object IDs across frames,
- A speed estimation module that converts per-frame displacements (in pixels) into real-world velocities using camera calibration or simple metric heuristics.

It is suitable for research, prototyping, and proof-of-concept vehicle speed detection from dashcams, surveillance cameras, or traffic monitoring feeds.

## Features
- Real-time detection using YOLOv8 models (nano/small/medium/large).
- Kalman-filter-based tracking for stable object IDs and smoother trajectories.
- Per-object instantaneous and averaged speed estimates (km/h or mph).
- Video and webcam input support.
- Optional homography / per-scene calibration to convert pixel displacement → meters.
- CLI and Python API examples for easy integration.

## How it works (high level)
1. Load YOLOv8 weights and run detection on each frame.
2. Associate detections to existing tracks using IoU/embedding + Hungarian matcher.
3. Update tracks via Kalman filter to smooth noise and predict positions when occluded.
4. Compute per-frame displacement for each tracked object, convert to meters using calibration, divide by time delta to get speed, and optionally apply smoothing/averaging.

## Requirements
- Python 3.8 or later
- GPU recommended for real-time inference (CUDA-enabled PyTorch). CPU works for small models but slower.

## Installation

1. Clone the repository:
   ```
   git clone https://github.com/Gunasekhar00212/Car_Speed_Detection_YOLO.git
   cd Car_Speed_Detection_YOLO
   ```

2. (Optional) Create and activate a virtual environment:
   ```
   python -m venv venv
   source venv/bin/activate   # macOS / Linux
   venv\Scripts\activate      # Windows
   ```

3. Install dependencies:
   - If repository includes `requirements.txt`:
     ```
     pip install -r requirements.txt
     ```
   - Minimal packages (example):
     ```
     pip install ultralytics opencv-python-headless numpy filterpy scipy lap matplotlib
     ```
   Note: `ultralytics` will pull a compatible PyTorch installation. If you need GPU support, install a CUDA-enabled torch first (`pip install torch --index-url https://download.pytorch.org/whl/cuXXX`), then `pip install ultralytics`.

4. Download YOLOv8 weights (if not included):
   - ultralytics usually downloads automatically when you supply a model name such as `yolov8n.pt`.
   - Or manually:
     ```
     wget https://github.com/ultralytics/assets/releases/download/v0.0/yolov8n.pt
     ```

## Usage

Replace filenames and paths with those present in the repository (examples below assume `speed_detection.py` or equivalent script exists).

Command-line example:
```
python speed_detection.py \
  --source videos/traffic.mp4 \
  --weights yolov8n.pt \
  --model-size small \
  --output results/output_with_speeds.mp4 \
  --fps 30 \
  --calibration config/calibration.json \
  --min-trk-length 3 \
  --unit kmh
```

Common flags:
- `--source` : video file path, directory of frames, or integer for webcam.
- `--weights` : YOLOv8 weights file or model string (e.g., `yolov8n.pt`).
- `--calibration` : JSON or YAML file containing homography/pixel-to-meter parameters (optional).
- `--fps` : frames per second of the input source (used for time delta).
- `--unit` : "kmh" or "mph".
- `--output` : path to save annotated video.

Python API example:
```python
from speed_detection import SpeedDetector  # adapt import to actual module name

detector = SpeedDetector(
    weights="yolov8n.pt",
    calibration_path="config/calibration.json",
    fps=30,
    unit="kmh"
)

results = detector.process_video("videos/traffic.mp4", save_path="results/out.mp4")
for track in results.tracks:
    print(f"ID {track.id}: avg_speed={track.avg_speed:.1f} km/h")
```
(Adapt class/method names to match the repository's implementation.)

## Calibration & Speed Conversion
Accurate speed estimation requires converting pixel displacements to real-world distances. Two common approaches:

1. Homography (recommended)
   - Identify four or more reference points in the scene (corners of a rectangle of known width/length on the road).
   - Compute a homography from image plane to a ground plane (birds-eye view).
   - Use ground-plane coordinates to measure meters traveled between frames.

2. Single-scale heuristic (simpler, less accurate)
   - Use a known object dimension (e.g., a road lane width) and estimate pixel→meter scale for a region of interest.
   - Convert pixel displacement using the local scale.

Important:
- Use the actual `fps` value from the video or camera stream. Wrong FPS leads to incorrect speeds.
- Perspective distortion matters: objects at different depths will have different pixel-to-meter scales.

## Configuration & Tuning
- Choose model size by performance/accuracy tradeoff (yolov8n → fastest, yolov8x → most accurate).
- Kalman filter parameters (process noise, measurement noise) can be tuned to make tracks smoother or more responsive.
- Matching strategy: IoU-only is simple, but combining it with appearance embedding reduces ID switches.
- Minimum track length threshold reduces spurious speed estimates for transient detections.

## Dependencies
A non-exhaustive list of key Python packages used or recommended:
- Python >= 3.8
- ultralytics (YOLOv8) — detection
- opencv-python or opencv-python-headless — video I/O and drawing
- numpy — numeric ops
- filterpy — Kalman filter implementation (or custom KF)
- scipy — utility functions (optional)
- lap or scipy.optimize — assignment (Hungarian)
- matplotlib — plotting and debugging (optional)
- pandas — logging and CSV export (optional)

If the repo contains a requirements file, install from it:
```
pip install -r requirements.txt
```

## Troubleshooting & Tips
- If detections are missing, try a larger YOLOv8 model or fine-tune on traffic images similar to your scene.
- For jittery speed estimates, ensure correct FPS and increase Kalman process noise or apply a moving average to speed values.
- If many ID switches occur, add an appearance feature or increase matching IoU thresholds and track confirmation time.
- For best results in real-world deployments, perform per-camera calibration and validate speeds against ground truth.

## Contributing
Contributions, bug reports, and feature requests are welcome. Please:
1. Open an issue describing the problem or feature.
2. Fork the repo and create a branch for your changes.
3. Submit a pull request with tests and documentation updates.

If you’d like help adding features such as:
- automated homography calibration tools,
- dataset training scripts,
- dashboard for per-vehicle stats,
I can help design and implement them.

## License
This project is released under the MIT License.

Full text (MIT):
Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies... (See LICENSE file for complete terms.)

If you prefer a different license (Apache-2.0, GPL, etc.), replace the LICENSE file accordingly.

---

If you need, I can:
- generate a matching `requirements.txt` for the repo based on the code, or
- craft example scripts (`speed_detection.py`, `calibrate.py`) and sample calibration JSON, or
- create a CI workflow to run basic linting/tests.
Tell me which of these you'd like me to add next and I'll prepare the files and instructions.
