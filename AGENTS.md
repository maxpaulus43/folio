# Folio — resume studio

Single-file resume builder (`index.html`, ~730 KB, no build step, no framework).
Owner: Max. Hosted at https://folio.maxpaul.us via GitHub Pages (`CNAME` in repo).
Also published as a Claude artifact at https://claude.ai/artifact/WZAwHu8NMxHrELTcS1BSAG —
that copy is where the AI features work (see "Two runtimes").

## What it does

- Sections: contact, summary, experience, education, projects, skills, certifications, involvement.
  Entries can be reordered (↑↓), collapsed, and hidden individually (`entry.off`).
  Sections can be reordered and hidden (`settings.order`, `settings.hidden`) via the desktop
  rail or the "⇅ Arrange sections" modal. Bullets reorder with ▲▼; Enter adds, Backspace on empty removes.
- Live US-letter preview (816×1056 px, scaled to fit) with approximate page-break guides.
  Two templates: `classic` (Spectral serif, centered header) and `modern` (Carlito sans, accent bar).
  Sizes 9 / 10.5 / 11.5 pt; `spacing()` maps size → vertical-spacing multiplier `--sp`.
- Export: PDF via pdfmake (real text, ATS-parseable), DOCX via docx.js, print, JSON backup/restore.
- Local health score (0–100) from heuristic checks in `healthChecks()` — action verbs, numbers, dates, length, etc.
- Versions: named snapshots in localStorage (`Versions` object); auto-snapshot before tailoring.
- AI tools (Claude runtime only): improve/write summary, improve bullets per entry, find weak
  bullets, tailor to a job posting (fit score, requirements, keywords, headline/summary/skills/
  order/emphasis/bullet-rewrite proposals with checkboxes), import from pasted resume text.
  Model tier selectable in the AI panel (`settings.aiTier`: quick | default | complex).

## File layout (all in index.html, in order)

1. `<head>`: Google Fonts link (IBM Plex Sans UI, Spectral, Carlito), pdfmake + docx from jsDelivr, CSS.
2. Markup: topbar, `.work` grid (rail / editor / preview), mobile tabs, modal, toast.
3. `<script>` FONT_VFS — base64 TTF subsets (Spectral + Carlito, 4 styles each, Latin only, OFL licensed).
4. `<script>` part 1: state (`S`), SAMPLE data, storage, rail, editor, preview, health score, versions, arrange/ask modals.
5. `<script>` part 2: `AI` object, exports (`pdfDef`, `exportPDF`, `exportDOCX`), download menu, mobile tabs, init.

## Data model

```
S = { contact:{name,title,email,phone,location,website,linkedin,github}, summary:'',
      experience:[{id,role,company,location,start,end,current,bullets:[],off}], education:[…],
      projects:[…], skills:[{id,category,items}], certifications:[…], involvement:[…],
      settings:{template,size,accent,order:[…7 keys],hidden:{},aiTier}, lastJD }
```
localStorage keys: `folio-resume-v1` (S), `folio-versions-v1`, `folio-theme`.
`vis(sec)` = entries with `!off`; use it (not `S[sec]`) anywhere that renders output.
Dates are free text; `dates(e)` formats "start – end/Present".

## Two runtimes — important

- **Claude artifact viewer:** `window.claude.use('sample')` gives AI calls (streaming `onText`,
  `.json()` for structured output, `modelTier`), `claude.use('downloads')` saves files.
  `window.confirm/alert` are blocked there → use `ask()` (in-app modal), never `confirm()`.
- **Plain web (GitHub Pages):** no `window.claude`; AI button and per-entry AI buttons are hidden
  (`AI.available=false`), downloads fall back to an `<a download>` blob link.
- Published-page CSP allows scripts only from cdnjs / cdn.jsdelivr.net / tailwind / jquery and
  stylesheets from fonts.googleapis.com. Keep it self-contained; no other external requests.
- Never add `[hidden]`-attribute elements without remembering `[hidden]{display:none!important}` exists
  (buttons have explicit `display`, which would otherwise override the attribute).

## Gotchas already solved (don't regress)

- **PDF line spacing:** pdfmake multiplies `lineHeight` by the font's *built-in* line height
  (Spectral 1.522 em, Carlito 1.2207 em). `defaultStyle.lineHeight` divides by those so the PDF
  matches the preview's CSS `line-height:1.22`. Compact uses 1.14.
- Preview margins (59 px) ≈ PDF margins (44 pt); content widths match (~699 px / 524 pt).
- Mobile font boosting inflated preview text → `.paper * { text-size-adjust:none }`.
- Page count: reset `paper.style.minHeight='0'` before measuring `scrollHeight`, else it sticks.
- Section pill bar re-renders on every edit; scrollLeft is saved/restored in `renderEditor()`.
- Pasted skills sometimes arrive URL-encoded (`%20`) → decoded on input when the string has no spaces.
- Entry titles in the editor must go through `esc()`.
- pdfmake fonts are passed explicitly: `createPdf(def, null, fonts, FONT_VFS)`.

## Conventions

- Vanilla JS, template strings, event delegation on `#editor`. `commit(rerender)` = save + re-render preview/rail/score.
- Design tokens on `:root`; dark mode via `prefers-color-scheme` guarded by `:root:not([data-theme="light"])` and `:root[data-theme="dark"]`.
- Accent teal `#0F6C64`; AI actions use the violet `--ai` token and a `✦` glyph.
- AI prompts live in `AI.RULES` + per-tool prompts; every AI result is reviewed with checkboxes before applying. Never let AI invent facts.
- To regenerate `FONT_VFS`: download static TTFs from github.com/google/fonts (ofl/spectral, ofl/carlito),
  `pyftsubset --unicodes="U+0020-007E,U+00A0-00FF,U+2010-2027,U+2030-203A,U+20AC,U+2122" --layout-features="kern,liga"`, base64 into the object.

## Testing

- Syntax: extract `<script>` blocks and `node --check`.
- Rendering/behaviour: Playwright against a copy with external `<script src>` stripped and Google Fonts link disabled.
- PDF: pass `pdfDef()` to pdfmake's Node `PdfPrinter` with Buffer fonts from FONT_VFS; check page count with pdfplumber and rasterize with `pdftoppm` to eyeball.
- DOCX: the page's docx instance hangs in jsdom; serialize with Node's `docx` Packer instead, then render with `soffice --headless --convert-to pdf`.

## Backlog / ideas

- Bring-your-own Anthropic API key mode so AI works on folio.maxpaul.us (store key in localStorage; API supports direct browser access with the `anthropic-dangerous-direct-browser-access` header).
- Import from an uploaded PDF (pdf.js from cdnjs → text → existing `AI.importText`).
- Cover-letter generator from tailor results; custom sections; drag-and-drop reordering; more templates.
- Sync data between the Claude-hosted and self-hosted copies (currently manual JSON backup/restore).
