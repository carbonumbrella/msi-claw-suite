# AGENTS.md - MSI Claw Suite Portal Operational Rules

## Project Overview
MSI Claw Suite is an open-source technical portal and community knowledge base for MSI Claw handheld consoles (MSI Claw 8 AI+, Claw 7 AI+, and Claw A1M).
The portal provides calibrated display resolutions, OS debloating and S4 hibernation tooling, background utilities (SaySo voice assistant, ClawAmbilight, kbdhaptic), and architectural guides comparing handheld software stacks.

Hosted on GitHub Pages from the repository `carbonumbrella/msi-claw-suite`.

## Dedicated Workspace Directory
- **Root Path**: `C:\ClaudeCode\msi-claw-suite`
- All site source files, stylesheets, images, and articles are strictly maintained in this directory.

## Site Structure & Multilingual Architecture
- **Russian Version (Root)**:
  - `index.html`: Portal home, software catalog, resolution specs, downloads.
  - `guides/setup-guide.html`: Step-by-step clean setup guide from scratch.
  - `guides/power-and-sleep.html`: Power management, S4 hibernation, TDP tuning.
  - `guides/vrr-display.html`: Display scaling, 16:10 custom resolutions, VRR.
  - `guides/memory-and-stability.html`: Unified LPDDR5x RAM, pagefile, HVCI.
  - `guides/software-choice.html`: ClawTweaks vs MSI Center M vs HandheldCompanion.
- **English Version (`/en/`)**:
  - `en/index.html`: Mirror home page translated into English.
  - `en/guides/setup-guide.html`: English setup guide.
  - `en/guides/power-and-sleep.html`: English power guide.
  - `en/guides/vrr-display.html`: English display & VRR guide.
  - `en/guides/memory-and-stability.html`: English memory & stability guide.
  - `en/guides/software-choice.html`: English software choice guide.
- **Assets**:
  - `assets/img/`: Screenshots and UI illustrations shared by both language versions.

## Language Switcher & Localization Logic
- Language toggle element: A compact pill button in the top navigation bar (`RU | EN`).
- Local preference persistence: `localStorage.getItem('site_lang')` stores user selection (`'ru'` or `'en'`).
- First-visit auto-detection: If `localStorage` is empty, query `navigator.language`. If it starts with `ru`, keep Russian; otherwise redirect/suggest English. Explicit user choice via the toggle overrides auto-detection permanently.

## Documentation & Code Style Rules
- **Punctuation**: STRICTLY ZERO dashes (`—` or `–`) anywhere in prose or HTML. Use colons, commas, dots, or parentheses. Hyphens in compound words are permitted.
- **Tone**: Technical, laconic, strictly fact-based. No marketing hype, superlatives, or AI tells.
- **Links**: All relative links must resolve cleanly across `/` and `/en/` hierarchies.

## Project Memory Files
- `CHANGELOG.md`: Accepted architectural decisions and reasons.
- `AGENTS.md`: Current operational context and rules.
