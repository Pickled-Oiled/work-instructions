# Work Instructions — Meta Ray-Ban Display web app

A URL-launched web app that turns PDF work instructions into large-type, one-step-at-a-time
cards on the glasses' 600×600 heads-up display. No app store, no native build — host the
folder, open the URL on the glasses.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The entire app (UI, PDF parsing, navigation, progress). Single file. |
| `manifest.json` | **Your library.** Edit this to list your documents. |
| `app.webmanifest`, `icon-96.png`, `icon-192.png` | App name and launcher icon. |
| `docs/*.pdf` | Sample work instructions, for testing. Replace with yours. |

## Adding your documents

The app opens on a list of work stations; selecting one lists that station's instructions.
Drop PDFs in `docs/` (or point at any HTTPS URL) and edit `manifest.json`:

```json
{
  "stations": [
    {
      "name": "Forklift",
      "documents": [
        {
          "id": "wi-1042",
          "title": "Daily Pre-Shift Inspection",
          "subtitle": "Class II trucks",
          "rev": "Rev C",
          "url": "docs/wi-1042.pdf"
        }
      ]
    },
    {
      "name": "Packaging / Shipping",
      "documents": []
    }
  ]
}
```

- Stations appear in the order you list them — put the busiest first.
- `id` — must be stable and unique across the whole file; it's the key used to remember
  reading progress. Renaming one resets that document's progress for every operator.
- `subtitle` / `rev` — optional, shown as the grey second line.
- `url` — relative path or absolute HTTPS URL. If the PDF is on another domain, that
  server must send `Access-Control-Allow-Origin`, otherwise the browser will block it.
- A station with no documents is hidden rather than shown as a dead end.

Deep links: `index.html?station=Forklift` opens that station, `index.html?doc=wi-1042` jumps
straight into one document.

The older flat format still loads — a top-level `documents` array is grouped automatically by
each entry's `station` field — so an existing manifest keeps working without edits.

## Hosting and loading on the glasses

1. Put the folder on any static HTTPS host (Vercel, Netlify, Cloudflare Pages, GitHub Pages,
   S3, or an internal web server). **HTTPS is required.**
2. In the Meta AI companion app on the paired phone, turn on developer mode for Web Apps.
3. Open the app's URL from the glasses.

Local testing on a laptop — the app is fully usable with a keyboard:

```bash
cd <this folder>
python3 -m http.server 8000
# then open http://localhost:8000 and drive it with arrow keys + Enter
```

Opening `index.html` directly from the filesystem will **not** work: `fetch` and pdf.js both
need a real HTTP origin.

## Controls

The Neural Band and the temple touch strip are delivered to the page as arrow keys and Enter.

| Where | Input | Action |
| --- | --- | --- |
| Stations | ↑ ↓ | Move between work stations |
| Stations | Enter | Open that station's instructions |
| Documents | ↑ ↓ | Move between documents |
| Documents | Enter | Open the document |
| Documents | ↑ from the top | Reach "← All work stations" |
| Reader | → or ↓ | Next step |
| Reader | ← or ↑ | Previous step |
| Reader | Enter | Open the menu |
| Menu | ↑ ↓ / Enter | Select / confirm |

Menu items: Continue, Jump to step, Text size (Small / Medium / Large), Restart document,
Back to *station*, All work stations.

Lists wrap, so the Back item at the top of a document list is always one Up press away from
the first document — there's no dedicated back gesture on the glasses to rely on.

## How PDFs become steps

1. pdf.js pulls the text layer with per-glyph coordinates; glyph runs are regrouped into
   lines by baseline, with spacing inferred from the horizontal gaps.
2. Running headers, footers and bare page numbers are removed — by repetition across pages,
   and by small type sitting in the top or bottom margin band.
3. Steps are detected from numbering (`1.`, `2)`, `Step 3`), requiring the numbers to run in
   sequence so that measurements like "12.5 Nm" or "2.0 Nm" aren't mistaken for step markers.
   Lines set in noticeably larger type become section labels (shown next to the step number).
4. If a document has no numbering, it falls back to paragraph blocks split on vertical gaps.
5. Each step is reflowed into paragraphs, then measured against the real card box and split
   across continuation cards (`3. 1/2`, `3. 2/2`) if it overflows. Re-measured when you
   change text size, and the reader stays on the same step.
6. Lines starting with WARNING / CAUTION / DANGER / NOTE are rendered in amber.

Progress is stored in `localStorage` per document id. Reopening a document offers
**Resume** at the last step or **Start from the beginning**, and the library shows a
percentage badge, or DONE.

## Design constraints this app is built around

The display is an **additive waveguide** — black pixels emit nothing and are effectively
transparent, so the UI is black-backed with bright text. Per Meta's guidance the app uses a
fixed 600×600 viewport with `overflow: hidden` and no scrolling in the reader, body text at
26–36 px, and 88 px minimum focus targets. There is no mouse, no touch, no text input and no
camera, so every action is reachable with four directions and a select.

## Known limits

- **Scanned PDFs won't work.** There must be a real text layer. Run OCR first; the app says
  so explicitly rather than showing a blank card.
- **Diagrams and photos are dropped** — this is a text-extraction reader by design. Keep the
  printed drawing at the station, or split figures into a separate reference.
- **Multi-column layouts** are read in reading order only when the PDF's text order is sane;
  heavily columned documents may interleave.
- **No offline support** on the glasses, so the host must be reachable on the shop-floor
  network.

## Verification performed

Step extraction is exercised headlessly against five work-instruction PDFs, including a
real Word-authored one whose list numbering reaches the text through a tab stop rather than
a space. The parser asserts that every extracted line is accounted for — no line may be
silently dropped — which is what caught steps 10 onward disappearing after a section heading.

Earlier coverage: four generated work-instruction PDFs
(single-page and three-page, numbered and unnumbered, with running headers and page footers),
asserting: headers and footers stripped, correct step counts, section labels carried across
page breaks, correct page references, no glued-together words from missed spaces, and
decimal values not misread as step numbers. All checks passed.
