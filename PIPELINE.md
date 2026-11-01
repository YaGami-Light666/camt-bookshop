# PIPELINE.md — CAMT Bookshop

Student ID: **692131008**

Name: **ZIXIAN SHU**

Repository: <https://github.com/YaGami-Light666/camt-bookshop>

Production URL: <https://camt-bookshop-lz482534829-9647.vercel.app>

## 1. Pipeline at a glance

```mermaid
flowchart LR
  PR[Pull request to main] --> S[smoke: local Chromium smoke]
  S --> DP[deploy-preview: Vercel preview]
  DP --> TP[test-preview: preview Chromium smoke]

  MAIN[Push to main / manual] --> T[test: local Chromium full suite]
  T --> DPROD[deploy-production: Vercel production]
  DPROD --> VP[verify-production: deployed smoke + build SHA]

  NIGHT[03:00 Bangkok / manual target] --> RT[resolve-target]
  RT --> MX[matrix-tests: 3 browsers x 3 shards]
  RT --> Q[quarantine-watch: non-blocking]
  MX --> MR[merge-reports: one HTML artifact]

  DP --> VPREVIEW[(Vercel Preview)]
  DPROD --> VPROD[(Vercel Production)]
```

## 2. Evidence

The account-backed pipeline evidence is recorded below.

| AC | What | Link / screenshot |
|---|---|---|
| AC-03.2 | Vercel deployments were made by Actions | [Vercel deployment list](https://vercel.com/lz482534829-9647/camt-bookshop/deployments) · [restored production deployment run](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41347919423) |
| AC-03.4 | Vercel Authentication on and automation bypass created | [Vercel deployment-protection settings](https://vercel.com/lz482534829-9647/camt-bookshop/settings/deployment-protection) (Vercel Authentication enabled; automation bypass secret configured) |
| AC-04.3–4 | Green PR gate with preview URL | [PR gate run 41345288588](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41345288588) · [Preview](https://camt-bookshop-huxv4f7nf-lz482534829-9647.vercel.app) |
| AC-04.6 | Warm-cache PR duration | [1 min 53 sec](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41347190688) |
| AC-05.1–3 | Branch protection / ruleset | [Active `main` ruleset](https://github.com/YaGami-Light666/camt-bookshop/rules/26325671) |
| AC-05.4 | PR blocked by a red required check | [Closed PR #3](https://github.com/YaGami-Light666/camt-bookshop/pull/3) · [red gate run 41345960029](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41345960029) (`smoke` failed; preview jobs skipped; merge disabled) |
| AC-06.3–4 | Green production deployment and verification | [Main pipeline run 41344916675](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41344916675) |
| AC-06.7 | Red main test with deployment skipped | [PR #4](https://github.com/YaGami-Light666/camt-bookshop/pull/4) · [red main run 41347376968](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41347376968) (`test` failed; both deployment jobs skipped) · [revert PR #5](https://github.com/YaGami-Light666/camt-bookshop/pull/5) · [restored green run](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41347919423) |
| AC-07.2 | Manual nightly runs for production, URL and local | [production 41346389899](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41346389899) · [URL 41346393170](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41346393170) · [local 41346396474](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41346396474) |
| AC-08.2 | Quarantine issue | <https://github.com/YaGami-Light666/camt-bookshop/issues/1> |

## 3. Timings

| Run | Cold cache | Warm cache | Budget | Within? |
|---|---:|---:|---:|---|
| PR gate | 2 min 7 sec | 1 min 53 sec | 8 min | Yes |
| Main pipeline | 2 min 6 sec | 2 min 13 sec | 10 min | Yes |
| Nightly, production | 2 min 47 sec | — | 20 min | Yes |
| Nightly, URL | 3 min 0 sec | — | 20 min | Yes |
| Nightly, local | 3 min 59 sec | — | 20 min | Yes |

Browser-cache saving: **The first green PR gate took 2 min 7 sec and the later PR-42 gate took 1 min 53 sec, an indicative 14-second (about 11%) whole-workflow saving. Deployment variance means this is not a controlled browser-install benchmark.**

Production versus local nightly: **Production took 2 min 47 sec; local took 3 min 59 sec, so local was 1 min 12 sec slower because each matrix runner also built and started the application.**

## 4. Triage record — quarantined test

| | |
|---|---|
| Symptom | The cart sometimes shows `Exchange rate unavailable` instead of the USD total. |
| Local measurement | 7 passed / 3 failed out of 10 Chromium runs |
| Classification | Environment / unreliable external exchange-rate service |
| Deciding evidence | The failing Playwright trace records `GET /api/rates` returning HTTP 503 `rates service unavailable`. |
| Containment | Tagged `@quarantine`, annotated with issue and expiry, excluded from blocking jobs, retained in nightly `quarantine-watch`. |
| Cure | Replace it later with controlled `page.route` success and outage tests. |
| Expiry date | 2026-11-15 |

## 5. The teammate's pull request (PR-42)

1. PR gate prediction: both smoke jobs pass and a preview is deployed because the boundary-price test is not tagged `@smoke`.
2. Main prediction: the full-suite test job fails at `member discount applies at exactly 2,000 THB`; production deployment is skipped.
3. Production prediction: no new Vercel production deployment is created and the previous good build remains live.
4. Restoration plan: use GitHub's Revert action, let the PR gate pass, merge the revert, and verify the next main run deploys.
5. Decision: tag the boundary test `@smoke`; it adds little time and catches this high-value pricing boundary before merge without running the entire suite twice.

Actual result: [PR #4](https://github.com/YaGami-Light666/camt-bookshop/pull/4) passed its PR gate in 1 min 53 sec and was merged. The [following main run](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41347376968) failed in `test` after 58 sec: both the exact-2,000-THB cart assertion and the discounted checkout-total assertion failed, while production deploy and verification were skipped. [Revert PR #5](https://github.com/YaGami-Light666/camt-bookshop/pull/5) passed the gate, and the [restoration run](https://github.com/YaGami-Light666/camt-bookshop/actions/runs/41347919423) completed test, production deployment, and deployed-SHA verification. The chosen prevention was then implemented by tagging the exact-boundary test `@smoke`.

## 6. Think-first answers

1. Predicted 6/10 passes. Actual 7/10. Classification: environment. Evidence: the trace shows `GET /api/rates` returned HTTP 503.
2. `.vercel/` would expose project/account linkage and pulled environment files; `blob-report/` would commit generated mergeable CI reports; `.env` would commit local credentials and configuration.
3. Without `{ retry: 0 }`, `$fetch` retries a failed GET once, so the visible failure probability is `0.35² = 12.25%`. An application retry hides the dependency failure and changes product latency without Playwright marking a flaky attempt; Playwright retries remain visible in reports and traces.
4. No. The password remains in Git history and existing clones. A real team must revoke/rotate it immediately, purge it from history (for example with `git filter-repo`), force-push the repaired history, and tell collaborators to re-clone.
5. With two retries: `0.35³ = 4.2875%`. With five retries: `0.35⁶ ≈ 0.1838%`. Still no: more retries mask a known unreliable dependency, waste time and make a green gate less trustworthy.
6. Vercel's Git integration could deploy the merged bug independently while the main Actions test is red; customers would see the broken build. Disabling automatic Git deployment keeps production behind the tested pipeline.
7. A private window shows Vercel Authentication because the preview is protected. Without a bypass the tests reach that login wall and fail before the app. The choices are disable protection or send the automation-bypass header; PL-03 chooses the bypass so previews stay private.
8. GitHub can offer a required check only after that job name has been registered by a workflow run. If the YAML later becomes invalid, the workflow never starts and the required checks remain expected/pending, so merging stays blocked.
9. The PR gate should pass; the main pipeline should fail. Somchai changed the discount boundary from `>= 2000` to `> 2000`; the non-smoke boundary and discounted-checkout tests catch it, while his four smoke tests do not cover exactly 2,000 THB.
10. Tagging the boundary test `@smoke` is fast and targets a high-risk business rule, but grows the smoke set. Running the full suite on every preview gives broader deployment confidence, but duplicates work and risks the 8-minute PR budget. Decision: tag this boundary test `@smoke`.

## 7. One thing I would change

I would replace the probabilistic exchange-rate test with deterministic contract tests that intercept `/api/rates` for both success and outage responses. I would measure success by a zero flake rate over 100 repeated CI runs while preserving assertions for both user-visible states.
