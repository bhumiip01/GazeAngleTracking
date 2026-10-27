# Gaze Angle Tracking

Real-time gaze direction tracking using webcam input. Detects facial landmarks via dlib's 68-point predictor, locates pupils, and computes the **gaze angle** — the direction each eye is looking, expressed in degrees relative to the horizontal axis.

Built on top of [antoinelame/GazeTracking](https://github.com/antoinelame/GazeTracking) with added angle calculation capabilities.

## What's Added

The original library provides pupil detection and left/right/center gaze classification. This fork adds:

- **`calculate_eye_angle(eye_center, eyeball_coords)`** — static method that computes the gaze direction angle using `arctan2`
- **`calculate_left_angle()`** / **`calculate_right_angle()`** — instance methods that return the absolute gaze angle (in degrees) for each eye
- No explicit input parameters needed — coordinates are pulled from the detected pupil and eye center positions internally

### How the Angle Calculation Works

```
gaze_vector = eyeball_coords - eye_center
angle = arctan2(gaze_vector.y, gaze_vector.x)
```

The gaze vector is computed from the eye center to the pupil position. The `arctan2` function converts this 2D vector into an angle in radians, which is then converted to degrees. The result maps to a polar coordinate system:

- **0°** → Looking right
- **90°** → Looking up
- **180°** → Looking left
- **270°** → Looking down

![Visual Field Diagram](docs/visual_field_diagram.jpg)

## Installation

Clone the project:

```bash
git clone https://github.com/bhumiip01/GazeAngleTracking.git
cd GazeAngleTracking
```

Requires **Python 3.10+**.

### pip

```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -e .
```

### Anaconda

```bash
conda env create --file environment.yml
conda activate GazeTracking
```

### Docker (Linux only)

```bash
./build_and_run.sh
```

> **Trouble installing dlib?** Install CMake first: `brew install cmake` (macOS) or `sudo apt install cmake build-essential` (Ubuntu).

### Dependencies

- `opencv-python` — webcam capture and image processing
- `dlib` — face detection and 68-point facial landmark prediction
- `numpy` — numerical operations for angle calculation

## Usage

### Quick Start

```bash
python example.py
```

Press `Esc` to exit.

### In Your Code

```python
import cv2
from gaze_tracking import GazeTracking

gaze = GazeTracking()
webcam = cv2.VideoCapture(0)

while True:
    _, frame = webcam.read()
    gaze.refresh(frame)

    # Pupil coordinates
    left_pupil = gaze.pupil_left_coords()
    right_pupil = gaze.pupil_right_coords()

    # Gaze direction (left/right/center)
    if gaze.is_right():
        print("Looking right")
    elif gaze.is_left():
        print("Looking left")

    # Gaze angles (degrees)
    left_angle = gaze.calculate_left_angle()
    right_angle = gaze.calculate_right_angle()
    if left_angle is not None:
        print(f"Left eye: {left_angle} deg, Right eye: {right_angle} deg")

    # Blink detection
    if gaze.is_blinking():
        print("Blinking")

    if cv2.waitKey(1) == 27:
        break
```

## API Reference

| Method | Returns | Description |
|--------|---------|-------------|
| `refresh(frame)` | `None` | Analyzes a new frame from the webcam |
| `pupil_left_coords()` | `(x, y)` | Left pupil coordinates in the frame |
| `pupil_right_coords()` | `(x, y)` | Right pupil coordinates in the frame |
| `horizontal_ratio()` | `float` | 0.0 (right) to 1.0 (left) |
| `vertical_ratio()` | `float` | 0.0 (top) to 1.0 (bottom) |
| `calculate_left_angle()` | `int` | Absolute gaze angle of left eye in degrees |
| `calculate_right_angle()` | `int` | Absolute gaze angle of right eye in degrees |
| `is_right()` | `bool` | True if looking right |
| `is_left()` | `bool` | True if looking left |
| `is_center()` | `bool` | True if looking at center |
| `is_blinking()` | `bool` | True if eyes are closed |
| `annotated_frame()` | `ndarray` | Frame with pupil crosshairs drawn |

## Project Structure

```
GazeAngleTracking/
├── gaze_tracking/
│   ├── __init__.py
│   ├── gaze_tracking.py      # Main GazeTracking class (modified with angle calc)
│   ├── eye.py                # Eye region extraction and analysis
│   ├── pupil.py              # Pupil detection via thresholding
│   ├── calibration.py        # Auto-calibration for threshold tuning
│   └── trained_models/
│       └── shape_predictor_68_face_landmarks.dat
├── docs/
│   ├── visual_field_diagram.jpg
│   └── calculation_notes.jpg
├── example.py
├── pyproject.toml
├── environment.yml
├── Dockerfile
├── LICENSE
└── README.md
```

## How It Works

1. **Face Detection** — dlib's frontal face detector locates the face in the grayscale frame
2. **Landmark Prediction** — the 68-point shape predictor identifies facial landmarks including eye regions
3. **Pupil Detection** — image processing (thresholding, erosion) isolates the pupil within each eye region
4. **Gaze Calculation** — the pupil position relative to the eye center determines horizontal/vertical ratios and gaze direction
5. **Angle Computation** — `arctan2` converts the gaze vector (eye center → pupil) into a directional angle in degrees

## Acknowledgments

- Original [GazeTracking](https://github.com/antoinelame/GazeTracking) library by [Antoine Lame](https://github.com/antoinelame)
- dlib's 68-point facial landmark predictor

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.
