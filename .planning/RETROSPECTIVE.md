# Project Retrospective

*A living document updated after each milestone. Lessons feed forward into future planning.*

## Milestone: v1.2 — Polish & Local-First UX

**Shipped:** 2026-05-29
**Phases:** 4 (6, 7, 8, 9) | **Plans:** 16 | **Commits:** 110 | **Timeline:** 44 days (2026-04-16 → 2026-05-29)

### What Was Built
- **Brand cleanup complete** — `dictus-*.wav` filenames, "Dictus Portable Mode" marker, runtime app-data path in DebugPaths; verify-sync.sh relocated to `.github/scripts/` and extended to 15 assertions (BRAND-01..04, SYNC-06)
- **Platform icons regenerated** — opaque navy-tile Linux PNG (black-corners impossible), 6-layer Windows ICO, `tauri.conf.json bundle.icon` with 7 entries, 1024×1024 RGBA source-of-truth (ICON-01..04)
- **macOS clean shutdown** — `flush_and_exit` helper releases CGEventTap on main runloop before `app.exit(0)`; debug-mode `simulate_updater_restart` validates updater-relaunch path; multi-day no-crash validation (SHUT-01..03)
- **Privacy / local-first UX** — platform-aware default provider, Local/Cloud tabs replacing cloud opt-in toggle, `docs/PRIVACY.md` documenting network surface, 25 i18n keys propagated across 19 sibling locales (PRIV-01..03)
- **Audit gap closure** — 33 clippy errors resolved across 14 files, retroactive 07-VERIFICATION.md authored, 08-UAT promoted to passed (AUDIT-01..03)

### What Worked
- **Wave-0 verify-sync.sh extension pattern** — Phase 6 Plan 01 extended `verify-sync.sh` with BRAND-01a/02a/03a assertions before any source change, giving downstream plans (02/03) immediate green-make targets. Same pattern as v1.1 SYNC validate.sh.
- **Diagnosis-before-fix gate (SHUT-01)** — Committing `## Diagnosis` to `07-01-PLAN.md` *before* writing code forced the team to rule out wrong suspects (path b: `std::process::exit`) and pick the right one (path a: explicit CGEventTap release on main thread). First commit on the Phase 7 branch was the diagnosis.
- **Single-script i18n propagation (08-10)** — One Node script (JSON.parse/stringify roundtrip + insertion-order preservation) propagated 25 new keys across 19 locales in 1m 56s. All 19 succeeded first try.
- **Retroactive verification backfill (Phase 9)** — Authoring `07-VERIFICATION.md` from `07-01-SUMMARY.md` + Pierre's multi-day observation produced a defensible audit artifact without re-running the original verification work. Pattern reusable for any phase that closed without VERIFICATION.md.
- **Audit-driven gap closure** — The 2026-05-28 audit produced 3 concrete gaps (AUDIT-01/02/03), each scoped tightly to a single deliverable; Phase 9 closed all three in 14 minutes of execution.

### What Was Inefficient
- **Phase 8 UAT iteration cost** — Initial Phase 8 (plans 08-01..05) shipped a cloud opt-in toggle + three-pillar marketing block, both rejected at UAT. Rework spanned plans 08-06..10 (5 additional plans across ~2 weeks), 6 commits per gap. A pre-UAT design contract (UI-SPEC.md was authored but not enforced against the cloud-toggle pattern) might have caught the pivots earlier.
- **Phase 7 crash non-reproducible after fix** — Fix is defensive but the original trigger was never fully isolated under clean env (likely multi-install pollution). Multi-day real-world observation served as final evidence, but a clean reproduction would have been more defensible.
- **Clippy debt accumulated invisibly** — 33 clippy errors had accumulated across 14 files before the v1.2 audit surfaced them. Phase 7 VALIDATION.md *claimed* the clippy gate as a sampling rate but the gate was never run in CI. Phase 9 closure was small (60min) but the deferred cost grew silently.
- **Nyquist VALIDATION.md never finalised** — All 4 phases authored VALIDATION.md at planning time but left them as `status: draft, nyquist_compliant: false`. Pattern repeats from v1.1 (Phase 5 VALIDATION.md draft). Authoring without enforcement = paperwork.

### Patterns Established
- **Audit-then-gap-closure-phase** — `/gsd:audit-milestone` produces concrete REQ-IDs (AUDIT-XX), gap-closure becomes its own phase. Cleaner than retro-amending phases or shipping with documented debt.
- **3-source requirement cross-reference** — `VERIFICATION.md status` × `SUMMARY frontmatter` × `REQUIREMENTS.md checkbox` must agree per REQ-ID. Caught Phase 7 (missing VERIFICATION.md) and Phase 8 (stale UAT frontmatter) in v1.2 audit.
- **Retroactive VERIFICATION.md from SUMMARY** — Defensible when SUMMARY is comprehensive + real-world observation exists. Document source-of-evidence transparently in VERIFICATION.md frontmatter.
- **TECH-XX debt naming** — Suppressed lints + deferred refactors get a TECH-XX entry in REQUIREMENTS.md "Future Requirements" with file:line + rationale + size estimate. Visible debt, not silent.
- **Sibling-locale propagation by script** — 19-locale changes done by one-shot Node script with JSON-roundtrip + per-locale failure log+continue. Faster than 19 manual edits, deterministic, audit-friendly.

### Key Lessons
1. **Author the diagnosis before the fix.** SHUT-01 forced this; it caught the wrong suspect early. Make it a gate, not a checkbox.
2. **UI-SPEC contracts must enforce against design pivots, not just describe layout.** Phase 8's cloud-toggle survived the UI-SPEC but failed UAT — the spec described layout, not interaction model. Future UI phases should specify "what the user does" not just "what the user sees."
3. **Don't claim a CI gate you don't run.** Phase 7 VALIDATION.md sampling rate listed `cargo clippy -- -D warnings` but no CI step enforced it. By v1.2 audit, 33 errors had accumulated. Either wire the gate or remove the claim.
4. **Audit-to-gap-closure-to-archive is a clean shipping flow.** Audit produces concrete gaps → gaps become a small surgical phase → re-audit confirms green → archive. Avoids the "ship with debt" trap.
5. **Nyquist VALIDATION.md as planner-stub is a smell.** Three milestones in a row (v1.0/v1.1/v1.2) authored VALIDATION.md but never finalised it. Either drop the artifact or make it a wave-0 gate.

### Cost Observations
- Model mix: not tracked per-session in this project (no telemetry capture configured)
- Sessions: spanned 44 days; longest single execution Phase 9 (14 min, 17 files) — surgical audit closure
- Notable: Phase 8 iteration cost (5 follow-up plans for UAT pivots) was 2× the original Phase 8 plan count

---

## Cross-Milestone Trends

### Process Evolution

| Milestone | Phases | Plans | Key Change |
|-----------|--------|-------|------------|
| v1.0 | 3 | 8 | First milestone — rebrand baseline established |
| v1.1 | 2 | 7 | Wave-0 pattern (validate.sh FAIL-first) introduced; triple-backup signing custody |
| v1.2 | 4 | 16 | Audit-driven gap-closure phase pattern; 3-source requirement cross-reference; UAT pivots motivating UI-SPEC enforcement gap |

### Cumulative Tech Debt

| Item | Origin | Status |
|------|--------|--------|
| Nyquist VALIDATION.md draft state | v1.0 | Carries through v1.0/v1.1/v1.2 (5 phases in draft) |
| `blob.handy.computer` CDN (INFR-01) | v1.0 | Documented in `docs/PRIVACY.md` v1.2; CDN migration still pending |
| Windows OS-level signing (INFR-03) | v1.1 | Deferred; SmartScreen warning accepted |
| Cargo binary rename `handy`→`dictus` (TECH-03) | v1.0 | Deferred; macOS permission risk |
| Module rename `handy_keys` (TECH-01) | v1.0 | Deferred; external crate `handy-keys` must not be touched |
| `llm_client.rs:137` 8-arg refactor (TECH-04) | v1.2 | Suppressed via `#[allow]`; 15-30 min estimated |
| `DictusLogo.tsx` i18next lint failure | pre-v1.2 | Pre-existing; out of scope |

### Top Lessons (Verified Across Milestones)

1. **Wave-0 / FAIL-first scripts make downstream plans testable from commit 1.** Used by v1.1 (validate.sh), v1.2 Phase 6 (verify-sync.sh assertions added before source change). Worth promoting as a workflow default.
2. **Identity integrity needs script-enforced guards through fork-sync merges.** v1.0 established the principle; v1.1 made it a CI gate (verify-sync.sh); v1.2 extended assertions when surfaces grew (BRAND-01a/02a/03a/ICON-02a). Without the script, every upstream sync would silently regress brand surfaces.
3. **VALIDATION.md without a CI hook is paperwork.** Verified across all 3 milestones: 5 of 9 phase VALIDATION.md files are in draft state. Either wire them as gates or stop authoring them.
4. **Audit-before-archive surfaces hidden debt cheaply.** v1.1 audit returned `tech_debt`; v1.2 audit caught 31 clippy errors + missing VERIFICATION.md + stale UAT. Cheap insurance against "ship and discover."
