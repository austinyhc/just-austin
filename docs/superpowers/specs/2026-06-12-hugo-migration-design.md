# Spec: Migrating personal blog from Quarto to Hugo

This design specification details the migration of the personal blog `just-austin` from Quarto to the Hugo static site generator.

## 1. Overview
The current blog is powered by Quarto and hosted on GitHub Pages. To align with the goal of learning Go and utilizing a Go-based blog framework, we are converting the repository to use Hugo (written in Go) with the minimal PaperMod theme.

## 2. Structural Changes

### 2.1 Files to Remove
The following Quarto files will be deleted:
- `_quarto.yml`
- `about.qmd`
- `index.qmd`
- `styles.css`

### 2.2 Files to Add or Modify
- [hugo.toml](file:///home/achen/workspace/just-austin/hugo.toml): The primary configuration file for the Hugo site.
- `content/about.md`: Migrated page from `about.qmd`.
- `content/posts/`: Main directory for all blog articles.
- `static/CNAME`: Contains `www.sensorai.link` to maintain custom domain routing on GitHub Pages.
- `.github/workflows/hugo-deploy.yml`: GitHub Actions workflow to compile and deploy the site automatically.

## 3. Theme Integration
We will use the **PaperMod** theme.
- Path: `themes/PaperMod` (added as a Git submodule from `https://github.com/adityatelange/hugo-PaperMod.git`).
- Config: Configured within [hugo.toml](file:///home/achen/workspace/just-austin/hugo.toml) to support social profiles (Twitter, LinkedIn, GitHub), menus, and default dark/light mode toggle.

## 4. Content Migration

### 4.1 Article Paths
All posts will be moved from `posts/` to `content/posts/` maintaining their directory names and resource structures (e.g. thumbnails):
- `posts/bytewise-data-alignment/index.qmd` -> `content/posts/bytewise-data-alignment/index.md`
- `posts/bytewise-declaring-a-string/index.qmd` -> `content/posts/bytewise-declaring-a-string/index.md`
- `posts/bytewise-exit/index.qmd` -> `content/posts/bytewise-exit/index.md`
- `posts/bytewise-pragmas/index.qmd` -> `content/posts/bytewise-pragmas/index.md`

### 4.2 Front Matter Mapping
We will retain the YAML front matter of the posts, converting `.qmd` extensions to `.md`. Example structure:
```yaml
---
title: "Data Alignment"
author: "Austin Chen"
date: "2022-12-01"
categories: [byte-wise, c]
toc: true
---
```

### 4.3 Syntax Translation
Quarto callout blocks (e.g., `:::{.callout-note}`) will be converted to standard Markdown blockquotes:
- `:::{.callout-note title="title"}` blocks -> `> 📝 **title**\n>`
- `:::{.callout-tip title="title"}` blocks -> `> 💡 **title**\n>`
- `:::info` / `:::warning` blocks -> `> ℹ️ / ⚠️`

## 5. Deployment Setup
A GitHub Actions workflow will build the site on every push to the `main` branch:
- Installs the latest Hugo (extended version).
- Runs `hugo --minify` to generate the production build into `./public`.
- Deploys the `./public` directory to the `gh-pages` branch using `peaceiris/actions-gh-pages@v4`.
