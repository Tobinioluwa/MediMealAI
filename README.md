# Website chat box (Gemini / Google AI Studio) on Cloudflare

A chat box that sits at the bottom-right of your website and answers clients using your AI Studio API key.
The key is stored as an encrypted Cloudflare secret, so nobody can copy it from your page source.

## Files
- `public/chat-widget.js` — the chat box. One script tag puts it on any page.
- `functions/api/chat.js` — the server bit (Cloudflare Pages Function at `/api/chat`). Holds the business info and the "how to talk" rules, and calls Gemini.
- `public/index.html` — a test page.
- `wrangler.toml` — Cloudflare config.

## Step 1 — Get your key and edit the business info
1. aistudio.google.com → **Get API key** → **Create API key**. Copy it.
2. Open `functions/api/chat.js` and edit the `BUSINESS` block at the top (services, hours, contact, how you work).
   The more specific it is, the smarter the bot sounds.

## Step 2 — Deploy (pick one)

### A. Through GitHub (easiest to update later)
1. Push this folder to a GitHub repo.
2. Cloudflare dashboard → **Workers & Pages** → **Create** → **Pages** → **Connect to Git** → choose the repo.
3. Build command: *leave empty*. Build output directory: `public`.
4. Deploy.

### B. From your computer
```bash
npm install
npx wrangler login
npx wrangler pages deploy
```

## Step 3 — Add the key as a secret
Dashboard → your Pages project → **Settings** → **Variables and Secrets** → **Add**:
- `GEMINI_API_KEY` — type **Secret** — your AI Studio key
- `ALLOWED_ORIGIN` — e.g. `https://yourdomain.com` (recommended: stops other sites using your bot)
- `GEMINI_MODEL` — optional. Default is `gemini-flash-latest`. Use `gemini-pro-latest` for deeper answers (slower, uses more quota).

Or from the terminal: `npx wrangler pages secret put GEMINI_API_KEY`

**Redeploy after adding secrets** (Deployments → latest → Retry deployment). Then open the site and click **Ask us anything**.

## Putting it on your existing site

**If your site is already a Cloudflare Pages project:** copy `public/chat-widget.js` into your site's output folder,
copy the `functions/` folder to the root of the project, and paste this before `</body>`:

```html
<script src="/chat-widget.js"
        data-endpoint="/api/chat"
        data-name="Pivot Tech"
        data-greeting="Hi, I'm the Pivot Tech assistant. What are you working on?"
        data-color="#0F3D2E"
        data-accent="#E8542A"
        data-whatsapp="2349020238263"
        data-suggestions="I need a website|Set up CCTV for my office|Microsoft 365 for my team"
        defer></script>
```

**If your site is somewhere else (WordPress, cPanel, etc.) and only uses Cloudflare for DNS:** deploy this folder as its own
Pages project, set `ALLOWED_ORIGIN` to your real domain, then use the full URLs in the script tag:
`src="https://your-project.pages.dev/chat-widget.js"` and `data-endpoint="https://your-project.pages.dev/api/chat"`.

## Testing on your computer
```bash
cp .dev.vars.example .dev.vars   # then put your key in .dev.vars (it's git-ignored)
npm install
npm run dev                      # opens http://localhost:8788
```

## Customising
- Colours: `data-color` (main) and `data-accent`.
- Quick-reply buttons: `data-suggestions`, separated by `|`.
- Tone and rules: `SYSTEM_PROMPT` in `functions/api/chat.js`. It already handles Pidgin, keeps answers short, never invents prices, and collects lead details when a client is ready.

## Notes
- The free AI Studio tier has rate limits. If many people chat at once, the widget shows a "busy, try again" message instead of breaking.
- On the free tier, Google may use the data to improve its products. For client conversations, enable billing on the key so the paid-tier terms apply.
- Chat history is kept only in the visitor's browser tab (sessionStorage) and is cleared when they close it.
- Logs: Pages project → **Functions** → real-time logs, if anything goes wrong.
