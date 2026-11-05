# Playbooks by problem type

Read only the section that matches the problem. Each section lists the signals, the bones to start with, the first checks (with commands), and the usual root causes. Replace `<placeholders>` before giving commands to the user.

## Contents
1. Crash or unhandled exception
2. Wrong output or logic bug
3. Build or compile failure
4. Failing or flaky tests
5. CI/CD pipeline failure
6. Works locally, fails elsewhere
7. Performance regression
8. Memory leak or out-of-memory
9. Deployment and Kubernetes failures
10. API, integration and network failures
11. Data and migration problems
12. Concurrency and race conditions
13. Mobile (Flutter, Android, iOS)
14. Web frontend
15. Universal tools (git, logs)

---

## 1. Crash or unhandled exception

**Signals**: stack trace, app closes, 500 errors, process exits non-zero.
**Start with**: Code, Data and state.

First checks:
- Read the full trace bottom-up through every "Caused by"; find the first frame in the project's own code.
- Identify the exact input or state at the crash (log the inputs around that frame).
- Check whether the crashing line or its callers changed recently: `git log -p -L <start>,<end>:<file>`

Usual root causes: null or missing data from an upstream source, unhandled error path, lifecycle misuse (using an object after dispose), type mismatch after an API change.

## 2. Wrong output or logic bug

**Signals**: no error, but results are wrong.
**Start with**: Code, Data and state.

First checks:
- Write the smallest failing test that asserts the expected value.
- Trace the value through the pipeline with logging or a debugger and find the first step where it goes wrong.
- Check boundary inputs: empty, zero, negative, max, duplicates, unusual characters.
- Find when it last worked and bisect (see section 15).

Usual root causes: wrong condition, off-by-one, rounding or precision, time zone conversion, stale cache, wrong assumption about input ordering.

## 3. Build or compile failure

**Signals**: build stops, dependency resolution errors, linker or signing errors.
**Start with**: Dependencies, Environment, Configuration.

First checks:
- Find the first error in the log, not the last; later errors are usually fallout.
- Compare toolchain versions with the last green build.
- Clean and rebuild to rule out stale artifacts.

Commands:
```bash
# Node
npm ls <package> ; npm explain <package> ; rm -rf node_modules && npm ci
# Flutter / Dart
flutter doctor -v ; flutter pub deps ; flutter pub outdated
flutter clean && flutter pub get
# Android / Gradle
cd android && ./gradlew assembleDebug --stacktrace --info
./gradlew app:dependencies --configuration releaseRuntimeClasspath
# iOS / CocoaPods
cd ios && pod deintegrate && pod install --repo-update
# Python
pip freeze ; pip check
```

Usual root causes: unpinned or transitive dependency update, lockfile out of sync, toolchain upgrade (Gradle, AGP, Kotlin, Xcode, JDK), min SDK or deployment target mismatch, cached artifacts.

## 4. Failing or flaky tests

**Signals**: test fails consistently after a change, or passes and fails without code changes.
**Start with**: Code, Environment, Process.

First checks:
- Consistent failure: did the test or the code under test change? Is the test asserting the right thing?
- Flaky: run it in a loop to measure the failure rate, then run it alone and in random order.

Commands:
```bash
# Loop a test until it fails (adjust the command)
for i in $(seq 1 50); do <test-command> || { echo "failed on run $i"; break; }; done
# Examples of a single-test command
flutter test test/<file>_test.dart --plain-name "<test name>"
npx jest <file> -t "<test name>"
pytest <file>::<test> -x
```

Usual root causes of flakiness: shared state between tests, order dependence, real time or timers (use fake clocks), network calls in tests, unawaited async work, animations in widget tests, random data without a fixed seed, parallel tests sharing resources.

## 5. CI/CD pipeline failure

**Signals**: passes locally, fails in CI; a pipeline step fails; deploy job breaks.
**Start with**: Environment, Configuration, Dependencies.

First checks:
- Compare the last green run with the first red run: commit diff, runner image version, action versions, cache keys.
- Re-run with debug logging.
- For self-hosted runners, check for leftover state from previous jobs (workspace, Docker images, disk space).

Commands:
```bash
gh run list --limit 10
gh run view <run-id> --log-failed
gh run rerun <run-id> --debug
# Self-hosted runner health
df -h ; docker system df ; free -m
```
In GitHub Actions you can also set the repository secret or variable `ACTIONS_STEP_DEBUG` to `true` for step-level debug logs.

Usual root causes: hosted runner image updated, an action referenced by a moving tag (`@v4`, `@main`) changed, missing or expired secret, cache restoring stale or corrupt artifacts, disk full on self-hosted runner, network or registry rate limits, concurrency between jobs deploying the same target.

## 6. Works locally, fails elsewhere

**Signals**: fine on the developer machine, broken in CI, staging, prod or on a user device.
**Start with**: Environment, Configuration, Data and state.

First checks: build a side-by-side diff of the two environments.
- Runtime and dependency versions (lockfile honoured in both?)
- Env vars and secrets (names, not values)
- Build mode (debug versus release, minification, obfuscation)
- Data: local seed data versus real data
- OS, architecture, locale, time zone, file-system case sensitivity
- Network access (internal hosts, proxies, firewalls)

Usual root causes: missing env var or secret, release-only build settings, different data, case-sensitive paths, a dependency resolved to a different version.

## 7. Performance regression

**Signals**: slower responses, jank, higher CPU, timeouts after a change.
**Start with**: Code, Data and state, Environment.

First checks:
- Quantify it: before and after numbers for the same scenario.
- Profile instead of guessing; find the hot path.
- Correlate with deploys, data growth and traffic changes.

Commands and tools:
```bash
curl -o /dev/null -s -w "dns:%{time_namelookup} connect:%{time_connect} ttfb:%{time_starttransfer} total:%{time_total}\n" <url>
kubectl top pod -n <namespace>
py-spy top --pid <pid>
```
Also: database `EXPLAIN` or `EXPLAIN ANALYZE` on slow queries; Flutter DevTools Performance view (profile mode, not debug); browser DevTools Performance tab; Node `--cpu-prof`.

Usual root causes: N+1 queries, missing index, data volume growth, unbounded loops or rebuilds, synchronous work on the UI or main thread, cache disabled or always missing, chatty network calls, CPU throttling from container limits.

## 8. Memory leak or out-of-memory

**Signals**: memory climbs over time, OOMKilled pods, app killed in background, slowdown over a session.
**Start with**: Code, Environment, Configuration.

First checks:
- Confirm the growth pattern: steady climb (leak) versus spikes (large allocations).
- Take two heap snapshots some time apart and compare what grew.

Tools:
```bash
# Kubernetes: was it OOMKilled?
kubectl describe pod <pod> -n <namespace> | grep -A5 "Last State"
# JVM
jmap -dump:live,format=b,file=heap.hprof <pid>
# Node
node --inspect <app>   # then take heap snapshots in Chrome DevTools
```
Also: Flutter DevTools Memory view, Android Studio Memory Profiler, Xcode Instruments (Leaks, Allocations).

Usual root causes: listeners, subscriptions, streams, timers or controllers never disposed; growing caches without eviction; holding references to large objects or contexts; loading full images instead of resized ones; memory limit set below real need.

## 9. Deployment and Kubernetes failures

**Signals**: CrashLoopBackOff, ImagePullBackOff, Pending pods, failed rollouts, readiness probe failures.
**Start with**: Configuration, Environment, Dependencies.

Commands:
```bash
kubectl get pods -n <namespace> -o wide
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl get events -n <namespace> --sort-by=.lastTimestamp
kubectl rollout status deployment/<name> -n <namespace>
kubectl rollout history deployment/<name> -n <namespace>
kubectl rollout undo deployment/<name> -n <namespace>   # mitigation
```

Map the state to likely causes:
- **CrashLoopBackOff**: app exits on start (missing env var or secret, bad config, failed migration, crash). Check `logs --previous`.
- **ImagePullBackOff**: wrong image tag, registry auth (`imagePullSecrets`), registry down, architecture mismatch.
- **Pending**: insufficient CPU or memory, node selectors or taints, unbound PersistentVolumeClaim.
- **OOMKilled**: memory limit too low or a leak (section 8).
- **Readiness or liveness failing**: wrong path or port, probe too aggressive for slow start, dependency not reachable.

## 10. API, integration and network failures

**Signals**: 4xx or 5xx from another service, timeouts, connection refused, TLS errors, CORS errors.
**Start with**: Dependencies, Configuration, Environment.

Commands:
```bash
curl -v <url>
dig +short <host>
echo | openssl s_client -connect <host>:443 -servername <host> 2>/dev/null | openssl x509 -noout -dates -subject
nc -vz <host> <port>
```

Read the status code first:
- **401 or 403**: token expired, wrong scope, rotated key, clock skew.
- **404**: wrong base URL or path, API version removed.
- **429**: rate limit; check retry and backoff behaviour.
- **5xx**: upstream problem; check their status page and your request payload.
- **Timeouts**: network path, DNS, slow upstream, timeout set too low, connection pool exhausted.

## 11. Data and migration problems

**Signals**: corrupted or missing records, errors only for some users, migration failures.
**Start with**: Data and state, Process, Code.

First checks:
- Find which records are affected and what they have in common (created when, by which version, which path).
- Check migration order relative to the deploy, and whether it ran in every environment.
- Back up before any repair; write repair scripts as idempotent and test them on a copy.

Usual root causes: migration and code deployed in the wrong order, non-backward-compatible schema change, missing validation at the write path, partial writes without transactions, duplicate event processing.

## 12. Concurrency and race conditions

**Signals**: intermittent, timing-dependent, disappears when you add logging, only under load.
**Start with**: Code, Environment.

First checks:
- Increase concurrency or add artificial delays to make it reproducible.
- Identify shared mutable state and check-then-act sequences.
- Log with timestamps and thread, isolate or request IDs to reconstruct ordering.

Usual root causes: shared state without locking, double-submit, out-of-order async responses overwriting newer state, missing idempotency, non-atomic read-modify-write in the database.

## 13. Mobile (Flutter, Android, iOS)

**Signals**: platform-specific crash, release-only bug, device-specific behaviour, store build rejected.
**Start with**: Configuration, Environment, Dependencies.

Commands:
```bash
flutter doctor -v
flutter run --release            # reproduce release-only issues
flutter run --verbose
adb logcat *:E                   # Android errors
adb logcat | grep -i <tag>
xcrun simctl spawn booted log stream --level error   # iOS simulator logs
```

Usual root causes:
- **Release-only crash**: R8 or ProGuard stripping classes used via reflection or platform channels (needs keep rules); obfuscation without symbol upload; asserts that only run in debug.
- **Platform channel errors** (`MissingPluginException`): plugin not registered, hot restart needed after adding a plugin, method name mismatch.
- **Permissions**: missing AndroidManifest entries or Info.plist usage descriptions; runtime permission not requested.
- **Version-specific**: behaviour changes in new Android API levels or iOS versions (background limits, storage access, notification permissions).
- **Lifecycle**: `setState` or context used after dispose, work continuing in the background.
- **Build**: Gradle, AGP, Kotlin or Xcode version mismatch with plugins; CocoaPods out of date.

## 14. Web frontend

**Signals**: blank page, works after hard refresh, works in one browser only, console errors.
**Start with**: Configuration, Environment, Data and state.

First checks:
- Browser console and Network tab: first error, failed requests, status codes.
- Hard refresh and incognito to rule out cache or service worker.
- Compare the deployed bundle version with the expected one.

Usual root causes: stale cached bundle or service worker, SPA routing not configured on the server (404 on refresh), wrong base path or env var baked in at build time, CORS misconfiguration, hydration mismatch, browser-specific APIs.

## 15. Universal tools (git, logs)

```bash
# What changed recently?
git log --since="3 days ago" --oneline
git diff <good-sha>..<bad-sha> --stat
git diff <good-sha>..<bad-sha> -- '*lock*' 'pubspec.lock' 'package-lock.json' 'yarn.lock'
# Who or what changed a specific string or function?
git log -S'<string>' --oneline
git log -p -L <start>,<end>:<file>
# Binary search for the breaking commit
git bisect start
git bisect bad <bad-sha>
git bisect good <good-sha>
# test, then mark each step: git bisect good | git bisect bad
git bisect run <test-script>     # automate if a script exits non-zero on failure
git bisect reset
```

Log tips: filter by request or correlation ID; search a window around the first failure, not just the latest; compare log volume and error rates before and after the suspected change.
