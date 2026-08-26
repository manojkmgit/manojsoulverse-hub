# Yaayavar Music Link Page

A mobile-first music catalogue and Instagram bio-link page for ManojYaayavar. The site presents a featured release, artist-level streaming links, and direct Spotify, Apple Music, YouTube Music, and Amazon Music links for every song.

- Live site: [https://yaayavar.com](https://yaayavar.com)
- GitHub repository: [manojkmgit/manojsoulverse-hub](https://github.com/manojkmgit/manojsoulverse-hub)

## Architecture

This is a static site with no framework, package manager, database, or server-side application. GitHub Pages serves the files directly from the repository root.

```text
Browser
  |
  +-- index.html             Page structure, responsive CSS, and rendering logic
  +-- site-data.json         Profile, artist links, and feature flags
  +-- songs.json             Song catalogue and song-level platform links
  +-- local image/audio      Profile image and optional preview clips
  +-- Font Awesome CDN       Platform icons
```

When the page loads, `index.html` fetches `site-data.json` and `songs.json` in parallel. A timestamp query parameter and `cache: "no-store"` are used to avoid stale JSON. JavaScript then renders the profile, featured release, artist links, searchable catalogue, and optional preview controls.

The catalogue displays 10 songs initially and loads additional songs in groups of 10, so it can comfortably support catalogues containing 30, 50, or more releases.

## Project files

| File | Purpose |
| --- | --- |
| `index.html` | Complete UI, responsive styling, search, pagination, link rendering, and preview-player logic. |
| `site-data.json` | Active site-wide profile data, artist-level streaming links, and feature flags. |
| `songs.json` | Active song catalogue. This is the main file to update when adding releases. |
| `face_pen_final.png` | Local profile image, favicon, social preview image, and fallback artwork. |
| `clip-whai-kissa.mp3` | Optional local preview audio for “Wahi Kissa.” |
| `CNAME` | GitHub Pages custom domain: `yaayavar.com`. |
| `data.json` | Legacy data retained for reference. It is not loaded by the current page. |
| `index.backup.html` | Local backup when present. It is intentionally not committed or deployed. |

Album artwork is currently loaded from external Spotify image URLs. Platform links open the corresponding music service in a new tab.

## Song data

Songs are stored in the `songs` array in `songs.json`:

```json
{
  "songs": [
    {
      "title": "Song Name - हिन्दी नाम",
      "albumArt": "https://example.com/cover.jpg",
      "featured": true,
      "preview": "optional-preview.mp3",
      "links": {
        "spotify": "https://open.spotify.com/...",
        "apple": "https://music.apple.com/...",
        "youtube": "https://music.youtube.com/...",
        "amazon": "https://music.amazon.com/..."
      }
    }
  ]
}
```

Notes:

- The text before ` - ` is displayed as the primary song title; the text after it is displayed as the secondary title.
- Set `featured` to `true` on the release that should appear in the featured section. If no song is marked, the first song is used.
- Keep only one song marked as featured to make the selection unambiguous.
- `preview` is optional and should point to an audio file in the repository or another accessible URL.
- A missing or `#` platform URL is ignored by the renderer.
- Song order in the JSON determines catalogue order.

## Site-wide data and previews

`site-data.json` contains:

- `profile`: display name, profile photo, Instagram handle, and social URLs.
- `listenAll`: artist-page links for the four supported platforms.
- `features.previewEnabled`: global switch for song previews.

Preview playback is currently disabled:

```json
"features": {
  "previewEnabled": false
}
```

Change it to `true` to show preview controls for songs that have a valid `preview` field. The player supports play/pause, a progress indicator, and animated equalizer bars.

The introductory profile wording shown on the page is currently defined in `index.html`; the profile identity, photo, and Instagram link come from `site-data.json`.

## Run locally

Serve the directory over HTTP because browser security rules can block JSON requests when `index.html` is opened directly from the filesystem.

From the project directory, run:

```powershell
python -m http.server 8000
```

Open:

```text
http://127.0.0.1:8000/
```

Stop the server with `Ctrl+C`.

There is no compilation or dependency installation. Changes to HTML or JSON do not require a server restart. Refresh the browser; use a hard refresh if an old page remains visible.

To validate edited JSON in PowerShell:

```powershell
Get-Content -Raw songs.json | ConvertFrom-Json | Out-Null
Get-Content -Raw site-data.json | ConvertFrom-Json | Out-Null
```

## GitHub Pages deployment

GitHub Pages is configured to publish from:

```text
Repository: manojkmgit/manojsoulverse-hub
Branch:     main
Directory:  / (repository root)
Domain:     yaayavar.com
```

Every push to `main` starts a GitHub Pages deployment. A normal update workflow is:

```powershell
git pull --rebase origin main
git add index.html songs.json site-data.json README.md
git commit -m "Describe the change"
git push origin main
```

Only add files that were intentionally changed. `index.backup.html` should remain local.

Check deployment in **GitHub repository → Settings → Pages**, or with the GitHub CLI:

```powershell
gh api repos/manojkmgit/manojsoulverse-hub/pages/builds/latest `
  --jq '{status: .status, commit: .commit, error: .error.message}'
```

A deployment is complete only when its status is `built` and its commit matches the pushed commit. Verify important data updates directly with a cache-busting URL such as:

```text
https://yaayavar.com/songs.json?version=TIMESTAMP
```

## Custom-domain DNS

The active DNS configuration for the apex domain uses GitHub Pages' four IPv4 addresses:

| Type | Name | Value |
| --- | --- | --- |
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| CNAME | `www` | `manojkmgit.github.io.` |

The GitHub domain-verification TXT record should also remain in DNS. Email-related MX, SPF, DMARC, DKIM, and bounce records are independent of GitHub Pages and should not be removed unless the related email service is intentionally decommissioned.

Do not add a GoDaddy `WebsiteBuilder Site` record at `@`. It connects the domain to GoDaddy's Coming Soon/website service and conflicts with GitHub Pages. Once DNS is correct and GitHub provisions the certificate, **Enforce HTTPS** should remain enabled in the Pages settings.

## Updating a release

1. Add or edit the song in `songs.json`.
2. Confirm the platform URLs and artwork URL.
3. Move `featured: true` to the intended featured release.
4. Validate the JSON.
5. Test at `http://127.0.0.1:8000/`, including a narrow mobile viewport.
6. Commit and push the change.
7. Wait for the GitHub Pages build to report `built`.
8. Verify both the live page and live `songs.json`.

Because the site is entirely static, click analytics, persistent counters, form handling, or other server-side features require an external analytics or backend service.
