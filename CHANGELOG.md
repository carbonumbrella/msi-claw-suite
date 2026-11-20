# CHANGELOG

## Decisions and Architectural Rules

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
