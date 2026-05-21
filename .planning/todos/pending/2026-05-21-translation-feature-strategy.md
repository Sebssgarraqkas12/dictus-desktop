---
created: 2026-05-21
title: Translation feature — strategy & UX redesign
area: product + ui + backend
files:
  - src-tauri/src/settings.rs:89-107 (LLMPrompt, PostProcessProvider)
  - src-tauri/src/settings.rs:524-600 (default providers)
  - src-tauri/src/settings.rs:705-755 (default bindings)
  - src-tauri/src/llm_client.rs (LLM pipeline — already complete)
  - src-tauri/src/managers/transcription.rs:66 (post_process_transcription)
  - src-tauri/src/managers/transcription.rs:543-552 (Whisper translate param)
  - src/components/settings/post-processing/PostProcessingSettings.tsx
  - src/components/settings/advanced/AdvancedSettings.tsx:60-70 (Experimental gate)
  - src/components/Sidebar.tsx:59-64 (post-processing tab visibility)
  - src/lib/constants/languages.ts (target language list)
related:
  - .planning/todos/pending/2026-04-14-privacy-local-first-ux-audit.md (Phase 8)
  - .planning/todos/pending/2026-04-14-handy-brand-cleanup.md
  - getdictus/dictus-ios issue #54 (Dictus Pro — Open Core roadmap)
  - getdictus/dictus-ios issue #79 (Smart Mode — Local LLM)
  - getdictus/dictus-ios issue #111 (Translated TTS / travel mode)
  - getdictus/dictus-ios issue #141 (free-tier polish vs Pro Smart Mode)
---

## Context

Brainstorming session held 2026-05-21 to define a Translation Mode feature on Dictus Desktop (FR → EN / ES / ZH / etc.) and the broader business model that frames it. This todo consolidates the decisions and the empirical evidence gathered during the session so the work can resume cold.

The user wants a Translation Mode where dictating in a source language produces text in a chosen target language, ideally via dedicated shortcuts (one per target language).

## Empirical findings — Whisper translation approaches

Tested in real conditions on a Mac M4 Pro / macOS 26.1 with the same French reference text across three approaches. Scores reflect translation quality on a single ~45-word paragraph with varied tenses (passé composé, plus-que-parfait, futur proche), modal expressions, and idiomatic phrasing.

| Direction | Whisper Turbo `language` token trick | Whisper Medium `language` token trick | Whisper native `translate: true` | **Apple Intelligence post-process** |
|---|---|---|---|---|
| FR → EN | 5/10 (3 erreurs sérieuses) | 8.5/10 (quasi-humain) | 8/10 (validé en usage réel) | **8/10** |
| FR → ES | 0/10 (sort du français) | 0/10 (sort du français) | impossible (Whisper ne sait pas) | **9/10** ✅ |
| FR → ZH | 0/10 (hallucinations) | 6.5/10 (lisible mais bugué) | impossible | **8/10** ✅ |

**Conclusions empiriques :**

1. Le trick `language = target` fonctionne **uniquement** vers l'anglais sur Whisper Medium. Toutes les autres paires échouent (FR→ES retombe sur du français pur, FR→ZH hallucinerait sur Turbo, marche partiellement sur Medium).
2. La tâche `translate: true` native de Whisper marche très bien mais est **English-only** par design — c'est documenté, c'est intrinsèque au training.
3. **Apple Intelligence en post-process** est la **seule approche qui livre une qualité acceptable sur EN, ES et ZH simultanément**, même avec le modèle anglais-first.
4. Le bug "cold-start" (premier mot avalé : `Hier`, `Ce matin`) est attribuable à Whisper, pas au LLM — observable même sans post-process.

## Decisions — Business model (session 1)

Brainstorming en cinq questions/réponses a abouti à :

### Positionnement projet
- **Type** : side project alimenté par passion + utilité personnelle quotidienne, avec ambition d'upgrade si ça décolle (entre "side hustle monétisé" et "startup naissante").
- **Plancher acceptable** : revenu couvrant 1-2 jours/mois de maintenance.
- **Scénario idéal** : milliers d'utilisateurs, plusieurs milliers €/mois, possibilité de bascule à plein temps.
- **Filet de sécurité acquis** : showcase commercial pour missions externes — bénéfice indépendant du revenu.
- **Référence pricing dans la tête de l'auteur** : 10-15 €/mois (SuperWhisper / WisprFlow déjà payés perso).

### Personas
**Payants identifiés :**
- Ami business international (FR/EN/ES/ZH au quotidien dans une entreprise multinationale) — mobile + desktop
- Développeurs / power users IA (le user lui-même + ses pairs) — mobile + desktop
- Pros sensibles à la confidentialité (médecins, commerciaux) — usage mixte selon mobilité

**Non-payants identifiés :**
- Conjoint / proches utilisateurs occasionnels — mobile uniquement, faible besoin
- Non-utilisateurs IA / non-bureautique — n'utiliseront probablement pas

### Modèle économique tranché

| | Desktop | Mobile Free | Mobile Pro |
|---|---|---|---|
| **Tarif** | Gratuit | Gratuit (bêta = tout débloqué) | ~quelques €/mois post-bêta |
| **Code** | Open source MIT | Open source MIT | Open source MIT |
| **Gating** | Aucun | Aucun | StoreKit 2 IAP |
| **Rôle** | Produit d'appel + showcase + bac à sable + outil quotidien perso | Acquisition grand public | Monétisation power users |

**Stratégie Open Core** (déjà documentée dans dictus-ios issue #54) :
- Tout le code reste public, le gating Pro se fait par vérification d'achat StoreKit
- Promesse "no bait-and-switch" : les features actuellement gratuites le restent à vie
- Pendant la bêta, toutes les features Pro sont débloquées gratuitement
- Modèle inspiré de Bitwarden / Cal.com / GitLab

### Différenciation vs concurrence (SuperWhisper, WisprFlow, Wispr Flow)
**Argument central** : open source MIT + 100% local sur toutes les plateformes → privacy garantie par construction, vérifiable indépendamment.

### Conséquences immédiates pour Dictus Desktop
- **Pas d'entitlement / licensing à coder** côté desktop — énorme simplification.
- **La traduction est first-class sur desktop**, pas premium, pas cachée derrière Experimental.
- **Le post-process LLM est first-class** — sortir le toggle PostProcessing du gate Experimental.
- **La friction technique Ollama est acceptable** sur desktop (audience power users). Pas besoin de model downloader intégré à court terme.
- **Desktop informe mobile** : ce qu'on développe et valide sur desktop sera porté en feature Pro sur mobile (Smart Mode templates, traduction, custom vocab, etc.).

### Mapping cross-plateforme des features (référence)

| Feature | Desktop | Mobile Free | Mobile Pro |
|---|---|---|---|
| Transcription locale | ✅ Gratuit | ✅ Gratuit | ✅ Inclus |
| Smart Mode base (polish léger) | ✅ Gratuit | ✅ Gratuit | ✅ Inclus |
| Smart Mode Pro (templates Email/SMS/Notes) | ✅ Gratuit | ❌ | ✅ Pro |
| **Traduction locale multi-langue** | ✅ **Gratuit** | ❌ | ✅ **Pro Tier 2** |
| Custom vocabulary | ✅ Gratuit | ❌ | ✅ Pro Tier 1 |
| Historique + recherche | ✅ Gratuit (déjà là) | ❌ | ✅ Pro Tier 1 |
| Audio file transcription | TBD desktop | ❌ | ✅ Pro Tier 1 |
| Voice actions / Voice shortcuts | TBD desktop | ❌ | ✅ Pro Tier 3 |

### Note de communication
Tension à expliquer dans le marketing : "*pourquoi mobile peut être payant si tout est open source ?*"
→ Réponse claire : modèle Open Core, le code reste auditable, le mobile vend le confort + l'écosystème + le soutien au projet, pas la fermeture du code. À muscler dans la FAQ landing au moment du lancement Pro.

## Open questions — Encore à trancher (sessions 2 et 3 à venir)

### Session 2 — Audit upstream Handy
15 commits Handy en avance sur notre `main` (jusqu'à `e3206aa`, v0.8.3 publié 2026-04-28). À classer en "à puller / à skipper / à inverser" avant tout chantier UX :
- `aee682f feat: add AWS Bedrock (Mantle) as post-processing provider` → **à skipper** (direction opposée à notre local-first)
- `a4d671a fix: improve German translation quality` → à investiguer (i18n UI ou translation feature ?)
- Reste majoritairement Nix, docs, GPU async — probablement neutre

### Session 3 — UX cible Translation Mode
**Choix UX à faire :**
- Nombre de slots de raccourcis traduction : fixe (3 hardcodés) vs extensible (l'utilisateur ajoute autant qu'il veut) ?
- Source language par slot : auto-détectée par Whisper vs imposée ?
- Le toggle "Translate to English" global existant : à garder, déprécier, ou fusionner dans le nouveau système ?
- Cohabitation avec les prompts custom power-user existants : présets + escape hatch advanced, ou présets only ?
- Refonte providers (chevauche Phase 8 GSD) : Apple Intelligence + Ollama (Custom) en première ligne, cloud regroupés sous "External (advanced)" ou totalement masqués ?
- Sortir le post-processing du gate Experimental — décision indépendante, peut se faire en parallèle.

**Choix backend à faire :**
- Étendre `bindings: HashMap<String, ShortcutBinding>` pour accepter N entrées `translate_to_<lang>` au lieu d'un seul `transcribe_with_post_process`.
- Mapping binding → prompt sélectionné (actuellement le prompt actif est global via `post_process_selected_prompt_id`).
- Préfabriquer des prompts "Translate to X" injectés au boot pour les langues principales.

### Bug Whisper cold-start
Identifié pendant les tests : premier mot avalé systématiquement (`Hier`, `Ce matin`). À investiguer **séparément** :
- `extra_recording_buffer_ms` dans les settings
- Délai entre `try_start_recording` (`actions.rs:438`) et arrivée du premier sample CPAL
- Mode `always_on_microphone` vs on-demand
→ Probablement un todo dédié, indépendant de la feature traduction.

### Prompt engineering — Patterns à corriger
Trois biais récurrents observés sur les traductions Apple Intelligence (à intégrer dans les prompts présets) :
1. Modalités / hedges parfois aplaties (`je crois que` → perdu) — ajouter `Preserve modal expressions and hedges`
2. Faux amis `ancien` → `old` (devrait être `former` / `antiguo` / `前`) — ajouter directive contextuelle
3. Calques de calque inversion grammaticale chinois (`没有见面了三年` au lieu de `三年没见面了`) — à voir si raffinable par prompt ou inhérent au modèle

## Suggested next steps (pas commencé)

1. **Session 2** — Lire les 15 commits upstream Handy, livrer un mini-rapport classant chaque commit. ~30 min.
2. **Session 3** — Wireframe / sketch UX cible (ASCII ou Figma rapide). 1-2h.
3. **Découpage en phases GSD** :
   - Phase 8 (existante, "Privacy / Local-First UX") → absorbe la refonte providers + sortir post-processing de Experimental
   - Phase 11 (nouvelle) à créer → "Translation Mode" : bindings extensibles + prompts présets + UX dédiée
   - Phase 12 ? (optionnelle) → cold-start Whisper + raffinement prompts
4. **Spike technique court** — preuve de concept hors GSD : brancher 1 binding `translate_to_es` câblé à un prompt Apple Intelligence préfabriqué, sans UX, juste pour valider la plomberie binding → prompt. Quelques heures de code.

## Acceptance criteria (pour quand on attaquera vraiment)

- [ ] Provider post-processing affiche Apple Intelligence + Ollama en première ligne, cloud regroupé/masqué
- [ ] Au moins 3 raccourcis traduction configurables avec langue cible présélectionnée (FR→EN, FR→ES, FR→ZH minimum)
- [ ] Prompts présets fournis par défaut, l'utilisateur n'a pas à les écrire pour les cas standards
- [ ] Mode "Custom prompt" reste accessible pour les power users qui veulent personnaliser
- [ ] Post-processing sorti du toggle Experimental, visible par défaut
- [ ] Aucune régression sur les features existantes (transcription pure, translate_to_english toggle natif Whisper, etc.)
- [ ] Privacy preserved : aucun appel réseau par défaut, cloud providers explicitement opt-in
- [ ] FAQ landing préparée pour expliquer le modèle Open Core (action marketing, hors-scope code)

## Notes

- Session de brainstorming menée le 2026-05-21, durée ~2h, alternant audit de code, tests empiriques en conditions réelles, et réflexion business.
- Tests effectués par l'utilisateur avec sa propre voix sur sa machine M4 Pro macOS 26.1, modèles Whisper Turbo et Medium, Apple Intelligence (modèle anglais).
- La feature est techniquement viable aujourd'hui : la plomberie LLM existe à 95%, le pipeline post-process est fonctionnel, le système de bindings est extensible par construction.
- Estimation grossière du chantier global (sans GSD strict) : 3-5 jours de travail focused pour livrer une feature utilisable, hors refonte providers (Phase 8) et hors investigation cold-start Whisper.
