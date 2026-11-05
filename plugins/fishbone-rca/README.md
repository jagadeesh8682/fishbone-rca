# fishbone-rca

Root cause analysis for software, the structured way. This plugin adds a skill that makes Claude find the real root cause of a software problem using a fishbone (Ishikawa) diagram and the 5 Whys, then fix it and stop it from coming back.

Paste an error, a stack trace, a failing CI log, or describe an incident. Claude will define the problem starting from "what changed?", map possible causes across six categories (code, data and state, dependencies, environment, configuration, process), verify each hypothesis with evidence instead of guessing, run 5 Whys down to a fixable gap, and deliver a fix, a regression test, a guard, and an alert. It can also write a full RCA or postmortem report.

It includes playbooks for crashes, logic bugs, build failures, flaky tests, CI/CD pipelines, Kubernetes, performance regressions, memory leaks, APIs and networking, data migrations, race conditions, Flutter, Android, iOS, and web frontends.

## Privacy and data

This plugin contains only Markdown instructions. It runs no code, installs no packages, connects to no servers, and sends no data anywhere. It does not read or store personal data.

## Usage

No command is needed. It triggers when you debug something, for example "why is this failing?" with a stack trace, "find the RCA for this CI failure", or "write a postmortem for yesterday's outage".

## Author and license

Built by Jagadeesh (https://github.com/jagadeesh8682). MIT licensed.
