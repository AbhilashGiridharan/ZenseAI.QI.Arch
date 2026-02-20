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
├── docs/
│   ├── 01-overview.md               # Problem statement, goals, constraints
│   ├── 02-architecture-at-a-glance.md   # Context diagram
│   ├── 03-ai-integration.md         # LLM + RAG pipeline
│   ├── 04-ui-ux.md                  # React / Next.js frontend
│   ├── 05-backend-apis.md           # FastAPI services & APIs
│   ├── 06-data-storage.md           # Data stores & ERD
│   ├── 07-security-compliance.md    # Identity, encryption, LLM safety
│   ├── 08-observability-qe.md       # Tracing, metrics, test agent
│   ├── 09-deployment.md             # CI/CD, IaC, environments
│   ├── 10-nfr.md                    # Non-functional requirements
│   └── 11-appendix.md              # OpenAPI, prompts, config
├── index.md                 # Landing page (links to all docs)
├── architecture.md          # Full E2E architecture (single page)
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

Replace the placeholder hero image at `assets/images/qi-offerings-overview.svg` with your own before deploying.

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
