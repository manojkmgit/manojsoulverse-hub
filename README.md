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

The profile identity, photo, bio, and Instagram link all come from `site-data.json`.

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

## Connect `yaayavar.com` to GitHub Pages

The domain is registered with GoDaddy, while the website is hosted directly by GitHub Pages. `yaayavar.com` must not use GoDaddy forwarding because it is the primary domain.

### 1. Configure the GitHub Pages publishing source

1. Open the [GitHub repository](https://github.com/manojkmgit/manojsoulverse-hub).
2. Select **Settings → Pages**.
3. Under **Build and deployment**, choose **Deploy from a branch**.
4. Select branch **main** and directory **/(root)**, then save.
5. Confirm that the site builds successfully at its default GitHub Pages address.

### 2. Verify ownership of the domain on GitHub

Domain verification protects the domain from being used by another GitHub account.

1. Open personal GitHub **Settings** from the profile menu. This is the account settings page, not the repository settings page.
2. Select **Pages → Add a domain**.
3. Enter `yaayavar.com`.
4. GitHub displays a TXT record. Add it in GoDaddy DNS.
5. Wait for DNS propagation, return to the same GitHub page, and select **Verify**.
6. Keep the verification TXT record permanently.

The current verification record is:

| Type | Name | Value |
| --- | --- | --- |
| TXT | `_github-pages-challenge-manojkmgit` | `4bed6df5439ff8dbc294a0eaa14ce8` |

If GitHub generates a different value during a future verification, use the value displayed by GitHub. See [GitHub's domain-verification documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/verifying-your-custom-domain-for-github-pages).

### 3. Add the custom domain to the repository

1. Open repository **Settings → Pages**.
2. Under **Custom domain**, enter `yaayavar.com` and select **Save**.
3. GitHub creates or updates the root `CNAME` file with `yaayavar.com` when publishing from a branch.
4. If GitHub created that commit remotely, run `git pull --rebase origin main` before making the next local commit.

GitHub recommends adding the custom domain in repository settings before pointing DNS to GitHub. See [GitHub's custom-domain documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site).

### 4. Configure GoDaddy DNS

Open GoDaddy **Domain Portfolio → yaayavar.com → DNS** and configure these records:

| Type | Name | Value |
| --- | --- | --- |
| A | `@` | `185.199.108.153` |
| A | `@` | `185.199.109.153` |
| A | `@` | `185.199.110.153` |
| A | `@` | `185.199.111.153` |
| CNAME | `www` | `manojkmgit.github.io.` |

The trailing dot in `manojkmgit.github.io.` is valid. The `www` CNAME must point directly to `manojkmgit.github.io`, without the repository name and without pointing to `yaayavar.com`.

Remove any conflicting root-domain records or services, especially:

- `A @ WebsiteBuilder Site`
- GoDaddy domain forwarding on `yaayavar.com`
- GoDaddy Airo, Coming Soon, parking, or Websites + Marketing connections
- Other `A` records at `@`
- Wildcard DNS records such as `*`

The final public A-record lookup must return only the four `185.199.*.153` GitHub addresses. Do not delete the GoDaddy `ns07.domaincontrol.com` and `ns08.domaincontrol.com` nameserver records. Preserve the GitHub verification TXT record and any required email MX, SPF, DMARC, DKIM, and bounce records.

### 5. Validate DNS and enable HTTPS

On Windows, verify the public records with:

```powershell
Resolve-DnsName yaayavar.com -Type A
Resolve-DnsName www.yaayavar.com -Type CNAME
```

Expected results:

- `yaayavar.com` returns only the four GitHub Pages IPv4 addresses.
- `www.yaayavar.com` is a CNAME for `manojkmgit.github.io`.
- GitHub repository **Settings → Pages** reports **DNS check successful**.

GitHub then provisions the TLS certificate. The **Enforce HTTPS** checkbox can remain unavailable while certificate provisioning is in progress; GitHub notes that it can take up to 24 hours. Once available, enable it and test both:

```text
https://yaayavar.com/
https://www.yaayavar.com/
```

GitHub automatically redirects the configured `www` alternate to the primary apex domain. See [GitHub's HTTPS documentation](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https).

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
