# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A single-file, browser-based high-frequency sound level meter (`splmeter.html`). It captures microphone input via the Web Audio API and displays the real-time and peak level (relative dBFS, not calibrated SPL) at a selected target frequency (19 kHz or 21 kHz). UI text and code comments are in Japanese.

There is no build system, package manager, linter, or test suite — HTML, CSS, and JS are all inline in one file with no external dependencies.

## Running

`getUserMedia` requires a secure context, so open the page via `localhost` or HTTPS (not `file://` on most mobile browsers):

- **VS Code Live Server** (ritwickdey, installed on the WSL side): click "Go Live", then open `http://127.0.0.1:5500/splmeter.html` in a Windows browser.
- **Without VS Code**:
  ```sh
  python3 -m http.server 8000   # then open http://localhost:8000/splmeter.html
  ```

Development happens in WSL2. WSL2 forwards ports to Windows `localhost`, so a Windows browser treats the page as a secure context and uses the Windows microphone — no mic setup is needed inside WSL. Use an external browser; VS Code's built-in preview panes (e.g. Live Preview) likely cannot get mic permission.

To test on a phone over the LAN, the page must be served over HTTPS, and WSL2 is not reachable from the LAN by default (needs Windows port forwarding or WSL mirrored networking).

## How the measurement works

- `toggleStart()` creates an `AudioContext` and requests the mic with `echoCancellation`, `noiseSuppression`, and `autoGainControl` all disabled — these must stay off, or browser processing distorts the high-frequency level.
- An `AnalyserNode` with `fftSize = 4096` feeds `getFloatFrequencyData()` (dB values) each `requestAnimationFrame` in `analyze()`.
- The target bin is `round(targetFreq / (sampleRate/2) * fftSize/2)`, and the reading is the max over that bin ±1 to tolerate small frequency drift.
- 21 kHz is only measurable when the device's `sampleRate` is ≥ 44.1 kHz (Nyquist ≥ 22.05 kHz); actual response near these frequencies depends heavily on the device's microphone.
- Peak is held in `peakDb` and is reset whenever the frequency changes (`setFrequency()` calls `resetPeak()`).
- The "rotate 180°" button toggles a `.rotated` class on `<body>` (for holding the device upside down so the mic points at the source).
