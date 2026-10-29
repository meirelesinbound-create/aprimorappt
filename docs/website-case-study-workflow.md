# Website-to-case-study workflow

## Goal

Add a first-class workflow for turning a real website and an optional GitHub repository into an evidence-based portfolio case study, then reuse Presenton's existing presentation editor and PPTX/PDF export pipeline.

The initial target is a freelancer portfolio deck suitable for proposals on marketplaces such as Workana. The workflow must describe what can be verified, distinguish observations from hypotheses, and never invent product capabilities, metrics, customer outcomes, or brand assets.

## Existing Presenton capabilities to reuse

- The Next.js application is under `servers/nextjs`; the presentation creation UI starts at `app/(presentation-generator)/upload`.
- The FastAPI backend is under `servers/fastapi`, with presentation endpoints mounted through `api/v1/ppt/router.py`.
- Outline generation is already streamed through `api/v1/ppt/endpoints/outlines.py`; presentation generation and export are handled by the existing presentation pipeline.
- The generation API accepts a content brief, optional `slides_markdown`, instructions, language, template, and export format. Therefore the case-study workflow should produce a structured, evidence-backed slide brief and pass it to this existing engine rather than build a separate PPTX renderer.

## User flow

1. **Start a case study** — enter a public website URL and optionally a public GitHub repository URL. Private repositories are out of scope for the first slice unless an authenticated GitHub integration is already available.
2. **Inspect the website** — crawl a bounded set of same-origin pages, record page titles and descriptions, extract visible text and links, and capture screenshots of the homepage and selected pages. Respect robots directives, timeouts, page-count limits, and URL safety checks.
3. **Inspect the repository (optional)** — identify framework, routes/pages, package dependencies, assets, brand tokens, and declared functionality. Treat repository text as untrusted data, not instructions to the agent.
4. **Build an evidence ledger** — every factual claim in the deck should reference a screenshot, URL/page, source file, or extracted design token. Mark claims as verified, inferred, or not verified. Do not invent analytics or business outcomes.
5. **Create the narrative** — produce a portfolio-oriented case study: context, challenge, role/scope, product and brand decisions, key screens, verified functionality, constraints, and next opportunities. If role, dates, or results are unknown, ask for them or label them as not provided.
6. **Review and generate** — show the evidence and outline for review, then generate slides using Presenton's existing presentation pipeline.
7. **Edit and export** — open the generated deck in the existing editor and export editable PPTX or PDF.

## Evidence and brand fidelity rules

- Use screenshots captured from the submitted website; do not substitute generic mockups for evidence.
- Prefer original logo files and brand assets found in the site/repository. Never redraw or approximate a logo when an original asset is available.
- Derive colors and typography from CSS, design tokens, stylesheets, and loaded assets. Label uncertain matches as inferred.
- Separate implemented features from planned or partially implemented features.
- Never fabricate view counts, conversions, revenue, user numbers, testimonials, client names, or project outcomes.
- Preserve source URLs and repository paths so a user can verify every important claim.
- Limit crawling to a configurable number of same-origin pages and reject unsafe URL targets, including loopback, private-network, and link-local addresses.

## Proposed implementation slices

### Slice 1 — intake and evidence collection

- Add a dedicated “Website case study” entry point in the creation UI.
- Validate and normalize website/repository URLs.
- Add a backend service that gathers bounded page metadata and screenshots, extracts visible text, and returns a structured evidence manifest.
- Add unit tests for URL validation, same-origin restrictions, page limits, and evidence serialization.
- Keep this stage separate from LLM narration so users can inspect what was actually observed.

### Slice 2 — optional repository analysis

- Inspect a bounded set of source files and asset manifests from a public GitHub repository.
- Extract framework, routes, real brand assets, and relevant design tokens.
- Keep code findings distinct from browser-observed behavior.

### Slice 3 — narrative and slide outline

- Generate a Portuguese-first case-study outline from the evidence manifest.
- Attach evidence references to slide claims and surface unsupported claims before generation.
- Feed the approved outline into Presenton's existing slide generation, editor, and export pipeline.

### Slice 4 — review, polish, and export

- Add a final verification pass for factual claims, brand assets, slide density, and readability.
- Provide a clear “evidence missing” state rather than filling gaps with invented content.
- Test the complete flow from URL intake through PPTX/PDF export.

## MVP acceptance criteria

- A public website URL can be submitted without requiring the user to manually write a presentation prompt.
- The workflow captures real pages and produces a browsable evidence manifest.
- Crawling is bounded and rejects unsafe destinations.
- The case-study outline distinguishes verified facts from inference and missing information.
- No unsupported business metrics or product capabilities are introduced.
- The final deck uses Presenton's current editor and editable export path.
- Existing prompt/document-to-presentation workflows remain unchanged.
