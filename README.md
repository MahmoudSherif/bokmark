# Bokmark

Sort your saved digital stuff — browser bookmarks, Messenger conversations, Facebook saved items, and X likes/bookmarks — into filterable tables, entirely in your browser.

One HTML file. No server, no accounts, no analytics, no build step.

## How it works

1. Export your data from each platform (instructions are inside the app, under "Where do I get these files?").
2. Open Bokmark and drop the export files onto the page.
3. Everything is parsed locally, normalized into one schema, and grouped into tables by type, source, person, thread, domain, month, or script. Filter, search, and sort freely.
4. Export the result as CSV, XLSX (one sheet per table), or JSON.

Supported inputs:

| Platform | File | Notes |
|---|---|---|
| Any browser | Exported bookmarks `.html` | Chrome, Firefox, Edge, Safari (Netscape format) |
| Chrome | Raw `Bookmarks` profile file | JSON, detected automatically |
| Facebook / Messenger | `message_1.json`, `your_saved_items.json` | From "Download your information" in **JSON** format. Non-English text is automatically repaired (Facebook's export garbles it) |
| X (Twitter) | `like.js`, `tweets.js` from your archive, or a `/2/users/:id/bookmarks` API response | Bookmarks require X's paid API; the archive route is free |

## AI folders

Click **AI sort → folders** and Bokmark groups whatever is currently on screen into named topic folders. Two ways, no account either way:

- **Smart sort** (default) — free for your visitors, powered by *your* Google Gemini or Anthropic key, held on a small Cloudflare Worker. Best folder names ("Web Development", "Recipes", "Job Hunt", in the items' own language). Only each item's title, domain and type are sent; URLs, message bodies, and the names of people and chats are not. Up to 1,500 items per run.
- **Private sort** — no server at all: a ~25 MB model downloads once and runs in the browser, so nothing leaves the device. No daily limit. Folder names come from each topic's key words. Also the automatic fallback when the daily budget runs out.

Power users can open "Use my own Anthropic key instead" to bypass the shared service entirely — that path talks to Anthropic directly from the browser and is independent of which provider your Worker uses.

Folders are renameable (✎ on each tab), removable, saved with your data, and included in every export. Group by "AI folder" for one table per topic.

## Deploy the sorting service

This is what makes Smart sort free for visitors. **Your API key must never go in `index.html`** — anyone could read it from the page source and spend your money. It lives on the Worker instead.

You need a free Cloudflare account and a model API key. The Worker speaks both
**Google Gemini** and **Anthropic Claude** — set one key and the provider is picked
automatically.

```bash
npm install -g wrangler
wrangler login

# 1. storage for the daily quota counters
wrangler kv namespace create RL          # paste the printed id into wrangler.toml

# 2. edit wrangler.toml: set ALLOWED_ORIGINS to your Pages URL

# 3. secrets, stored encrypted — never in the repo
wrangler secret put GOOGLE_API_KEY       # from aistudio.google.com
wrangler secret put IP_SALT              # any long random string
#   for Claude instead: wrangler secret put ANTHROPIC_API_KEY  (and set PROVIDER = "anthropic")

# 4. ship it
wrangler deploy
```

Wrangler prints a URL like `https://bokmark-sort.yourname.workers.dev`. Put it in the `BOKMARK_API` constant near the top of the script in `index.html`, commit, and Smart sort turns on. Visit `/health` on the Worker to confirm the provider, key, quota storage, and origins are all wired up.

### Notes on Google Gemini

- Get the key at **aistudio.google.com** → Get API key. No credit card needed to start.
- Google's **free tier** covers the Flash and Flash-Lite models with per-minute and daily request caps, which is why `DAILY_GLOBAL_LIMIT` defaults to 800 — keep it under Google's daily cap so your own quota trips first with a friendly message instead of a hard API error.
- **Free-tier prompts may be used by Google to improve its products; paid-tier prompts are not.** If other people's bookmark titles are flowing through your service, enable billing before you promote it, and say which tier you're on in your privacy note.
- Model IDs move fast. Leave `MODEL` unset to use the Worker's default, or check the current list in AI Studio. A **Flash-Lite** model is more than enough for filing bookmarks and costs a fraction of Flash.
- Gemini 3.x models think by default and reasoning tokens share the output budget. If batches come back truncated, set `THINKING_BUDGET = "0"` in `wrangler.toml`.

If you point a custom domain at the Worker, also add that domain to `connect-src` in the CSP meta tag at the top of `index.html`.

### Protecting your account

The Worker is built so a free public endpoint can't become an expensive one:

- The key stays server-side; the browser never sees it.
- The browser can only send **structured items**, never a prompt. The Worker writes the prompt itself, so nobody can turn your endpoint into free general-purpose Claude access.
- Per-visitor and global **daily quotas** (`DAILY_IP_LIMIT`, `DAILY_GLOBAL_LIMIT` in `wrangler.toml`). If quota storage is ever missing, the Worker refuses to run rather than leaving the key uncapped.
- Requests are accepted only from origins you list, item text is stripped of control characters and truncated, oversized bodies are rejected, and upstream error text is never passed through to visitors.
- Visitor IPs are salted and hashed for counting, never stored in the clear. Keep `IP_SALT` a secret rather than a committed variable — a public salt would let someone brute-force which IPs used your service.

**Also set a spend limit in your provider's console** (Google Cloud budget alerts, or the Anthropic console limit). That is your real backstop, and the one control that cannot be bypassed by a bug in this code. Start the limits low — a 50-item batch on a Flash-Lite or Haiku model costs a fraction of a cent, so the defaults (20 runs per visitor, 800 per day) are already generous — and raise them once you see real traffic.

If abuse ever does appear, add Cloudflare Turnstile in front of `/sort`, or move quota counting to a Durable Object for strict rather than eventual consistency.

## A note on becoming a data processor

The moment visitors' data passes through your Worker, you are handling other people's data, not just your own. The Worker stores none of it — items are forwarded to Anthropic and the response returned, with only hashed-IP counters kept — and that is worth keeping true. If you promote this widely, especially in the EU or UK, publish a short privacy page saying what is sent, what is not, that nothing is retained, and who the subprocessor is (Google or Anthropic, depending on your setup). Keeping Private sort prominent gives privacy-sensitive users a real alternative and is a genuine selling point.

## Deploy to GitHub Pages

1. Create a new repository and add `index.html` (and this README) to the root of the `main` branch.
2. Repository **Settings → Pages → Source: Deploy from a branch → main / (root) → Save**.
3. Your copy is live at `https://<username>.github.io/<repo>/` within a minute or two.

It also works from any static host (Netlify, Cloudflare Pages) or simply double-clicked as a local file.

## Deliberately not included (yet)

- **Dead-link checking** — would require per-URL network requests, which browsers block cross-origin anyway and which would leak your reading list to every saved site. Possible later as an explicit opt-in.
- **Live X sync (OAuth)** — feasible from a static page with PKCE, but requires registering an app with X and depends on their paid API tiers. Uploading fetched JSON works today.
- **ZIP ingestion** — for now, unzip the Facebook/X archives and drop the inner files (you can drop many at once).

## Files

| File | What it is |
|---|---|
| `index.html` | The whole app. Deploy to GitHub Pages. |
| `worker.js` | The sorting service. Deploy to Cloudflare Workers. Optional — without it, Private sort still works. |
| `wrangler.toml` | Worker config: provider, origins, quotas, model. Secrets are *not* here. |
| `.gitignore` | Keeps `.dev.vars` and Wrangler's local state out of the repo. |

## Roadmap ideas

Saved views, per-table export buttons, ZIP support, folder merging via drag-and-drop, Turnstile on the sorting endpoint, a Tauri desktop wrapper.

## License

Suggested: MIT. Add a `LICENSE` file before publishing if you want others to reuse it.
