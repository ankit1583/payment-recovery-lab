# Changelog

## 0.6.0 — Explainable recovery analyst

- Added `GET /api/v1/runs/{run_id}/analysis`, an evidence-grounded deterministic analysis endpoint.
- Added recovery recommendations for unavailable evidence, duplicate completed captures, pending captures, no confirmed capture, confirmed success, and client/audit disagreement.
- Added an evidence explorer action and analysis panel showing findings, confidence, recommendation, actions, and disclaimer.
- Explicitly disclose that the analyst is rule-based decision support, not a generative AI model and not a real-provider integration.
- Added backend acceptance tests for duplicate escalation, incomplete evidence, successful capture analysis, and missing run handling.
- Backend verification in this build: 40 tests pass; Python compile check passes.
- Frontend dependency installation/build/test/lint and browser QA were not run in this environment because `node_modules` is absent and registry access is unavailable. Run the frontend checks locally before release.


## [0.5.0] - 2026-11-07

### Fixed
- Oracle gives definite invariant violations precedence over pending captures and emits structured violations.
- Client view records a received initial capture response; runs now report terminal client outcome and outcome-agreement/scorecard fields.
- Configurable bounded retry budget gates the retry; backoff is virtual and does not sleep.
- Reconcile-first validates a visible capture before adopting it.
- Pre-existing duplicate scenario seeds duplicate records before recovery begins.
- Matrix runs no longer evict single-run history.
- HTTP, 404/405 and validation failures share a common error envelope.
- Restored the full MIT license disclaimer.
- Corrected Vitest setup, React Testing Library cleanup, mobile drawer focus, null evidence rendering, download URL cleanup, minimum small-text sizes and reduced-motion support.

### Verification limits
- Backend: 35 tests pass; compileall and API smoke checks pass.
- Frontend dependencies could not be installed in this network-restricted environment; frontend build, tests, lint, browser and accessibility checks remain unverified.
- No lockfile was fabricated. Generate `package-lock.json` with `npm install` on a connected machine, then use `npm ci` in CI.
- This is still a local provider-neutral simulator, not a payment processor or LLM agent.
