# Architecture & Notebook Walkthrough

This document maps each notebook section to what it does and why, for anyone reviewing the code section by section.

| # | Section | Purpose |
|---|---|---|
| 0 | Environment Check | Confirms GPU (T4) is attached; falls back to CPU with a warning if not |
| 1 | Install Dependencies | Installs YOLOv8, Gradio, Folium, Geopy, headless OpenCV |
| 2 | Imports & Config | Centralizes all tunable constants (severity thresholds, colors, paths) in one place |
| 3 | Load YOLOv8 Model | Loads trained weights if present; otherwise flags fallback mode |
| 3B | Fine-Tune (optional) | Template for training YOLOv8 on a Roboflow pothole dataset |
| 4 | Core CV Utilities | `classify_severity()` + `detect_potholes_classical()` fallback detector |
| 5 | Geospatial Utilities | EXIF GPS extraction, mock-GPS generation, Nominatim reverse geocoding |
| 6 | Folium Map Builder | Builds the severity-coded interactive map |
| 7 | Reporting | In-memory detection log + CSV export |
| 8 | Master Pipeline | `run_pipeline()` — orchestrates detection → severity → GPS → map → log |
| 9 | Gradio Web App | 3-tab UI (Detection / Map / Data Export) wired to one Analyze button |
| 10 | Notes & Roadmap | Known limitations and production next steps |

## Key design decision: dual-mode detection

The pipeline checks for trained weights at `MODEL_PATH` on load:

- **Found** → loads real YOLOv8 weights, runs true object detection.
- **Not found** → falls back to a classical OpenCV heuristic (adaptive thresholding + contour filtering on dark, irregular blobs).

Both paths return detections in the exact same `(x1, y1, x2, y2, confidence)` format, so every downstream function (severity, drawing, mapping, logging) is completely unaware of which detector ran. This keeps the notebook demoable without a trained model while leaving a clean, single-point upgrade path to production accuracy.

## Data flow summary

```
Image → Detect boxes → Severity per box → Overall severity (worst-case)
      → Resolve GPS (EXIF or mock) → Reverse geocode → Build map
      → Append to log → Export CSV → Return to Gradio UI
```
