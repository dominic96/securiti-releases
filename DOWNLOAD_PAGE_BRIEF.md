# Securiti Website — Download Page Brief

> **For: Claude Design / Web Designer**
> This document contains everything needed to build the `/download` page of the Securiti website.
> The download page serves two things: the Windows Desktop Server installer and the mobile/web app links.

---

## 1. Page Goal

Users arriving at `/download` are one of two types:

1. **New customer** — just had cameras installed, needs the Desktop Server to make them work.
2. **Returning user** — needs to update to a newer version, or is finding the app link.

The page should make the primary action — **downloading the Windows installer** — immediately obvious. App links are secondary.

---

## 2. Installer File — Where It Lives

The installer is hosted on GitHub Releases:

| Field | Value |
|---|---|
| **GitHub repo** | `https://github.com/dominic96/securiti-releases` |
| **Latest release page** | `https://github.com/dominic96/securiti-releases/releases/latest` |
| **Direct download (latest)** | Use `assets[0].browser_download_url` from the API — filename is versioned (e.g. `Securiti_Desktop_Setup_1.2.0.exe`) |
| **GitHub Releases API** | `https://api.github.com/repos/dominic96/securiti-releases/releases/latest` |

### Current Version

| Field | Value |
|---|---|
| Version tag | **v1.2.0** |
| Release date | June 2026 |
| File name | `Securiti_Desktop_Setup_1.2.0.exe` |
| File size | **377 MB** |
| Platform | Windows 10 / Windows 11 (64-bit) |

---

## 3. How to Auto-Load the Latest Version (Dynamic Integration)

> **Important:** Do not hardcode the version number, file size, or release date on the page.
> Use the GitHub API so the version table updates automatically every time a new release is published — no website edits needed.

### API Endpoint

```
GET https://api.github.com/repos/dominic96/securiti-releases/releases/latest
```

No authentication required (public repo).

### API Response — Fields to Use

```json
{
  "tag_name": "v1.2.0",
  "name": "Securiti Desktop Server v1.2.0",
  "published_at": "2026-05-21T17:20:38Z",
  "assets": [
    {
      "name": "Securiti_V1_Setup.exe",
      "browser_download_url": "https://github.com/dominic96/securiti-releases/releases/download/v1.2.0/Securiti_V1_Setup.exe",
      "size": 395411456,
      "download_count": 0
    }
  ]
}
```

### What to Display (mapped from API)

| On-page element | API field | Format |
|---|---|---|
| Version badge | `tag_name` | Display as-is: `v1.2.0` |
| Release date | `published_at` | Format as: `May 2026` or `21 May 2026` |
| File size | `assets[0].size` | Convert bytes → MB: `Math.round(size / 1048576) + " MB"` |
| Download button URL | `assets[0].browser_download_url` | Direct link — use as `href` on the download button |
| Download count | `assets[0].download_count` | Optional — can show as social proof: "X downloads" |

### Implementation (JavaScript fetch — runs in the browser)

```javascript
async function loadLatestRelease() {
  const res = await fetch(
    'https://api.github.com/repos/dominic96/securiti-releases/releases/latest'
  );
  const data = await res.json();
  const asset = data.assets[0];

  document.getElementById('version-badge').textContent = data.tag_name;
  document.getElementById('release-date').textContent  = new Date(data.published_at)
    .toLocaleDateString('en-GB', { month: 'long', year: 'numeric' });
  document.getElementById('file-size').textContent     = Math.round(asset.size / 1048576) + ' MB';
  document.getElementById('download-btn').href         = asset.browser_download_url;
}

loadLatestRelease();
```

### Fallback (if API is unreachable)

Hardcode a fallback directly on the button so users can still download if the API call fails:

```html
<a id="download-btn"
   href="https://github.com/dominic96/securiti-releases/releases/latest"
   download>
  Download for Windows
</a>
```

The `/releases/latest/download/` URL always redirects to the newest release automatically — so even without the API call, this URL stays valid forever.

---

## 4. Page Layout & Copy

### Hero / Top Section

**Background:** Dark navy `#060061`
**Logo:** White wordmark (top-left in nav)

```
[Section eyebrow chip]  FREE DOWNLOAD

[H1]  Get Securiti on every device.

[Sub-heading]
The Desktop Server runs on your Windows PC at your property.
The app runs on your phone — anywhere in the world.
```

---

### Section A — Desktop Server (PRIMARY, above the fold)

This is the most important section. Display it prominently.

```
[H2]  Securiti Desktop Server for Windows

[Body paragraph]
The Desktop Server is the brain of Securiti. It runs silently in the background
on any Windows PC at your property, connects to your CCTV cameras, runs all AI
inference locally, and keeps your Securiti app in sync with everything happening
on-site. Install once — it starts automatically every time Windows starts.
```

**Download card — design as a prominent card with navy border or fill:**

```
┌──────────────────────────────────────────────────────────────┐
│  [Windows logo icon]   Securiti Desktop Server               │
│                                                              │
│  Version: [v1.2.0 ← from API]                               │
│  Released: [May 2026 ← from API]                             │
│  Platform: Windows 10 / 11  ·  64-bit                        │
│  Size: [377 MB ← from API]                                   │
│                                                              │
│  [↓  Download for Windows]  ← PRIMARY CTA BUTTON (navy fill) │
│                                                              │
│  ⚠  Requires Administrator privileges to install             │
└──────────────────────────────────────────────────────────────┘
```

**Below the card — "What gets installed?" accordion/expandable:**

```
[Expandable section heading]  What does the installer set up?

When you run the installer it automatically installs and configures
three Windows background services:

  ● Nginx               — local reverse proxy (routes traffic between services)
  ● Securiti Web Server — local backend (auth, AI events, Firebase sync, web portal)
  ● Securiti Video Server — C++ AI engine (connects cameras, runs AI, live streams)

Also installed:
  ● AI model files — YOLOv8n, ArcFace, SCRFD face detector, fire detector, Re-ID models
  ● Visual C++ Redistributable (no-op if already installed on your PC)
  ● Local web portal — accessible from your browser at http://localhost:8000
```

**System Requirements table (below the accordion):**

| Requirement | Minimum | Recommended |
|---|---|---|
| Operating System | Windows 10 64-bit | Windows 11 64-bit |
| Processor | Intel Core i5 / AMD Ryzen 5 (quad-core) | Intel Core i7 / AMD Ryzen 7 |
| RAM | 8 GB | 16 GB (for 4+ cameras with AI) |
| Storage | 2 GB free | 10 GB (for event snapshot cache) |
| Network | Wi-Fi — same network as cameras | Wired Ethernet (more reliable for RTSP) |
| Camera | Any RTSP-capable IP camera | Any brand — Hikvision, Dahua, Reolink, etc. |

**Call to action after requirements:**

```
[Info box]
Not sure how to get started? The Getting Started Guide walks you through
installation and your first live camera in under 10 minutes.

[→ Read the Getting Started Guide]   (links to /guides/getting-started)
```

---

### Section B — All Version History

Below the main download card, include a collapsible or secondary table of all published versions. This lets users downgrade if needed and shows the product is actively maintained.

**Table:**

| Version | Release Date | Platform | Size | Download |
|---|---|---|---|---|
| v1.2.0 *(Latest)* | June 2026 | Windows 10/11 64-bit | 377 MB | [Download] |

> This table should also be loaded from the GitHub Releases API:
> `GET https://api.github.com/repos/dominic96/securiti-releases/releases`
> Returns an array — map each item the same way as the latest release.

---

### Section C — App Downloads

**H2:** `Securiti App — Android, iOS & Browser`

**Body:**
```
Watch live streams, review AI alerts, manage visitors, track guards,
and control your cameras — from anywhere in the world.
```

**Three download cards (side by side on desktop, stacked on mobile):**

**Card 1 — Android**
```
[Google Play badge image — official asset from developer.android.com]
Minimum Android 8.0
[Download on Google Play]  → [TBD — Play Store URL]
```

**Card 2 — iOS**
```
[App Store badge image — official asset from developer.apple.com]
Minimum iOS 14
[Download on the App Store]  → [TBD — App Store URL]
```

**Card 3 — Browser**
```
[Globe / browser icon]
No install needed.
Works on Chrome, Safari, Edge, Firefox.
[Open Web App →]  → [TBD — Web App URL]
```

> **Designer note:** Until the app store links are live, render the Android and iOS badges
> as greyed-out / reduced opacity with a `cursor: not-allowed` style and a "Coming Soon"
> tooltip on hover. Do NOT link them to dead URLs. The browser web app link may be available
> sooner — confirm with developer before launch.

---

### Section D — Installation Steps (optional, compact)

A compact 3-step visual below the app cards reinforces how easy setup is:

```
Step 1                    Step 2                    Step 3
[Download icon]           [Plug/connect icon]       [Phone icon]
Download & install        Add your cameras          Monitor anywhere
Run Securiti_V1_Setup.exe Add cameras by RTSP URL   Open the app on your
as Administrator.         in the app or local       phone or browser.
Takes under 5 minutes.    web portal.               Live. From anywhere.
```

---

## 5. Brand & Style Notes for This Page

| Element | Value |
|---|---|
| Page background | White `#FFFFFF` |
| Hero/header section | Navy `#060061` with white text |
| Primary download button | Navy `#060061` fill, white text, rounded corners (`border-radius: 8px`) |
| Button hover state | `#040040` (slightly darker navy) |
| System requirements table | Light navy tint `#E8E8F5` header row |
| Accordion/expandable | Soft grey border, white bg, navy chevron icon |
| App store cards | White cards with `1px` light grey border, shadow on hover |
| Section spacing | `80px` vertical padding between major sections on desktop, `48px` on mobile |

**Version badge style:**
```css
background: #E8E8F5;
color: #060061;
border: 1px solid #060061;
border-radius: 4px;
font-size: 12px;
font-weight: 600;
padding: 2px 8px;
letter-spacing: 0.05em;
```

---

## 6. SEO Metadata for This Page

```html
<title>Download Securiti Desktop Server — Windows Installer</title>
<meta name="description" content="Download the Securiti Desktop Server for Windows. Connects your CCTV cameras to AI-powered cloud monitoring. Free download. Windows 10/11 64-bit.">
<meta property="og:title" content="Download Securiti Desktop Server">
<meta property="og:description" content="AI-powered CCTV surveillance for Windows. Free download.">
```

---

## 7. Full Download Button — Copy Variants

Use the primary. Alternatives are for A/B testing:

| Label | Usage |
|---|---|
| `↓  Download for Windows` | **Primary — use this** |
| `↓  Download Securiti Desktop Server` | If more descriptive label needed |
| `↓  Free Download — Windows 10/11` | If "free" needs to be emphasised |
| `Get Securiti for Windows →` | CTA in nav / hero section |

---

## 8. File & Asset Checklist for the Developer

Before the download page goes live, confirm:

- [x] GitHub release `v1.2.0` is published and `Securiti_V1_Setup.exe` is attached ✅
- [x] API call `https://api.github.com/repos/dominic96/securiti-releases/releases/latest` returns the asset ✅
- [x] Direct URL `https://github.com/dominic96/securiti-releases/releases/latest/download/Securiti_V1_Setup.exe` triggers a download in the browser ✅
- [ ] Google Play Store URL confirmed and live — **TBD**
- [ ] Apple App Store URL confirmed and live — **TBD**
- [ ] Web App URL confirmed — **TBD**
- [ ] Official Google Play badge downloaded from `developer.android.com/distribute/marketing-tools`
- [ ] Official App Store badge downloaded from `developer.apple.com/app-store/marketing/guidelines`

---

## 9. Publisher Information

| Field | Value |
|---|---|
| App name (installer) | Securiti Desktop |
| Publisher | Colours Universal Inc |
| Website | universesecurity.app |
| Support email | *(TBD — add before launch)* |

---

*This brief was prepared by Dominic Mundirewa — June 2026.*
*All technical details are accurate to the shipped v1.2.0 installer.*
