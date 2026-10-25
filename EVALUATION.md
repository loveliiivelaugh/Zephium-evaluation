# Evaluation status: installation and execution paused

This is Michael Woodward's evaluation fork of [Zephium](https://github.com/zephium-browser/Zephium). It exists to retain a reviewable baseline and track adoption blockers. It is not a security certification, endorsed distribution, or approved daily-use browser.

As of October 25, 2026, the decision is **HARD PAUSE on installation, builds, and execution**. Creating this fork does not lift that pause. Further runtime evaluation requires an explicit decision to resume and an isolated environment.

## Baseline

- Upstream main: `0eaad6d79703ca53c1f77201f7a396dad47552ef`.
- Release reviewed: `v1.0.3`, source commit `e3d3337469e5a7e87c050f777f26f9220f5ac702`.
- Review type: focused source inspection, public CI/issue review, and downloaded Mac artifact validation. No application code or repository scripts were executed, no dependencies were installed, and no browser profiles or credentials were imported.
- GitHub Actions is disabled at the repository level. Do not enable inherited workflows or provide signing/deployment credentials as part of ordinary evaluation.
- No automatic upstream synchronization or release publishing has been configured. Reassess each candidate commit deliberately.

## Blocker 1: installer provenance discrepancy

The Mac v1.0.3 installer downloaded on October 25 matches that release's published checksum and passes Apple code-signature, Gatekeeper, and stapled-notarization checks. However, its current digest has no matching GitHub build attestation; verification returned HTTP 404.

Current `Zephium-macOS-arm64.dmg` SHA-256:

`348bceea73c76ac060c0549fd474c936ed6ae9ec3ea729d441ff53509b4b3b3f`

The verified SLSA statement retrieved for the updater archive lists this different DMG digest:

`50286372277d6eb55c91ac84ea4ff20d43a8ffda355898f42879f889700d2700`

The updater archive itself matches its published checksum and successfully verifies against the release workflow and source commit above:

`Zephium-macOS-arm64.app.tar.gz`: `5cb2e413cc1aed92396e6c9d6dc6c43e9caaa4432bcc81d8b177dc38217abc0a`

This is an unexplained provenance gap, not proof of malicious modification. Apple signing and build provenance establish different facts. Do not treat one as a substitute for the other.

- [ ] Establish why the distributed DMG differs from the attested DMG, with evidence binding the actual distributed bytes to their build and post-build transformations.
- [ ] Verify a matching attestation for the actual installer or independently validate an adequately documented alternative provenance chain.
- [ ] Revalidate signature, notarization, checksum, source commit, and provenance for the exact candidate being considered.

Sources: [release](https://github.com/zephium-browser/Zephium/releases/tag/v1.0.3), [attested workflow](https://github.com/zephium-browser/Zephium/actions/runs/40121241988).

## Blocker 2: blocker fuzz failure

The [October 25 nightly run](https://github.com/zephium-browser/Zephium/actions/runs/40177068972) for the baseline commit failed in the blocker `source_admission` fuzz target. It hit a debug assertion comparing `converted_trailing_separator` and `has_trailing_separator` in `vendor/adblock/src/content_blocking.rs:474`.

This is confirmed CI evidence. It is not a locally reproduced exploit or proof of a production browser crash: the assertion is debug-only, and production reachability/impact has not been established. The other extended nightly failures were formatting differences in vendored code. Exact-commit push CI and CodeQL passed; a green main CI does not resolve the fuzz finding.

- [ ] Investigate the failing input and determine production behavior and reachability.
- [ ] Review a fix or a supported explanation, including a regression case.

## Usability and feature gates

The following are open user reports as of the review date, not failures independently reproduced by this fork:

- [Bitwarden popup on macOS](https://github.com/zephium-browser/Zephium/issues/160).
- [1Password/passkey behavior on macOS](https://github.com/zephium-browser/Zephium/issues/141).
- [New tabs failing to load on macOS](https://github.com/zephium-browser/Zephium/issues/131).
- [Windows startup failures](https://github.com/zephium-browser/Zephium/issues/158).
- [Windows Work admission failures and misleading completion](https://github.com/zephium-browser/Zephium/issues/150).

The current `crates/zephium-app/src/work_commands.rs` explicitly disables arbitrary commands and coding-agent handoffs despite broader README language. Extension compatibility and native platform qualification also require workflow-specific evidence. Do not assume full Chrome extension compatibility or Chromium-equivalent site isolation.

## Conditions for a later isolated trial

These are future evaluation steps, not permission to execute now.

- [ ] Explicitly resume runtime evaluation after reviewing the provenance evidence.
- [ ] Use a disposable test machine or VM without personal browser data, SSH/cloud credentials, development signing identities, or confidential host mounts. A browser profile alone is not OS isolation.
- [ ] Review dependency/build-script execution before installing or building; refresh vulnerability information for the exact locked graph.
- [ ] Test public browsing, new windows, redirects, session restoration, downloads, and resource use before adding extensions or accounts.
- [ ] Test required password-manager and passkey flows with test credentials.
- [ ] Evaluate AI context disclosure with synthetic data. Cloud-model use sends selected context to providers; backend retention claims were not independently verified.
- [ ] Review each MCP server separately. The inspected Unix stdio adapter starts configured programs with an explicit environment and process lifecycle controls, but does not provide filesystem/network confinement.
- [ ] Keep release builds, signing, publishing, and production-account access outside the evaluation scope.

## Review limits

No full source audit, fresh complete dependency advisory scan, traffic audit, sandbox penetration test, runtime benchmark, or hostile-page test was performed. Signing and notarization do not certify software safety. This fork contains upstream source for inspection; inherited product claims and build instructions below the README notice do not override this evaluation status.
