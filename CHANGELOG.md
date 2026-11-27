# CHANGELOG

## Decisions and Architectural Rules

### 2026-11-27: Apply Audited Factual Corrections and Tighten Portal Copy Across All 14 Pages
- **Decision:** Reconciled every factual discrepancy found during the portal audit against Intel/Notebookcheck docs, local machine state, and source repositories, and compressed prose across both index pages and all 12 RU/EN guides without removing technical parameters:
  - Corrected Lunar Lake Intel XeSS execution path (`Intel XMX` matrix engines with `DP4a` fallback only for non-XMX GPUs).
  - Clarified stock Intel Arc driver (`COMPATIBLE`, `60..120 Hz` LFC threshold near 60 FPS) vs `ArcSyncSwitch` (`EXCELLENT`, `48..120 Hz`), pruning unverified `56..62 FPS` hysteresis oscillation wording.
  - Standardized `ClawAmbilight` CPU load estimate to `~1.5%`, updated factory white point to `~7852K` (Notebookcheck), and documented Windows `Calibration Loader` scheduled task behavior.
  - Corrected audio cleanup details in the memory guide to optional `VendorAudio` disabling DTS (`DtsApo4Service`) and MSI intelliGo (`IgoAudioService`) (`~90 MB`), removed duplicate `Win+Ctrl+Shift+B` restart instructions in `setup-guide`, added the `Fast Boot` disabled condition to `RB + RT` BIOS recovery, and removed screen Ambilight attribution from `ClawTweaks` feature lists.
  - Updated catalog versions (`SaySo v0.3.17`, `ClawAmbilight v0.9.9`, `kbdhaptic v0.9.3`), aligned `1600x1000` load reduction to `30.5%`, updated Windows version compatibility (`24H2/25H2`), and added `game-tuning.html` to `AGENTS.md`.
- **Reason:** Eliminate unverified claims while cutting reading bloat so handheld users can scan specifications, commands, and workflows quickly without losing technical detail.

### 2026-11-27: Compile Comprehensive Context Facts and Eliminate Speculative Deductions
- **Decision:** Compile a comprehensive, strictly empirical `CONTEXT_FACTS.md` document covering:
  - Hardware specifications and environmental constraints for MSI Claw 8 AI+.
  - Intel Arc Sync driver profiles (`EXCELLENT` vs `COMPATIBLE` vs `OFF`) and verified driver behavior.
  - Low Framerate Compensation (LFC) control flags via `D3DKMTEscape`.
  - Impact of performance telemetry overlays on 60 FPS smoothness.
  - Explicit pruning of speculative deductions (ClawTweaks VRR toggle warning and stock LFC frequency threshold speculation).
  - Portal architecture, design checklist standards (32px controls, SVG arrow centering, zero em/en dashes), and suite application catalog.
- **Reason:** Provide an authoritative, detailed factual memory file prior to context compaction, eliminating all unverified claims.

### 2026-11-27: Document Verified Performance Overlay Impact on 60 FPS Smoothness
- **Decision:** Document the verified hardware and driver interaction regarding performance overlays:
  - VRR and Display guide (`guides/vrr-display.html`): in `#lfc-boundary`, document that active on-screen performance/statistics overlays over the game window disrupt frame delivery smoothness at 60 FPS, advising users to disable overlays if 60 FPS stutter occurs.
  - Game tuning guide (`guides/game-tuning.html`): add a targeted note under 60 FPS capping to disable on-screen telemetry overlays.
- **Reason:** Accurately convey verified user findings on how active telemetry overlays affect frame pacing at 60 FPS on MSI Claw 8 AI+.

### 2026-11-27: Integrate ArcSyncSwitch and Document Suite-Wide Bilingual Support
- **Decision:** Integrate ArcSyncSwitch into the portal and document bilingual (RU / EN) support across all suite applications:
  - Portal showcase (`#apps`): add dedicated ArcSyncSwitch application card with ControlLib.dll driver integration, D3DKMT LFC controls, 48..120 Hz range unlocking, and preview screenshot in Russian and English index pages.
  - Bilingual localization: document official RU / EN bilingual support with automatic Windows language matching across all suite tools (SaySo, ClawAmbilight, kbdhaptic, ArcSyncSwitch).
  - Downloads catalog (`#downloads`): add ArcSyncSwitch v1.0.0 with direct installer and repository links.
  - VRR and Display guide (`guides/vrr-display.html`): update `#lfc-boundary` to document the root cause of 60 Hz micro-stutter (stock COMPATIBLE 60..120 Hz clamp) and the permanent fix via ArcSyncSwitch (EXCELLENT 48..120 Hz profile).
  - Game tuning guide (`guides/game-tuning.html`): update 60 FPS capping strategy in Section 5 with ArcSyncSwitch integration.
  - Pruned unverified warnings: removed speculative ClawTweaks VRR toggle and MPO overlay warnings from guides.
- **Reason:** Provide users with the driver-level solution for smooth 60 FPS gameplay on Lunar Lake and advertise bilingual software capabilities without unverified assumptions.

### 2026-11-22: Align Header Controls Geometry and Catalog Third-Party Utilities
- **Decision:** Perfect header layout geometry and expand the downloads catalog across the portal:
  - Header controls alignment: strictly lock `.lang-switch` and `.btn-back` outer height to 32px with box-sizing border-box across all 14 files, eliminating visual jumping and height mismatch.
  - Arrow centering: replace Unicode arrows with crisp 14x14 SVG vector arrows (`.btn-back-arrow`) inside `.btn-back`, achieving pixel-perfect vertical and horizontal alignment across all browsers and display scalings.
  - Mobile header clearance: hide `.logo-badge` and reduce `.nav-brand` gap to 8px under `@media (max-width: 768px)`, ensuring at least 45px of clear separation between `.lang-switch` and `.btn-back` on compact handheld screens.
  - Catalog expansion: add ClawTweaks and OptiClick to the downloads catalog (`#downloads`) in `index.html` and `en/index.html` with an amber `.tag-external` badge, official release links, and GitHub repository references.
- **Reason:** Prevent header control collision on narrow viewports, align control heights flush, and provide direct access to verified third-party utilities.

### 2026-11-22: Responsive Layout Audit and Touch Usability Enhancements
- **Decision:** Apply comprehensive layout and touch accessibility fixes across all 14 portal HTML files based on subagent audit findings:
  - Touch targets: expand `.lang-switch .lang-btn` tap target to minimum 38x36 px across all pages to meet mobile and handheld touchscreen accessibility standards.
  - Header responsiveness: introduce dual-label `.btn-back` (`.btn-back-desktop` and `.btn-back-mobile`) in all 12 guide pages, rendering concise "← Назад" / "← Back" on mobile to eliminate header button multi-line text wrapping on 375 px viewports.
  - Anchor offset consistency: standardize `scroll-margin-top: 80px` across all content sections and anchor targets (including `#lfc-boundary`), removing 24 px mobile regressions to guarantee sticky header clearance.
  - Mobile container alignment: harmonize mobile side padding to `0 16px` across `en/index.html` and all 10 guide pages under `@media (max-width: 768px)`.
  - Table and code formatting: unify `.table-responsive` and `.downloads-table-wrap` styles in `en/index.html` with explicit min-widths and custom scrollbars, and add dedicated monospace `<code>` styling to index pages.
- **Reason:** Guarantee smooth touchscreen navigation, eliminate layout shifts and header collisions on narrow mobile viewports, and maintain consistent styling standards.

### 2026-11-22: Expand 16:10 Resolution Spectrum in Game Tuning Guides
- **Decision:** Replace the binary 1200p vs 800p framing in Section 6 of `guides/game-tuning.html` and `en/guides/game-tuning.html` with the complete 16:10 resolution ladder:
  - Standard Windows modes: 1680x1050 (-23% GPU load) and 1440x900 (-44% GPU load).
  - Custom pack modes: 1600x1000 (-31% load, 1.20x scale) and 1536x960 (-36% load, 1.25x scale) bridging the gap with high pixel density.
  - Low-load mode: 1280x800 (-56% load) for extreme battery savings.
- **Reason:** Eliminate the false dichotomy between 1200p and 800p, guiding users to intermediate resolutions that preserve image sharpness while improving framerates.

### 2026-11-22: Clarify Discrete Framerate Values and Target Ranges in Guides
- **Decision:** Eliminate ambiguous colon ranges (`63:65`, `50:54`) in capping recommendations across Russian and English guides:
  - Replaced with discrete target values (e.g. 116 FPS ceiling, 63 FPS or any target from 63 to 116, 50 or 54 FPS or any target from 45 to 54) to reflect that UI frame limiters accept only individual scalar numbers.
  - Clarified that the unstable dead zone applies to any selected value from 56 to 62 FPS (specifically 58, 59, and 60).
- **Reason:** UI accuracy and clear user instruction for software limiters (RTSS, ClawTweaks, in-game).

### 2026-11-22: Document 116 FPS Upper VRR Ceiling and Comprehensive Safe Capping Tiers
- **Decision:** Document the Blur Busters standard 116 FPS upper limit and comprehensive safe capping tiers across Russian and English guides:
  - In `vrr-display.html`: define the full VRR envelope (`#lfc-boundary`) including the 116:117 FPS upper cap (providing ~0.3 ms margin to keep VRR continuously engaged without V-Sync buffer latency) alongside the 56:62 FPS LFC boundary zone.
  - In `game-tuning.html`: integrate full capping tiers into Section 5: 116 FPS ceiling for esports, 63:65 FPS for native 1x VRR, 50:54 FPS for stable 2x LFC (100:108 Hz), avoiding 56:62 FPS, and fixed 60 Hz mode when strictly 60 FPS is required.
- **Reason:** Provide exact, actionable frame capping standards for competitive smoothness and tear-free VRR operation.

### 2026-11-22: Document LFC 56:62 FPS Boundary Flutter and Remedies
- **Decision:** Document the Intel Arc Adaptive Sync Plus (ASP) LFC multiplier instability around 56:62 FPS across Russian and English guides:
  - In `vrr-display.html`: concisely explain driver-side 60 Hz LFC boundary oscillation caused by lack of hysteresis, linking to game tuning tactics.
  - In `game-tuning.html`: document actionable FPS capping rules: cap at 63 FPS for native 1x VRR, cap at 50 FPS for stable 2x LFC (100 Hz), or set a fixed 60 Hz display refresh with V-Sync if exactly 60 FPS is desired.
- **Reason:** Eliminate micro-tearing and stuttering reported at 58:60 FPS caps on 120 Hz VRR panels while maintaining clean separation of concerns without duplication.

### 2026-11-22: Correct Frame Generation Effective Multiplier and Base Overhead
- **Decision:** Correct theoretical 2.0x claims in `game-tuning.html` and `en/guides/game-tuning.html` to reflect physical frame generation performance:
  - Document that frame generation (XeFG and LSFG) incurs an intrinsic compute tax, lowering native rasterization speed by 5:15%.
  - Clarify that 45 baseline FPS translates into roughly 78:82 displayed FPS with native XeFG (~1.75x:1.80x effective boost) and 70:78 displayed FPS with external LSFG (~1.7x boost), rather than an exact 2.0x doubling.
- **Reason:** Physical accuracy and honest technical benchmarks without marketing claims of lossless 2.0x doubling.

### 2026-11-22: Action-First Streamlining of Game Tuning Guide and ClawTweaks Anti-Lag
- **Decision:** Streamline `game-tuning.html` and `en/guides/game-tuning.html` to place actionable settings and check-lists upfront:
  - Document that Low Latency Mode (Anti-Lag: On / Boost) and Display Scaling (GPU Scaling / Preserve Aspect Ratio) are accessible directly inside the ClawTweaks widget as well as Intel GCC.
  - Clarify that while recent OptiScaler builds support FSR 4 models running on Intel XMX cores (FSR 4 XMX / Light), native XeSS remains superior on Lunar Lake because FSR 4 XMX takes 5+ ms per frame versus 2+ ms for XeSS.
  - Shorten upscaler, frame-gen, framerate capping, and resolution sections down to concise, practical steps.
- **Reason:** Provide immediate, actionable guidance for handheld users with minimal reading friction.

### 2026-11-22: Strip Prose Fluff, Wordiness, and Marketing Epithets Across All Guides
- **Decision:** Remove non-essential prose, expository filler, pseudo-mathematical claims, and evaluative marketing adjectives across all Russian and English portal pages.
  - Replaced marketing terms ("инженерный", "математически выверенный", "математическая кратность", "сверхскоростной", "безупречный", "идеальный") with dry, factual technical descriptors ("архитектурный", "практический", "кастомные разрешения 16:10", "четкий").
  - Streamlined lead paragraphs and section intros in `game-tuning.html`, `memory-and-stability.html`, `software-choice.html`, `power-and-sleep.html`, `vrr-display.html`, and `index.html` to focus directly on actionable settings and hardware constraints.
  - Preserved 100% of factual technical parameters, hotkeys, commands, and numbers (1950 MHz, 48:120 Hz, 16384 MB, 1000p/960p, 65W/100W PD, 80°C/85% and 88°C/120%, 20-30 sec EC reset, RB + RT).
- **Reason:** Minimize reading friction on handheld/mobile screens while maintaining technical depth and zero AI tells.

### 2026-11-19: Remove BIOS VRAM Configuration and Marketing Buzzwords
- **Decision:** Eliminate all instructions requiring users to enter BIOS for VRAM/Shared GPU Memory on MSI Claw.
- **Reason:** Intel Core Ultra 200V (Lunar Lake) uses unified Memory-on-Package LPDDR5x (32 GB). The standard MSI Claw BIOS contains no UMA Frame Buffer setting. Windows Display Driver Model (WDDM) dynamically allocates up to 50% of system RAM (16 GB) as Shared GPU Memory automatically.
- **Decision:** Remove marketing fluff and evaluative adjectives ("выверенная для геймеров", "идеальный комфорт", "математически выверенные") across all guides and portal pages.
- **Reason:** Strict adherence to factual, concise, and laconic technical documentation standards without AI tells.

### 2026-11-19: Dedicated Software Choice Guide (ClawTweaks vs MSI Center M vs HandheldCompanion)
- **Decision:** Extract handheld management software comparison into dedicated guide `guides/software-choice.html` and link from `memory-and-stability.html` and portal index.
- **Reason:** Provide detailed hardware-level comparison: EC bus saturation by high-frequency WMI polling in HandheldCompanion, persistent background service memory overhead (350-600+ MB) and disk logging in MSI Center M, versus ClawTweaks out-of-the-box readiness, native Xbox Game Bar overlay, and ClawTweaksCenterFSE launcher with multi-store aggregation.

### 2026-11-20: Multilingual Portal Architecture and Dedicated Workspace
- **Decision:** Add full English version located in `/en/` (`en/index.html` and `en/guides/*.html`) alongside root Russian pages. Introduce language toggle `[RU | EN]` and `localStorage` preference persistence with initial `navigator.language` fallback.
- **Reason:** Broaden reach for international MSI Claw community while maintaining fast static loading, zero layout flicker (FOUT), and clean search engine indexing on GitHub Pages. Formalize `C:\ClaudeCode\msi-claw-suite` as the dedicated repository and working directory for all website assets.

### 2026-11-20: Add Display ICC Color Profile and Notebookcheck Calibration Source
- **Decision:** Host calibrated ICC color profile for panel CSW PN8007QB1-2 (`assets/calib/PN8007QB1_2_cal.icm`) and link to the Notebookcheck laboratory review across all portal index pages, download catalogs, and display/setup guides.
- **Reason:** Enable users to easily download the verified 6500K sRGB profile in one click and reference laboratory display measurements directly.

### 2026-11-22: Content Deduplication and Guide Scope Realignment
- **Decision:** Eliminate large-scale duplicate content across guides:
  - Remove Low Latency Mode and Intel Endurance Gaming sections from `vrr-display.html`, consolidating complete technical breakdowns in `game-tuning.html`.
  - Remove redundant upscaling/frame-gen section 5 from `software-choice.html`.
  - Remove standalone management software comparison section 4 from `memory-and-stability.html`, embedding a concise memory-footprint note referencing `software-choice.html`, and renumbering sections 1-6.
  - Streamline framerate capping and Endurance Gaming mentions in `power-and-sleep.html` into compact cross-references to `game-tuning.html`.
  - Streamline setup checklist steps in `setup-guide.html` and portal index cards to keep practical instructions while referencing authoritative deep-dive guides.
- **Reason:** Clean separation of concerns across documentation modules, preventing contradictory advice and reducing redundant reading volume while strictly preserving all facts, settings, and practical steps.

### 2026-11-22: Dedicated Game Tuning Guide, ClawTweaks Controller Subsystems, and Hardware Insights
- **Decision:** Extract game performance optimization, upscaling, and frame generation into a standalone monolithic guide `guides/game-tuning.html` (and `/en/guides/game-tuning.html`), linking it from the main portal grid and related guides.
- **Reason:** Users prioritize a single comprehensive reference on tuning games for maximum FPS and smoothness: Intel XeSS vs AMD FSR on Lunar Lake, native frame generation with OptiClick (OptiScaler), external LSFG 2x generation, Low Latency Mode On vs Boost, framerate capping over wattage throttling, and custom 16:10 resolutions (1000p/960p).
- **Decision:** Document full ClawTweaks controller emulation modes (Xbox 360, DualShock, Steam Controller), hardware Right Stick Gyroscope emulation, and Battery Charge Limit (60%/80%) across Russian and English software guides.
- **Reason:** Clarify that ClawTweaks enables universal gyro aiming via right stick mapping in any controller-compatible title without mouse emulation, alongside native gyro data pass-through for Sony titles and Steam Input.
- **Decision:** Document dock station power starvation with factory 65W charger (recommending 100W PD) and eGPU plug-and-play behavior over Thunderbolt 4 without DDU driver removal.
- **Reason:** Prevent battery drain during docked gaming sessions and simplify external GPU setup.

### 2026-11-21: Correct PL2 Switch Nomenclature and Maximum GPU Frequency
- **Decision:** Rename references from "TDP Boost" to "PL2 switch" (the exact name in ClawTweaks, disabled by default) and update GPU monitoring description to flat maximum 1950 MHz in Russian and English power guides.
- **Reason:** Accurate UI terminology matching ClawTweaks v2.x overlay controls and eliminating imprecise frequency qualifiers.
