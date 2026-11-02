# Daily Ditch: Live Audio-Visual Analyzer

A single-page web app at **https://dailyditch.com**. The phone camera shows the colors it sees on a color wheel and plays them as sound. Each color's light frequency is dropped about 40 octaves into hearing range (green light at 563 THz becomes about 513 Hz).

> **Status: paused (2026-11-01).** The site is live and working. See [Open items](#open-items) for what to pick up next.

---

## At a glance

| | |
|---|---|
| Live site | https://dailyditch.com (`www` redirects here) |
| Repo | `mediavanity-dev/dailyditch` (public) |
| Hosting | GitHub Pages, served from `main` at `/` (root). No build step or Actions workflow |
| Domain registrar / DNS | GoDaddy |
| HTTPS | GitHub-issued certificate, **Enforce HTTPS on**. The camera only works over HTTPS |
| Dependencies | None. Plain HTML, CSS and JS in one file |

## Repo layout

| File | Purpose |
|---|---|
| `index.html` | The entire app: markup, styles and script |
| `CNAME` | Contains `dailyditch.com`. Tells Pages which domain to serve. **Don't delete it.** |
| `.nojekyll` | Makes Pages serve files as-is, with no Jekyll processing |
| `README.md` | This file |

## Updating and deploying

1. Edit `index.html`.
2. Commit and push to `main`. Pages redeploys in about a minute.
3. Check that the build finished: `gh api repos/mediavanity-dev/dailyditch/pages/builds/latest --jq .status` should return `built`.

Commits use the GitHub noreply address (`350796533+mediavanity-dev@users.noreply.github.com`), set in this repo's local git config, so a personal email never appears in history.

### Running locally
The camera needs HTTPS or `localhost`, so serve the folder instead of opening the file:
```
npx serve .
```
To test on a phone against a local copy, you need an HTTPS tunnel. Otherwise, push and test on the live site.

---

## Hosting and DNS configuration

### GitHub Pages (repo → Settings → Pages)
- Source: **Deploy from a branch**, `main` / `(root)`
- Custom domain: `dailyditch.com`
- Enforce HTTPS: **on**

### GoDaddy DNS records for dailyditch.com
**Website records (point at GitHub Pages):**

| Type | Name | Value |
|---|---|---|
| A | @ | 185.199.108.153 |
| A | @ | 185.199.109.153 |
| A | @ | 185.199.110.153 |
| A | @ | 185.199.111.153 |
| CNAME | www | mediavanity-dev.github.io |
| TXT | `_github-pages-challenge-mediavanity-dev` | code from GitHub's domain verification (keep it) |

For the TXT record, enter only the part of the hostname before `.dailyditch.com` in GoDaddy's Name field, because GoDaddy adds the domain itself.

**Email records (Microsoft 365 / GoDaddy). Don't touch these:**

| Type | Name | Value |
|---|---|---|
| MX | @ | dailyditch-com.mail.protection.outlook.com |
| CNAME | autodiscover | autodiscover.outlook.com |
| CNAME | email | email.secureserver.net |
| CNAME | calendar | calendar.secureserver.net |
| CNAME | fax | fax.secureserver.net |

Leave any other TXT records (SPF and similar) alone as well.

The site used to be on GoDaddy web hosting at 50.63.7.223. That IP should no longer appear anywhere in DNS.

---

## How it works

```
camera frame ──► analyze visible area ──► 36-bin hue wheel ──► pick tones ──► Web Audio
                 (white balance, HWB split)  (+ white/black mix)   (peaks or all)   (limiter → speakers)
```

1. **Capture.** The camera is analyzed at 96 px wide, about 20 times a second. Only the part shown in the (cropped) preview is analyzed.
2. **Lighting adjustment.** The light's tint is estimated from lit, near-grey pixels and cancelled with per-channel gains. This is partial, clamped and smoothed, and it's skipped when less than 10% of the view is near-grey.
3. **Color / white / black split (HWB).** For each pixel:
   - chroma (`max − min`) is the colored light
   - `min` is white light
   - `255 − max` is the missing light, i.e. black

   Only chroma above a noise floor is placed on the wheel.
4. **Wheel.** 36 hue wedges of 10° each, with red at the top and hues running clockwise. Wedge area follows each hue's share of the color in view, and each wedge is drawn in the real average camera color for that hue.
   - Hues 270–360° are **purples**, which have no single wavelength; they're shaded on the wheel.
   - The center pie shows the scene's color / white / black mix.
5. **Hue → wavelength → pitch.**
   - Hue maps to an approximate spectral wavelength through the `HUE_NM` table (425–660 nm).
   - The light's frequency (c / λ) is divided by 2^(octaves). With Spread above 1×, the pitch intervals are also widened around the center of the range.
   - Purples play as a **violet + red pair**, shifting from mostly violet next to blue to mostly red next to red.
6. **Sound.**
   - **Tones:** Main colors (up to 4 wheel peaks, played on voices that glide between pitches) or Full spectrum (every hue present).
   - **Loudness:** follows each color's share and how vivid the scene is.
   - **White:** plays as a soft red + green + blue chord, or as band-limited noise. Black is silent.
   - **Output:** everything passes through master volume and then a limiter.
7. **Interaction.**
   - Tap a wedge to dim everything else on the video and show the conversion chain, e.g. `468 nm → 641 THz → 583 Hz (D5) → 0.59 m sound wave`.
   - Tap the center, or the same wedge again, to clear.

---

## User settings (Settings sheet)

| Setting | Default | Options / range | What it does |
|---|---|---|---|
| Volume | 70% | 0–100% | Master volume, adjustable while sound is playing |
| Octaves below light | 40 | 37–43 | How far light's frequency is dropped. At 40, red to violet plays at 410–649 Hz (G#4–E5) |
| Spread | 1× | 1×, 2×, 4× | 1× keeps the true frequency ratios. 2× and 4× widen the intervals for clarity, which isn't physically accurate |
| Tone | Pure | Pure (sine), Bright | Bright adds overtones only at 2× and 4× (exact octaves) for phone speakers |
| What plays | Main colors | Main colors (up to 4), Full spectrum | Peaks only, or every hue present |
| White and grey play as | Soft chord | Soft chord, Noise, Silence | How white light sounds |
| Adjust for room lighting | On | on / off | Cancels the light's tint (color constancy) |
| Let vivid colors dominate | On | on / off | Weights hue by chroma², so dull beige, wood and skin don't swamp vivid colors |
| Snap tones to chromatic notes | Off | on / off | Merges tones onto the nearest musical note |
| Show note names around the wheel | Off | on / off | Rim labels show the notes played instead of wavelengths |

Settings aren't saved between visits; every visit starts from these defaults.

## Tuning constants (in `index.html`)

| Constant / location | Value | Effect |
|---|---|---|
| `NUM_BINS` | 36 | Number of wheel wedges (10° each) |
| `ANALYSIS_W` | 96 | Analysis width in pixels. Higher is finer but slower |
| `MIN_CHROMA` | 24 | Noise floor on channel spread (0–255). **Raise it** if dim or noisy areas still show false color; **lower it** if faint real colors disappear |
| `HUE_NM` | 7-point table | Hue → wavelength map (0° = 660 nm … 270° = 425 nm) |
| `MIN_NM` / `MAX_NM` | 420 / 665 | Sets the center for Spread and the noise band |
| `SPECTRAL_END` | 270 | Hue where purples begin |
| `updateWhiteBalance()` | strength 0.85, gains clamped 0.6–1.7, needs ≥10% near-grey, smoothing 0.1 | Lighting adjustment behavior. A pixel counts as near-grey when its channel spread is < 40% of its brightness and its brightness is > 50 |
| Frame rate (in `mainLoop`) | every 50 ms | Analysis rate |
| Tone gain | `0.45 × colorLevel × √share` | Loudness of the color tones |
| White chord gain | `0.2 × √(white / 3)` | Loudness of the white chord |
| Limiter | threshold −10 dB, ratio 12, attack 3 ms | Prevents clipping |
| `brightWave` | harmonics 1, 0.45 at 2×, 0.2 at 4× | Timbre of the Bright tone |

## Testing

There's no automated test suite in the repo. Changes were checked in headless Edge (Playwright) with fake cameras, using `--use-fake-device-for-media-stream` and `--use-file-for-fake-video-capture=<file>.y4m`. The test videos were synthetic: a color wheel, a warm-lit room, a full-frame red scene, and grey and dark-noise scenes. The checks covered three things:
- **Wheel shares by hue sector.** A test copy of the page exposed the smoothed histogram as `window.__smooth`.
- **Frequencies and levels of every oscillator.** Recorded by wrapping `AudioContext.prototype.createOscillator`.
- **The actual audio spectrum.** Measured with an `AnalyserNode` tapped after the limiter.

Reference results from the last session (warm-lit room, red/orange share): both new settings off: 77%. Lighting adjustment only: 37%. Vivid weighting only: 71%. Both on: 18%.

---

## Open items

- [ ] **Test on a real phone.** All tuning so far used synthetic scenes. Check:
  - whether orange still dominates real indoor scenes (tap the biggest wedge to see which surfaces cause it)
  - whether `MIN_CHROMA` needs adjusting
  - whether Pure is loud enough on a phone speaker, or Bright should be the default
  - the white chord level
  - whether the rim labels are clipped at narrow widths
- [ ] **Confirm GitHub domain verification is complete:** GitHub profile → Settings → Pages → `dailyditch.com` shows as verified.
- [ ] **Cancel the old GoDaddy web hosting plan** if it hasn't been already. Keep the domain registration and the email services.

### Ideas not built yet
- Save settings between visits (`localStorage`).
- A short "how this works" note on the page (the camera measures hue, not a true spectrum).
- A favicon, meta description and social-preview tags.
- Use a perceptual color space (OKLCH) for hue and chroma instead of HSV/HWB.

### History of major changes
1. Moved from GoDaddy hosting to GitHub Pages with the custom domain and enforced HTTPS.
2. Analysis covers only the visible preview area, the axis was fitted to the hue range, bars got real colors, and tap-to-highlight was added.
3. Sound engine rebuilt around light's real frequency (octaves below light, Spread, Tone, What plays, white as sound, a limiter, gliding voices).
4. Mic mode removed. Color / white / black (HWB) split added, which fixed false orange from dark noise.
5. Both bar charts replaced with the color wheel, Chart.js removed, purples play as violet + red, and Bright changed to octave-only overtones (removing a stray ~1760 Hz tone).
6. Lighting adjustment and vivid weighting added, to stop warm lighting and dull warm materials from dominating.
