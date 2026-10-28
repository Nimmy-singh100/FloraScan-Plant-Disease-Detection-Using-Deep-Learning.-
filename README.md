# FloraScan AI

A responsive React, Vite, TypeScript, and Tailwind frontend for plant disease exploration. It includes a FastAPI integration scaffold, but the Kaggle notebook model is not bundled. Until a real model endpoint is configured, scan results are clearly labeled illustrative examples and do not analyze uploaded photos.

## Run the frontend

```powershell
npm install
npm run dev
```

Open the local URL printed by Vite. Camera scanning requires browser camera permission and a secure context (`localhost` is supported).

## Connect a model API

1. Open the Kaggle notebook while signed into Kaggle. Run the notebook and identify its trained model artifact and the ordered class labels used during training.
2. Export/download the trained artifact from the notebook output. For a Keras `.keras` artifact, copy it to `backend/model/plant_disease_model.keras`. If the notebook uses H5 or PyTorch, update `MODEL_FILE` and implement the matching loader in the backend adapter.
3. Implement preprocessing and inference in `backend/app/main.py` (or a separate model service). Match the same resize, color order, and normalization used during training. Map the model's output index through its training class-label list; do not guess the order.
4. Return JSON from `POST /api/v1/predict` in this shape:

```json
{
  "plant": "Tomato",
  "disease": "Early blight",
  "confidence": 82.4,
  "symptoms": "...",
  "prevention": "...",
  "treatment": "...",
  "grad_cam_url": null
}
```

`confidence` is a percentage from 0 to 100. Add an uncertainty or abstain rule for low-confidence and out-of-distribution images. Model predictions are not guaranteed diagnoses.

5. Start the API from this folder in PowerShell:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r backend\requirements.txt
uvicorn backend.app.main:app --reload --port 8000
```

6. Copy `.env.example` to `.env.local` in the project root. Set `VITE_API_BASE_URL=http://localhost:8000`. Restart Vite. The UI will report a model connection configured; `/health` can be used to check backend/model readiness.

The Kaggle URL provided is an editable notebook page, not a hosted prediction endpoint. This environment could not access its notebook contents, so the model framework, artifact format, preprocessing, and label order still need to be confirmed from that notebook before real inference can be connected. Do not upload Kaggle credentials or API tokens to the frontend.

## Privacy and data

Demo-mode images stay in the browser. Configured model requests send the selected image to the configured backend. Scan history is stored in `sessionStorage` and resets when the browser session ends. Dashboard charts and library guidance are illustrative examples.
