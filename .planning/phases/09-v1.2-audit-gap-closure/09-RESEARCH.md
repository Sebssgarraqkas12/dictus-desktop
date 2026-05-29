# Phase 9: v1.2 Audit Gap Closure — Research

**Researched:** 2026-05-28
**Domain:** Rust clippy remediation, GSD verification-doc authoring, UAT frontmatter refresh
**Confidence:** HIGH — all findings verified directly from source (live clippy run, git log, file inspection)

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| AUDIT-01 | `cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings` exits 0 from repo root | 33 distinct errors mapped to 14 source files; all are style/idiom lints with known fixes; no unsafe or logic rewrites needed |
| AUDIT-02 | `.planning/phases/07-macos-clean-shutdown/07-VERIFICATION.md` exists with `status: passed` frontmatter and populated truths/artifacts/link tables sourced from `07-01-SUMMARY.md` | 07-01-SUMMARY.md is complete and self-contained; template from `~/.claude/get-shit-done/templates/verification-report.md` confirmed; 08-VERIFICATION.md is the living same-phase exemplar to follow |
| AUDIT-03 | `.planning/phases/08-privacy-local-first-ux/08-UAT.md` frontmatter reads `status: passed` and body references closure commits from plans 08-08, 08-09, 08-10 | All closure commits located on `feat/phase-08-local-first-ux` branch; exact SHAs documented below |
</phase_requirements>

---

## Summary

Phase 9 is a three-task documentation-and-linting cleanup. No new features ship. The entire scope is:
1. Fix 33 cargo clippy errors spread across 14 Rust source files (all style/idiom lints — zero logic rewrites required)
2. Author `07-VERIFICATION.md` from the already-complete `07-01-SUMMARY.md` using the project's standard verification-report template
3. Update `08-UAT.md` frontmatter from `status: diagnosed` to `status: passed` and add references to the three closure commits

The clippy sweep is the only task touching source code. All clippy errors are machine-readable style lints; approximately half have inline `help:` replacement suggestions from clippy itself. The one structural lint (`items after a test module` in `recorder.rs`) and the one architectural lint (`too many arguments` in `llm_client.rs`) require hand-edits. `cargo fix --lib -p dictus --allow-dirty` compiled cleanly (finished dev profile in 2.58s) but did NOT apply any fixes, confirming all 33 errors require explicit code edits rather than automated rewriting.

The `07-VERIFICATION.md` does not exist yet. The template at `~/.claude/get-shit-done/templates/verification-report.md` defines the schema. The best living example in this repo is `08-VERIFICATION.md` (passed, 11/11 truths, re-verification structure). The content source — `07-01-SUMMARY.md` — is comprehensive and covers all three requirements (SHUT-01/02/03) with exact commits, exact file line numbers, and validation narrative.

The `08-UAT.md` update is the simplest task: the body content is already accurate (all six gap-closure items are documented); only the YAML frontmatter `status` field and the source references need updating to reflect plans 08-08/09/10.

**Primary recommendation:** Fix clippy errors file-by-file starting with `settings.rs` (6 errors, highest density), then `lib.rs` (4 errors), then the remaining 12 files. Author 07-VERIFICATION.md in a single write from 07-01-SUMMARY.md. Patch 08-UAT.md frontmatter last.

---

## AUDIT-01: Clippy Error Inventory

### Current State (verified 2026-05-28)

Running `cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings` exits with 33 errors on `lib test` (31 on `lib`).

### Error Map — by file and lint

| File | Line(s) | Error | Fix Strategy |
|------|---------|-------|-------------|
| `src/portable.rs` | 9 | empty line after doc comment | Remove blank line between `///` and `static` |
| `src/portable.rs` | 162 | using `write!()` with format string ending in newline | Use `writeln!()` instead |
| `src/apple_intelligence.rs` | 63 | calling `CStr::new` with a byte string literal | Change `CStr::from_bytes_with_nul(b"Unknown error\0").unwrap()` to `c"Unknown error"` |
| `src/audio_toolkit/audio/recorder.rs` | 190 | this can be `std::io::Error::other(_)` | Change `Error::new(ErrorKind::Other, ...)` to `Error::other(...)` |
| `src/audio_toolkit/audio/recorder.rs` | 353 | items after a test module | Move `run_consumer` function (lines 395–402) to before the `mod tests` block |
| `src/audio_toolkit/audio/recorder.rs` | 444 | loop could be `while let` | Rewrite `loop { let chunk = match ... { Ok(c) => c, Err(_) => break, ...} }` as `while let Ok(chunk) = sample_rx.recv()` |
| `src/audio_toolkit/text.rs` | 162 | `map_or` can be simplified | Likely `.map_or(false, |x| x == y)` → `.is_some_and(|x| x == y)` |
| `src/commands/audio.rs` | 24 | `map_or` can be simplified | Same pattern |
| `src/llm_client.rs` | 137 | function has too many arguments (8/7) | Group related params into a struct (e.g., `ChatCompletionParams`) — hand-edit required |
| `src/managers/history.rs` | 336, 341, 345 | unneeded `return` statement (×3) | Remove `return` keyword from each tail expression |
| `src/managers/model.rs` | 657 | borrowed expression implements required traits | Remove unnecessary `&` borrow |
| `src/managers/transcription.rs` | 119 | `map_or` can be simplified | Same pattern as above |
| `src/overlay.rs` | 310 | `let` binding has unit value | Remove or rewrite the unit binding |
| `src/settings.rs` | 182, 198, 204, 271, 285, 302 | `impl` can be derived (×6) | Add `#[derive(Default)]` to the six enum/struct impls and remove manual `impl Default` blocks |
| `src/settings.rs` | 251, 255 | `to_*` method with `&self` should take `self` by value (Copy type) (×2) | Change `&self` to `self` in those two `to_*` methods |
| `src/settings.rs` | 863 | `contains_key` followed by `insert` on HashMap | Rewrite as `.entry(key).or_insert(value)` |
| `src/shortcut/mod.rs` | 286 | `if` statement can be collapsed | Merge nested `if` into single `if a && b` |
| `src/transcription_coordinator.rs` | 65 | `map_or` can be simplified | Same pattern |
| `src/transcription_coordinator.rs` | 167 | `map_or` can be simplified | Same pattern |
| `src/lib.rs` | 232 | borrowed expression implements required traits | Remove unnecessary `&` |
| `src/lib.rs` | 345 | expression creates reference immediately dereferenced | Remove double-ref |
| `src/lib.rs` | 393 | used consecutive `str::replace` calls | Chain as `.replace(['.', '-'], "_")` |
| `src/lib.rs` | 612, 651, 681 | expression creates reference immediately dereferenced (×3) | Remove double-ref at each site |

**Total:** 33 errors across 14 files

### Fix Strategy Classification

| Category | Count | Approach |
|----------|-------|----------|
| Mechanical one-liner (remove `return`, remove `&`, replace `write!` → `writeln!`) | ~12 | Edit in place — clippy `help:` provides exact replacement |
| Pattern substitution (`map_or` → `is_some_and`, `entry()`, `while let`, c-string literal) | ~10 | Edit in place — clippy `help:` provides exact replacement |
| `#[derive(Default)]` replacement (settings.rs ×6) | 6 | Add derive attr + delete manual `impl Default` block for each |
| Structural (move `run_consumer` before `mod tests`) | 1 | Move function block, verify tests still compile |
| Method signature (`to_*` with Copy type: `&self` → `self`) | 2 | Change param type — check all call sites for ownership impact |
| Architectural (`too many arguments` in `llm_client.rs`) | 1 | Group 8 params into a `ChatCompletionRequest` struct |

**None of the 33 errors require logic changes, algorithm rewrites, or unsafe code.** All are code-style and idiom improvements.

### Critical Note: `items_after_test_module` in `recorder.rs`

The `run_consumer` function at line 395 falls after the `mod tests` block at line 353. The fix is to move the function definition to before the test module. This is a structural edit requiring care: verify that `run_consumer` is still reachable from its callers after the move (it is — Rust modules do not affect private-function call order within the same file).

### Critical Note: `too_many_arguments` in `llm_client.rs`

`send_chat_completion_with_schema` at line 137 takes 8 arguments (limit is 7). Options:
- Option A (minimal): Add `#[allow(clippy::too_many_arguments)]` attribute — acceptable for audit cleanup, minimal diff
- Option B (proper): Introduce a `ChatCompletionRequest` struct grouping related params

For an audit cleanup phase, Option A is the lowest-risk approach (no API surface change). Option B is the "correct" refactor but risks breaking callers. The planner should choose — both are documented here.

### Verification Command

```bash
cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings
```

Expected exit code after fixes: `0`

---

## AUDIT-02: 07-VERIFICATION.md Authoring

### File Does Not Exist

Confirmed: `ls .planning/phases/07-macos-clean-shutdown/` shows: `07-01-PLAN.md`, `07-01-SUMMARY.md`, `07-CONTEXT.md`, `07-RESEARCH.md`, `07-VALIDATION.md`. No `07-VERIFICATION.md`.

### Template Location

`~/.claude/get-shit-done/templates/verification-report.md` — full schema confirmed. Key sections:

- YAML frontmatter: `phase`, `verified`, `status`, `score`
- `## Goal Achievement` with three sub-tables: Observable Truths, Required Artifacts, Key Link Verification
- `## Requirements Coverage` table
- `## Anti-Patterns Found` table
- `## Human Verification Required` section
- `## Gaps Summary` section
- `## Verification Metadata` footer

### Best Living Exemplar

`08-VERIFICATION.md` (status: passed, 11/11, re-verification pass) — use this as the structural reference. It shows how to handle a re-verification with `re_verification:` frontmatter block and gap closure table.

### Source Material for 07-VERIFICATION.md

Everything needed is in `07-01-SUMMARY.md`. Mapping:

| VERIFICATION.md section | Source in 07-01-SUMMARY.md |
|------------------------|---------------------------|
| Observable Truths | §Accomplishments bullets (SHUT-01/02/03) |
| Required Artifacts | `key-files:` frontmatter + §Files Created/Modified |
| Key Link Verification | §Decisions Made + frontmatter `provides:` |
| Requirements Coverage | frontmatter `requirements-completed: [SHUT-01, SHUT-02, SHUT-03]` |
| Human Verification | §Manual-Only Verifications in 07-VALIDATION.md (crash non-reproducible under clean env, 3× tray-quit observed) |
| Verification Metadata | `duration: ~1h coding + multi-day`, `completed: 2026-04-23` |

### Frontmatter Values

```yaml
phase: 07-macos-clean-shutdown
verified: 2026-04-23T00:00:00Z
status: passed
score: 3/3 must-haves verified
```

### Truths to Document

| # | Truth | Status | Evidence Source |
|---|-------|--------|----------------|
| 1 | SHUT-01: `## Diagnosis` section committed to `07-01-PLAN.md` before any code change | VERIFIED | `grep -q "## Diagnosis" 07-01-PLAN.md` passes; commit `26d4755` is the first commit in the plan |
| 2 | SHUT-02: `flush_and_exit()` helper at both exit sites (tray quit + no-tray CloseRequested); logs flushed + CGEventTap released on macOS before `app.exit()` | VERIFIED | `grep -n "flush_and_exit" src-tauri/src/lib.rs` finds ≥3 hits (definition + 2 call sites); commit `e46d75c` |
| 3 | SHUT-03: `simulate_updater_restart` command registered; `SimulateUpdaterRestart.tsx` conditionally rendered on `settings.debug_mode`; macOS tray-quit crash dialog not observed over multi-day validation window | VERIFIED (human) | commit `722a5b2`; multi-day real-world validation documented in 07-01-SUMMARY.md §Issues Encountered |

### Task Commits

| Task | SHA | Message |
|------|-----|---------|
| SHUT-01 diagnosis | `26d4755` | `docs(07): capture diagnosis of quit-unexpectedly crash (SHUT-01)` |
| SHUT-02 flush_and_exit | `e46d75c` | `fix(shutdown): flush logs and run cleanup before exit on macOS (SHUT-02)` |
| SHUT-03 simulate_updater_restart | `722a5b2` | `feat(debug): add simulate_updater_restart trigger for SHUT-03 validation` |
| UPSTREAM.md protection | `cc72158` | `docs(upstream): protect lib.rs quit-exit divergence from future syncs (SHUT-02)` |

### Human Verification Note

The crash dialog is observable only on macOS with a physical device. 07-01-SUMMARY.md §Decisions Made item 4 documents the rationale for closing without a second reproduction: multi-day real-world use, no crashes observed, fix is defensible on first principles. The `07-VERIFICATION.md` `## Human Verification Required` section should capture this as the one manual observation.

---

## AUDIT-03: 08-UAT.md Frontmatter Refresh

### Current State

`08-UAT.md` YAML frontmatter:
```yaml
status: diagnosed
phase: 08-privacy-local-first-ux
source:
  - 08-01-SUMMARY.md
  - 08-02-SUMMARY.md
  - 08-03-SUMMARY.md
  - 08-04-SUMMARY.md
started: 2026-05-22T00:00:00Z
updated: 2026-05-22T00:00:00Z
diagnosis_note: |
  Root cause is a design pivot rejected by user, not a code bug — skipped
  parallel debug agents (no investigation needed). Closure split per user
  decision (2026-05-22): UI-only items handled as a Phase 8 sub-phase
  (gap-closure plan); embedded LLM runtime items deferred to a new
  milestone "Local-First Models" promoted ahead of v1.3 Smart Modes.
```

### Target State

```yaml
status: passed
phase: 08-privacy-local-first-ux
source:
  - 08-01-SUMMARY.md
  - 08-02-SUMMARY.md
  - 08-03-SUMMARY.md
  - 08-04-SUMMARY.md
  - 08-08-SUMMARY.md
  - 08-09-SUMMARY.md
  - 08-10-SUMMARY.md
started: 2026-05-22T00:00:00Z
updated: 2026-05-28T00:00:00Z
```

The `diagnosis_note` block should be removed (or replaced with a `closure_note`) since the phase is now `passed`.

### Closure Commits to Reference in Body

These commits live on `feat/phase-08-local-first-ux` (not yet merged to main as of 2026-05-28):

| Plan | SHA | Commit message |
|------|-----|----------------|
| 08-08 | `47688cc` | `fix(08-08): make ProviderPicker invoke renderRowExtras for every row` |
| 08-08 | `bbe82db` | `fix(08-08): inline Apple Intelligence Alert, fix Ollama link visibility, hide API key for Custom (local)` |
| 08-08 | `dbf0a26` | `docs(08-08): complete post-processing UI gap-closure plan` |
| 08-09 | `dd45f4f` | `feat(08-09): replace cloud toggle with Local/Cloud tabs in ProviderPicker` |
| 08-09 | `3433f9e` | `refactor(08-09): hoist library to top, remove pillars, wire tab state in PostProcessingSettings` |
| 08-09 | `4750bd2` | `chore(08-09): update en/translation.json — add tabs.*, drop pillars.* + cloudToggle.*, update cloudSelectedNotice` |
| 08-09 | `cea7eb9` | `docs(08-09): complete local-cloud tabs restructure + library hoist + pillars removal` |
| 08-10 | `54b9a85` | `chore(08-10): propagate i18n key changes to 19 sibling locales` |
| 08-10 | `6d34e18` | `docs(08-10): complete sibling-locale i18n propagation plan` |

The `08-UAT.md` body's `## Gaps` section already contains the correct closure detail for all 6 gaps (confirmed in file inspection). The planner only needs to update the frontmatter block and add a `## Closure` section referencing the SHAs above.

---

## Standard Stack (this phase only)

| Tool | Version | Use |
|------|---------|-----|
| `cargo clippy` | Rust stable (current) | Lint-gated compilation check |
| `cargo test --lib` | — | Verify no regression in Rust unit tests after fixes |
| `bun run lint` | — | Confirm frontend lint unaffected |

**No new libraries, no new dependencies.** This phase modifies only:
- Rust source files (clippy fixes)
- One new markdown file (`07-VERIFICATION.md`)
- One frontmatter update (`08-UAT.md`)

---

## Architecture Patterns

### Verification Doc Pattern

Every completed phase in this repo has a `{NN}-VERIFICATION.md` with this schema (confirmed from Phases 1–6, 8):
- YAML frontmatter: `phase`, `verified` (ISO timestamp), `status` (`passed`|`gaps_found`|`human_needed`), `score` (N/M)
- `## Goal Achievement` → truths table, artifacts table, link-wiring table
- `## Requirements Coverage` → maps REQUIREMENTS.md IDs to status
- `## Human Verification Required` → any tests needing real device/human
- `## Gaps Summary` → "No gaps found." or itemized critical/non-critical list
- `## Verification Metadata` footer

**Phase 7 specifics:** Three requirements (SHUT-01/02/03) all map to a single plan (07-01). The truths table has 3 rows. SHUT-03 is human-verified (crash dialog observation). The file must be authored as the initial (non-re-verification) pass with `re_verification: false`.

### Cargo Clippy Fix Pattern

- Fix errors file-by-file rather than all at once
- Run `cargo clippy` after each file to confirm progress
- Run `cargo test --lib settings::tests` after `settings.rs` changes (derives affect serialization)
- Run full `cargo clippy --all-targets` before committing

### UAT Frontmatter Update Pattern

The `status` field in UAT files is the single field that drives `/gsd:audit-milestone` status checks. The body content is documentation; the frontmatter `status` is machine-readable. Change only the frontmatter `status`, `updated`, `source` list, and optionally replace `diagnosis_note` with a `closure_note`.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead |
|---------|-------------|-------------|
| Clippy auto-fixes | Custom sed/awk scripts | `cargo clippy` `help:` suggestions applied manually (cargo fix does not apply these particular lints) |
| Verification doc content | Scraping files with scripts | Read `07-01-SUMMARY.md` directly; map sections by hand to template |

---

## Common Pitfalls

### Pitfall 1: `cargo fix` Does Not Fix These Lints

`cargo fix --lib -p dictus --allow-dirty` compiled cleanly (dev profile, 2.58s) but applied **zero** fixes. All 33 clippy errors persist after running it. These particular lints (`map_or`, `derive`, `return`, `items_after_test_module`, etc.) are not in clippy's machine-applicable fix set for this Rust version. **Every fix requires a manual edit.**

### Pitfall 2: `settings.rs` Derive Changes Break Serde

Adding `#[derive(Default)]` to settings enums/structs and deleting manual `impl Default` blocks must be verified against the existing `settings::tests` suite. Run `cargo test --lib settings::tests` after changes to settings.rs. The existing test `default_enable_cloud_providers_is_false` already guards one such default; the new derives must produce identical default values.

### Pitfall 3: `to_*` by Value May Break Callers

`settings.rs:251,255` — changing `&self` to `self` for Copy-typed `to_*` methods affects all call sites. Since `self` is `Copy`, callers already have a value and the change is safe, but inspect call sites before committing.

### Pitfall 4: `items_after_test_module` Move Must Keep Function Visible

Moving `run_consumer` from after `mod tests { ... }` to before it in `recorder.rs` is safe because function order within a Rust file doesn't affect visibility or call resolution. But confirm the function is not called only from within tests (in which case it could stay inside `mod tests`).

### Pitfall 5: 08-UAT.md Is on a Different Branch

The `feat/phase-08-local-first-ux` branch is not yet merged to `main`. The `08-UAT.md` file on this branch (`feat/phase-09-v1.2-audit-gap-closure`) comes from `main`, which predates the gap closure. The plan must either:
- Cherry-pick or merge the Phase 8 branch first, then update `08-UAT.md`, OR
- Update `08-UAT.md` in the Phase 9 plan with awareness that the source file for UAT is on the Phase 8 branch

**This is a dependency the planner must address.** The Phase 9 plan likely needs to merge/incorporate Phase 8 work before Phase 9 AUDIT-03 can be satisfied on the same branch.

### Pitfall 6: 07-VERIFICATION.md Is a New File

The file does not exist. It must be written with the Write tool, not Edit. The planner should confirm the target path: `.planning/phases/07-macos-clean-shutdown/07-VERIFICATION.md`.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | `cargo clippy` (Rust static analysis) + `cargo test` (Rust unit tests) |
| Config file | `src-tauri/Cargo.toml` |
| Quick run command | `cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings` |
| Full suite command | `bun run lint && cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings && cargo test --lib settings::tests` |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| AUDIT-01 | `cargo clippy -- -D warnings` exits 0 | static | `cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings` | ✅ |
| AUDIT-01 | No Rust unit test regression | unit | `cargo test --lib settings::tests` | ✅ |
| AUDIT-02 | `07-VERIFICATION.md` exists with `status: passed` frontmatter | doc inspection | `test -f .planning/phases/07-macos-clean-shutdown/07-VERIFICATION.md && grep -q "status: passed" .planning/phases/07-macos-clean-shutdown/07-VERIFICATION.md` | ❌ Wave 0 (file to be created) |
| AUDIT-03 | `08-UAT.md` frontmatter `status: passed` | doc inspection | `grep -q "status: passed" .planning/phases/08-privacy-local-first-ux/08-UAT.md` | ✅ (file exists, needs update) |

### Sampling Rate

- Per task commit: `cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings`
- Per wave merge: `bun run lint && cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets -- -D warnings && cargo test --lib settings::tests`
- Phase gate: Full suite green + AUDIT-02 file existence + AUDIT-03 frontmatter check before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] `.planning/phases/07-macos-clean-shutdown/07-VERIFICATION.md` — covers AUDIT-02 (file must be created by Wave 0 task)

---

## Open Questions

1. **Branch dependency for AUDIT-03**
   - What we know: `08-UAT.md` on `feat/phase-09` currently has `status: diagnosed`. The closure work is on `feat/phase-08-local-first-ux`. The Phase 8 branch is NOT merged into main or the current Phase 9 branch.
   - What's unclear: Should Phase 9 merge Phase 8's branch first? Or should Phase 9 update `08-UAT.md` independently on its own branch?
   - Recommendation: The planner should include a task to merge `feat/phase-08-local-first-ux` into `feat/phase-09-v1.2-audit-gap-closure` (or to main first, then rebase Phase 9) before AUDIT-03. Updating `08-UAT.md` on Phase 9 without the Phase 8 source changes risks a divergent file state on merge.

2. **`too_many_arguments` in `llm_client.rs`: allow vs. refactor**
   - What we know: Option A (`#[allow(clippy::too_many_arguments)]`) is a 1-line fix. Option B (struct extraction) is the idiomatic fix but changes the function signature.
   - What's unclear: Are there callers in other crates or tests that would need updating?
   - Recommendation: Option A for audit closure; add a TECH-DEBT item to the REQUIREMENTS.md Future section for Option B.

---

## Sources

### Primary (HIGH confidence)

- Live `cargo clippy` run on `feat/phase-09-v1.2-audit-gap-closure` — 2026-05-28 — all 33 errors confirmed, file/line locations verified
- `07-01-SUMMARY.md` — complete Phase 7 summary; all SHUT requirements covered
- `08-UAT.md` — current frontmatter `status: diagnosed` confirmed; body closure details confirmed accurate
- `08-VERIFICATION.md` — living exemplar for verification report format; `status: passed` re-verification
- `~/.claude/get-shit-done/templates/verification-report.md` — official template schema
- `git log --all --format="%H %s"` — Phase 8 closure commit SHAs confirmed
- `.planning/milestones/v1.2-MILESTONE-AUDIT.md` — audit gaps and tech_debt items confirmed

### Secondary (MEDIUM confidence)

- None

---

## Metadata

**Confidence breakdown:**
- Clippy error inventory: HIGH — sourced from direct live run
- 07-VERIFICATION.md template: HIGH — template file confirmed, exemplar confirmed
- Phase 8 closure SHAs: HIGH — `git log --all` confirmed all SHAs
- Branch dependency issue: HIGH — `git merge-base` confirmed Phase 8 NOT merged into Phase 9

**Research date:** 2026-05-28
**Valid until:** 2026-06-28 (Rust clippy lints are stable; template is project-owned)
