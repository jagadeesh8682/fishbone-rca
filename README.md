# fishbone-rca

**Root cause analysis for software, the structured way.**
A Claude skill that uses the fishbone (Ishikawa) diagram and 5 Whys to find the *real* root cause of a software problem, fix it, and stop it from coming back.

Paste an error, a stack trace, a failing CI log or describe an incident, and Claude will:

1. Pick **quick mode** (clear, reproducible bug) or **full mode** (incidents, intermittent or unclear problems)
2. Recommend **mitigation first** if production is down, while preserving evidence
3. Write a precise **problem statement**, starting from *"what changed?"*
4. Build a **fishbone** across six software categories
5. **Verify hypotheses with evidence** (git bisect, env diffs, logs) instead of guessing
6. Run **5 Whys**, blamelessly, down to a fixable system gap
7. Deliver the **fix + regression test + guard + alert**
8. Output a short summary or a full **RCA / postmortem report**

## The six software bones

| Bone | Covers |
|---|---|
| Code | Logic, edge cases, nulls, async and concurrency, error handling |
| Data and state | Input shape, schema and migrations, caches, persisted state, time zones |
| Dependencies | Libraries, SDKs, plugins, third-party APIs, version drift |
| Environment | OS, device, runtime, network, resources, containers, CI runners |
| Configuration | Env vars, secrets, flags, build flavors, permissions, certificates |
| Process | Missing tests, review gaps, deploy steps, manual changes |

## Built-in playbooks

Crashes · logic bugs · build failures (Gradle, CocoaPods, pub, npm) · failing and flaky tests · CI/CD (GitHub Actions, self-hosted runners) · works-locally-fails-elsewhere · performance regressions · memory leaks / OOM · Kubernetes (CrashLoopBackOff, ImagePullBackOff, OOMKilled, probes) · APIs and networking · data and migrations · race conditions · Flutter / Android / iOS · web frontend

## Install

### Claude Code (plugin marketplace)

```
/plugin marketplace add jagadeesh8682/fishbone-rca
/plugin install fishbone-rca@jagadeesh-skills
```

### Claude Code (manual)

```bash
git clone https://github.com/jagadeesh8682/fishbone-rca.git
mkdir -p ~/.claude/skills
cp -r fishbone-rca/plugins/fishbone-rca/skills/fishbone-rca ~/.claude/skills/
```

### Claude.ai / Claude desktop app

Download `fishbone-rca.skill` from the [latest release](https://github.com/jagadeesh8682/fishbone-rca/releases/latest) and upload it in the Skills section of Claude's settings.

## Usage

No special command needed. It triggers on things like:

- *"Why is this failing?"* + a stack trace
- *"Find the RCA for this CI failure"* + a log
- *"Pods are in CrashLoopBackOff after the deploy"*
- *"Works on my machine but breaks in release build"*
- *"Write a postmortem for yesterday's outage"*

## Repository layout

```
.claude-plugin/marketplace.json          # marketplace catalog
plugins/fishbone-rca/
├── .claude-plugin/plugin.json           # plugin manifest
└── skills/fishbone-rca/
    ├── SKILL.md                         # the workflow
    └── references/
        ├── categories.md                # common causes per bone
        └── playbooks.md                 # per-problem-type checks and commands
```

## Contributing

Contributions are welcome, especially new playbooks (Go, Rust, .NET, databases, cloud providers) and real-world RCA examples. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Author

Built by **Jagadeesh** · [LinkedIn](https://www.linkedin.com/in/jagadeesh8682/) · [GitHub](https://github.com/jagadeesh8682)

## License

[MIT](LICENSE) © 2026 Jagadeesh
