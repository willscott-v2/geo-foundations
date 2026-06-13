# GEO Foundations

A [Claude Code](https://claude.com/claude-code) **Skill** that makes a web page (or whole site) structurally ready to be **found, understood, and cited** by AI search engines (AI Overviews, AI Mode, ChatGPT, Perplexity) and browser agents — the technical *plumbing* layer beneath the content.

This is the foundation work, not the copy. This skill makes sure a crawler can reach the page, an LLM can resolve what entity it's about, and an agent can act on it. Great answer-first copy on a page with no entity graph and no crawl access still won't get cited.

It's **stack-agnostic** (Next.js, WordPress, Astro, hand-rolled HTML, or a no-code platform) and works **one page at a time**, so you can harden a homepage, then the top service/product page, then the next.

---

## The seven foundations

| # | Foundation | Scope | What "done" means |
|---|------------|-------|-------------------|
| 1 | Crawlability & AI access | Site | `robots.txt` doesn't block the major AI bots; page is in `sitemap.xml` |
| 2 | Connected entity schema | Page | One JSON-LD `@graph` whose nodes cross-link by `@id`; every `sameAs`/`knowsAbout` verified, never guessed |
| 3 | Answer-engine readiness (AEO) | Page + site | Answer-first content; `FAQPage` ↔ visible Q&A parity; a root `/llms.txt` |
| 4 | Web MCP layer | Page | Read-only `document.modelContext.registerTool` tools, feature-detected |
| 5 | Social / share meta | Page | Open Graph + Twitter Card with **absolute** image URLs + canonical |
| 6 | Accessibility (WCAG 2.1 AA) & structure | Page | One `<h1>`, landmarks, skip link, contrast, focus, reflow, reduced-motion |
| 7 | Modern delivery | Page | `<picture>` WebP + fallback; lazy-load below the fold |

**If you only do three:** crawl access (1), connected schema (2), and answer-first + FAQ parity (3) carry most of the citation weight. The rest is upside.

Full principles, acceptance criteria, and "where it lives in an implementation" notes are in [SKILL.md](SKILL.md).

---

## Installation

Pure instructions — no runtime to build. Place the folder where Claude Code discovers skills.

**Personal** (all your projects):

```bash
git clone https://github.com/willscott-v2/geo-foundations.git /tmp/geo-foundations && \
  cp -R /tmp/geo-foundations ~/.claude/skills/geo-foundations
```

**Project** (shared with a specific repo):

```bash
cp -R geo-foundations /path/to/your/project/.claude/skills/geo-foundations
```

However you get the files there, the result should be `~/.claude/skills/geo-foundations/` containing `SKILL.md`.

### Verify

In Claude Code, run `/skills` — you should see `geo-foundations` listed. Claude also invokes it automatically when your request matches its triggers.

---

## Requirements

- **Claude Code** (CLI, desktop, web, or an IDE extension) with web-fetch capability.
- No Python, Node, or other runtime to install. (The runbook's optional JSON-LD parser uses Node or Python if present; the schema validator works without either.)

---

## Usage

Point Claude Code at a page in natural language:

> Make this page AI-citable: https://example.com/services/roof-repair

> Audit the GEO foundations on our homepage and tell me what's missing.

> Add connected schema + WebMCP to https://example.com — verified data only.

Trigger phrases include *"GEO foundations,"* *"AI search readiness,"* *"agent-ready page,"* *"make this page AI-citable,"* *"WebMCP,"* *"llms.txt,"* *"connected schema,"* *"AEO foundations,"* and *"get cited by ChatGPT / Perplexity / AI Overviews."*

### What Claude does

1. **Scopes** the target (single URL, template, or site) and identifies the primary entity type — local business (`LocalBusiness`), brand/SaaS (`Organization`), or individual (`Person`).
2. **Gathers verified facts** for that entity (nothing in schema may be model-invented).
3. **Applies the seven foundations** in order — site-scope items once, per-page items per page.
4. **Verifies against the runbook** ([references/runbook.md](references/runbook.md)) — every page passes its checklist before it's called done.
5. **Reports** what passed (with evidence), what's deferred and why, and the site-scope items still outstanding.

### Notes

- **One page per run** for hardening; on large sites, templatize the schema/meta/WebMCP into the layout once and verify a representative sample per template.
- **Verified data only.** Every `sameAs` is a profile the entity provably owns; every `knowsAbout` Wikidata QID is derived from the canonical Wikipedia page, not hand-picked. Invented entities poison resolution.
- **Constrained / no-code stacks** (Webflow, Squarespace, Wix, locked WordPress) may not allow custom scripts — the skill applies what the platform permits and explicitly defers (never fakes) what it can't inject.

---

## When NOT to use this

- You want the **article / copy** written or refreshed → use a content-writing workflow first, then apply these structural checks.
- You want **one** rich-result type (e.g. Product stars) → use focused schema/rich-result guidance.
- You want to **find what's broken** across an existing site → run an SEO audit first, then come back here to fix the structural gaps it surfaces.

---

## Repository layout

```
geo-foundations/
├── SKILL.md                          # The skill: the seven foundations, how to run, acceptance criteria
├── README.md                         # This file
└── references/
    ├── runbook.md                    # Per-page acceptance checklist + copy-paste verification commands
    └── schema-and-webmcp.md          # Connected @graph, WebMCP, share meta, llms.txt templates
```

---

## A note on the references

`SKILL.md` describes where each foundation's logic "lives in an implementation" (a schema builder, a page renderer, a build step). These are generic layer descriptions — the patterns transfer to any stack. The included templates in [references/schema-and-webmcp.md](references/schema-and-webmcp.md) are the copy-paste starting points; fill the `{{…}}` slots from verified data only.
