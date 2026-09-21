# Folio — resume studio

[folio.maxpaul.us](folio.maxpaul.us)

A single-file resume builder: structured editing for contact, summary, experience,
education, projects, skills, certifications and involvement; live letter-size preview
with page-break guides; text-based PDF and Word (.docx) export; local autosave and
saved versions.

Everything is in `index.html` — no build step. Data stays in the browser (localStorage);
use *Download → Back up data* to move it between machines.

## AI features

The AI tools (tailor to a job, improve bullets, import) run through Claude's artifact
runtime and are only available when the app is opened inside Claude. When hosted here
they are hidden automatically; the rest of the app works normally.

## Hosting (GitHub Pages)

`CNAME` points this site at `folio.maxpaul.us`. Add a DNS `CNAME` record for `folio`
targeting `<username>.github.io`, then enable *Enforce HTTPS* in Settings → Pages.

## Fonts

The PDF exporter loads Latin subsets of Spectral and Carlito (SIL Open Font License) from
`fonts/` on first export. Keep that folder next to `index.html`.
