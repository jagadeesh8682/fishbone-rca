# Fishbone bones: common software causes

Use these as prompts when filling each bone. Pick only what fits the problem; never paste the whole list into the output.

## Contents
- Code
- Data and state
- Dependencies
- Environment
- Configuration
- Process

## Code

- Null, undefined or optional values not handled; force-unwrap (`!`) on data that can be missing
- Wrong type assumptions, implicit casts, integer overflow, float precision
- Off-by-one, boundary values, empty collections, first and last item
- Wrong condition or operator, inverted boolean, missing `else` or `default` branch
- Async mistakes: missing `await`, unhandled promise or future, callback after dispose or unmount
- Concurrency: race conditions, shared mutable state, deadlocks, non-atomic check-then-act
- Error handling: swallowed exceptions, catch-all hiding the real error, wrong retry logic
- Resource leaks: unclosed streams, listeners, subscriptions, controllers, connections
- State management: stale state, rebuild or re-render loops, lifecycle ordering
- Recent refactor that changed behaviour (check `git log -p` on the file)
- Copy-paste errors, dead code paths suddenly reached

## Data and state

- Unexpected input shape: missing fields, extra fields, nulls, empty strings
- Schema or migration mismatch between code and database
- API response format changed; client model out of sync with server contract
- Cache serving stale or corrupt data (CDN, Redis, in-memory, HTTP cache, service worker)
- Persisted local state from an older app version (SharedPreferences, SQLite, localStorage)
- Encoding: UTF-8 versus other encodings, emoji, RTL text, BOM
- Time: time zones, DST, locale date formats, clock skew, leap years, epoch seconds versus milliseconds
- Large or unusual data volumes: pagination edges, very long strings, huge payloads
- Duplicate or out-of-order events and messages
- Data written by a buggy older version still present

## Dependencies

- Library upgraded (directly or transitively) with breaking or changed behaviour
- Version ranges (`^`, `~`, `latest`) resolving to a new version without a code change
- Lockfile not committed or not respected in CI
- Conflicting versions of the same dependency
- Plugin or native SDK incompatible with the framework or platform version
- Third-party API changed, deprecated an endpoint, rate-limited, or is down
- Internal downstream service degraded, slow or returning errors
- Package registry, mirror or artifact repository outage
- Deprecated API removed in a new runtime or OS version

## Environment

- Runtime version differs (Node, Python, Java, Dart or Flutter SDK, JDK, Xcode)
- OS or device differences: OS version, CPU architecture (arm64 versus x86_64), screen size, locale
- Network: DNS, proxy, firewall, VPN, TLS interception, IPv6, timeouts, flaky connectivity
- Resource limits: memory, CPU throttling, disk full, file descriptors, inode exhaustion
- Container differences: base image updated, missing system libraries, different user or permissions
- CI runner differences: hosted runner image updated, self-hosted runner dirty state, cache poisoning
- Clock and time zone of the machine
- Load: traffic spike, noisy neighbour, autoscaling lag
- Case-sensitive versus case-insensitive file systems; path separators

## Configuration

- Missing, wrong or misspelled environment variable
- Secret rotated, expired or not present in that environment
- Feature flag in a different state than expected
- Wrong build flavor, scheme, profile or mode (debug versus release)
- Release-only settings: minification or obfuscation (R8, ProGuard), tree shaking, AOT compilation
- Infra-as-code drift, or a manual change made outside it
- IAM, RBAC or file permissions changed
- Certificate expired or wrong chain
- CORS, CSP or allowed-hosts settings
- Kubernetes: resource requests and limits, probes, ConfigMap or Secret not mounted, wrong image tag
- Mobile: AndroidManifest permissions, Info.plist keys, entitlements, signing, min SDK or deployment target

## Process

- No test covering this path, or tests use unrealistic fixtures
- Flaky tests ignored or retried until green, hiding real failures
- Change merged without review, or the review missed context
- Manual deploy step skipped or done out of order
- Migration run in the wrong order relative to the deploy
- Hotfix made directly in production and never back-ported
- Missing monitoring or alerts, so the problem was found late
- Unclear ownership between teams or services
- Documentation out of date, so someone followed old instructions
- Time pressure leading to skipped checks (note it blamelessly, as a system gap)
