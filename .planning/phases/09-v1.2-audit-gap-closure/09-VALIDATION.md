---
phase: 09
slug: v1-2-audit-gap-closure
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-05-28
---

# Phase 09 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | `cargo clippy` (Rust static analysis) + `cargo test` (Rust unit tests) + `bun run lint` (frontend ESLint) |
| **Config file** | `src-tauri/Cargo.toml`, `eslint.config.js` |
| **Quick run command** | `cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings` |
| **Full suite command** | `bun run lint && cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings && cargo test --manifest-path src-tauri/Cargo.toml --lib settings::tests` |
| **Estimated runtime** | ~60 seconds (clippy ~30s cold / ~5s warm, settings tests ~5s, eslint ~10s) |

---

## Sampling Rate

- **After every task commit:** Run `cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings`
- **After every plan wave:** Run full suite command above
- **Before `/gsd:verify-work`:** Full suite must be green; both audit doc files must exist with correct frontmatter
- **Max feedback latency:** ~60 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 09-01-01 | 01 | 1 | AUDIT-01 | static (lint) | `cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings` | ✅ | ⬜ pending |
| 09-01-01 | 01 | 1 | AUDIT-01 | unit (regression) | `cargo test --manifest-path src-tauri/Cargo.toml --lib settings::tests` | ✅ | ⬜ pending |
| 09-01-02 | 01 | 1 | AUDIT-02 | doc inspection (file exists) | `test -f .planning/phases/07-macos-clean-shutdown/07-VERIFICATION.md` | ❌ W0 (to be created by this task) | ⬜ pending |
| 09-01-02 | 01 | 1 | AUDIT-02 | doc inspection (frontmatter) | `grep -q "^status: passed$" .planning/phases/07-macos-clean-shutdown/07-VERIFICATION.md` | ❌ W0 | ⬜ pending |
| 09-01-03 | 01 | 1 | AUDIT-03 | doc inspection (frontmatter) | `grep -q "^status: passed$" .planning/phases/08-privacy-local-first-ux/08-UAT.md` | ✅ (file exists, needs update) | ⬜ pending |
| 09-01-03 | 01 | 1 | AUDIT-03 | doc inspection (closure refs) | `grep -qE "(47688cc\|bbe82db\|dbf0a26\|dd45f4f\|3433f9e\|4750bd2\|cea7eb9\|54b9a85\|6d34e18)" .planning/phases/08-privacy-local-first-ux/08-UAT.md` | ✅ | ⬜ pending |
| 09-phase-gate | — | — | AUDIT-01/02/03 | audit re-run | `/gsd:audit-milestone v1.2` reports `status: passed` | — | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `.planning/phases/07-macos-clean-shutdown/07-VERIFICATION.md` — new file required for AUDIT-02 (created by task 09-01-02 from `07-01-SUMMARY.md`)

*No new test infrastructure needed — existing `cargo clippy`, `cargo test`, and `bun run lint` toolchains cover all checks.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| `/gsd:audit-milestone v1.2` re-run reports `status: passed` | Phase-gate (AUDIT-01/02/03 collectively) | Audit command runs interactively against the planning tree; success is the integration test for all three audit fixes | After all three tasks land, run `/gsd:audit-milestone v1.2` and confirm "no partial requirements, no integration gaps" |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references (07-VERIFICATION.md creation tracked)
- [ ] No watch-mode flags
- [ ] Feedback latency < 60s
- [ ] `nyquist_compliant: true` set in frontmatter (after planner reviews and confirms)

**Approval:** pending
