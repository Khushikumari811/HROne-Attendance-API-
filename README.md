# Employee Attendance & Analytics API

FastAPI + MongoDB implementation of HROne's Employee Attendance & Analytics API contract v2.0.

## Run locally

```bash
python -m venv .venv
# Windows: .venv\\Scripts\\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
# Set MONGO_URI and MONGO_DB in your environment (or a local .env file).
uvicorn app.main:app --port 8000
```

MongoDB 6.0+ is required. The app creates indexes idempotently at startup. `GET /health` pings MongoDB and returns 503 if it is unavailable. Interactive API docs are at `/docs`.

## Seed sample data

Run `python sample_seed.py` after setting `MONGO_URI` and `MONGO_DB`. The seed script loads the sample Extended JSON documents; use a disposable database when reseeding.

## Submission notes

- All application endpoints and helpers are in `app/main.py`, as requested.
- `REVIEW.md` records defects found in the starter; `DECISIONS.md` records the design choices.
- Do not commit `.env`, credentials, virtual environments, or database dumps.
- Verify the service against MongoDB 7.0 and the complete hidden-style edge cases before submission. The sample dataset alone does not prove correctness.
