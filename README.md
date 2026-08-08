# Shoebox

Sort your saved digital stuff — browser bookmarks, Messenger conversations, Facebook saved items, and X likes/bookmarks — into filterable tables, entirely in your browser.

One HTML file. No server, no accounts, no analytics, no build step.

## How it works

1. Export your data from each platform (instructions are inside the app, under "Where do I get these files?").
2. Open Shoebox and drop the export files onto the page.
3. Everything is parsed locally, normalized into one schema, and grouped into tables by type, source, person, thread, domain, month, or script. Filter, search, and sort freely.
4. Export the result as CSV, XLSX (one sheet per table), or JSON.

Supported inputs:

| Platform | File | Notes |
|---|---|---|
| Any browser | Exported bookmarks `.html` | Chrome, Firefox, Edge, Safari (Netscape format) |
| Chrome | Raw `Bookmarks` profile file | JSON, detected automatically |
| Facebook / Messenger | `message_1.json`, `your_saved_items.json` | From "Download your information" in **JSON** format. Non-English text is automatically repaired (Facebook's export garbles it) |
| X (Twitter) | `like.js`, `tweets.js` from your archive, or a `/2/users/:id/bookmarks` API response | Bookmarks require X's paid API; the archive route is free |

## Privacy model

- The page ships a Content-Security-Policy with `connect-src 'none'` — the browser itself refuses to let the page make network requests. Your data *cannot* be uploaded. Verify in DevTools → Network.
- The only external resource is the SheetJS library from cdnjs (for XLSX export). Everything else, including fonts, is local. No favicons are fetched, so the sites you saved never learn you're organizing them.
- Data persists in your browser's IndexedDB on your device only. "Clear all data" wipes it.
- All rendered content is inserted as text, never as HTML, so a malicious bookmark title can't run code in the page.

## Deploy to GitHub Pages

1. Create a new repository and add `index.html` (and this README) to the root of the `main` branch.
2. Repository **Settings → Pages → Source: Deploy from a branch → main / (root) → Save**.
3. Your copy is live at `https://<username>.github.io/<repo>/` within a minute or two.

It also works from any static host (Netlify, Cloudflare Pages) or simply double-clicked as a local file.

## Deliberately not included (yet)

- **Dead-link checking** — would require per-URL network requests, which browsers block cross-origin anyway and which would leak your reading list to every saved site. Possible later as an explicit opt-in.
- **Live X sync (OAuth)** — feasible from a static page with PKCE, but requires registering an app with X and depends on their paid API tiers. Uploading fetched JSON works today.
- **ZIP ingestion** — for now, unzip the Facebook/X archives and drop the inner files (you can drop many at once).

## Roadmap ideas

Saved views, opt-in AI auto-tagging, per-table export buttons, ZIP support, a Tauri desktop wrapper.

## License

Suggested: MIT. Add a `LICENSE` file before publishing if you want others to reuse it.
