# bill18.github.io

Personal GitHub Pages site for Wenhua Wang (Quant Finance Developer), built with Hugo 0.74.3.

## Key facts

- **No build/test/lint/CI infrastructure.** Hugo source files are not tracked — generated HTML/CSS/JS are committed directly to `master`. Editing HTML files and committing is the deploy step.
- **`.gitignore`** only excludes `_site/`.
- **`dist/`** contains prebuilt CSS/JS assets — do not edit.
- **Images** live at root `images/` (shared) or `post/<slug>/images/` (per-post).
- **All pages are noindex** — `<meta NAME="ROBOTS" CONTENT="NOINDEX, NOFOLLOW">` on every page.
- **Root static files:** `404.html`, `robots.txt`, `sitemap.xml`, `index.xml` (RSS/Atom feed per section).

## Key pages

| Path | Content |
|---|---|
| `/` | Home page |
| `/about/` | Bio |
| `/contact/` | Contact form |
| `/post/` | Projects listing (7 projects) |
| `/post/ql-risk-lab/` | QuantLib AI Risk Lab |
| `/post/tax-pipeline/` | OCR-LLM Tax Pipeline |
| `/post/distributex/` | DistributeX article |
| `/post/ai-margin-service/` | AI Margin Service |
| `/post/serverless-finance/` | Serverless Computing |
| `/post/project-1/` | Project 1 |
| `/post/project-2/` | Project 2 |
