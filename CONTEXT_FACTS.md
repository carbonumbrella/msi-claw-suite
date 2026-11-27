# Project Memory & Context Facts: MSI Claw 8 AI+ & Portal Ecosystem

This document compiles verified empirical facts, hardware architectures, software implementations, user corrections, and project constraints established across the project. It avoids speculative deductions and unverified assumptions.

---

## 1. Hardware Architecture & System Constraints (MSI Claw 8 AI+)

- **Device Model**: MSI Claw 8 AI+ (Model A2VM), Lunar Lake architecture, Intel Core Ultra 7 258V processor (8C/8T), 32GB LPDDR5x 8533 MT/s unified memory, Intel Arc 140V integrated GPU (Xe2).
- **Display Subsystem**:
  - Panel Model: CSW PN8007QB1-2, 8.0-inch IPS, native resolution 1920x1200 pixels (16:10 aspect ratio), native 120 Hz refresh rate, eDP internal connection.
  - DisplayID 2.0 Descriptor: Timing controller (TCON) exposes hardware adaptive sync range of 48..120 Hz (exposed through `ControlLib.dll` monitor info as `48.0..120.0 Hz`, mapped to `MonMinHz`/`MonMaxHz` in `ArcSyncEngine.cs`).
- **Sleep & Power Constraints**:
  - Windows S0 Modern Standby is intentionally disabled and replaced with S4 hibernation in OS optimization (`Optimize-Handheld`).
  - When `can_sleep()` returns false, `gemma_agent.py` routes voice sleep requests to hibernation (`shutdown.exe /h`).
- **Desktop & Window Management**:
  - Handheld form-factor rule: Launched and focused application windows are maximized (`SW_MAXIMIZE` = 3).
  - Window activation (`activate()` in `gemma_agent.py`) uses `AllowSetForegroundWindow(-1)`, `SW_MAXIMIZE`, `SetForegroundWindow`, and `SwitchToThisWindow`. (`keybd_event(0x12, ...)` is used for `Alt+Enter` fullscreen toggling.)
  - System Process Protection: Desktop shell `explorer.exe` (and `applicationframehost`, `dwm`, etc.) is in `PROTECTED`. Closing explorer windows targets only `CabinetWClass` handles via `SC_CLOSE`.

---

## 2. Display Refresh Rates, VRR & ArcSyncSwitch

### 2.1 Intel Graphics Driver Profiles (`ControlLib.dll`)
Intel graphics driver manages Variable Refresh Rate (VRR / Intel Arc Sync) through internal profiles:
- **Profile 2 (`EXCELLENT`)**:
  - Active range: `48..120 Hz` (`ActiveMinHz = 48`, `ActiveMaxHz = 120`, step 0 us in preview UI).
  - Unlocks native 1x VRR down to the hardware floor of 48 Hz.
- **Profile 4 (`COMPATIBLE`)**:
  - Stock driver profile by default.
  - Active range: `60..120 Hz` (`ActiveMinHz = 60`, `ActiveMaxHz = 120`).
  - Clamps the active 1x VRR floor to 60 Hz.
- **Profile 5 (`OFF`)**:
  - Active range: `120..120 Hz` (VRR disabled, fixed refresh rate).

### 2.2 Verified Driver Behavior in 2D vs 3D
- In 2D Windows desktop mode, Advanced Display Settings reports `60..120 Hz`. This is governed by Windows Dynamic Refresh Rate (DRR) and panel flicker prevention on static content.
- When launching a 3D application in fullscreen, the stock Intel driver switches to profile `COMPATIBLE` (`60..120 Hz`).
- **Correction of prior inaccuracy**: The stock driver **does not** automatically lift the 60 Hz limit to 48 Hz in games. The active floor remains 60 Hz in stock. Unlocking the true hardware `48..120 Hz` range requires switching the driver profile to `EXCELLENT` via `ArcSyncSwitch`.

### 2.3 Low Framerate Compensation (LFC)
- LFC is controlled via private driver escape commands through the Windows Graphics Kernel Subsystem (`D3DKMTEscape`).
- Driver flags: `LowFpsSolutionEnabled` (Op 3 to enable, Op 4 to disable) and `HighFpsSolutionEnabled` (Op 5 to enable, Op 6 to disable).
- When active, the driver duplicates or triplicates refresh cycles when render framerates drop below the minimum threshold (e.g., 35 FPS runs at 70 Hz).
- Toggling LFC on/off can be performed independently of the VRR profile in `ArcSyncSwitch`.

### 2.4 Performance Statistics Overlays
- **Verified Empirical Fact**: Running an active on-screen performance/statistics overlay (FPS counter, telemetry HUD, RivaTuner/RTSS) over a game window disrupts display smoothness at 60 FPS.
- If stuttering or pacing issues occur near 60 FPS, the statistics overlay should be completely turned off first before modifying VRR profiles or LFC.

### 2.5 Pruned Unverified Claims (User Corrections)
- **ClawTweaks VRR Profile Warning**: Pruned from portal guides per user confirmation (unverified in practice; only active statistics overlays were confirmed to break 60 FPS smoothness). Note: `ArcSyncSwitch/README.md`, `README_RU.md`, and `src/Lang.cs` (`TipVrr`) still retain the older ClawTweaks VRR and `Independent Flip / MPO` text.
- **LFC Triggering Below 60 FPS in Stock**: Pruned. Speculation that stock LFC kicks in strictly below 60 FPS was a theoretical deduction, not an empirically observed metric. The documented panel floor for frame doubling is 48 FPS.

### 2.6 Utility Implementation (`ArcSyncSwitch`)
- Path: `C:\ClaudeCode\ArcSyncSwitch`.
- Language & Runtime: C#, .NET 10 (WPF), version `1.0.0`.
- Architecture:
  - Calls `ControlLib.dll` to read and set Intel VRR profiles.
  - Calls `D3DKMTEscape` to query and toggle LFC flags.
  - Does not modify EDID in registry or hardware EEPROM. Custom resolutions (`1600x1000`, `1536x960`) remain completely intact.
  - Scheduled Task (`--apply` flag): Silently reapplies the user-chosen profile at user logon without admin prompts, then immediately terminates with zero background overhead.
  - Bilingual UI: Russian and English with automatic OS language detection (`auto`) and manual override.

---

## 3. Web Portal `msi-claw-suite` Structure & Standards

- **Project Location**: `C:\ClaudeCode\msi-claw-suite`.
- **Git Repository**: Remote `origin/main` on GitHub (`carbonumbrella/msi-claw-suite`), branch `main`.
- **Structure (14 HTML Pages)**:
  - Root: `index.html` (Russian portal homepage)
  - English Root: `en/index.html` (English portal homepage)
  - Russian Guides:
    - `guides/setup-guide.html`: Step-by-step clean installation and driver sequence.
    - `guides/power-and-sleep.html`: Hibernation, TDP balancing (PL1/PL2), fan curves, power bank specs.
    - `guides/game-tuning.html`: FPS capping strategy, XeSS/FSR upscaling, OptiClick/LSFG frame generation.
    - `guides/memory-and-stability.html`: Pagefile allocation, Shared VRAM (16 GB), memory integrity.
    - `guides/software-choice.html`: Ecosystem comparison (ClawTweaks vs MSI Center M), Handheld Companion.
    - `guides/vrr-display.html`: CSW PN8007QB1-2 panel analysis, DisplayID 2.0, VRR ranges, LFC physics, color calibration.
  - English Guides:
    - `en/guides/setup-guide.html`
    - `en/guides/power-and-sleep.html`
    - `en/guides/game-tuning.html`
    - `en/guides/memory-and-stability.html`
    - `en/guides/software-choice.html`
    - `en/guides/vrr-display.html`
  - Note: `msi-claw-suite/AGENTS.md` is slightly outdated and lists 5 guides (omitting `game-tuning.html`).

### 3.1 Portal Design & Quality Checklist
- **Strict Prohibition on Dashes**:
  - Never use em dash or en dash anywhere in user-facing content, UI copy, or chat responses. Use colons, commas, dots, or parentheses.
- **Navigation Controls Sizing**:
  - Back button (`.btn-back`) and language switcher (`.lang-switch`) in guide headers must have identical height: `32px` (`height: 32px; box-sizing: border-box;`).
  - Back button contains an inline SVG arrow (`<svg class="btn-back-arrow" width="14" height="14" ...>` styled to `13px`), vertically and horizontally centered with flexbox.
  - Responsive behavior: Under `@media (max-width: 768px)`, `.btn-back-desktop` ("На главную" / "Back Home") is hidden and `.btn-back-mobile` ("Назад" / "Back") is shown, preventing overlap with the language switch.
- **Suite Software Catalog (Actual Repo Versions vs Portal Table)**:
  - `SaySo`: Repo/installer `v0.3.17` (portal `#downloads` currently lists `v0.3.7`). Supports RU / EN / Auto speech recognition and commands; WPF Settings UI (`SettingsWindow.xaml`) is currently Russian-only.
  - `ClawAmbilight`: Repo/installer `v0.9.9` (portal `#downloads` currently lists `v0.9.8`). Bilingual UI (RU / EN) with OS language auto-detection.
  - `kbdhaptic`: Repo release `v0.9.3` (portal `#downloads` currently lists `v0.9.2`). Bilingual UI (RU / EN) with OS language auto-detection.
  - `ArcSyncSwitch`: Repo/installer & portal `v1.0.0`. Bilingual UI (RU / EN) with OS language auto-detection.
  - `Optimize-Handheld`: `v1.2.0`; `MSI Claw Resolution Pack`: `v1.0.0`; ICC color profile: `CSW0801` (`PN8007QB1_2_cal.icm`).
  - Third-party utilities cataloged in Downloads: `ClawTweaks` and `OptiClick` (both explicitly badged as "Стороннее" / "Third-party").
- **Asset Links**:
  - `assets/img/arcsync-preview.png` (copied from ArcSyncSwitch docs preview).

---

## 4. Custom Resolutions & Display Scaling

- **Native Resolution**: `1920x1200` (16:10).
- **Custom 16:10 Modes** (written as Windows Registry `EDID_OVERRIDE` descriptors without modifying hardware EEPROM or DisplayID 2.0 TCON blocks):
  - `1600x1000` (scale factor 1.20x, -30.5% exact / -31% rounded GPU raster load vs 1200p).
  - `1536x960` (scale factor 1.25x / 80%, -36% GPU raster load vs 1200p).
  - Standard Windows 16:10 resolutions: `1680x1050` (-23% GPU load), `1440x900` (-44% GPU load), `1280x800` (-56% GPU load).
- **Intel Graphics Command Center / ClawTweaks Scaling**:
  - Configured to `GPU Scaling` and `Preserve Aspect Ratio` (`Сохранять пропорции`) to prevent stretching.

---

## 5. Local Voice Stack (`SaySo` / `C:\AI\openvino-npu`)

- **ASR Model**: Active model on Claw NPU is OpenVINO `whisper-large-v3-turbo-fp16-ov` (`stt_state.json`; `turbo int8` triggers `ZE_RESULT_ERROR_DEVICE_LOST` on Lunar Lake NPU). `whisper-small-int8-ov` remains available as a lightweight fallback.
- **Tray Supervisor & Dictation Daemon**: Managed by `SaySo.exe` tray app (`C:\ClaudeCode\SaySo`); elevated task `SaySoHotkeys` forwards hotkey events over UDP `47321` to `C:\AI\openvino-npu\dictate.py` (`Win+Ctrl+Alt+W` dictation, `Win+Ctrl+Shift+G` assistant, plus one-button mode). `dictate.py` hot-reloads `gemma_agent.py` via `importlib.reload`.
- **LLM Server**: OpenVINO `OpenVINO/gemma-3-4b-it-int4-cw-ov` (`npu_server.py`, port `8081`), **unloaded by default** (`gemma_state.txt = off`) and started on demand. Optional cloud provider (`gemini-3.5-flash-lite`) is also supported.
- **Voice Agent**: `C:\AI\openvino-npu\gemma_agent.py` uses deterministic Python `fast_route()` for most commands and a 32-line `SYSTEM` prompt (~845 tokens measured with schema/examples, 11 built-in tools plus dynamic plugin/user tools). Notifications display via custom WPF `OverlayWindow` (UDP `47330`, toast fallback); TTS supports Windows system voices (Irina) and optional Silero TTS.

---

## 6. Verification Scripts & Tools

- Location: `c:\ClaudeCode\WinMCPHuawei\BootCD\scratch/`.
- `verify_all.py`: Validates zero em/en dashes, valid internal `#anchor` targets, and valid relative file links across all 14 HTML files.
- `verify_checklist.py`: Validates UI control heights (`32px`), SVG arrows (`btn-back-arrow`), and catalog items (`ClawTweaks`, `OptiClick`).
- `audit_vrr_claims.py`: Scans HTML files for VRR, 48-120 Hz, and automatic transition claims.

---

## 7. Current In-Progress Task: VRR Display Guide Surgical Correction

### Objective
Correct the inaccurate statement in `guides/vrr-display.html` and `en/guides/vrr-display.html` regarding automatic 60 Hz limit removal in games.

### Affected Files and Lines
1. `guides/vrr-display.html` (lines 680-683, Section 2, sub-item "Автоматический переход в играх", plus table cell at line 646):
   - Current text (lines 680-683):
     "При запуске трехмерной игры в полноэкранном режиме графический драйвер переключается в специализированный игровой профиль. Ограничение 60 Гц мгновенно снимается, открывая полный диапазон 48-120 Гц и подключая алгоритм компенсации низкой частоты кадров (LFC)."
   - Target replacement:
     Clarify that upon launching a 3D title, the stock Intel driver switches to profile COMPATIBLE (clamped to 60..120 Hz). The 60 Hz limit is not automatically removed on its own. To lift the 60 Hz clamp and unlock the true hardware 48..120 Hz range, the user must switch the driver into profile EXCELLENT via ArcSyncSwitch.
2. `en/guides/vrr-display.html` (lines 603-606, Section 2, sub-item "Automatic Gaming Transition", plus table cell at line 569):
   - Current text (lines 603-606):
     "Launching a 3D title in exclusive or borderless fullscreen switches the driver into its gaming profile. The 60 Hz floor lifts, opening the full 48-120 Hz hardware window and engaging Low Framerate Compensation (LFC)."
   - Target replacement:
     Mirror the Russian clarification in English: in stock, the driver remains in COMPATIBLE mode (60..120 Hz). Opening the full 48..120 Hz window requires switching to EXCELLENT via ArcSyncSwitch.

### Verification Requirements
- Execute `python c:\ClaudeCode\WinMCPHuawei\BootCD\scratch\verify_all.py` to ensure zero em/en dashes and 100% link resolution.
- Commit to `msi-claw-suite` and push immediately to `origin/main`.
