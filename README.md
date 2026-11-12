# Payment Recovery Lab

A local reliability workbench for studying how an automated payment workflow recovers from uncertain network outcomes, with an explainable evidence-based recovery analyst.

**Version:** 0.6.0

## Scope and naming
This build is a **standalone simulator**. It does not connect to PayPal, use PayPal credentials, process real payments, or claim affiliation with PayPal. The name is intentionally provider-neutral because no PayPal integration is included.

## Requirements
- Python 3.11 or newer
- Node.js 20 or 22 (LTS) and npm 10 or newer
- npm

## Windows setup

### Backend — terminal 1
From the repository root:

```powershell
cd backend
py -3.13 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements-dev.txt
uvicorn app.main:app --reload
```

If PowerShell blocks activation:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
.\.venv\Scripts\Activate.ps1
```

API documentation: http://127.0.0.1:8000/docs

### Frontend — terminal 2
From the repository root:

```powershell
cd frontend
npm install
npm run dev
```

Open http://127.0.0.1:5173

### Tests — terminal 3
From the repository root:

```powershell
cd backend
.\.venv\Scripts\Activate.ps1
pytest -q
```

Then build the frontend:

```powershell
cd ..\frontend
npm run build
```

## Recovery analyst
After running an experiment, open its evidence record and select **Generate recovery analysis**. The backend returns a deterministic, explainable assessment based on the run's client observations, independent audit verdict, capture counts, and event outcomes. It identifies duplicate-capture risk, pending states, missing confirmation, client/audit disagreement, and safe escalation. It provides bounded next-step recommendations and exposes the evidence used.

This analyst is a **rule-based decision-support component, not a generative AI/LLM**. It makes no external model calls and does not provide production payment instructions. Do not describe it as an LLM-powered agent.

## API routes
- `GET /api/v1/health` — service status and version.
- `POST /api/v1/experiments/capture` — execute one simulator run.
- `GET /api/v1/runs` and `GET /api/v1/runs/{run_id}` — inspect bounded run history.
- `GET /api/v1/runs/{run_id}/analysis` — generate evidence-grounded recovery analysis for a saved run.
- `POST /api/v1/matrix` — run the 40-cell strategy/provider-semantics matrix.
- `DELETE /api/v1/runs` — clear in-memory history.

## Experiment
In the UI, choose:
- Strategy: Baseline or Reconcile before retry (reference algorithms)
- Fault: Request dropped, response dropped after provider processing, or no fault
- Amount: positive USD amount with at most two decimal places
- Retry budget: 1–5 attempts (default 2); backoff is virtual and does not sleep

The simulator models capture idempotency: the same idempotency key maps to the same capture. This is an explicit local assumption for comparing recovery strategies, not a statement about any real provider's API.

Expected behavior for the response-drop case:
- Baseline uses a new key on retry and creates two completed captures: oracle FAIL.
- Reconcile-first finds the existing capture and does not send another capture: oracle PASS.

For request-drop:
- No capture exists when reconciliation runs.
- Reconcile-first retries using the original key and creates one capture: oracle PASS.

## What verdicts mean
- PASS: the simulator's complete run-scoped evidence shows exactly one completed capture with the expected amount and currency.
- FAIL: the evidence shows a definite invariant violation, such as zero/multiple completed captures or mismatched payment data. Definite violations outrank pending records.
- Each run also reports `agent_outcome`, `agreement`, `outcome_class`, `safety`, `liveness`, `strategy_assessment`, and `protected_by` so a safe escalation or incomplete attempt is not conflated with an unsafe duplicate.
- INCONCLUSIVE: the evidence source is marked incomplete, so a reliable verdict cannot be established.

## Important limitations
- This is a deterministic simulator, not a live payment integration. It is not a PayPal integration and does not meet a competition requirement for actual PayPal API usage.
- Baseline and hardened are reference recovery algorithms, not LLM agents. The recovery analyst is deterministic rule-based decision support, not a generative model.
- In-memory state resets when the backend restarts.
- Run records are kept in the browser session; use Export JSON to save individual records. Matrix runs are not added to single-run history.
- The simulator's idempotency and capture rules are simplified and must not be assumed to match PayPal or another provider.
- Do not expose the development server to public networks.
