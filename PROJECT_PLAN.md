# Project Plan — Safeguard (`ux-oss-safeguard`)

Last refreshed by a full `/team` review.

## Current Objective

Position Safeguard as a public model demo and consulting calling card
for OpenAI's `gpt-oss-safeguard-20b` — a transparent, policy-as-prompt
moderation interface that competing APIs do not expose. Keep the
codebase small, observable, and defensible while the public deploy at
`https://dr.eamer.dev/io/safeguard/` runs.

## Architecture Snapshot

- `proxy-server.js` — Zero-dep Node HTTP server on port `3456`.
  Serves the HTML frontends, exposes `/health` and a fake
  `/api/tags`, and translates an Ollama-shaped `/api/chat` request
  to HuggingFace's OpenAI-compatible `/v1/chat/completions`
  endpoint (SSE → newline-delimited JSON).
- `safeguard.html` — Policy evaluator SPA. Splits the streamed
  response on `assistantfinal` into Analysis and Verdict panels,
  decodes structured labels (`D-SP4.a` → "Phishing · Fraud &
  Deception · Flagged for review").
- `ollama-chat.html` — Companion chat UI sharing the proxy; streams
  `message.content` and live `message.thinking`.
- `gpt-oss-safeguard/app.py` — Gradio variant. **Currently targets a
  local Ollama endpoint with model `glm-4.7-flash`, not the
  safeguard model.** Decision pending — see Open Decisions below.

## Positioning

Audience: trust-and-safety engineers, product managers, and ML
practitioners evaluating policy-as-prompt moderation as an alternative
to fixed-category APIs (OpenAI Moderation, Perspective, Azure AI
Content Safety, AWS Bedrock Guardrails).

Wedge: visible reasoning trace and a user-editable policy textarea.
Competing commercial APIs return a categorical score; Safeguard shows
the model thinking and then decodes its verdict.

Monetization: not from the app itself. Safeguard is top-of-funnel for
consulting inquiries (`luke@lukesteuber.com`), newsletter reach, and
the broader IO Suite at `dr.eamer.dev/io/`. Any paid tier in the
future ("managed policy packs + audit logs") is a fork, not this
repo.

## Cursor Skill Pack

Eight project-scoped skills in `.cursor/skills/` standardize recurring
workflows so we don't reinvent verification every time we touch the
codebase:

- `proxy-cors-regression-check`
- `streaming-jsonline-debugger`
- `safeguard-verdict-split-validator`
- `ui-accessibility-smoke-check`
- `local-stack-smoke-test`
- `release-readiness-safeguard`
- `proxy-error-path-audit`
- `safeguard-policy-fixture-check`

These are explicit-invocation only (`disable-model-invocation: true`).
Run them with `/skill-name` in Cursor chat.

## Verification Workflow

1. Syntax checks: `node --check proxy-server.js` and
   `python -m py_compile gpt-oss-safeguard/app.py`.
2. Run the relevant Cursor skill(s) for changed surfaces.
3. Manually verify the browser flows for chat and safeguard against
   `dr.eamer.dev/io/safeguard/` in both light and dark modes.

## Risks (current)

1. **HuggingFace inference quota drain.** `/api/chat` has no auth,
   no rate limit, and accepts any client-provided `max_tokens` via
   `options.num_predict`. A motivated griefer in the CORS allowlist
   can drain the HF token's quota and the public deploy goes silent
   with no fallback.
2. **Verdict-screenshot laundering.** The UI does not display a
   policy fingerprint, so a screenshot of "PASS" with a custom
   permissive policy can look like a model endorsement of harmful
   content. Trivial to mitigate; meaningful to leave open.
3. **App Store rejection risk.** No native iOS code exists. A
   WKWebView wrapper of the public URL is exposed to Apple 4.2
   (Minimum Functionality) / 4.3 (Spam) rejections. See
   `APP_STORE_LAUNCH_PACK.md`.
4. **Verdict split brittleness.** No automated test asserts that
   `assistantfinal` splitting handles partial-chunk boundaries or
   that `extractVerdict` covers new severity codes. Two regressions
   are likely without coverage.
5. **Trademark surface near "OpenAI".** OG title currently reads
   "OpenAI OSS-Safeguard" and the repo contains a fork of OpenAI's
   HuggingFace Space. Apple App Store reviewers flag co-brand naming.

## Kill / Pivot / Sunset Criteria

- **Web ship-blocker.** If `/api/chat` is exercised by a
  non-allowlisted origin in production without being blocked, pause
  the public deploy until rate-limiting lands in front of it.
- **Pivot trigger.** If no T&S team has run a real-world policy
  through the public deploy 90 days after launch, drop the "product"
  framing and rebrand as model demo / blog companion.
- **iOS sunset trigger.** If the iOS wrapper isn't started within 60
  days of `APP_STORE_LAUNCH_PACK.md` landing on `main`, delete the
  launch pack and strip iOS framing from public docs.

## Open Decisions

- **`gpt-oss-safeguard/app.py`.** Route through the proxy (gain
  portability, lose direct-Ollama development convenience) or
  delete it and link to OpenAI's canonical HuggingFace Space. Also
  prune `requirements.txt` to the deps the code actually imports
  (`gradio`, `requests`).
- **Naming.** Pick one product display name across `package.json`
  (`ux-oss-safeguard`), README ("Safeguard"), and OG tags
  ("OpenAI OSS-Safeguard"). Recommend "Safeguard · powered by
  `gpt-oss-safeguard-20b`".

## Outstanding Tasks

- [ ] Clamp `max_tokens` server-side in `proxy-server.js`
      (`Math.min(num_predict, 4096)` or similar).
- [ ] Add per-IP rate limiting in front of `/api/chat`.
- [ ] Display a policy fingerprint (hash of the active policy) next
      to the verdict so screenshots cannot launder a permissive
      policy as a model endorsement.
- [ ] Add lightweight fixture inputs for verdict-split edge cases
      (missing `assistantfinal`, partial chunk boundary,
      `D-SP4.a+ESCALATE` decoding).
- [ ] Add a minimal GitHub Action that runs `node --check` and
      `python -m py_compile` on PR.
- [ ] Resolve the `app.py` and naming decisions above.
- [ ] Decide whether to keep, shrink, or delete
      `APP_STORE_LAUNCH_PACK.md` — the sunset trigger above gives
      this a 60-day clock once the doc lands on `main`.
- [ ] Squash or rewrite the `session checkpoint` commits in history
      before any GitHub release tag is cut.

## Done

- Migrated the proxy from Ollama-only to HuggingFace Inference API
  while preserving the Ollama-shaped wire protocol for the
  frontends.
- Added the 8-skill Cursor pack for repeatable verification.
- Drafted `APP_STORE_LAUNCH_PACK.md` and the `app-store/` asset
  workspace.
- Refreshed workspace-root `CLAUDE.md` and `AGENTS.md` so they
  reflect the current HuggingFace stack on port `3456`.
