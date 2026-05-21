---
created: 2026-05-21
title: Upstream sync — May 2026 audit + strategy review
area: process + planning
files:
  - UPSTREAM.md (existing runbook — solid, no change needed)
  - .github/upstream-sha.txt (last sync = fdc8cb7, 2026-04-14)
  - .github/workflows/upstream-sync.yml (the cron generating tracker issues #4/#7/#9/#10)
  - .planning/ROADMAP.md (Phase 9 + Phase 10 to reconsider)
related:
  - getdictus/dictus-desktop issue #1 (Upstream sync strategy — strategic discussion)
  - getdictus/dictus-desktop issues #4, #7, #9, #10 (auto-generated trackers)
  - .planning/todos/pending/2026-04-14-simplify-upstream-sync-workflow.md
---

## Part 1 — May 2026 audit of 15 upstream commits

### Context
Audit done during the 2026-05-21 brainstorming session. Decision: **defer the merge**. Capture the analysis so it can be picked up cold in a later focused session.

### Delta state
- **Stored upstream SHA** (`.github/upstream-sha.txt`): `fdc8cb7` (last sync = 2026-04-14, "merge cjpais/Handy post-v0.8.2 delta")
- **Upstream HEAD** at audit time: `e3206aa` (2026-05-12)
- **Delta**: 15 commits, ~1 month of upstream activity (not 8 months as initially miscounted)

### Commits classified

| # | SHA | Title | Verdict | Notes |
|---|---|---|---|---|
| 1 | `f26fe0d` | Fix (linux) overlay problem in kde (#1121) | 🟢 Accept | Linux bug fix, 4/12 lines in `overlay.rs`, zero risk |
| 2 | `0392b7b` | docs(readme): Linux startup troubleshooting (#1266) | 🟢 Accept + brand cleanup | README only, but references `HANDY_NO_GTK_LAYER_SHELL` (brand leak in docs) |
| 3 | `966ff99` | query gpu async (#1246) | 🟢 Accept | Perf — async GPU enumeration, 25/2 lines across 3 files |
| 4 | `11311be` | fix(overlay): parse HANDY_NO_GTK_LAYER_SHELL as boolean (#1269) | 🟡 Brand decision needed | Bug fix correct, but keeps `HANDY_` env var prefix. Decision: accept as-is (don't break user configs), or rename to `DICTUS_*` with backward-compat alias. Add to `handy-brand-cleanup.md` |
| 5 | `564fbc8` | docs: unify CLAUDE.md and AGENTS.md (#1272) | 🟡 Hot conflict expected | Upstream merges CLAUDE.md content into AGENTS.md. Our CLAUDE.md is Dictus-customized (CLI parameters, debug mode, platform notes). Strategy: take AGENTS.md additions, keep OUR CLAUDE.md verbatim |
| 6 | `aee682f` | feat: add AWS Bedrock (Mantle) as post-processing provider (#1288) | 🔴 SKIP | Adds a cloud provider (10 lines in `settings.rs`). Direct violation of local-first policy. Drop the +10 lines during merge conflict resolution |
| 7 | `a4d671a` | fix: improve German translation quality (#1292) | 🟢 Accept | i18n UI strings only (`de/translation.json`), 16 replacements. **Not** the translation feature — purely UI |
| 8 | `c1e11fa` | refactor(nix): rustPlatform.bindgenHook (#1255) | 🟢 Accept passive | Nix-only, we don't ship Nix, doesn't break us |
| 9 | `af6ec6c` | chore(nix): bun node_modules normalization (#1256) | 🟢 Accept passive | Nix-only |
| 10 | `4b7bb4e` | docs(audio): mic-init timing log (#1330) | 🟢 Accept | 5 lines of comments in `audio.rs`, zero risk |
| 11 | `8346bc2` | fix(nix): macOS build for nixpkgs (#1316) | 🟢 Accept passive | Nix-only |
| 12 | `085cd53` | release v0.8.3 | 🟡 Conflict — REJECT their version | Bumps 0.8.2 → 0.8.3 in `package.json`, `Cargo.toml`, `tauri.conf.json`. We're on Dictus 0.1.2 — keep our version |
| 13 | `a385371` | refactor(nix): cargo-tauri.hook standard phases (#1335) | 🟢 Accept passive | Nix-only |
| 14 | `1d042f3` | docs(agents): GitHub workflow rules for AI assistants (#1393) | 🟢 Accept (after #5) | 16 lines in AGENTS.md. Depends on clean merge of `564fbc8` |
| 15 | `e3206aa` | refactor(nix): drop redundant LD_LIBRARY_PATH (#1392) | 🟢 Accept passive | Nix-only |

### Summary
- 🟢 **10 commits accept directly** (Linux fixes, GPU perf, docs, i18n, Nix x4)
- 🟡 **3 commits with known conflicts** (CLAUDE.md, version bump, env var prefix decision)
- 🔴 **1 commit to skip** (AWS Bedrock cloud provider)
- ⚠️ **1 cross-cutting policy decision** — `HANDY_*` env var prefix

### Real value for our user base
For the current primary audience (macOS power users):
- `f26fe0d` overlay KDE → 0 value (we're not on Linux/KDE)
- `966ff99` GPU async → marginal (M-series Apple Silicon already fast)
- `11311be` env var bool → 0 value (Linux niche)
- `085cd53` v0.8.3 tag → cosmetic
- i18n de → 0 value (not German-speaking)
- Nix x4 → 0 value (we don't ship Nix)

For Linux users (silent but real): `f26fe0d` brings a real fix for KDE compositor overlay issues. Niche but real.

### Merge cost estimate
- Total time: **30-60 minutes** (not 1-3h as originally feared)
- Identified conflicts: 3 (all with clear resolution path)
- AWS Bedrock removal: 30 seconds
- Most commits trivially merge

### Decision (session 2)
**Defer the merge.** Reasons:
1. 1-month delta is not urgent
2. Low functional value for primary audience (macOS)
3. User wants to keep focus on Translation feature (higher ROI)
4. Risk of merge becoming harder after UX refactor is mitigated — the 15 commits barely touch the files Phase 8/translation work will modify (except `settings.rs` AWS Bedrock which we skip anyway)

### When to actually do this sync
- Either: before starting heavy UX refactor work, to ensure clean baseline
- Or: after the Translation feature ships, as a maintenance batch
- Or: when a Linux user reports the KDE overlay bug (gets a free user-visible fix)

The runbook `UPSTREAM.md` is mature and ready — execution is purely a focus/timing question.

---

## Part 2 — Strategy review: do we need automated upstream-sync infrastructure?

### Current state
We have planned Phase 9 ("Sync Infrastructure Refactor") and Phase 10 ("Claude Code Agent Layer") on the GSD roadmap. Combined goal: weekly cron generates draft PRs labeled `upstream-sync`, verify-sync.sh runs as CI gate, Claude Code adapter agent applies identity fixes automatically, auditor agent posts review comment.

### What's actually happening
- Cron `.github/workflows/upstream-sync.yml` runs weekly, generates **tracker issues** (#4, #7, #9, #10) — currently 4 open issues piling up
- They aren't being consumed (the manual merge is rare, ~1 per month max)
- No draft PR automation yet (Phase 9 not done)
- No agent layer yet (Phase 10 not done)

### Reflection — Are Phase 9 and Phase 10 still worth doing?

**Original assumption**: upstream syncs are frequent enough to justify automation (draft PRs + agents).

**Actual observation after 1 month of operating**: 15 commits in 1 month, of which only 3-4 have real value, and the merge takes 30-60 min when batched. **Manual cadence is fine.**

**Cost of Phase 9/10 if executed**:
- Phase 9: ~1-2 weeks of work (workflow refactor, CI gate, PR template, branch policy)
- Phase 10: ~1-2 weeks of work (adapter agent allow-list, auditor agent, OAuth token, prompt design)
- Total: ~4 weeks of engineering for infrastructure we'd rarely use

**Cost of staying manual**:
- 1h/month for routine merges
- The mature `UPSTREAM.md` runbook is already enough

**Conclusion**: Phase 9 and Phase 10 are **over-engineered** for the actual upstream cadence. Worth considering:
- **Cancel both phases** outright
- **Or downscope Phase 9** to "improve verify-sync.sh CI gate + remove the auto-tracker cron" (small scope)
- **Or merge Phase 9 + 10 into a single lightweight phase**: "Manual upstream sync — improve runbook, simplify tracker UX, add a `make sync-audit` helper script"

### Trackers #4 / #7 / #9 / #10 — what to do
These are auto-generated by the weekly cron. They accumulate because we don't consume them in real time. Two options:

**Option A — Disable the cron, consume manually**
- Stop the weekly cron
- Replace by a `make sync-audit` script that the maintainer runs when wanting to check
- Close all 4 trackers as "replaced by manual audit process"

**Option B — Keep the cron but consume properly**
- Cron continues running weekly
- Each new tracker auto-closes the previous one (so we always have at most 1 open)
- The single open tracker is the canonical "delta status" at any time
- The maintainer treats it as a backlog item, addresses when bandwidth allows

**Option B is probably better** because:
- It keeps the visibility of "where we stand vs upstream"
- It doesn't require new scripts
- The "auto-close previous tracker" tweak is small (a few lines in the workflow)

### Action items for the future "sync strategy" session

1. Decide: cancel Phase 9/10, downscope, or merge them
2. Update `.planning/ROADMAP.md` accordingly (likely remove Phase 9 + 10 as currently scoped)
3. Decide tracker policy (Option A vs B above)
4. If Option B: patch `upstream-sync.yml` to auto-close previous trackers
5. Close all 4 current trackers (#4, #7, #9, #10) regardless — they're stale
6. Comment on issue #1 with the updated strategy decision
7. Update `STATE.md` to reflect the new plan
8. Consider whether the existing todo `2026-04-14-simplify-upstream-sync-workflow.md` is now covered by this review (likely yes — close it as superseded)

### Note on Phase 6 absorption
Reminder: Phase 9 originally also included SYNC-06 (relocate `verify-sync.sh` to `.github/scripts/`). That work **was already absorbed by Phase 6** (completed 2026-04-16). So whatever remains of Phase 9 is the cron/PR/CI gate work only — which is what we'd cancel or downscope.

---

## Suggested session sequencing

1. ✅ Session 1 (2026-05-21) — Translation feature business model
2. ✅ Session 2 (2026-05-21) — Upstream audit (this todo)
3. 🔜 **Session 3 — Upstream sync strategy review** (Phase 9/10 fate, tracker policy, ROADMAP update)
4. 🔜 **Session 4 — Translation Mode UX cible** (originally session 3, decaled)
5. Then: actual implementation

## Why this matters
The user's mental model is now: *"the auto-sync ambitions were over-engineered, let's simplify before scaling further"*. This is a healthy retreat from premature automation. The output of session 3 will free up roadmap space for the Translation feature without scope creep.
