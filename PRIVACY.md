# Privacy Policy

**Drydock does not collect, store, transmit, or sell any personal data or telemetry.**

Drydock is a local Claude Code plugin. It runs entirely inside your own Claude Code session, on your own machine. The plugin itself contains no analytics, tracking, telemetry, or "phone-home" behavior of any kind.

## What stays on your machine

- **Your code and prompts** never leave your environment because of Drydock. They are processed by Claude Code / the Anthropic API exactly as they would be without the plugin — Drydock adds no additional collection or transmission.
- **The `drydock/` workspace** (architecture docs, receipts, pipeline state, compound-learning notes) is written to your project directory and stays there. Nothing is uploaded.
- **Hooks** (`secret-guard.sh`, `session-guard.sh`) run locally. `secret-guard` scans your staged changes to **block** secrets from being written or committed — it does not record, copy, or transmit any secret it finds.

## Network access

Drydock makes no network calls on its own. Network activity only happens through the standard tools the agents use **at your request** during a run — for example:

- `WebSearch` to verify volatile facts (model IDs, CVEs, package versions) before implementing.
- External security/quality tools the pipeline may invoke if you have them installed (e.g. `semgrep`, `trivy`, `gitleaks`, OWASP ZAP).

Any data handling by those third-party tools, by Claude Code, or by the Anthropic API is governed by their own respective privacy policies, not this one.

## No accounts, no third-party sharing

Drydock has no accounts, no servers, and no backend. It shares nothing with the author or any third party.

## Changes

If this policy ever changes, the update will be committed to this file in the repository's history.

## Contact

Questions about this policy or about security:

- Open a [GitHub Security Advisory](https://github.com/sundarshahi/drydock/security/advisories/new), or
- Email **shahithakurisundar@gmail.com**.
