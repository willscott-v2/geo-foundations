# Templates — Connected Schema, WebMCP, llms.txt

Copy-paste-able, stack-agnostic patterns. Fill the `{{…}}` slots from **verified** data only.

---

## 1. Connected entity `@graph`

One `<script type="application/ld+json">` per page. Nodes are joined by `@id` — that linkage is the whole point.

**Pick the primary entity first** — it drives everything below:
- **Local business** → `LocalBusiness` subtype if one fits (`Dentist`, `HairSalon`, `RoofingContractor`, `HVACBusiness`, `LegalService`, `HealthAndBeautyBusiness`, …). The fully-worked example below.
- **Non-local brand / SaaS** → `Organization`. Drop `address`/`geo`/`hours`/service-area; the page-type node is usually `WebPage`+`Product`/`SoftwareApplication` or `Article`, not `Service`.
- **Individual** (consultant, creator, author) → `Person`, with verified `sameAs` to their real profiles.

The local-business graph below is the example, not a requirement — swap the primary entity and page-type node to match.

**Server-render this** in page source or the framework head API. Avoid injecting it via Google Tag Manager — GTM-injected JSON-LD is not reliably parsed by all crawlers. On no-code platforms, use the platform's custom-code embed or SEO plugin.

`@id` convention: `{{origin}}#business`, `{{origin}}#website`, `{{pageUrl}}#webpage`, `{{origin}}#area-{{slug}}` (locations), `{{origin}}#service-{{slug}}`, `{{pageUrl}}#breadcrumb`, `{{pageUrl}}#faq`.

**Entity Home in one line:** define each entity's full node once at its home (the org + locations on the homepage; each service at the page it's about), and reference it by `@id` everywhere else. Locations are first-class `Place` nodes, not anonymous `{"name": "…"}` strings buried in `areaServed`.

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "LocalBusiness",
      "@id": "{{origin}}#business",
      "name": "{{Business Name}}",
      "url": "{{origin}}",
      "telephone": "{{+1XXXXXXXXXX}}",
      "address": {
        "@type": "PostalAddress",
        "streetAddress": "{{street}}",
        "addressLocality": "{{city}}",
        "addressRegion": "{{region}}",
        "postalCode": "{{zip}}"
      },
      "geo": { "@type": "GeoCoordinates", "latitude": {{lat}}, "longitude": {{lng}} },
      "aggregateRating": {
        "@type": "AggregateRating",
        "ratingValue": {{rating}},
        "reviewCount": {{count}}
      },
      "sameAs": [
        "{{official website}}",
        "{{https://www.google.com/maps?cid=...}}"
      ],
      "knowsAbout": [
        {
          "@type": "Thing",
          "@id": "{{https://www.wikidata.org/wiki/QXXXXX}}",
          "name": "{{Concept name}}",
          "sameAs": "{{https://en.wikipedia.org/wiki/Concept}}"
        }
      ]
    },
    {
      "@type": "WebSite",
      "@id": "{{origin}}#website",
      "url": "{{origin}}",
      "name": "{{Business Name}}",
      "publisher": { "@id": "{{origin}}#business" }
    },
    {
      "@type": "WebPage",
      "@id": "{{pageUrl}}#webpage",
      "url": "{{pageUrl}}",
      "name": "{{Page Title}}",
      "isPartOf": { "@id": "{{origin}}#website" },
      "about": { "@id": "{{origin}}#business" },
      "primaryImageOfPage": "{{absolute image URL}}",
      "dateModified": "{{YYYY-MM-DD}}"
    },
    {
      "@type": "BreadcrumbList",
      "@id": "{{pageUrl}}#breadcrumb",
      "itemListElement": [
        { "@type": "ListItem", "position": 1, "name": "Home", "item": "{{origin}}" },
        { "@type": "ListItem", "position": 2, "name": "{{Section}}", "item": "{{sectionUrl}}" },
        { "@type": "ListItem", "position": 3, "name": "{{Page Title}}", "item": "{{pageUrl}}" }
      ]
    },
    {
      "@type": "Place",
      "@id": "{{origin}}#area-{{slug}}",
      "name": "{{Area}}",
      "url": "{{origin}}#service-areas"
    },
    {
      "@type": "Service",
      "@id": "{{origin}}#service-{{slug}}",
      "name": "{{Service}}",
      "serviceType": "{{Service}}",
      "description": "{{one sentence}}",
      "provider": { "@id": "{{origin}}#business" },
      "areaServed": { "@id": "{{origin}}#area-{{slug}}" },
      "mainEntityOfPage": { "@id": "{{pageUrl}}#webpage" }
    },
    {
      "@type": "FAQPage",
      "@id": "{{pageUrl}}#faq",
      "isPartOf": { "@id": "{{pageUrl}}#webpage" },
      "about": { "@id": "{{origin}}#business" },
      "mainEntity": [
        {
          "@type": "Question",
          "name": "{{Question?}}",
          "acceptedAnswer": { "@type": "Answer", "text": "{{Answer.}}" }
        }
      ]
    }
  ]
}
```

**Swap the page-type node by page kind:** `Service` for a service page; `Article`/`BlogPosting` for a post; `Product` (+`Offer`, real `AggregateRating`) for a product. Keep business + WebSite + WebPage on every page; add `BreadcrumbList` on every non-homepage page (omit it on the homepage).

**Locations — define once, reference everywhere.** Emit one `Place` node per location served, homed on the homepage (`{{origin}}#area-{{slug}}`); every service references it via `areaServed` → `{"@id": "…#area-{{slug}}"}`. On a location×service page, include the relevant `Place` node alongside the service so the reference resolves on that page too (same `@id` → crawlers merge it). Add `sameAs` → the location's Wikidata QID *only* when verified from its canonical Wikipedia page — otherwise omit it; never guess a place QID.

**Anchor abstract concepts, don't re-home them.** For the *abstract* service concept (the kind of work, not the place-specific offering), add `about` → a verified Wikidata concept (`{"@type":"Thing","@id":"…wikidata…","name":"…","sameAs":"…wikipedia…"}`) on the `Service` node, or carry it on the business's `knowsAbout`. Omit `about` when you don't have a verified QID — `serviceType` alone is fine.

**Article/BlogPosting — get the author entity right.** This is where content-site E-E-A-T lives, so don't leave the author as a bare string:
```json
{
  "@type": "Article",
  "@id": "{{pageUrl}}#article",
  "headline": "{{Title}}",
  "datePublished": "{{YYYY-MM-DD}}",
  "dateModified": "{{YYYY-MM-DD}}",
  "author": { "@id": "{{origin}}#author-{{slug}}" },
  "publisher": { "@id": "{{origin}}#business" },
  "isPartOf": { "@id": "{{pageUrl}}#webpage" }
}
```
…with a real `Person` node whose `sameAs` are verified profiles:
```json
{ "@type": "Person", "@id": "{{origin}}#author-{{slug}}", "name": "{{Author}}",
  "sameAs": [ "{{verified LinkedIn}}", "{{verified author site / org bio}}" ] }
```
Keep `datePublished`/`dateModified` real and current — freshness is an AEO input.

**FAQPage caveat:** since Google's 2023 change, `FAQPage` no longer yields FAQ rich results for most sites (gov/health authorities only). Keep it — its value is now AEO answer-extraction and parity with visible on-page Q&A — but don't promise a client SERP FAQ snippets.

### Hard rules (these are what make it trustworthy)
- **Verified data only.** Every field traces to a real source (GBP, the client's own site, reviews). Nothing model-invented — invented `sameAs`/credentials poison entity resolution.
- **QIDs from the canonical page.** To add a `knowsAbout` concept: open its English Wikipedia page → grab the QID via the MediaWiki `pageprops` API (`?action=query&prop=pageprops&titles=...&format=json` → `pageprops.wikibase_item`). A bare Wikidata 200 only proves a QID exists, not that it names the right concept. (Real miss caught this way: `Q748069` is a Hungarian village, not "Medical spa.")
- **Escape the block.** Replace `</script` → `<\/script`, and HTML-escape any string that could contain `< > &`. The serializer should also strip the JSON-LD-unsafe Unicode line/paragraph separators.

---

## 2. WebMCP — read-only browser-agent tools

Inline `<script>` near end of `<body>`. Feature-detected; silent where unsupported. Pull `B` from the **same verified data** as the schema. Keep tools read-only. (Emerging API — a Chrome origin trial as of this review; the surface may change. Where a no-code platform forbids an end-of-body script, defer this and note it — don't fake it.)

```html
<script>(function () {
  var mc = (self.document && document.modelContext) ||
           (self.navigator && navigator.modelContext);
  if (!mc || !mc.registerTool) return;            // feature-detect, no-op otherwise
  var B = {{businessJSON}};                         // JSON.stringify(...).replace(/<\/script/gi,"<\\/script")
  var ro = { readOnlyHint: true };
  try {
    mc.registerTool({ name: "get_business_info", title: "Business info",
      description: "Name, category, phone, address, rating, service areas for " + B.info.name + ".",
      annotations: ro, execute: async function () { return B.info; } });
    mc.registerTool({ name: "get_services", title: "Services",
      description: "Services offered by " + B.info.name + ", with page links.",
      annotations: ro, execute: async function () { return B.services; } });
    mc.registerTool({ name: "get_hours", title: "Business hours",
      description: "Opening hours by day for " + B.info.name + ".",
      annotations: ro, execute: async function () { return B.hours; } });
    mc.registerTool({ name: "get_contact", title: "Contact options",
      description: "Phone, SMS and map link for " + B.info.name + ".",
      annotations: ro, execute: async function () { return B.contact; } });
  } catch (e) {}
})();</script>
```

`B` shape (all values already validated; omit/`null` anything you can't confirm):
```json
{
  "info": { "name": "", "category": "", "phone": null, "address": null,
            "city": null, "region": null, "rating": null, "reviews": null,
            "serviceAreas": [] },
  "services": [ { "name": "", "page": null } ],
  "hours": [ { "day": "Monday", "value": "9:00 AM – 5:00 PM" } ],
  "contact": { "phone": null, "sms": null, "tel": null, "map": "" }
}
```
**Generalize beyond local business:** for a SaaS/brand, expose `get_product_info`, `get_pricing`, `get_docs_links`. The contract is the same — read-only, verified, JSON. Choose tools an agent would actually call to act on the page.

---

## 3. Social / share meta (head)

```html
<link rel="canonical" href="{{pageUrl}}">
<meta property="og:title" content="{{title}}">
<meta property="og:description" content="{{description}}">
<meta property="og:type" content="website">
<meta property="og:image" content="{{ABSOLUTE image URL}}">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="{{title}}">
<meta name="twitter:description" content="{{description}}">
<meta name="twitter:image" content="{{ABSOLUTE image URL}}">
```
Image URLs must be absolute (resolve against the canonical origin). Use `summary_large_image` when there's a real hero image; `summary` otherwise.

---

## 4. Modern image delivery

```html
<picture>
  <source type="image/webp" srcset="{{/path/img.webp}}">
  <img src="{{/path/img.jpg}}" alt="{{meaningful alt}}" loading="lazy" width="{{w}}" height="{{h}}">
</picture>
```
Generate the `.webp` sibling at build (e.g. sharp at `quality: 80`). Above-the-fold hero: drop `loading="lazy"`.

---

## 5. llms.txt  *(site root, `text/plain`)*

```markdown
# {{Business / Site Name}}

> {{One-line summary — what it is, where, the one credibility fact (rating/review count or category+location).}}

## About
{{2–4 sentences.}}

## Services            ## or Products / Offerings
- {{Service}} — {{one line}}

## Service areas        ## (local only)
{{Area, Area, Area}}

## Hours
Monday: {{…}}
Tuesday: {{…}}

## Contact
Phone: {{(XXX) XXX-XXXX}}
Address: {{…}}
Website: {{origin}}

## Key pages
- [{{Page}}]({{url}})
```
Build it from the same verified facts as the schema and WebMCP — one source of truth across all three surfaces (schema, WebMCP, llms.txt) so an agent gets a consistent answer wherever it looks.
