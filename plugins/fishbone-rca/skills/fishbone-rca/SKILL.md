---
name: fishbone-rca
description: Root cause analysis (RCA) and resolution for any software problem, using a fishbone (Ishikawa) diagram plus 5 Whys, then fixing the root cause and preventing recurrence. Use this whenever the user is debugging or troubleshooting code or systems - bugs, crashes, exceptions, stack traces, failing or flaky tests, build or compile errors, CI/CD pipeline failures, deployment or Kubernetes issues, performance regressions, memory leaks, "works locally but not in prod", API or integration failures, data problems, race conditions, mobile (Flutter, Android, iOS) or web frontend issues, and production incidents or postmortems. Trigger even when the user just pastes an error log or says "why is this failing", "find the root cause", "RCA", "fix this bug", "debug this" or "write a postmortem", without mentioning fishbone.
license: MIT
metadata:
  author: Jagadeesh
  repository: https://github.com/jagadeesh8682/fishbone-rca
  version: 1.0.0
---

# Fishbone RCA for software

Goal: find the real root cause of a software problem with evidence, fix it, and make sure it cannot quietly come back. A symptom fix ("added a null check") is never the whole answer on its own; always ask why the bad state was possible in the first place.

## Pick the mode first

**Quick mode** is for a single, reproducible bug with clear evidence: a stack trace pointing at a line, a compile error, a failing assertion. Skip the full diagram and do: problem, evidence, cause, one or two whys, fix, regression test. Keep the output short.

**Full mode** is for anything intermittent, production incidents, regressions with an unclear cause, "works on my machine", problems with several plausible causes, explicit requests for an RCA or postmortem, or when quick mode did not find the cause.

When unsure, start in quick mode and escalate to full mode as soon as the first hypothesis fails to hold up.

## Step 0: Stop the bleeding (production only)

If users are affected right now, recommend mitigation before investigation: roll back, turn the feature flag off, scale up, fail over, or revert the hotfix. Preserve evidence first where possible (logs, heap dumps, `kubectl describe` output, pod state), because restarting or redeploying often destroys it. Record what was done and when, then do the RCA.

## Step 1: Define the problem

Write a one-paragraph problem statement covering:

- **What**: the exact symptom and error text, expected versus actual behaviour.
- **Where**: service, module, screen or job; environment (local, CI, staging, prod); platform and version.
- **When**: first seen, frequency (always or intermittent, roughly what %), since which deploy, commit or date.
- **Impact**: who is affected and how badly.
- **What changed**: deploys, dependency bumps, config, infra, data, traffic, OS or device updates, and time-based triggers (DST, certificate expiry, month end, token expiry).

"What changed?" is the highest-yield question in software RCA, because most regressions trace back to a recent change. Treat "nothing changed" as a claim to verify: lockfiles, base images, runner images, managed services, certificates and data all change without anyone touching the code.

Gather evidence: the full error text, the complete stack trace including every "Caused by" section, logs around the timestamp, the relevant code, versions (lockfiles), config, and recent git history. If key information is missing, ask for all of it in one message and give the exact command to get each item (see `references/playbooks.md`). If there is already enough to form hypotheses, proceed and state assumptions explicitly.

## Step 2: Build the fishbone

Use these six software bones. For the checklist of common causes on each bone, read `references/categories.md`. For the problem type (crash, CI failure, Kubernetes, flaky test, and so on), read the matching section of `references/playbooks.md`.

| Bone | Covers |
|---|---|
| Code | Logic, edge cases, nulls and types, async and concurrency, error handling, recent commits |
| Data and state | Input shape, schema and migrations, caches, stale or persisted state, encoding, time zones |
| Dependencies | Libraries, SDKs, plugins, third-party APIs, internal services, version drift |
| Environment | OS, device, runtime version, network, CPU, memory, disk, containers, CI runners |
| Configuration | Env vars, secrets, feature flags, build flavors, infra-as-code, permissions, certificates |
| Process | Missing tests, review gaps, deploy steps, manual changes, unclear ownership or docs |

For each relevant bone, list two to four concrete candidate causes specific to this problem, not generic ones. For a bone that clearly does not apply, say so in one line rather than padding it.

**Displaying the fishbone**: if a diagram or visualization tool is available, render it with the problem at the head and the six bones. Otherwise use this text format:

```
PROBLEM: <one-line problem statement>
├─ Code:          <cause> [?] · <cause> [?]
├─ Data/state:    <cause> [?]
├─ Dependencies:  <cause> [?]
├─ Environment:   <cause> [?]
├─ Configuration: <cause> [?]
└─ Process:       <cause> [?]
```

Status markers: `[?]` unverified, `[✓]` confirmed, `[✗]` ruled out. Update them as evidence comes in.

## Step 3: Rank and verify hypotheses

Rank candidates by likelihood given the evidence and by how cheap they are to check, and check cheap ones first. Anything that lines up with "what changed" moves to the top.

For each top hypothesis, write a prediction: "If X is the cause, then Y should be true." Then pick a check that would confirm it or rule it out:

- Reproduce with a minimal case.
- `git bisect` between a known good and a known bad commit.
- Diff config, env vars and versions between a working and a broken environment.
- Add targeted logging or assertions around the suspect path.
- Toggle the suspect: revert the commit, pin the old version, disable the flag.
- Correlate timestamps of the failure with deploys, config changes and metrics.

If you can run code (sandbox or repo access), run the checks yourself. If not, give the user exact, copy-pasteable commands and say what output would confirm or rule out each hypothesis, then wait for the results.

Do not declare a root cause without at least one piece of confirming evidence. If confirmation is impossible, label it "most likely cause (unconfirmed)" and say what would confirm it.

Incidents often need two or three things to go wrong together. Separate the **trigger** (what set it off), the **root cause** (the underlying defect) and **contributing factors** (what made it worse or let it through).

## Step 4: 5 Whys on the confirmed cause

Starting from the confirmed technical cause, ask "why?" repeatedly until you reach something systemic you can change. This usually takes three to five levels. Stop when the answer is a design or process gap with an actionable fix.

Keep it blameless. "The developer forgot" is never a root cause; ask why the system allowed the mistake (no test, no type check, no lint rule, unclear API, no review step).

Example:
1. The app crashes on the profile screen → `user.address` is null.
2. Why is it null? → The API returns no address for users who signed up with Google.
3. Why doesn't the app handle that? → The model treats `address` as required and parsing force-unwraps it.
4. Why wasn't it caught? → Test fixtures only contain email-signup users.
5. Why? → There is no contract between the API schema and the client model.

Root cause: the client model drifted from the API contract, and nothing checks that contract.

## Step 5: Resolve

Deliver three layers:

1. **Fix**: the actual code or config change, as a concrete diff or a full replacement block, minimal and focused. Explain why it addresses the root cause rather than silencing the symptom.
2. **Prevent**: a regression test that fails before the fix and passes after it, plus a guard where it fits (type or lint rule, CI check, schema or contract validation, pinned version).
3. **Detect sooner** (for production issues): the log line, metric or alert that would have surfaced this earlier.

Avoid fixes that only hide symptoms: blanket try/catch, infinite retries, raising timeouts without knowing why, force-unwrapping, skipping or deleting tests, `--force` installs. If a temporary workaround is genuinely needed, label it as a workaround and list the real fix as a follow-up action.

## Step 6: Verify the fix

- Reproduce the original failure, apply the fix, and confirm it is gone.
- The new regression test passes, and so does the existing suite.
- For intermittent problems, run the scenario many times in a loop or under load, and for production fixes define a monitoring window after deploy.
- Check that the fix did not move the problem to another bone (performance, memory, another platform).

## Output format

**Quick mode:**

```
**Problem:** <one line>
**Root cause:** <one or two lines> (evidence: <what confirmed it>)
**Why it happened:** <one or two whys>
**Fix:** <diff or code>
**Regression test:** <code>
**Verify:** <exact command(s)>
```

**Full mode** (RCA report):

```
# RCA: <short title>
## Summary            (2-3 lines: what broke, root cause, fix)
## Problem statement
## Timeline           (prod incidents: detected, mitigated, resolved)
## Fishbone           (diagram or text format)
## Hypotheses tested  (table: hypothesis | check | result)
## Root cause and 5 Whys
## Contributing factors
## Resolution         (fix, prevention, detection)
## Verification
## Action items       (action | owner | due)
```

Keep the report as short as the problem allows. All commands and code must be copy-pasteable.

## Principles

- Evidence over intuition; cheapest check first.
- Change one variable at a time.
- Read the whole error, including every "Caused by" in the chain; the deepest cause is often the real one.
- Verify "nothing changed" instead of accepting it.
- Stay blameless and system-focused.
- If you are stuck after the obvious checks, go back to the fishbone and look at the bones you dismissed.
