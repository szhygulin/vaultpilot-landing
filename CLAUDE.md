# Repo conventions for Claude Code

> **Generic process rules live in `~/.claude/CLAUDE.md`** (auto-loaded by Claude Code from the private [claude-md-global](https://github.com/szhygulin/claude-md-global) repo). The rules below are project-specific.

## Deploy-on-push: every merge to `main` goes live immediately
- This repo is the source for `vaultpilot-mcp.ai`. GitHub Pages serves the `main` branch root with no build step. **A merge to `main` deploys to production within ~60 seconds — no staging, no preview, no rollback except by pushing another commit.**
- Validate the rendered HTML in a browser BEFORE merging. Open the file directly (`file://`) or run a local static server (`npx serve .` / `python3 -m http.server`) over the worktree. Mistakes (broken layout, missing image, typo in headline, dead link) hit the live site at the speed of `git push`.
- Treat every PR merge as a production deploy. The "don't watch CI" default in the global rules does NOT apply here — there's no CI, the merge IS the deploy.

## CNAME is pinned to `vaultpilot-mcp.ai`
- The `CNAME` file is a single line containing `vaultpilot-mcp.ai`. GitHub Pages reads it to route the custom domain. **Removing or editing this file breaks the live site and the SSL cert** (Let's Encrypt re-issuance is gated on the CNAME being stable).
- If a domain change is genuinely intended, coordinate three things in the same change: DNS records at the registrar, the `CNAME` file edit, and forcing a new SSL cert issuance via GitHub Pages settings. None of those land safely in isolation.

## `robots.txt` and `sitemap.xml` are coupled
- `robots.txt` references `sitemap.xml` by URL; `sitemap.xml` enumerates every public route on the site.
- **Adding a new HTML page** (e.g. a future `pricing.html`) requires adding it to `sitemap.xml` so search engines index it. Otherwise the page exists but isn't discoverable.
- **Removing a page** requires removing it from `sitemap.xml` (and optionally adding a `Disallow` line to `robots.txt`). Stale sitemap entries cause 404s in Google Search Console — visible only after the next crawl, when the warning's already accumulated.
- **Renaming a page** is "remove old + add new" — both files updated.

## `og-card.html` ↔ `og-card.png` coupling
- `og-card.png` is the social-preview image rendered when the site is shared on X / Slack / Discord / iMessage. It's a screenshot generated from `og-card.html` (1200×630 viewport, headless browser). The HTML is the source; the PNG is the artifact.
- **Editing `og-card.html` (headline, layout, colors) requires regenerating `og-card.png`** — otherwise the social preview stays stale and the live site shows the old card on every share. The README does not currently document the regeneration command; add it here when one is settled (likely `puppeteer` or `playwright` headless screenshot, or a manual browser-DevTools full-page screenshot at the right viewport).
- The `og-card.html` file itself has `<meta name="robots" content="noindex,nofollow">` — keep that. The card is a render-source, not a public page; it should not appear in search results.

## Cloudflare Web Analytics is privacy-respecting (no cookies, no IP retention)
- Cloudflare Web Analytics was added explicitly for the no-cookie, no-IP-retention property — see commit `feat: add Cloudflare Web Analytics — privacy-respecting, no cookies`.
- **Do not replace with a cookie-dropping or IP-retaining tracker** (default Google Analytics, Hotjar, Mixpanel, Plausible-with-default-config) without surfacing the privacy trade-off explicitly. Replacement candidates that preserve the property: self-hosted Plausible with anonymization, Fathom Lite, server-side log analysis. Anything that drops a cookie or retains client IPs is a regression on the explicit design choice — flag it as a trade-off, don't merge silently.
