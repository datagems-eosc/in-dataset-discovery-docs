# in-dataset-discovery Docs

Documentation site for the in-dataset-discovery project, built with MkDocs and Material.

## Overview

This repository contains the documentation source (docs/) and a GitHub Actions workflow (.github/workflows/deploy-docs-on-demand.yml) that builds and deploys the site. The workflow can deploy versioned docs to `in-dataset-discovery` repo (gh-pages) using mike.

## Quickstart (local)

Prerequisites: Python 3.x, mkdocs, mike.

Install deps:
```
pip install mkdocs mkdocs-material mike pymdown-extensions mkdocstrings
```

Serve locally:
```
mkdocs serve
```

Build:
```
mkdocs build --site-dir site
```

## Deployment

To deploy the documentation site, use `mike` to publish to GitHub Pages.


## Contributing

Edit files under `docs/` and open a PR. Use the workflow to trigger on-demand deployments for preview or production updates.
