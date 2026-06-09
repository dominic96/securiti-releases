# Securiti Desktop Server — Releases

> **"Nowhere to hide for the bad guys"**

This repository hosts the official installer releases for the **Securiti Desktop Server** — the on-premises Windows service that connects your CCTV cameras to the Securiti AI surveillance platform.

---

## Latest Release

See the [**Releases**](https://github.com/dominic96/securiti-releases/releases/latest) page to download the latest version.

---

## System Requirements

| Requirement | Value |
|---|---|
| Operating System | Windows 10 / Windows 11 (64-bit) |
| Processor | Intel Core i5 / AMD Ryzen 5 or better (quad-core minimum) |
| RAM | 8 GB minimum — 16 GB recommended for 4+ cameras with AI |
| Storage | 2 GB for installation |
| Network | Wired or Wi-Fi on the same network as your CCTV cameras |
| Camera compatibility | Any RTSP-capable IP camera (Hikvision, Dahua, Reolink, etc.) |

---

## What Gets Installed

The installer sets up three Windows background services that start automatically with Windows:

- **Nginx** — local reverse proxy
- **Securiti Web Server** — Flask application (auth, AI events, Firebase sync, local web portal)
- **Securiti Video Server** — C++ engine (RTSP capture, AI inference, MJPEG/WebSocket streaming)

AI model files and the Visual C++ Redistributable are also installed automatically.

---

## Installation Guide

Full step-by-step installation instructions are available on the Securiti website:

👉 **[Getting Started Guide](https://getsecuriti.com/guides/getting-started)** *(link live on website launch)*

---

## Get the App

| Platform | Link |
|---|---|
| Android | [Google Play Store](#) *(coming soon)* |
| iOS | [Apple App Store](#) *(coming soon)* |
| Browser | [Open Web App](#) *(coming soon)* |

---

## Version History

| Version | Release Date | Notes |
|---|---|---|
| [v1.2.0](https://github.com/dominic96/securiti-releases/releases/tag/v1.2.0) | June 2026 | Initial public release |

---

## Support

For help, installation issues, or feature requests, contact us at **support@getsecuriti.com** *(placeholder — update before launch)*.

---

*© 2026 Securiti. This repository contains installer binaries only — no source code.*
