# QI Offerings – Architecture Documentation

GitHub Pages site for the **QI Offerings** end-to-end architecture (LLM + RAG).

## Structure

```
Architecture/
├── _config.yml              # Jekyll configuration
├── _layouts/
│   └── default.html         # Page layout with Mermaid support
├── assets/
│   ├── css/
│   │   └── style.css        # Custom styles
│   └── images/
│       └── qi-offerings-overview.png   # ← Add your hero image here
├── index.md                 # Landing page
├── architecture.md          # Full E2E architecture document
├── Gemfile                  # Ruby dependencies
├── .gitignore
└── README.md                # This file
```

## Local Development

```bash
# Install dependencies (requires Ruby ≥ 3.0)
bundle install

# Serve locally
bundle exec jekyll serve --livereload

# Open http://localhost:4000/Architecture/
```

## Deploy to GitHub Pages

1. Push this folder to a GitHub repository.
2. Go to **Settings → Pages → Source** and select the branch/folder.
3. The site will be available at `https://<org>.github.io/<repo>/Architecture/`.

## Hero Image

Drop your hero image into `assets/images/qi-offerings-overview.png` before deploying.

## Key Sections

- Overview & capability map
- Architecture context diagram (Mermaid)
- AI Integration (LLM + RAG pipeline)
- UI/UX (React / Next.js)
- Backend & APIs (FastAPI)
- Data & Storage (ERD)
- Security & Compliance
- Observability & QE
- Deployment & CI/CD
- Non-Functional Requirements
- Appendix (OpenAPI, prompts, config)
