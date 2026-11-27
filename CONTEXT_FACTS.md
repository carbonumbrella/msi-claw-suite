# Project Memory & Context Facts: MSI Claw 8 AI+ & Portal Ecosystem

This document compiles all verified empirical facts, hardware architectures, software implementations, user corrections, and project constraints established across the project. It contains zero speculative deductions or unverified assumptions.

---

## 1. Hardware Architecture & System Constraints (MSI Claw 8 AI+)

- **Device Model**: MSI Claw 8 AI+ (Model A2VM), Lunar Lake architecture, Intel Core Ultra 7 258V / 200V series processor, 32GB LPDDR5x 8533 MHz memory, Intel Arc 140V integrated GPU (Xe2).
- **Display Subsystem**:
  - Panel Model: CSW PN8007QB1-2, 8.0-inch IPS, native resolution 1920x1200 pixels (16:10 aspect ratio), native 120 Hz refresh rate, eDP internal connection.
  - DisplayID 2.0 Descriptor: Timing controller (TCON) reports hardware timing parameters of 48..120 Hz adaptive sync (`MonMinHz = 48.0`, `MonMaxHz = 120.0`).
- **Sleep & Power Constraints**:
  - Modern Standby (Connected Standby / S0ix) is disabled and unsupported on this device.
  - Any sleep command or power button trigger must execute hibernation (`shutdown.exe /h`).
- **Desktop & Window Management**:
  - Handheld form-factor rule: All launched and focused application windows must be maximized (`SW_MAXIMIZE` = 3, `SC_MAXIMIZE`).
  - Releasing foreground lock (`SPI_SETFOREGROUNDLOCKTIMEOUT`) from background daemons requires simulated Alt key events (`keybd_event(0x12, ...)`).
  - System Process Protection: Desktop shell `explorer.exe` is protected. Closing file explorer windows targets only `CabinetWClass` window handles without killing the shell process.

---

## 2. Display Refresh Rates, VRR & ArcSyncSwitch

### 2.1 Intel Graphics Driver Profiles (`ControlLib.dll`)
Intel graphics driver manages Variable Refresh Rate (VRR / Intel Arc Sync) through internal profiles:
- **Profile 2 (`EXCELLENT`)**:
  - Active range: `48..120 Hz` (`ActiveMinHz = 48`, `ActiveMaxHz = 120`, step 0 us).
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
- **ClawTweaks VRR Profile Warning**: Pruned. The user confirmed this was unverified in practice; the only verified cause of 60 FPS stutter was active statistics overlays.
- **LFC Triggering Below 60 FPS in Stock**: Pruned. Speculation that stock LFC kicks in strictly below 60 FPS was a theoretical deduction, not an empirically observed metric. The documented matrix floor for frame doubling is 48 FPS.

### 2.6 Utility Implementation (`ArcSyncSwitch`)
- Path: `C:\ClaudeCode\ArcSyncSwitch`.
- Language & Runtime: C#, .NET 10 (WPF).
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

### 3.1 Portal Design & Quality Checklist
- **Strict Prohibition on Dashes**:
  - Never use em dash or en dash anywhere in user-facing content, UI copy, or chat responses. Use colons, commas, dots, or parentheses.
- **Navigation Controls Sizing**:
  - Back button (`.btn-back`) and language switcher (`.lang-switch`) in guide headers must have identical height: `32px` (`height: 32px; box-sizing: border-box;`).
  - Back button contains an inline SVG arrow (`<svg class="btn-back-arrow" width="14" height="14" viewBox="0 0 24 24"...>`), vertically and horizontally centered with flexbox.
  - Responsive behavior: On mobile (<480px), `.btn-back-desktop` ("На главную" / "Back Home") is hidden and `.btn-back-mobile` ("Назад" / "Back") is shown, preventing overlap with the language switch.
- **Suite Software Catalog**:
  - `SaySo` (v1.0.0, voice dictation and control, RU / EN).
  - `ClawAmbilight` (v1.0.0, dynamic RGB backlight from display edges, RU / EN).
  - `kbdhaptic` (v1.0.0, haptic tactile vibration on virtual keyboard input, RU / EN).
  - `ArcSyncSwitch` (v1.0.0, Intel Arc Sync profile and LFC switcher, ControlLib.dll, D3DKMT, RU / EN).
  - Third-party utilities cataloged in Downloads: `ClawTweaks` and `OptiClick` (both explicitly badged as "Стороннее ПО" / "Third-Party").
- **Asset Links**:
  - `assets/img/arcsync-preview.png` (copied from ArcSyncSwitch docs preview).
  - All suite apps include official bilingual support (RU / EN) with automatic OS language detection.

---

## 4. Custom Resolutions & Display Scaling

- **Native Resolution**: `1920x1200` (16:10).
- **Custom 16:10 Modes** (injected via registry overrides without modifying EDID hardware blocks):
  - `1600x1000` (scale factor 1.20x, -31% GPU raster load vs 1200p).
  - `1536x960` (scale factor 1.25x, -36% GPU raster load vs 1200p).
  - Standard Windows 16:10 resolutions: `1680x1050` (-23% GPU load), `1440x900` (-44% GPU load).
- **Intel Graphics Command Center Scaling**:
  - Configured to "Maintain Aspect Ratio" (Сохранить соотношение сторон) or "Centered" (По центру) on the Display tab to prevent stretching.

---

## 5. Local Voice Stack (`BootCD` / `C:\AI\openvino-npu`)

- **ASR Model**: OpenVINO Whisper small int8 running locally on Intel NPU (`C:\AI\openvino-npu\whisper-small-int8-ov`).
- **Dictation Daemon**: `C:\AI\openvino-npu\dictate.py` (Push-to-talk on rear M1 paddle / Ctrl+Alt+Win+W; agent invocation on Ctrl+Win+Shift+G). Hot-reloads `gemma_agent.py` on each voice trigger via `importlib.reload`.
- **LLM Server**: OpenVINO `gemma-3-4b-it-int4-cw-ov` running on resident FastAPI NPU server at `http://127.0.0.1:8081` (`npu_server.py`, port 8081).
- **Voice Agent**: `C:\AI\openvino-npu\gemma_agent.py` (~35 lines system prompt, ~250 tokens, 16 tools). Direct Win32 API execution, deterministic CPU math, Windows TTS (Irina, Russian).

---

## 6. Verification Scripts & Tools

- Location: `c:\ClaudeCode\WinMCPHuawei\BootCD\scratch/`.
- `verify_all.py`: Validates zero em/en dashes, 100% valid anchor targets, and 100% valid relative links across all 14 HTML files.
- `verify_checklist.py`: Validates UI control heights (32px), SVG arrows, and catalog items.
- `audit_vrr_claims.py`: Validates consistency of VRR and LFC text across the portal.
