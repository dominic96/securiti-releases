# Designer Update — Brief Corrections #001

> **Date:** June 2026
> **Applies to:** `/download` page
> **Priority:** Apply before launch — these affect correctness of download links and displayed data

This document lists every correction to the original `DOWNLOAD_PAGE_BRIEF.md` brief.
Read this alongside the updated brief — it is now the source of truth.

---

## Summary of Changes

Three things changed after the original brief was written:

1. **Installer filename is now versioned** — the `.exe` name changes with every release
2. **No values on the download page should be hardcoded** — version, date, size, and download URL must all come from the GitHub API
3. **The fallback download button URL changed** — the old static filename URL no longer works

---

## Change 1 — Installer Filename Is Now Versioned

### What changed
The installer filename pattern changed from a fixed name to a versioned name:

| | Filename |
|---|---|
| **Before** | `Securiti_V1_Setup.exe` |
| **After** | `Securiti_Desktop_Setup_{version}.exe` |

Current release example: `Securiti_Desktop_Setup_1.2.0.exe`

Future release example: `Securiti_Desktop_Setup_1.3.0.exe`

### What to update in your implementation
- Remove any reference to `Securiti_V1_Setup.exe` from the codebase
- Do **not** hardcode a filename anywhere — always read it from `assets[0].name` via the GitHub API
- The download button `href` must always come from `assets[0].browser_download_url` (the API returns the full correct URL)

---

## Change 2 — All Download Card Values Must Be Dynamic

### What changed
The original brief listed the current version's values as fixed reference points. These must not be hardcoded in the website — they change with every release.

| Field | ❌ Do not hardcode | ✅ Load from API field |
|---|---|---|
| Version badge | `v1.2.0` | `data.tag_name` |
| Release date | `June 2026` | `new Date(data.published_at).toLocaleDateString(...)` |
| File size | `377 MB` | `Math.round(assets[0].size / 1048576) + ' MB'` |
| Download URL | *(any URL with a filename)* | `assets[0].browser_download_url` |
| File name (if displayed) | `Securiti_V1_Setup.exe` | `assets[0].name` |

### The correct JavaScript (unchanged from brief — verify this is what you have)

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

---

## Change 3 — Fallback Download Button URL Changed

### What changed
The fallback `href` on the download button (used when the API call fails) previously used a static filename in the URL. Since the filename is now versioned and changes per release, that pattern no longer works as a permanent fallback.

| | URL |
|---|---|
| **Before** | `https://github.com/dominic96/securiti-releases/releases/latest/download/Securiti_V1_Setup.exe` |
| **After** | `https://github.com/dominic96/securiti-releases/releases/latest` |

### What to update
Change the download button's initial/fallback `href` to point to the releases page. This is a valid fallback — if the API call fails, clicking the button takes the user to the GitHub releases page where they can download manually.

```html
<!-- Fallback href — JavaScript will replace this with the direct download URL on load -->
<a id="download-btn"
   href="https://github.com/dominic96/securiti-releases/releases/latest"
   download>
  ↓  Download for Windows
</a>
```

---

## Change 4 — Version History Table Must Be Fully Dynamic

### What changed
The version history table (showing all past releases) must not contain any hardcoded rows. Every row — including v1.2.0 — must be loaded from the GitHub Releases API.

### API endpoint
```
GET https://api.github.com/repos/dominic96/securiti-releases/releases
```
Returns an array of all published releases. Map each item the same way as the latest:

```javascript
async function loadReleaseHistory() {
  const res = await fetch(
    'https://api.github.com/repos/dominic96/securiti-releases/releases'
  );
  const releases = await res.json();
  const tbody = document.getElementById('version-history-body');

  releases.forEach((release, index) => {
    const asset = release.assets[0];
    if (!asset) return; // skip releases with no attached file

    const row = document.createElement('tr');
    row.innerHTML = `
      <td>${release.tag_name}${index === 0 ? ' <span class="badge-latest">Latest</span>' : ''}</td>
      <td>${new Date(release.published_at).toLocaleDateString('en-GB', { month: 'long', year: 'numeric' })}</td>
      <td>Windows 10/11 64-bit</td>
      <td>${Math.round(asset.size / 1048576)} MB</td>
      <td><a href="${asset.browser_download_url}">Download</a></td>
    `;
    tbody.appendChild(row);
  });
}

loadReleaseHistory();
```

---

## Checklist — Apply Before Launch

- [ ] Remove all hardcoded references to `Securiti_V1_Setup.exe`
- [ ] Remove all hardcoded references to `377 MB`
- [ ] Remove all hardcoded version strings (`v1.2.0`, `1.2.0`)
- [ ] Remove all hardcoded release dates (`June 2026`, `May 2026`)
- [ ] Verify `loadLatestRelease()` populates version, date, size, and download URL correctly
- [ ] Verify download button fallback `href` points to `/releases/latest` (not a `.exe` URL)
- [ ] Verify version history table is built from `GET /releases` array (not static HTML rows)
- [ ] Smoke test: open browser DevTools → Network tab → confirm the GitHub API call returns `200` with assets
- [ ] Smoke test: disable JavaScript → confirm fallback `href` on button points to the releases page

---

## No Other Changes

All other content in `DOWNLOAD_PAGE_BRIEF.md` remains correct:
- Page layout, copy, and section structure — unchanged
- Brand colours and typography — unchanged
- System requirements table — unchanged (these are hardware specs, not release data)
- App download cards (Android / iOS / Browser) — unchanged (still TBD pending store listings)
- SEO metadata — unchanged
- Publisher information — unchanged

---

*Corrections confirmed by Dominic Mundirewa — June 2026.*
