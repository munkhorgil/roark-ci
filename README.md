# roark-ci

Scratch repo for exercising the [Roark simulation gate action](https://github.com/roarkhq/simulation-action) the way a customer would.

## What is wired up

`.github/workflows/roark-simulation.yml` runs on every push to `main` and on demand from the Actions tab. It calls the agent with simulated customers, waits for the run, and turns Roark's verdict into the build's exit status.

| Setting | Where it lives | Value |
| :-- | :-- | :-- |
| API key | repo secret `ROARK_API_KEY` | set |
| Run plan | repo variable `ROARK_PLAN_ID` | `e6cd6ee1-ec09-4729-a80c-71ade579b5dd` |
| Action ref | workflow | `roarkhq/simulation-action@main` (no release yet; becomes `@v1`) |

The plan carries two checks, `user_effort_score_check` and `sentiment_score_avg_3_threshold_17`, both held to the 80% default. It is a one-simulation plan, so each check needs its single call to pass.

## Running it

**On merge to main.** Any push to `main` triggers it:

```bash
git commit --allow-empty -m "trigger the gate" && git push
```

**By hand.** Actions → *Roark simulation gate* → **Run workflow**. Two optional inputs:

- `plan-id` to gate on a different plan for one run
- `min-pass-rate` to hold every check to a higher bar for this run only (it tightens, it never loosens)

## Reading the result

| Outcome | Exit | Means |
| :-- | :-- | :-- |
| `PASSED` | green | Every check cleared its own minimum |
| `FAILED` | **red** | The run completed and a check fell short: the agent regressed |
| `SKIPPED` | green, with a warning | The run never produced a judgeable result (a Roark outage, dead simulations, a check that never ran) |

Only `FAILED` blocks a merge, deliberately: if Roark's own problems can turn the build red, people learn to re-run until green and stop reading the gate. Set `fail-on-run-error: true` on the step to block on those too.

A plan with no pass/fail check is the one exception: it fails the build, because a gate that can never fail is worse than a red one nobody notices.
