# bill18.github.io

Personal GitHub Pages site for Wenhua Wang (Quant Finance Developer), built with Hugo 0.74.3.

## Structure

- **Built output is committed** — only `_site/` is gitignored. The repo contains generated HTML/CSS/JS directly on `master` (standard GitHub Pages workflow).
- **No Hugo source files** in this repo — no `content/`, `layouts/`, `config.toml`, or theme files are tracked.
- `distributex/` — standalone article ("Scaling Legacy Quant Engines: Design & Code with DistributeX WSPP") with multi-language demos (Python, C++, Java, C#). Its own `README.md` and `post.md` serve as entrypoints. Published at `/post/distributex/`.

## Key pages

| Path | Content |
|---|---|
| `/` | Home page |
| `/about/` | Bio |
| `/post/` | Projects listing (4 projects) |
| `/contact/` | Contact form |
| `/post/ql-risk-lab/` | QuantLib AI Risk Lab |
| `/post/tax-pipeline/` | OCR-LLM Tax Pipeline |
| `/post/distributex/` | DistributeX WSPP article |

## distributex/ demos

All demo scripts assume they are run from the `distributex_wspp` root (not this repo root). Setup requires cmake, g++, python3, and related dependencies:

```bash
bash distributex/demos/demo_setup.sh        # system deps check
python3 distributex/demos/python/run_demo.py # Python demo
```

See `distributex/demos/README.md` (375 lines) for full walkthrough.

## Notes

- No build, test, lint, or CI infrastructure exists in this repo.
- `.gitignore` only excludes `_site/`.
