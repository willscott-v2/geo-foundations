---
name: geo-foundations
metadata:
  version: 1.2.0
  lastReviewed: 2026-06
  author: Will Scott
description: When the user wants to make a web page (or site) structurally ready to be found, understood, and cited by AI search and agents — the technical foundation layer beneath content. Use when the user mentions "GEO foundations," "AI search readiness," "agent-ready page," "make this page AI-citable," "web MCP" / "WebMCP," "llms.txt," "agents.json" / "agent manifest," "connected schema" / "entity graph," "AEO foundations," "structured data for AI," or "get cited by ChatGPT / Perplexity / AI Overviews." Stack-agnostic; works page-by-page.
---

# GEO Foundations

Make a page structurally ready to be **found, understood, and cited** by AI search engines (AI Overviews, AI Mode, ChatGPT, Perplexity) and browser agents — independent of the content itself or the tech stack it ships on.

This is the *plumbing* layer: it makes sure a crawler can reach the page, an LLM can resolve what entity it's about, and an agent can act on it. Great answer-first copy on a page with no entity graph and no crawl access still won't get cited.

## What this skill is for

Apply or audit the eight foundations below on **one page at a time**. A client site rarely gets rebuilt all at once — you harden the homepage, then the top service/product page, then the next. Each page is independently shippable. Four of the eight are site-scope prerequisites (robots, sitemap, llms.txt, agents.json); the rest are per-page.

**If you only do three:** (1) crawl access — Foundation 1, (2) connected entity schema — Foundation 2, (3) answer-first + FAQ parity — Foundation 3. Those carry most of the citation weight; foundations 4–8 are upside. Triage to them first on a small budget or a quick pass.

**Templatize at scale.** "One page at a time" is the right unit for *hardening and verifying*. But on a site with dozens or hundreds of pages, you don't hand-edit each one — inject the schema graph, share meta, and WebMCP script into the layout/`<head>` template once, parameterized per page, then verify a representative sample per template (homepage, a service/product page, an article). The runbook checks still run per URL.

Each foundation below names the *layer* where the logic typically lives — copy the **pattern**, not any one stack's code, onto whatever the site runs (Next.js, WordPress, Astro, hand-rolled HTML, or a no-code platform).

## When NOT to use this
- The user wants the *article/copy* written or refreshed → use a content-writing workflow first, then apply these structural checks.
- The user only wants one rich-result type (e.g. Product stars) → use focused schema/rich-result guidance.
- The user wants to find what's broken across an existing site → run an SEO/technical audit first, then use this to fix the structural gaps it surfaces.

## How to run it

1. **Scope.** Confirm the target: a single URL, a page template, or a whole site. **Identify the primary entity type** — local business (`LocalBusiness`+subtype), non-local brand/SaaS (`Organization`), or an individual (`Person`) — because it drives the schema graph, the WebMCP tools, and the llms.txt shape. The local-business shape below is the default *example*, not an assumption. Confirm the stack: how pages get their `<head>`, and how JSON-LD gets injected.
   - **Prefer server-rendered JSON-LD** (in the page source / framework head API) over injecting it via Google Tag Manager — GTM-injected structured data is not reliably parsed by all crawlers.
   - **Constrained / no-code stacks** (Webflow, Squarespace, Wix, locked-down WordPress) may not allow an arbitrary end-of-`<body>` script (Foundation 4) or custom `<head>`. Apply what the platform permits — schema via a custom-code embed or SEO plugin, share meta via the platform's fields — and explicitly defer (don't fake) what you can't inject. Note the deferral in the report.
2. **Gather verified facts** for the page's primary entity (see Foundation 2 — nothing in schema may be model-invented).
3. **Apply the eight foundations** in order, per page. Site-scope items (1, 3-llms.txt, 8) once per site.
4. **Verify against the runbook** — `references/runbook.md`. Every page must pass its checklist before you call it done.
5. **Report** what passed, what's deferred, and (if possible) run the AI-visibility outcome check.

Templates to copy: `references/schema-and-webmcp.md`.

---

## The eight foundations

Each: the principle, why it matters for AI search, the acceptance criteria (what "done" means), and where the logic typically lives in an implementation.

### 1. Crawlability & AI access  *(site-scope)*
**Principle.** AI crawlers must be allowed, and every page must be discoverable.
**Why.** If `robots.txt` blocks `GPTBot` / `ClaudeBot` / `PerplexityBot` / `Google-Extended`, the page cannot be cited — full stop. A page absent from `sitemap.xml` may never be discovered.
**Acceptance.** `robots.txt` does not disallow the major AI user-agents; the page's URL appears in `sitemap.xml` with a `<lastmod>`. (Many hosts — Cloudflare included — ship a *default* AI-bot block; check it explicitly.)
**Where it lives.** A build/SEO step writes `robots.txt` + `sitemap.xml`. Note: managed hosts (Cloudflare among them) often ship a *default* AI-bot block — disable it explicitly if you want the page citable.

### 2. Connected entity schema  *(per-page — with one site-scope prerequisite)*
**Principle.** Give every entity a home: define it once at a canonical URL with a stable `@id`, then reference it everywhere else by `@id`. Ship one JSON-LD `@graph` per page whose nodes are cross-linked that way, not a flat island of one type.
**Why.** A connected graph lets an LLM resolve "what real-world entity is this page about, who publishes it, and how do its parts relate" in one pass. Flat `LocalBusiness` with no links is far weaker for entity resolution. An Entity Home gives each thing one authoritative definition to resolve to, instead of N scattered copies a crawler has to reconcile.

**Prerequisite (site-scope): the Entity Home map.** Before hardening pages, decide *where each entity lives*. An Entity Home is the one canonical URL that authoritatively defines an entity. It does **not** require a dedicated page — an existing page, or a section of one addressed by a URL fragment, can be a home. Map it once per site:

| Entity | Home URL (existing page) | `@id` |
|---|---|---|
| The organization / person | homepage | `{origin}#business` |
| Each location served | a Service-Areas section, or a location page if one exists | `{origin}#area-{slug}` |
| Each service *as delivered* | the service or service×location page it's about | `{pageUrl}#service` |
| Each author / staff person | their bio page, or an About section | `{origin}#person-{slug}` |

Rules for the map:
- **No new pages required.** Pick the best URL you already ship; a fragment on an existing page is a valid home. A dedicated indexable page is a *stronger* home (own content, internal links, its own ranking surface) — choose per budget and how much that entity needs to rank on its own.
- **Don't invent homeless entities.** If a concept has no page and no section to sit behind (e.g. an *abstract* service like "Roofing" on a site whose only pages are location×service intersections), don't give it a fake internal home. Anchor it to its external concept instead — `about` / `knowsAbout` → the Wikidata QID — and let `serviceType` carry the label across the pages that share it.
- **Home what you own; anchor what you don't.** Specific places you serve and real people are worth internal homes (they disambiguate against external authorities). Abstract concepts belong to external knowledge bases — point at them, don't re-home them.

**Acceptance.**
- An **Entity Home map** exists for the site (see prerequisite); every entity in the graph has a designated home URL and a stable `@id`.
- The graph contains: the **primary entity** (`Organization` / `LocalBusiness`+subtype / `Person`), a **`WebSite`** node, a **`WebPage`** node for *this exact URL*, the **page-specific type** (`Service` / `Article` / `Product` / …), a **`BreadcrumbList`** for non-homepage pages, a **`FAQPage`** when the page has Q&A, and — for multi-location businesses — a **`Place`** node per location served.
- **Define at home, reference elsewhere.** On a page that *is* an entity's home, emit the **full node**. On any other page that uses that entity, emit a **thin reference** (`{"@id": "…"}`) — don't restate the full node. Each non-trivial node carries `url` (and `mainEntityOfPage` where it applies) pointing at its home, so a crawler landing anywhere can find the canonical definition.
- Nodes reference each other by `@id` (e.g. `WebPage.about` → `#business`, `WebPage.isPartOf` → `#website`, `Service.provider` → `#business`, `Service.areaServed` → `#area-{slug}`). Use the **same `@id` origin on every page** so crawlers merge references into one entity, not N copies.
- **Locations (multi-location):** each `Place`/`City` node is defined once at its home (a Service-Areas section or a location page) and referenced by `@id` from every page scoped to it. Add a verified `sameAs` → the **Wikidata QID for that place** *only* when you have it (derived from the canonical Wikipedia page, same discipline as `knowsAbout`); never guess a place QID.
- **Freshness:** `WebPage` carries `dateModified`; `Article`/`BlogPosting` carries both `datePublished` and `dateModified` (real dates, kept current). Freshness is an AEO input.
- **Author/publisher (content sites):** `Article.author` → a `Person` node *defined at its home* with verified `sameAs` profiles; `Article.publisher` → the business. The author entity is where content-site E-E-A-T strength comes from — don't leave it as a bare string.
- **Every `sameAs` and `knowsAbout` value is verified, never guessed.** `sameAs` = URLs you can confirm the entity owns (official site, Google CID, real social profiles). `knowsAbout` Wikidata QIDs are **derived from the canonical Wikipedia page** (MediaWiki `pageprops` API) — a bare 200 only proves a QID exists, not that it names the right concept.
- JSON-LD validates and is properly escaped (no `</script>` breakout, no raw user strings).

**Sequencing (retrofits).** Harden **home pages first** — homepage, then any hub that's a home — before the child pages that reference them, so thin references resolve to a real definition instead of dangling.

**Transitional fallback.** During a partial retrofit, before an entity's home page is hardened, it's safe to **repeat the full node** on the child page (with the same stable `@id`) rather than ship a dangling reference. Repeat-with-stable-`@id` is the safe mid-migration default; thin-reference-to-home is the clean end state you converge to once the home exists. Both are valid — the difference is *when* in the migration you are.

**Where it lives.** A schema builder that assembles the per-page `@graph` (with `@id` helpers) from the Entity Home map, backed by an HTTP-verified Wikipedia/Wikidata catalog for `knowsAbout` (and, where available, for location `sameAs`). Copy-paste templates in `references/schema-and-webmcp.md`.

### 3. Answer-engine readiness (AEO)  *(per-page + site-scope llms.txt)*
**Principle.** Lead with the answer; mirror real questions in `FAQPage` schema *and* on the page; publish a machine-readable site summary.
**Why.** Answer engines extract the span that directly answers the query. Answer-first + Q&A parity gives them a clean, attributable chunk. `llms.txt` is a fast path for agents to grok the site, and (as of this review) a signal in Lighthouse's experimental Agentic-Browsing audit — adoption by the engines themselves is still early, so treat it as low-cost insurance, not a proven ranking input.
**FAQ caveat.** Since Google's 2023 change, `FAQPage` no longer renders FAQ rich results for most sites (restricted to authoritative gov/health domains). Its value now is **AEO extraction and answer parity**, not SERP snippets — don't promise a client FAQ rich results.
**Acceptance.** Key question answered in the first screen of content; `FAQPage` schema items have matching visible on-page Q&A (e.g. `<details>/<summary>`); a root `/llms.txt` returns `200 text/plain` with H1 + summary + the page's section facts + key links.
**Where it lives.** The build/SEO step emits `/llms.txt`; FAQs render as visible `<details>/<summary>` accordions that mirror the `FAQPage` schema. Pair the copy work with an answer-first content pass.

### 4. Web MCP layer  *(per-page)*
**Principle.** Expose the page's entity as **read-only** tools to browser AI agents via `document.modelContext.registerTool`, feature-detected so it's silent where unsupported.
**Why.** This is an emerging agentic-browsing surface — a Chrome origin trial as of this review (API surface and availability are likely to change; verify current status before relying on it). It lets an in-browser agent *call* the page for structured facts (contact, services, hours, info) instead of scraping the DOM — a cleaner contract than HTML parsing. Low-cost to add now, feature-detected so it's inert where unsupported; don't oversell it to a client as a live ranking factor.
**Acceptance.** A small inline script registers a few read-only tools (`annotations:{readOnlyHint:true}`) drawn from the *same verified data as the schema*; it guards on `mc && mc.registerTool` and never throws where the API is absent; tool data contains nothing private or invented. Where the stack forbids an end-of-body script (no-code platforms), defer this foundation and say so — don't fake it.
**Where it lives.** A small inline script in the page renderer registers the tools; its payload is shaped at build time from the same verified data as the schema. Template in `references/schema-and-webmcp.md`.

### 5. Social / share meta  *(per-page)*
**Principle.** Open Graph + Twitter Card tags with **absolute** image URLs and a canonical link.
**Why.** Share unfurls and several crawlers read OG/Twitter; relative image paths break them. Canonical prevents duplicate-URL dilution of the entity signal.
**Acceptance.** `og:title`/`og:description`/`og:type`/`og:image`, `twitter:card` (+ title/description/image), `<link rel="canonical">`; the image URL is fully qualified against the canonical origin.
**Where it lives.** The page renderer's `<head>` block; a helper absolutizes the image URL against the canonical origin.

### 6. Accessibility (WCAG 2.1 AA) & agent-friendly structure  *(per-page)*
**Principle.** Meet WCAG 2.1 AA. Clean semantic structure is also the agent-readable structure — the same outline assistive tech walks is the one an agent reads.
**Why.** AA is the de-facto legal/contract baseline (ADA, Section 508, EAA). And the a11y tree it produces is exactly what Lighthouse's Agentic-Browsing audit and browser agents consume — accessibility and agent-readiness are the same work.
**Acceptance.**
*Structure (the floor):*
- Exactly one `<h1>`; semantic `<main>` / `<header>` / `<nav>` / `<footer>`; clean heading order (no skipped levels).
- `<html lang>` set; every meaningful image has `alt`; decorative images/icons have empty `alt`/`aria-hidden="true"`.
- No dead links (`href="#"` / `tel:#`) — omit the element instead. *(1.3.1, 2.4.x, 3.1.1, 4.1.2)*

*WCAG AA adds:*
- **1.4.3 Contrast.** Body text ≥ **4.5:1**, large text (≥24px, or ≥19px bold) ≥ **3:1**, against its actual background. Audit text-on-bg, muted/secondary text, and text-on-button.
- **1.4.11 Non-text contrast.** UI borders, icons, and focus indicators ≥ **3:1**.
- **2.4.7 Focus visible.** A visible focus indicator on every interactive element — never `outline:none` without an equivalent replacement (`:focus-visible`).
- **2.4.1 Bypass blocks.** A "Skip to content" link to `#main` as the first focusable element.
- **2.1.1 Keyboard.** Everything operable by keyboard; native `<a>`/`<button>`/`<details>` are fine — custom widgets need correct ARIA + key handling.
- **1.4.10 Reflow / 1.4.4 Resize.** Usable at 320px and at 200% zoom without horizontal scroll or clipping; use relative units.
- **2.3.3 / prefers-reduced-motion.** Honor `@media (prefers-reduced-motion: reduce)` — kill smooth-scroll and non-essential transitions/animations.
- **Embeds.** `<iframe>` has a `title`.

**Where it lives.** The section/render layer enforces the structural floor (single `<h1>`, `<main id="main">`, skip link, `:focus-visible`, reduced-motion block, CTAs that omit themselves when undialable). A small contrast helper (`contrastRatio()` / a per-theme check) is the deterministic gate, run in QC. When colors are theme-driven, run the contrast check at design-token time, not per page.

### 7. Modern delivery  *(per-page)*
**Principle.** Serve modern image formats with a fallback; lazy-load below the fold.
**Why.** Performance is a ranking and UX input; a `<picture>` with WebP + raster fallback is the low-risk win. (Modern-format coverage is a common grader ding.)
**Acceptance.** Raster images served via `<picture>` with a WebP `source` + original `<img>` fallback; offscreen images `loading="lazy"`; above-the-fold hero is *not* lazy-loaded and has explicit `width`/`height` (or aspect-ratio) to protect CLS.
**Scope boundary.** This foundation is the low-risk image win, not full performance tuning. Core Web Vitals (LCP, CLS, INP) matter for ranking and UX but a real CWV pass — render-blocking resources, font loading, JS cost — is its own engagement. Flag a failing CWV result here; fix it outside this skill.
**Where it lives.** An image helper in the renderer emits the `<picture>`; a build step generates the WebP sibling (e.g. sharp).

### 8. Agent action manifest (agents.json + auto-discovery)  *(site-scope)*
**Principle.** Publish a machine-readable manifest of the **actions** an agent can take on the site — at `/.well-known/agents.json` — and make it (plus `llms.txt`) discoverable from the page `<head>`.
**Why.** `llms.txt` and schema tell an agent what the site *is*; `agents.json` tells it what it can *do* and how to chain the calls. It's the executable layer of agent-readiness and a check in Lighthouse's experimental Agentic-Browsing audit (and third-party graders). Adoption by the engines is still early — treat it as low-cost, forward-looking insurance, not a proven ranking input.
**Truthfulness gate.** A manifest must only describe **real, reachable, typed** operations. Never invent endpoints. On a brochure/static site with no API, expose the facts you already publish as a small read-only JSON endpoint (mirror the WebMCP/`llms.txt` data — one source of truth) and describe *that*. Do not fabricate booking/checkout POST actions a site can't actually serve.
**Acceptance.**
- `/.well-known/agents.json` returns `200 application/json` and conforms to the agents.json spec (v0.1.0): top-level `agentsJson`, `info` (`title`/`version`/`description` — put the natural-language **agent instructions** in `description`), `sources`, and `flows`.
- **Actions are typed:** each `flows[].actions[]` references an `operationId` defined in a real OpenAPI `sources[].path` (also served on the domain), whose operations carry typed request/response schemas. The endpoints the OpenAPI names actually resolve.
- A copy is also reachable at `/agents.json` (some agents/graders probe the root as well as `/.well-known/`).
- **Auto-discovery:** the page `<head>` carries `<link>` tags pointing to both `/llms.txt` and `/.well-known/agents.json`; `robots.txt` does not block either path.
- One source of truth: the manifest's data matches `llms.txt` and the WebMCP tools (Foundation 4).
**Conform to the real spec — don't cargo-cult a grader.** Third-party "AI-readiness" checkers invent their own rubrics; build to the published spec, not a tool's idiosyncrasies. Two traps seen in the wild:
- **Typed actions live in `flows[].actions[]` → OpenAPI operations. Do NOT add a top-level `actions` array.** Some graders look for one and report a spec-correct file as "no actions." That array isn't in the agents.json spec; adding it to pass a test spreads a non-standard shape. If a grader demands it, fix the grader, not the file.
- **Don't invent an instructions file (e.g. `agent-instructions.md`).** It isn't a standard. Agent behavioral guidance belongs in agents.json `info.description` and `llms.txt`. The real named standard, `AGENTS.md`, is repo/coding-agent oriented (build/test/style rules), not for web-browsing agents.
**Adoption reality (don't oversell).** schema.org, `robots.txt`, and `sitemap.xml` are consumed by crawlers today — the proven layer. `llms.txt` and `agents.json` are publisher-side and not yet systematically fetched by the major agents; treat them as low-cost, forward-looking bets. Of the two, `llms.txt` is the stronger bet (referenced repeatedly in Google docs). Never tell a client agents.json is a live ranking input.
**Where it lives.** A site-scope build/SEO step writes the three files (data endpoint, OpenAPI source, manifest + root copy); the render/`<head>` template emits the two discovery `<link>` tags. Spec: github.com/wild-card-ai/agents-json. Worked reference: `getsightline.com`. Template in `references/schema-and-webmcp.md`.

---

## Output

When you finish a page, state plainly: which of the eight now pass (with the evidence — quote the tag / show the validator result), which are deferred and why, and the site-scope items still outstanding. Don't claim a foundation is in place without showing the check. If something can't be verified, say so.
