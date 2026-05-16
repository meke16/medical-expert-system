# Medical Expert System

A full-stack medical expert system with a FastAPI backend and a React + Vite frontend. The backend exposes diagnosis and report APIs, and the frontend provides the user interface for symptom-based medical guidance.

## Project Structure

```text
medical-expert-system/
├── backend/
│   ├── main.py
│   ├── requirements.txt
│   ├── data/
│   └── engine/
└── frontend/
    ├── package.json
    └── src/
```

## Requirements

- Python 3.12+ recommended
- Node.js 18+ recommended
- `npm`

## Backend Setup

From the `backend/` directory:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn main:app --reload
```

The backend runs at `http://127.0.0.1:8000` by default.

### Backend API

- `GET /health` - health check
- `GET /api/symptoms` - list available symptoms
- `POST /api/diagnose` - run diagnosis based on symptoms, age, gender, and language
- `POST /api/report` - generate a report from diagnosis results

### Example Payloads

```json
{
  "symptoms": ["fever", "headache"],
  "age": 30,
  "gender": "male",
  "lang": "en"
}
```

```json
{
  "patient": {"age": 30, "gender": "male"},
  "symptoms": ["fever", "headache"],
  "results": [{"disease": "...", "score": 0.9}]
}
```

## Frontend Setup

From the `frontend/` directory:

```bash
npm install
npm run dev
```

The Vite dev server runs at `http://localhost:5173` by default.

## Running Both Apps

1. Start the backend in `backend/`.
2. Start the frontend in `frontend/`.
3. Open the frontend in the browser and use the UI to interact with the backend APIs.

## Troubleshooting

- If `uvicorn main:app --reload` fails with `ModuleNotFoundError: No module named 'fastapi'`, install backend dependencies inside a virtual environment first.
- If port `8000` is already in use, stop the process using that port or start Uvicorn on another port with `--port 8001`.

## Notes

- Backend CORS is configured for local frontend development on `http://localhost:5173` and `http://localhost:3000`.
- The repository includes backend data under `backend/data/` and diagnosis logic under `backend/engine/`.