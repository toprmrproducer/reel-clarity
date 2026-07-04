# ReelClarity — Agent Notes

Standalone AstroJS single-tool site. Enhances video resolution and sharpness
entirely client-side in the browser (no server, no uploads).

## Why this is single-purpose (deliberate SEO strategy)

This project is a full split-out from a multi-tool site (AnyConvert). It
contains ONE tool only. Each tool in this family gets its own domain and
its own tightly-matched keyword focus ("video upscaler" / "enhance video
resolution") instead of living as one page among a dozen unrelated tools.
One keyword-matched domain per tool is the deliberate SEO strategy for this
whole site family — do not add other tools, a multi-tool nav, or unrelated
utilities to this repo. If a new tool is needed, it gets its own new
standalone project under `~/iCloud/website/<tool-name>/`, following this
same pattern.

## Stack

- Astro 7, static output (`output: 'static'`), zero server runtime.
- Tailwind v4 via `@tailwindcss/vite` (no separate tailwind.config — tokens
  live in `src/styles/global.css` under `@theme`).
- `@astrojs/sitemap` for `sitemap-index.xml`.
- Core video logic: `upscaler` + `@upscalerjs/esrgan-slim` (ESRGAN model,
  loaded lazily per-scale via dynamic `import()`), with a canvas
  resample-plus-unsharp-mask fallback when the AI model is unavailable or
  too slow on-device. Frames are captured via `<canvas>.captureStream()`
  and re-encoded with the browser's native `MediaRecorder` into WebM.
  100% client-side — nothing is ever uploaded.
- No backend, no database, no API routes.

## Design tokens (mandatory — shared family look, do not deviate)

- Background: `#FAF6EC` (warm cream)
- Card/surface: `#FFFFFF` with a warm `#E8DFC8` border (never gray)
- Headings: `#2B2013` (warm espresso brown, not pure black) — font: **Fraunces**
- Body text: `#5C4F3D` (warm brown-gray) — font: **Inter**
- Accent / CTA / links / active state: `#C9982E` (warm gold), hover `#B8860B`
- Both fonts loaded via Google Fonts `<link>` in `src/layouts/Layout.astro`
  head, `font-display: swap`. Headings use `font-weight: 600–700` and
  slightly negative letter-spacing.
- Generous whitespace, soft shadows only, rounded-xl corners, no gradients,
  no dark mode. The cream-only aesthetic is intentional — do not add a
  dark mode toggle or muddy the palette with grays.

## Pages

- `/` — the tool itself (upload, preview, full run, download).
- `/about`, `/privacy`, `/faq` — required for AdSense eligibility across
  this whole site family. Linked from the footer on every page.
- `/404` — custom not-found page.

## Security

Any user-controlled text rendered via `innerHTML` MUST go through the
`escapeHtml()` helper defined inline in `index.astro`'s script block first
(currently used for the uploaded file's name in the preview stats). This is
a standing requirement across the whole tool family — never interpolate
raw user/file-derived strings into `innerHTML`.

## iCloud sync hygiene

`node_modules` is renamed to `node_modules.nosync` with a symlink named
`node_modules` pointing at it, so iCloud does not try to sync the dependency
tree (it chokes on the file count). `tsconfig.json`'s `exclude` includes
BOTH `"node_modules"` and `"node_modules.nosync"` — without the second entry,
`astro check` crashes trying to walk into the real (nosync) directory via
the symlink.

On a fresh checkout on the other Mac: `npm install` regenerates
`node_modules.nosync` from scratch; re-run the rename+symlink step if it
doesn't already exist, or just let a fresh `npm install` recreate the
`node_modules` dir, then manually convert it back to the nosync+symlink
pattern.

## Dev

```bash
npm install
npm run dev   # http://localhost:4343
```

Port `4343` is this project's assigned dev port. Siblings anyconvert/
reelshift/scriptflow use ports in the 4330s, and 4340/4341/4342 were found
claimed by other concurrently-scaffolded sibling tools (vitalcalc,
clarity-lens/wordtally, luckytoss) at build time — 4343 was confirmed free
and picked to avoid collisions.

## Build & deploy

```bash
npm run build          # outputs to dist/, static output, zero errors expected
source ~/.claude/credentials/netlify.env
export NETLIFY_AUTH_TOKEN="$NETLIFY_API_KEY"
netlify deploy --prod --dir=dist
```

Site is linked via `.netlify/state.json` (site name `reel-clarity`, or
whatever unique name Netlify assigned if that name was taken).
