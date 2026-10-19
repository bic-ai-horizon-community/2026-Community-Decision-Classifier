# Community Decision Classifier

A full-stack app that takes user text, sends it to an AI-backed classifier, and returns:

- the **most fitting emoji**
- a **confidence score**

The project has a React frontend and a FastAPI backend.

## What the application does

1. You enter text in the frontend UI.
2. The frontend sends that text to the backend API.
3. The backend uses `EmojiClassifier` (`emoji/main.py`) and a predefined emoji catalog (`emojis.json`).
4. The API responds with:
   - `input` (original text)
   - `emoji` (selected emoji)
   - `confidence` (model confidence score)

## Architecture and request flow

- **Frontend (React + Vite + TypeScript)**
  - UI: `frontend/src/App.tsx`
  - API client: `frontend/src/api/emoji.ts`
  - Reads API URL from `VITE_API_URL`

- **Backend (FastAPI)**
  - API entrypoint: `backend/app.py`
  - Endpoint: `POST /predict`
  - Uses `EmojiClassifier` from `emoji/main.py`

- **Classifier module**
  - Loads emoji criteria from `emojis.json`
  - Calls `typesafe_sdk` with `TYPESAFE_API_KEY`

### End-to-end flow

`App.tsx` → `predictEmoji()` → `POST VITE_API_URL` → `/predict` in FastAPI → `EmojiClassifier.predict()` → JSON response to frontend

## Prerequisites

The repository does not pin runtime versions, but you need:

- **Python 3**
- **Node.js + npm**
- A valid **TypeSafe API key** for `TYPESAFE_API_KEY`

## Environment configuration

This repo includes example env files:

- Root: `.env.example`
- Frontend: `frontend/.env.example`

### 1) Backend env (`.env` in repository root)

Create `/home/runner/work/2026-Community-Decision-Classifier/2026-Community-Decision-Classifier/.env` from `.env.example`:

```bash
cp .env.example .env
```

Set:

```env
TYPESAFE_API_KEY=your_api_key_here
```

### 2) Frontend env (`frontend/.env`)

Create `/home/runner/work/2026-Community-Decision-Classifier/2026-Community-Decision-Classifier/frontend/.env` from `frontend/.env.example`:

```bash
cd frontend
cp .env.example .env
```

Set `VITE_API_URL` to the full prediction endpoint URL (not just host):

```env
VITE_API_URL=http://127.0.0.1:8000/predict
```

## Installation

### Backend

From repository root:

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

> Note: `emoji/main.py` imports `python-dotenv`, but `requirements.txt` currently lists only `typesafe-sdk`, `uvicorn`, and `fastapi`. If needed, install:
>
> ```bash
> pip install python-dotenv
> ```

### Frontend

From `frontend/`:

```bash
npm install
```

## Run in development

You need two terminals.

### Terminal 1: Start API

From repository root:

```bash
uvicorn backend.app:app --reload
```

API default URL: `http://127.0.0.1:8000`

### Terminal 2: Start frontend

From `frontend/`:

```bash
npm run dev
```

Vite default URL (typically): `http://localhost:5173`

## API usage

### `POST /predict`

- **Path**: `/predict`
- **Content-Type**: `application/json`
- **Request body**:

```json
{
  "text": "I just shipped a big feature and I'm excited!"
}
```

- **Response shape** (`EmojiResult`):

```json
{
  "input": "I just shipped a big feature and I'm excited!",
  "emoji": "🎉",
  "confidence": 0.92
}
```

### Example curl

```bash
curl -X POST "http://127.0.0.1:8000/predict" \
  -H "Content-Type: application/json" \
  -d '{"text":"This is amazing"}'
```

## Project structure

```text
.
├── backend/
│   └── app.py                  # FastAPI app and /predict endpoint
├── emoji/
│   └── main.py                 # EmojiClassifier + EmojiResult model
├── frontend/
│   ├── src/
│   │   ├── api/emoji.ts        # Frontend API call helper
│   │   ├── types/emoji.ts      # Shared frontend request/response types
│   │   └── App.tsx             # Main UI
│   ├── package.json            # Frontend scripts and dependencies
│   └── .env.example            # Frontend env example
├── emojis.json                 # Emoji-to-description criteria catalog
├── requirements.txt            # Backend Python dependencies
├── .env.example                # Backend env example
└── main.py                     # Local script example using EmojiClassifier
```

## Scripts and quality checks

### Frontend scripts (`frontend/package.json`)

- `npm run dev` – start dev server
- `npm run build` – TypeScript compile + Vite build
- `npm run lint` – run ESLint
- `npm run preview` – preview production build

### Backend checks

No dedicated lint/test script is currently defined in this repository for backend code.

## Testing

There is currently no automated test suite committed in this repository.

For now, validate by running both services and submitting sample text through the UI or `curl`.

## Troubleshooting

### `ModuleNotFoundError: No module named 'dotenv'`

Install missing dependency in your active backend environment:

```bash
pip install python-dotenv
```

### Frontend cannot reach backend

- Confirm backend is running on `http://127.0.0.1:8000`
- Confirm `frontend/.env` has:

```env
VITE_API_URL=http://127.0.0.1:8000/predict
```

### CORS errors in browser

Backend currently allows origin:

- `http://localhost:5173`

If you run frontend on another origin, update `allow_origins` in `backend/app.py`.

### `FileNotFoundError` for `emojis.json`

Run the backend from the repository root so relative file loading in `emoji/main.py` can find `emojis.json`.

## Contributing

1. Fork/branch your changes.
2. Keep changes focused and tested manually.
3. Run frontend lint/build when frontend code changes:

```bash
cd frontend
npm run lint
npm run build
```

## License

No license file is currently present in the repository.
