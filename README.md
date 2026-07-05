# WisprFlow WIN

A Wispr-Flow-style dictation utility for **Windows 10/11**: press a global
hotkey, speak, and the transcribed text is pasted straight into whatever app
has focus. Speech-to-text runs on your own LAN GPU box via an
OpenAI-compatible `/v1/audio/transcriptions` endpoint (speaches /
faster-whisper-server / anything compatible).

Windows port of [wisprflow-lnx](https://github.com/spital/wisprflow-lnx). Built with
**Tauri v2 + Rust** (cpal/WASAPI → rubato → hound → reqwest) and a small
vanilla-TS web UI.

```
hotkey ──▶ record mic (cpal/WASAPI) ──▶ 16 kHz mono WAV ──▶ POST /v1/audio/transcriptions
                                                                   │
     focused app ◀── SendInput paste / Unicode type / clipboard ◀── {"text": …}
```

## Features

- **Global hotkey** (default `Ctrl+Alt+Space`), two modes:
  - *Toggle* — press to start, press to stop
  - *Push-to-talk* — hold while speaking
- **Insertion strategies** (all native Win32 `SendInput`, no external tools):
  - clipboard + `Ctrl+V` paste (default; old clipboard restored afterwards)
  - `KEYEVENTF_UNICODE` typing (layout-independent, full Unicode) with
    **auto-paste** upgrade for Chromium-family processes
  - clipboard-only with a notification
- **Elevated-app fallback**: Windows UIPI silently drops synthetic input into
  Administrator windows, so the app detects an elevated foreground process up
  front, leaves the transcript on the clipboard and notifies you to press
  Ctrl+V yourself.
- **Overlay pill** at the bottom of the screen with live level bars and
  elapsed time — never takes focus, clicks pass through.
- **Tray icon** with state (idle / recording / transcribing), toggle, health
  check and settings.
- **Settings window**: API URL/token, model + language (fetched from
  `/v1/models`), hotkey capture, input device, prompt, autostart, …
- **CLI control** of the running instance:
  `wisprflow-win --toggle | --start | --stop | --cancel | --quit`
- Robustness: request retry, max-recording watchdog, too-short discard,
  clipboard fallback when injection fails.

## Requirements

- Windows 10 or Windows 11 (x64)
- Microsoft Edge WebView2 Runtime (preinstalled on current Windows;
  the installer bootstraps it if missing)
- A working microphone, with desktop-app microphone access allowed:
  Settings → Privacy & security → Microphone → *Let desktop apps access
  your microphone*
- A reachable ASR endpoint, e.g. speaches serving
  `FILM6912/whisper-large-v3-turbo-ct2`

