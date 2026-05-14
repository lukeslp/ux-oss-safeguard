# Project Plan

## Current Objective
- Improve development speed and reliability for the local safeguard/chat stack by adding reusable project-scoped Cursor skills.
- Prepare an iPhone/iPad App Store public launch pack with submission-ready icon, screenshot, and app preview specifications.

## Architecture Snapshot
- `proxy-server.js`: Serves static HTML and proxies model API requests.
- `safeguard.html`: Policy evaluation UI with streamed analysis/verdict split handling.
- `ollama-chat.html`: Streaming chat UI with reasoning display support.
- `gpt-oss-safeguard/app.py`: Gradio safeguard variant.

## Skill Pack Goals
- Standardize recurring debugging and verification workflows.
- Reduce regressions in streaming parsing and verdict split behavior.
- Improve accessibility QA consistency across UI changes.
- Keep proxy/CORS behavior explicit and testable.

## Added Skills (Project-Scoped)
- `proxy-cors-regression-check`
- `streaming-jsonline-debugger`
- `safeguard-verdict-split-validator`
- `ui-accessibility-smoke-check`
- `local-stack-smoke-test`
- `release-readiness-safeguard`
- `proxy-error-path-audit`
- `safeguard-policy-fixture-check`

## Verification Workflow
1. Run syntax checks (`node --check proxy-server.js`, `python -m py_compile gpt-oss-safeguard/app.py`).
2. Run targeted skill workflows for changed surfaces.
3. Manually verify browser flows for chat and safeguard.

## Outstanding Tasks
- Add lightweight fixture inputs for verdict split edge cases.
- Add optional Playwright-based regression script for automated browser checks.
- Evaluate whether to package these as a Claude plugin after stabilizing usage.
- Produce final launch artwork exports and localized screenshot/video variants defined in `APP_STORE_LAUNCH_PACK.md`.
- Run final App Store Connect pre-submission QA for readability, accessibility, and metadata consistency.
