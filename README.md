# Face-Recognition Attendance System

Mark attendance by **looking at a webcam** instead of signing a sheet. The app
detects a face, recognises who it is, and logs the name, roll number, and
timestamp — then exports the day's attendance as both a CSV and a PDF report.

<p>
  <img alt="Python" src="https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white" />
  <img alt="Flask" src="https://img.shields.io/badge/Flask-000000?style=flat&logo=flask&logoColor=white" />
  <img alt="OpenCV" src="https://img.shields.io/badge/OpenCV-5C3EE8?style=flat&logo=opencv&logoColor=white" />
  <img alt="scikit-learn" src="https://img.shields.io/badge/scikit--learn-F7931E?style=flat&logo=scikitlearn&logoColor=white" />
  <img alt="NumPy" src="https://img.shields.io/badge/NumPy-013243?style=flat&logo=numpy&logoColor=white" />
  <img alt="pandas" src="https://img.shields.io/badge/pandas-150458?style=flat&logo=pandas&logoColor=white" />
</p>

---

## Why it matters

Manual roll calls are slow, easy to fake (proxy attendance), and painful to
tally. This system makes attendance **contactless and automatic**: register
once, then every future check-in is a glance at the camera, with a clean daily
report ready to download.

## Features

- **Webcam enrolment** — capture a set of face images per person, labelled by
  `Name_Roll`, straight from the browser-driven Flask app.
- **On-device recognition** — Haar-cascade face detection (OpenCV) feeds a
  **K-Nearest-Neighbours** classifier trained on the registered faces.
- **Data augmentation** — training randomly flips, rotates, and scales the face
  crops so the model generalises from only a handful of captures.
- **Automatic logging** — each recognised person is written once per day to a
  dated CSV with name, roll, and time.
- **PDF reports** — the day's attendance is rendered to a printable PDF via
  ReportLab.

## How it works

```mermaid
flowchart LR
    C["Webcam frame"] --> D["Haar cascade<br/>face detection"]
    D --> X["Crop + resize 50x50"]
    X --> K["KNN classifier"]
    K --> I["Identified as Name_Roll"]
    I --> L["Append to daily CSV<br/>(once per person)"]
    L --> P["ReportLab PDF report"]
```

### Two phases: enrol, then recognise

```mermaid
flowchart TD
    subgraph Enrol
      A1["Capture N face images"] --> A2["Save to static/faces/Name_Roll/"]
      A2 --> A3["train_model()<br/>augment + fit KNN"]
      A3 --> A4["face_recognition_model.pkl"]
    end
    subgraph Attend
      B1["Live frame"] --> B2["detect + crop"]
      B2 --> B3["KNN predict"]
      B3 --> B4["log name / roll / time"]
    end
    A4 -.loads.-> B3
```

## Tech stack

| Concern | Tool |
| ------- | ---- |
| Web UI | Flask + Jinja templates |
| Face detection | OpenCV Haar cascade (`haarcascade_frontalface_default.xml`) |
| Recognition | scikit-learn `KNeighborsClassifier` (k = 5) |
| Data wrangling | NumPy + pandas |
| Model persistence | joblib (`.pkl`) |
| Reporting | ReportLab (PDF) |

## Quickstart

```bash
python -m venv env && source env/bin/activate   # Windows: env\Scripts\activate
pip install -r requirements.txt

python app.py
```

Then, from the web UI:

1. **Register** a person — this captures their face images and retrains the model.
2. **Take Attendance** — the webcam recognises registered people and logs them.
3. Grab the day's report from `Attendance/Attendance-<date>.csv` / `.pdf`.

> The model and captured faces live under `static/`, and daily logs under
> `Attendance/`. Both folders are created automatically on first run.

## Project structure

```
app.py                                  Flask app: detection, KNN training, logging, PDF export
haarcascade_frontalface_default.xml     OpenCV face-detection cascade
templates/                              Flask HTML templates
static/
├── faces/<Name_Roll>/                  captured face images per person
└── face_recognition_model.pkl          trained KNN model
Attendance/
└── Attendance-<date>.csv / .pdf        generated daily reports
```

## Screenshots

![Home / attendance view](https://user-images.githubusercontent.com/79561540/228186638-617a8fdd-d722-4830-ae1e-46602215dde2.png)
![Registering a user](https://user-images.githubusercontent.com/79561540/228185832-a18bc010-35e1-45d9-8691-8310ba7526f7.png)
![Live recognition](https://user-images.githubusercontent.com/79561540/228186033-fad84b90-7baf-436c-955b-d428a3ab38ef.png)
![Attendance table](https://user-images.githubusercontent.com/79561540/228186374-5dda2a3c-2860-4866-b05f-5fa397c4bb55.png)
![PDF report](https://user-images.githubusercontent.com/79561540/228186460-4aaf6114-42b1-4c53-b084-b820094c9649.png)
