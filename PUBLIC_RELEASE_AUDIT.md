# Public release audit: geo-foundations

Date: 2026-06-13

## Scope

Created a standalone public copy of the `geo-foundations` skill from the private `searchinfluence/ai-skills` repository. The public copy includes only:

- `README.md`
- `SKILL.md`
- `references/runbook.md`
- `references/schema-and-webmcp.md`
- `LICENSE`

No private git history was copied; the public repo was initialized as a fresh repository with a single initial commit.

## Sanitization performed

Removed or generalized internal references:

- `author: Search Influence` → `author: Will Scott`
- Removed references to sibling/internal skills: `aeo-geo-content`, `schema-markup`, `seo-audit`
- Removed implementation-specific note: “working implementation ... Node/Playwright site generator”
- Reworded “sibling content skill” and internal dependency language into generic public guidance
- Updated install command to clone `https://github.com/willscott-v2/geo-foundations.git`

## Secret/internal dependency scan

Checks run locally before publishing:

- File inventory confirmed only 5 public files.
- Regex scan for obvious secrets:
  - API keys
  - tokens
  - passwords
  - bearer tokens
  - authorization headers
  - client secrets
  - private keys
  - GitHub/AWS/Slack token patterns
- High-entropy token heuristic scan.
- Targeted internal reference scan for:
  - `Search Influence`
  - `searchinfluence`
  - `aeo-geo-content`
  - `schema-markup`
  - `seo-audit`
  - `working implementation`
  - `Node/Playwright`
  - `proprietary`

Result: no obvious secrets or targeted internal references found in the public copy.

## License

Added `LICENSE` with MIT License text.

## Published repository

- URL: https://github.com/willscott-v2/geo-foundations
- Visibility: PUBLIC
- Default branch: `main`
- Initial commit: `51a648e9c09a455f1758ac79af35de81ec1d73fe`
