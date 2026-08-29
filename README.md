# PhaseGuard

**Electrical fault classifier dashboard** — a machine learning powered web app that detects and classifies electrical faults on a 3-phase power line in real time.

Built as a capstone project combining machine learning (scikit-learn), a FastAPI backend, and a hand-coded vanilla HTML/CSS/JS frontend with live Canvas 2D waveform visualization.

**Live demo:** https://sehrish0508.github.io/PhaseGuard/
---

## What it does

PhaseGuard takes six real-time electrical readings — current (Ia, Ib, Ic) and voltage (Va, Vb, Vc) on a 3-phase power line — and predicts which type of fault, if any, is occurring. The dashboard lets you simulate these readings using sliders and see:

- A live 3-phase current waveform on an HTML canvas
- The predicted fault type and label
- The model's confidence score
- Per-phase (and ground) fault status, color-coded

No page reloads, no submit button — every slider movement instantly redraws the waveform and re-queries the trained model.

## Fault types detected

The model classifies six conditions, based on the standard fault categories used in power systems:

| Code | Meaning |
|---|---|
| No Fault | Normal, healthy operation |
| AG | Line A to Ground fault |
| ABG | Line A, B to Ground fault |
| BC | Line B to Line C fault |
| ABC | Three-phase fault |
| ABCG | Three-phase to Ground fault |

## Tech stack

**Machine Learning**
- Python, pandas, scikit-learn
- Random Forest Classifier
- Trained on the [Electrical Fault Detection and Classification](https://www.kaggle.com/datasets/esathyaprakash/electrical-fault-detection-and-classification) dataset (Kaggle)

**Backend**
- FastAPI
- Pydantic for request validation
- joblib for model serialization
- CORS-enabled REST API (`/predict`)

**Frontend**
- Vanilla HTML, CSS, and JavaScript (no frameworks, no libraries)
- HTML5 Canvas 2D for real-time waveform rendering
- `fetch()` with `async`/`await` for backend communication

## How it works

1. The user adjusts one or more of the six sliders (Ia, Ib, Ic, Va, Vb, Vc).
2. On every slider change:
   - The three current values are scaled and redrawn as a live 3-phase sine wave on the canvas.
   - All six raw values are sent via a `POST` request to the FastAPI `/predict` endpoint.
3. The backend loads the pre-trained Random Forest model, runs a prediction, and returns:
   - `fault_type`, `fault_label`, `confidence`, and a per-phase `phase_status` object.
4. The frontend updates the prediction panel and color-codes each phase (red = faulted, green = healthy) instantly.

## Model performance

- **Training accuracy:** 100%
- **Test accuracy:** 86.3% overall
- Near-perfect precision/recall (0.98–1.00) on No Fault, AG, ABG, and BC classes.
- Confusion is concentrated almost entirely between **ABC** and **ABCG** faults, because both are symmetric three-phase conditions with very similar instantaneous electrical signatures — the ground-involvement signal is inherently subtle in raw current/voltage readings for these two classes.

## A note on confidence during manual testing

Confidence scores may appear moderate (around 30–50%) when using the sliders. This is expected: sliders can only approximate a value, while the model was trained on exact, high-precision readings. On the original test dataset, using exact values, the model achieves 98–100% accuracy and confidence. Lower slider-based confidence reflects input precision, not model quality.

## Design choices

- **Current, not voltage, drives the waveform.** Instantaneous current is a more reliable single-glance fault indicator than instantaneous voltage, since fault current spikes are large relative to normal load current, whereas voltage near zero can be ambiguous (a real dip or a normal zero-crossing).
- **Voltage values are normalized.** The dataset's voltage columns are small, zero-centered values (roughly -0.6 to 0.6) rather than raw volts — this reflects how the source dataset was generated, not a project design decision. Current values remain in their raw, real form.
- **The waveform amplitude is scaled, not raw.** Real current values range roughly ±900, but healthy/no-fault current typically sits well within ±100. The waveform uses a linear scale calibrated so that ±450 maps to the canvas's safe visual bounds — values beyond that continue proportionally and may visually clip off-canvas, which itself signals an especially severe fault.
- **No "Predict" button.** Predictions update instantly on every slider change, giving immediate cause-and-effect feedback.

## Known limitations

- Trained on six fault types only; does not cover every theoretically possible fault combination.
- ABC and ABCG faults can be harder to distinguish due to their similar three-phase symmetry.
- No phasor diagram for voltage (deferred due to project time constraints).
- Sliders provide approximate, not exact, input precision compared to the original dataset.

## Running locally

**Backend**
```bash
cd backend
pip install fastapi uvicorn scikit-learn pandas joblib
uvicorn main:app --reload
```

**Frontend**

Open `index.html` directly in a browser, or serve it with any static file server. Ensure the `fetch()` URL in `script.js` points to your running backend (`http://127.0.0.1:8000/predict` for local use).

## Dataset

[Electrical Fault Detection and Classification](https://www.kaggle.com/datasets/esathyaprakash/electrical-fault-detection-and-classification) by esathyaprakash on Kaggle — `classData.csv` (7,861 rows, features: G, C, B, A fault indicators, Ia, Ib, Ic, Va, Vb, Vc).

---

Built as a hands-on capstone project to learn FastAPI, Canvas 2D, and full-stack integration — all frontend code was written by hand, without AI-generated code, as a deliberate learning constraint.
