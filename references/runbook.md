# GEO Foundations — Runbook & Acceptance Checklist

Run this per page before calling it done. It's stack-agnostic: the checks read the *rendered HTML* and the served files, so they work whether the page came from Next.js, WordPress, or static files.

Set two variables first:

```sh
URL="https://example.com/the-page/"      # the exact page you're hardening
ORIGIN="https://example.com"             # its scheme+host
PAGE="$(curl -fsSL "$URL")"              # rendered HTML (use a headless render if JS-built)
```

> If the site is JS-rendered, replace the `curl` capture with a headless fetch (Playwright/`wget`-after-render) so the JSON-LD and the WebMCP script are present in `$PAGE`.

---

## Site-scope prerequisites (once per site)

**1. AI crawlers allowed**
```sh
curl -fsSL "$ORIGIN/robots.txt"
```
PASS = no `Disallow: /` aimed at `GPTBot`, `ClaudeBot`, `OAI-SearchBot`, `PerplexityBot`, `Google-Extended`, `CCBot`, `Bytespider`. Hosts (Cloudflare etc.) often inject a default AI-bot block — confirm it's off if you want citations.

**2. Page is in the sitemap**
```sh
# normalize trailing slash before comparing; many sitemaps differ on it
curl -fsSL "$ORIGIN/sitemap.xml" | grep -F "${URL%/}"
```
PASS = the URL is listed (ideally with `<lastmod>`). If it's a sitemap index, follow the child sitemap that should contain this URL. A miss here is often a trailing-slash or percent-encoding mismatch, not a true absence — confirm the canonical form before flagging it.

**3. llms.txt present**
```sh
curl -fsSI "$ORIGIN/llms.txt" | grep -i '200\|content-type'
```
PASS = `200` and `text/plain`. Body should carry an H1, a one-line summary, and links to the key pages.

---

## Per-page checklist

### Foundation 2 — Connected entity schema

Extract every JSON-LD block. **Don't use `grep -o '<script…>.*</script>'`** — `grep` is line-by-line, so it silently misses pretty-printed (multi-line) JSON-LD and returns a false "no schema." Pull and pretty-print the blocks with a real parser:
```sh
# Node: print every ld+json block, parsed (works for multi-line + minified)
printf '%s' "$PAGE" | node -e '
  const h=require("fs").readFileSync(0,"utf8");
  const re=/<script[^>]*type=["\x27]application\/ld\+json["\x27][^>]*>([\s\S]*?)<\/script>/gi;
  let m,n=0; while((m=re.exec(h))){ n++; try{ console.log(JSON.stringify(JSON.parse(m[1]),null,2)); }catch(e){ console.log("INVALID JSON in block "+n+": "+e.message); } }
  if(!n) console.log("NO ld+json blocks found");'
```
(No Node? `python3 -c` with the same regex + `json.loads`, or just paste the page URL into the validator below.)

Then confirm:
- [ ] An `@graph` (or linked set) with: primary entity (`Organization`/`LocalBusiness`+subtype/`Person`), `WebSite`, `WebPage` for **this** URL, the page-type node (`Service`/`Article`/`Product`/…), `BreadcrumbList` on non-homepage pages, and `FAQPage` if the page has Q&A.
- [ ] Nodes cross-link by `@id` (`WebPage.about`→business, `WebPage.isPartOf`→website, page-type `.provider`/`.author`/`.publisher`→business). Same `@id` origin across pages so the entity merges, not duplicates.
- [ ] `WebPage` has `dateModified`; `Article`/`BlogPosting` has real `datePublished` + `dateModified`; the author is a `Person`/`Organization` node (not a bare string) with verified `sameAs`.
- [ ] Every `sameAs` URL is one the entity verifiably owns. **No guessed profiles.**
- [ ] Every `knowsAbout`/`about` Wikidata QID was derived from the canonical Wikipedia page (not hand-picked). Spot-check one: open the Wikipedia page → "Wikidata item" → confirm the QID matches.
- [ ] Validates: paste into <https://validator.schema.org/> (or Google Rich Results Test) — 0 errors.
- [ ] No `</script>` breakout / unescaped user strings in the block.

### Foundation 3 — Answer-engine readiness
- [ ] The page's primary question is answered in the first screen of content (answer-first, not buried).
- [ ] `FAQPage` schema items each have a matching **visible** on-page Q&A.
```sh
printf '%s' "$PAGE" | grep -c '<details'      # visible FAQ accordions (one pattern)
printf '%s' "$PAGE" | grep -o '"@type":"FAQPage"'
```
PASS = visible Q&A count ≥ schema `Question` count (no schema-only FAQs).

### Foundation 4 — Web MCP layer
```sh
printf '%s' "$PAGE" | grep -o 'registerTool'
printf '%s' "$PAGE" | grep -o 'readOnlyHint'
```
- [ ] `registerTool` present; tools are read-only (`readOnlyHint:true`).
- [ ] Script is feature-detected (guards on `modelContext`/`registerTool`) and can't throw where unsupported.
- [ ] Tool payload matches the schema facts and contains nothing private/invented.

### Foundation 5 — Social / share meta
```sh
printf '%s' "$PAGE" | grep -oE '<meta (property="og:[^"]+"|name="twitter:[^"]+")[^>]*>'
printf '%s' "$PAGE" | grep -o '<link rel="canonical"[^>]*>'
```
- [ ] `og:title`, `og:description`, `og:type`, `og:image` present.
- [ ] `twitter:card` (+ title/description/image) present.
- [ ] `og:image`/`twitter:image` are **absolute** URLs (start with `http`).
- [ ] `<link rel="canonical">` present and correct.

### Foundation 6 — Accessibility (WCAG 2.1 AA) & structure
```sh
printf '%s' "$PAGE" | grep -c '<h1'                 # must be 1
printf '%s' "$PAGE" | grep -o '<main'               # must exist
printf '%s' "$PAGE" | grep -o 'id="main"'           # skip-link target
printf '%s' "$PAGE" | grep -o 'href="#main"'        # the skip link itself
printf '%s' "$PAGE" | grep -o '<html[^>]*lang='     # lang set
printf '%s' "$PAGE" | grep -oE '<img[^>]*>' | grep -vc 'alt='   # imgs missing alt → want 0
printf '%s' "$PAGE" | grep -oE 'href="#"|href="tel:#"|href="sms:#"'  # dead CTAs → want none
printf '%s' "$PAGE" | grep -o '<iframe' && printf '%s' "$PAGE" | grep -o '<iframe[^>]*title='  # every iframe titled
printf '%s' "$PAGE" | grep -o 'prefers-reduced-motion'   # reduced-motion honored
printf '%s' "$PAGE" | grep -o ':focus-visible'           # visible focus styles
```
*Structure (floor):*
- [ ] Exactly one `<h1>`; `<main>`/`<header>`/`<nav>`/`<footer>` landmarks; no skipped heading levels.
- [ ] `<html lang>` set; 0 meaningful images without `alt`; decorative icons `aria-hidden`.
- [ ] No dead links — undialable phone / placeholder anchors omitted, not `#`.

*WCAG AA:*
- [ ] **Contrast (1.4.3):** body text ≥ 4.5:1, large text ≥ 3:1 vs. its real background. Check with any stack: axe DevTools, WAVE, or the Lighthouse a11y audit against the rendered page. If your colors are theme-driven (a fixed palette), a deterministic build-time gate — a small `contrastRatio()` helper run over the theme tokens — catches failures before render; otherwise sample the page with the tools above.
- [ ] **Non-text contrast (1.4.11):** borders, icons, focus rings ≥ 3:1.
- [ ] **Focus visible (2.4.7):** every interactive element shows a focus indicator; no bare `outline:none` (look for `:focus-visible`).
- [ ] **Bypass blocks (2.4.1):** a "Skip to content" link to `#main` is the first focusable element.
- [ ] **Keyboard (2.1.1):** tab through the page — all controls reachable/operable, no traps.
- [ ] **Reflow/resize (1.4.10/1.4.4):** no horizontal scroll at 320px or 200% zoom.
- [ ] **Reduced motion (2.3.3):** `@media (prefers-reduced-motion: reduce)` disables smooth-scroll + non-essential animation.
- [ ] **Embeds:** every `<iframe>` has a `title`.

> Tooling: for a full per-page audit run Lighthouse (a11y category) or axe-core. The `grep`/`a11y.js` checks above are the fast deterministic gate; an axe/Lighthouse pass is the confirmation before sign-off.

### Foundation 7 — Modern delivery
```sh
printf '%s' "$PAGE" | grep -o '<picture'
printf '%s' "$PAGE" | grep -o 'type="image/webp"'
printf '%s' "$PAGE" | grep -oc 'loading="lazy"'
```
- [ ] Raster images wrapped in `<picture>` with a WebP `source` + `<img>` fallback.
- [ ] Offscreen images `loading="lazy"`.

---

## Outcome check (optional, the real scoreboard)

Foundations are inputs; citation is the outcome. After hardening, test whether AI engines name the entity for its core queries:

- Ask the natural questions a prospect would (`"best <category> in <city>?"`, `"where can I get <service> near <city>?"`) across **more than one engine** — each crawls and cites differently: Perplexity (or its API), ChatGPT with browsing/search, and Google AI Overviews / AI Mode. Don't generalize from a single engine.
- Record whether the entity is named and, if cited, which URL.
- Re-run after a crawl cycle; this is a trend, not an instant result (rollouts are gradual).
- **You can't cleanly A/B this** — there's no holdout, engines roll out gradually, and answers vary run to run. Treat it as a directional before/after, not a controlled experiment.

Don't report a lift you didn't measure. "Foundations in place; visibility check pending next crawl" is the honest status until you have a before/after.

---

## Done = report

State, per page: foundations passing (with evidence — quote the tag or show the validator result), foundations deferred (and why), site-scope items outstanding. No green check without the check.
