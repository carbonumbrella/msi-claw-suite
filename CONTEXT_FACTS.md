# Project Memory & Context Facts: MSI Claw 8 AI+ & Portal Ecosystem

This document compiles verified empirical facts, hardware architectures, software implementations, user measurements, external specifications, and project constraints established across the project.

---

## 1. Hardware Architecture & System Constraints (MSI Claw 8 AI+)

- **Device Model**: MSI Claw 8 AI+ (Model A2VM), Lunar Lake architecture, Intel Core Ultra 7 258V processor (8C/8T), 32GB LPDDR5x 8533 MT/s unified memory, Intel Arc 140V integrated GPU (Xe2, 8 Xe-cores, official max dynamic GPU clock `1950 MHz`).
  - Reference sibling model: MSI Claw A1M (Core Ultra 7 155H, Meteor Lake) official max dynamic GPU clock is `2250 MHz`.
- **GPU Matrix Architecture (XeSS)**:
  - Lunar Lake Arc 140V (Xe2) includes dedicated hardware **Intel XMX** matrix engines. Intel **XeSS runs natively on XMX** on this chip (`DP4a` is only a fallback path on GPUs lacking XMX).
- **Shared GPU Memory Override**:
  - Out of the box in Intel Graphics Software on 32 GB Lunar Lake, the default `Shared GPU Memory Override` is **57%** (~18.2 GB, vs Windows WDDM default 50% / 16 GB; Intel notes a known issue when setting below 50%).
  - On the local reference Claw 8 AI+ (driver `32.0.101.9175`), `HardwareInformation.qwMemorySize` is manually raised to 28.0 GB (`30064771072` bytes).
- **Display Subsystem**:
  - Panel Model: CSW PN8007QB1-2, 8.0-inch IPS, native resolution 1920x1200 pixels (16:10 aspect ratio), native 120 Hz refresh rate, eDP internal connection.
  - DisplayID 2.0 Descriptor: Timing controller (TCON) exposes hardware adaptive sync range of 48..120 Hz (exposed through `ControlLib.dll` monitor info as `48.0..120.0 Hz`, mapped to `MonMinHz`/`MonMaxHz` in `ArcSyncEngine.cs`).
  - Color calibration: Notebookcheck laboratory review measured a cool out-of-the-box white point of **7,852 K** (the portal notes ~7200K) and published `PN8007QB1_2_cal.icm` targeting 6500K sRGB. On the local system, the Windows `Calibration Loader` scheduled task is `Ready` and loads the ICC `vcgt` LUT at logon.
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
- **Upper VRR Ceiling (Blur Busters G-SYNC 101)**: On 120 Hz VRR displays, capping at `116` or `117 FPS` (~0.3 ms below 120.0 Hz) keeps VRR engaged and prevents V-Sync buffer latency spikes at the ceiling.

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

## 3. Game Tuning, User Benchmarks & Handheld Software

### 3.1 User-Verified Frame Generation & Latency Measurements
- **OptiScaler (`XeSS` vs `FSR 4 XMX`)**: Measured on Lunar Lake: native XeSS takes ~2+ ms per frame vs ~5+ ms for FSR 4 XMX.
- **Frame Generation Overhead**:
  - Native XeFG incurs ~5:10% base render cost (~1.75x:1.80x effective multiplier; 45 baseline FPS -> ~78:82 displayed FPS).
  - External Lossless Scaling (`LSFG`) incurs ~10:15% base render cost (~1.7x effective multiplier; 40:45 baseline FPS -> ~70:78 displayed FPS).
- **Low Latency Mode**: `On + Boost` can cause micro-stutters on battery at low TDP (15:17 W); `On` is preferred on battery.
- **CPU-Bound Engines**: User verified Ubisoft Anvil titles require CPU boost enabled and higher PL2 headroom.
- **Intel Endurance Gaming (Official Spec)**: Active when unplugged; presets `Performance` (60 FPS cap), `Balanced` (40 FPS cap), `Battery Saver` (30 FPS cap).

### 3.2 ClawTweaks (`v1.0.7.42`) & MSI Center M Verified Details
- **MSI Center M Factory Fan Curves (Verified in Registry `HKLM:\SOFTWARE\WOW6432Node\MSI\MSI Center M\Component\User Scenario`)**:
  - `Default_Temp`: `0;50;60;70;80;88` (6 temperature points per fan).
  - `Default_Fan`: `0;40;49;58;67;75` (confirming stock 80°C = `67%` and 88°C = `75%`).
  - `High_Fan`: `0;40;58;70;85;110`.
  - `ECversion`: `1T52EMS1.109...` (`1T52` = Claw 8 AI+ A2VM).
- **ClawTweaks Capabilities (Verified in Binaries/UI)**:
  - `Extended range (>75%)` (`MsiFan_Extended`) removes the 75% MSI ceiling (`MSI max 75%`) and `Apply fan curve` writes custom curves. Local `settings.json` records `MsiFan_FactoryDuty6 = "0,40,58,70,85,110"` and 75%-capped `MsiFan_Curve3 = "1|0,40,58,70,75,75|0,40,58,70,75,75"`.
  - Built-in `A2vmPl1MinusOneToggle` (`PL2 = PL1 + 1W`), `MsiCenterManager` (plus standalone `MsiCenterToggle` utility in ecosystem), virtual controller targets (`Xbox 360`, `DualShock4`, `DualSense`, `Steam Deck`), `Gyro-to-Stick`, and `Battery Charge Limit` (`MsiChargeLimit_Pct`).
  - LED modes in ClawTweaks (`MsiLedModeCombo`): `Static`, `Breathing`, `Color Cycle`, `Wave`, `Battery`, `Rainbow`. ClawTweaks does **not** include screen-reactive Ambilight (`ClawAmbilight` is a separate utility).
- **MSI Center M Verified Behavior**:
  - On the local machine, `MSI_Center_M_Server` and `MSI_Center_M_Updater` tasks are `Disabled` (as required when ClawTweaks manages hardware).
  - Historical `MSI Center M` OSD debug logs (`C:\Program Files (x86)\MSI\MSI_Center_M\OSD\intelGEDll_debug_20261014..17.txt`) measured `10.2 MB` to `17.9 MB` per daily log (12.7 MB, 17.9 MB, 10.2 MB, 4.7 MB).

---

## 4. Web Portal `msi-claw-suite` Structure & Standards

- **Project Location**: `C:\ClaudeCode\msi-claw-suite`.
- **Git Repository**: Remote `origin/main` on GitHub (`carbonumbrella/msi-claw-suite`), branch `main`.
- **Structure (14 HTML Pages)**:
  - Root: `index.html` (Russian portal homepage)
  - English Root: `en/index.html` (English portal homepage)
  - Russian Guides: `guides/setup-guide.html`, `guides/power-and-sleep.html`, `guides/game-tuning.html`, `guides/memory-and-stability.html`, `guides/software-choice.html`, `guides/vrr-display.html`.
  - English Guides: mirrored under `en/guides/`.
  - Verified local OS build: Windows 11 Home SL build `26300` (25H2 branch).

### 4.1 Portal Design & Quality Checklist
- **Strict Prohibition on Dashes**: Never use em dash or en dash anywhere in user-facing content, UI copy, or chat responses.
- **Navigation Controls Sizing**: `.btn-back` and `.lang-switch` locked to `32px` height; `@media (max-width: 768px)` switches `.btn-back-desktop` to `.btn-back-mobile`.
- **Suite Software Catalog (Actual Repo Versions vs Portal Table)**:
  - `SaySo`: Repo/installer `v0.3.17` (portal `#downloads` currently lists `v0.3.7`). Full bilingual UI (RU / EN) with OS auto-detection implemented in `C:\ClaudeCode\SaySo-Dev` (`src/Lang.cs`, UI, wizard, tray, overlay, installer; `C:\ClaudeCode\SaySo` mirrors pre-l10n snapshot).
  - `ClawAmbilight`: Repo/installer `v0.9.9` (portal `#downloads` currently lists `v0.9.8`). Bilingual UI (RU / EN) with OS language auto-detection; CPU usage documented uniformly as `~1.5%` of one CPU core (until live in-game verification).
  - `kbdhaptic`: Repo release `v0.9.3` (portal `#downloads` currently lists `v0.9.2`). Bilingual UI (RU / EN) with OS language auto-detection.
  - `ArcSyncSwitch`: Repo/installer & portal `v1.0.0`. Bilingual UI (RU / EN) with OS language auto-detection.
  - `Optimize-Handheld`: `v1.2.0`; `MSI Claw Resolution Pack`: `v1.0.0`; ICC profile: `CSW0801` (`PN8007QB1_2_cal.icm`).
  - Third-party utilities cataloged in Downloads: `ClawTweaks` and `OptiClick`.

---

## 5. Custom Resolutions & Display Scaling

- **Native Resolution**: `1920x1200` (16:10).
- **Custom 16:10 Modes** (written as Windows Registry `EDID_OVERRIDE` descriptors without modifying hardware EEPROM or DisplayID 2.0 TCON blocks):
  - `1600x1000` (scale factor 1.20x, -30.5% exact / -31% rounded GPU raster load vs 1200p).
  - `1536x960` (scale factor 1.25x / 80%, -36% GPU raster load vs 1200p).
  - Standard Windows 16:10 resolutions: `1680x1050` (-23% GPU load), `1440x900` (-44% GPU load), `1280x800` (-56% GPU load).
- **Intel Graphics Command Center / ClawTweaks Scaling**: `GPU Scaling` and `Preserve Aspect Ratio`.

---

## 6. Local Voice Stack (`SaySo` / `C:\AI\openvino-npu`)

- **ASR Catalog & Default**: `WHISPER` catalog in `voice_settings.py` defines `small` (`OpenVINO/whisper-small-int8-ov`, 244 MB, NPU/GPU/CPU), `turbo` (`OpenVINO/whisper-large-v3-turbo-int8-ov`, 789 MB, GPU/CPU), and `turbo-npu` (`OpenVINO/whisper-large-v3-turbo-fp16-ov`, 1552 MB, NPU). Under `stt_model: "auto"`, **`small` (`whisper-small-int8-ov`) is the default on NPU and CPU** (while `turbo` is default on GPU). On the local Claw (`stt_state.json`), `whisper-large-v3-turbo-fp16-ov` is currently active on NPU (`turbo int8` fails with `ZE_RESULT_ERROR_DEVICE_LOST` on Lunar Lake NPU).
- **Tray Supervisor & Dictation Daemon**: Managed by `SaySo.exe` tray app (`C:\ClaudeCode\SaySo`); elevated task `SaySoHotkeys` forwards hotkey events over UDP `47321` to `C:\AI\openvino-npu\dictate.py` (`Win+Ctrl+Alt+W` dictation, `Win+Ctrl+Shift+G` assistant, plus one-button mode). `dictate.py` hot-reloads `gemma_agent.py` via `importlib.reload`.
- **LLM Server**: OpenVINO `OpenVINO/gemma-3-4b-it-int4-cw-ov` (`npu_server.py`, port `8081`), **unloaded by default** (`gemma_state.txt = off`) and started on demand. Optional cloud provider (`gemini-3.5-flash-lite`) is also supported.
- **Voice Agent**: `C:\AI\openvino-npu\gemma_agent.py` uses deterministic Python `fast_route()` for most commands and a 32-line `SYSTEM` prompt (~845 tokens measured with schema/examples, 11 built-in tools plus dynamic plugin/user tools). Notifications display via custom WPF `OverlayWindow` (UDP `47330`, toast fallback); TTS supports Windows system voices (Irina) and optional Silero TTS.

---

## 7. Verification Scripts & Tools

- Location: `c:\ClaudeCode\WinMCPHuawei\BootCD\scratch/`.
- `verify_all.py`: Validates zero em/en dashes, valid internal `#anchor` targets, and valid relative file links across all 14 HTML files.
- `verify_checklist.py`: Validates UI control heights (`32px`), SVG arrows (`btn-back-arrow`), and catalog items (`ClawTweaks`, `OptiClick`).
- `audit_vrr_claims.py`: Scans HTML files for VRR, 48-120 Hz, and automatic transition claims.

---

## 8. Pending Tasks / Confirmed Website Fixes Needed

1. **VRR Display Guide (`guides/vrr-display.html`, `en/guides/vrr-display.html`)**:
   - Fix Section 2 table (RU L646, EN L569) and "Автоматический переход в играх" / "Automatic Gaming Transition" (RU L680-683, EN L603-606): stock driver runs `COMPATIBLE` (`60..120 Hz`); `48..120 Hz` requires switching to `EXCELLENT` via `ArcSyncSwitch`.
   - Prune unverified `56..62 FPS` / lack of hysteresis LFC oscillation claim in `#lfc-boundary` (RU L729, EN L651) and `guides/game-tuning.html` Section 5 (RU/EN L647).
   - Clarify `Calibration Loader` scheduled task applies the ICC LUT at logon (vs saying no loader is involved).
2. **Game Tuning Guide (`guides/game-tuning.html`, `en/guides/game-tuning.html`)**:
   - Fix XeSS on Lunar Lake description (L545): Arc 140V runs XeSS natively on **Intel XMX** matrix cores, not `DP4a` fallback.
3. **Memory Guide (`guides/memory-and-stability.html`, `en/guides/memory-and-stability.html`)**:
   - Fix audio services section (RU L714-717, EN L680-683): `Optimize-Handheld` disables optional **DTS (`DtsApo4Service`) and MSI intelliGo (`IgoAudioService`)** (~90 MB), not `Nahimic` / `Realtek Audio Effects` (250 MB).
   - Add `Fast Boot` off caveat to `RB + RT` BIOS combo.
4. **Software Choice Guide (`guides/software-choice.html`, `en/guides/software-choice.html`)**:
   - Remove erroneous attribution of screen-reactive Ambilight to `ClawTweaks` (L670; `ClawAmbilight` provides screen Ambilight).
5. **Index Pages (`index.html`, `en/index.html`)**:
   - Update catalog versions to actual releases (`SaySo v0.3.17`, `ClawAmbilight v0.9.9`, `kbdhaptic v0.9.3`).
   - Standardize `ClawAmbilight` CPU load wording across comparison table and card to `~1.5%` of one CPU core (until live in-game verification).
   - Align `1600x1000` load reduction percentage (`30.5%` vs `31%` in `game-tuning.html`).
6. **Setup, White Point & Repository Docs (`setup-guide.html`, `vrr-display.html`, `index.html`, `AGENTS.md`)**:
   - Remove duplicate `Win + Ctrl + Shift + B` driver restart step after `Run_As_Admin.cmd` (which already restarts the graphics driver).
   - Update factory white point reference from `~7200K` to Notebookcheck's measured `7,852 K`, and note Windows `Calibration Loader` task for ICC LUT loading.
   - Align footer Windows version (`25H2` for build `26300`) and add `game-tuning.html` to `msi-claw-suite/AGENTS.md`.
