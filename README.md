# Vehicle Speed Detection using YOLOv8 and Kalman Filter

A Python project to detect vehicles in video streams, track them using a Kalman filter, and estimate their speed in real-time, even handling stationary vehicles accurately.

---

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Configuration](#configuration)
- [How it Works](#how-it-works)
- [Requirements](#requirements)
- [Acknowledgements](#acknowledgements)
- [License](#license)

---

## Overview
This project detects and tracks vehicles in a video using **YOLOv8**. It estimates vehicle speeds using a combination of **perspective transformation** and **Kalman filter-based tracking**. It also handles stationary vehicles to avoid false speed readings.

---

## Features
- Real-time vehicle detection using YOLOv8
- Multi-object tracking with Kalman filter
- Perspective transformation to map pixels to real-world meters
- Speed estimation in km/h
- Handles stationary vehicles and resets speed
- Visual annotations on video frames with vehicle ID and speed

---

## Installation
1. Clone the repository:
2. Create a virtual environment (optional but recommended):
3. Install dependencies:

---

## Usage
1. Place your video in the project folder and update `VIDEO_PATH` in the script if necessary.
2. Run the script:
3. On first run, click 4 points in the video window to calibrate perspective (Top-Left, Top-Right, Bottom-Right, Bottom-Left). Press **Enter** or **Space** when done.
4. Press **q** to quit the program.

---

## Configuration
You can adjust the following parameters in the script:
- `CONFIDENCE_THRESHOLD`: YOLO detection confidence
- `FRAME_SKIP`: Number of frames to skip for faster processing
- `REAL_WORLD_WIDTH_METERS` & `REAL_WORLD_HEIGHT_METERS`: Real-world size of the monitored road section
- `STATIONARY_PIXEL_THRESHOLD` & `STATIONARY_RESET_THRESHOLD_FRAMES`: Adjust for stationary vehicle detection

---

## How it Works
1. **YOLOv8** detects vehicles in frames.
2. Each detected vehicle is tracked using a **Kalman filter**.
3. A **perspective transform** maps pixel coordinates to real-world measurements.
4. Speed is calculated as the distance traveled over time, considering stationary vehicle thresholds.
5. Results are displayed on the video with bounding boxes, ID, and speed.

---

## Requirements
- Python 3.9+
- OpenCV
- Numpy
- Ultralytics YOLOv8
- Scipy

Install all dependencies:


---

## Acknowledgements
- [YOLOv8 by Ultralytics](https://github.com/ultralytics/ultralytics)
- OpenCV for video processing and Kalman filter utilities

---

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

