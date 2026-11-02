# Daily Ditch: Live Audio-Visual Analyzer

A single-page web app at **dailyditch.com**. The camera turns the colors it sees into an estimated light spectrum, and that spectrum can be played as sound.

There's no server or build step. The site is plain `index.html`, served by GitHub Pages straight from the `main` branch.

## Files
- `index.html`: the whole page (HTML, CSS, JS). It loads Chart.js from jsDelivr.
- `CNAME`: tells GitHub Pages to serve the site at `dailyditch.com`. Don't delete it.
- `.nojekyll`: serves files as-is, with no Jekyll processing.

## Updating the site
Edit `index.html`, then commit and push to `main`. The change goes live in about a minute.

## Running locally
The camera needs HTTPS or localhost, so serve the folder instead of opening the file:

```
npx serve .
```

## Domain
The domain is registered at GoDaddy, with DNS pointed at GitHub Pages:
- `A` records for `@`: 185.199.108.153, 185.199.109.153, 185.199.110.153, 185.199.111.153
- `CNAME` for `www`: `mediavanity-dev.github.io`
