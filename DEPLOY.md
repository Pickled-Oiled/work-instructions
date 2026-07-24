# Deploying to GitHub Pages

Browser upload, public repo. No git install required. About five minutes end to end.

## 1. Create the repository

1. Sign in at [github.com](https://github.com) and go to **[github.com/new](https://github.com/new)**.
2. **Repository name**: something URL-safe — `work-instructions` is fine. This becomes part of
   your app URL, so keep it short.
3. Set visibility to **Public**. (GitHub Pages on a private repo needs a paid plan.)
4. Leave **Add a README file**, **.gitignore** and **license** all unchecked. The folder
   already contains a README, and starting empty avoids a merge conflict on the first upload.
5. Click **Create repository**.

You land on an empty repo page with setup instructions.

## 2. Upload the files

1. On that page, click the **uploading an existing file** link. (If you've navigated away:
   **Add file → Upload files**.)
2. Open the app folder in File Explorer.
3. Select everything — `index.html`, `manifest.json`, `app.webmanifest`, `icon-96.png`,
   `icon-192.png`, `README.md`, `DEPLOY.md`, **and the `docs` folder itself** — and drag the
   whole selection onto the upload area. Dragging the folder preserves the `docs/` structure,
   which the manifest paths depend on.
4. Confirm the file list shows `docs/wi-1042.pdf` and friends with the `docs/` prefix.
5. Scroll down, type a commit message like `Initial upload`, and click **Commit changes**.

**Do not** upload the app folder as a single parent folder (e.g. `outputs/`) — `index.html`
must sit at the repository root, not one level down.

## 3. Turn on GitHub Pages

1. In the repo, go to **Settings** (top nav) → **Pages** (left sidebar).
2. Under **Build and deployment → Source**, choose **Deploy from a branch**.
3. Set **Branch** to `main` and the folder to **`/ (root)`** — *not* `/docs`. That dropdown
   option refers to a source directory and has nothing to do with this app's `docs/` PDF
   folder; picking it will break the site.
4. Click **Save**.

Wait about a minute. Refresh the Pages settings screen and it will show:

```
Your site is live at https://<your-username>.github.io/work-instructions/
```

The **Actions** tab shows the deploy running if you want to watch it.

## 4. Test it

1. Open that URL in a desktop browser first. You should see the library screen listing four
   documents. Drive it with **arrow keys** and **Enter** — the same events the Neural Band
   sends.
2. Open a document and step through it. If a document fails to open, the error card names the
   cause (missing file, no text layer, CORS).
3. On the glasses: enable developer mode for Web Apps in the Meta AI companion app on the
   paired phone, then open the same URL. GitHub Pages serves HTTPS by default, which the
   glasses require.

## 5. Adding your own work instructions later

Two small edits, both doable in the browser:

- **Add PDFs**: open the `docs` folder in the repo → **Add file → Upload files** → drag your
  PDFs in → commit.
- **Update the library**: click `manifest.json` → the pencil icon → add an entry for each
  PDF → **Commit changes**.

```json
{
  "id": "wi-4001",
  "title": "Spindle Bearing Replacement",
  "station": "Maintenance",
  "rev": "Rev A",
  "url": "docs/wi-4001.pdf"
}
```

Pages redeploys automatically, usually within a minute. A hard refresh on the glasses picks
up the change.

### Gotchas when adding files

- **Filenames are case-sensitive on the server.** `docs/WI-4001.pdf` in the repo will not
  match `docs/wi-4001.pdf` in the manifest, even though Windows treats them as the same.
  Keep everything lowercase.
- **Avoid spaces in PDF filenames.** Use hyphens; it saves URL-encoding problems.
- **Keep `id` values stable.** Reading progress is stored against the `id`, so renaming one
  resets that document's progress for every operator.
- **Scanned PDFs need OCR first.** No text layer means nothing to extract.

## A note on making this public

Everything in a public repo is readable by anyone who finds the URL, and GitHub Pages sites
are indexable by search engines. The sample PDFs are fictional, so the repo is harmless as
shipped — but before you add real controlled documents, consider whether they should be on
the open internet.

If they shouldn't be, the app is built to handle it: keep this repo public with the app code
only, delete the sample `docs/` folder, and point each `url` in `manifest.json` at your
internal document server instead. That server needs to send an
`Access-Control-Allow-Origin` header permitting your Pages origin, and it must be reachable
from the shop-floor network the glasses are on. To discourage indexing of the app itself, add
a `robots.txt` containing:

```
User-agent: *
Disallow: /
```
