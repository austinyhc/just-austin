# Hugo Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert the current Quarto blog to a Hugo-powered blog using the PaperMod theme and prepare it for GitHub Pages deployment.

**Architecture:** Initialize Hugo in the root directory, configure the PaperMod theme, migrate existing posts with front matter mapping and callout syntax fixes, clean up old Quarto files, and set up a GitHub Actions CD workflow.

**Tech Stack:** Hugo (Extended), Go, PaperMod Theme, GitHub Actions, Git.

---

### Task 1: Initialize Hugo Site & Theme Submodule

**Files:**
- Create: `hugo.toml` (initialized automatically)
- Modify: `.gitmodules` (created/updated by submodule command)

- [ ] **Step 1: Initialize new Hugo site structure**

Run this command in the workspace root to initialize Hugo without overriding existing files:
```bash
hugo new site . --force
```

- [ ] **Step 2: Install PaperMod theme as a Git submodule**

Run:
```bash
git submodule add --force https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

- [ ] **Step 3: Commit initial initialization**

Run:
```bash
git add hugo.toml themes/ .gitmodules
git commit -m ":wrench: chore(hugo): initialize Hugo site and add PaperMod theme submodule"
```

---

### Task 2: Configure Hugo Site (`hugo.toml`)

**Files:**
- Modify: `hugo.toml`

- [ ] **Step 1: Write configuration**

Overwrite [hugo.toml](file:///home/achen/workspace/just-austin/hugo.toml) with the following content:
```toml
baseURL = 'https://www.sensorai.link/'
languageCode = 'en-us'
title = 'AUSTIN CAN HELP'
theme = 'PaperMod'

[params]
  defaultTheme = "auto"
  showShareButtons = true
  showCodeCopyButtons = true
  
  [params.profileMode]
    enabled = false

  [params.homeInfoParams]
    Title = "AUSTIN CAN HELP"
    Content = "I am Austin; A System Software Engineer by passion. Data Science Practitioner at heart. Avid reader. Essentialist. Nerd. Newbie Rustacean. Virgo. ♍"

  [[params.socialIcons]]
    name = "github"
    url = "https://github.com/austinyhc"
  [[params.socialIcons]]
    name = "x"
    url = "https://twitter.com/austinyht"
  [[params.socialIcons]]
    name = "linkedin"
    url = "https://www.linkedin.com/in/austinyhc/"

[menu]
  [[menu.main]]
    name = "Posts"
    url = "posts/"
    weight = 10
  [[menu.main]]
    name = "About"
    url = "about/"
    weight = 20
```

- [ ] **Step 2: Build verification**

Run:
```bash
hugo
```
Expected: Success with `public/` directory created.

- [ ] **Step 3: Commit configuration**

Run:
```bash
git add hugo.toml
git commit -m ":wrench: chore(hugo): configure hugo.toml settings and menus"
```

---

### Task 3: Migrate and Clean Up Posts

**Files:**
- Create: `content/posts/bytewise-data-alignment/index.md`
- Create: `content/posts/bytewise-declaring-a-string/index.md`
- Create: `content/posts/bytewise-exit/index.md`
- Create: `content/posts/bytewise-pragmas/index.md`
- Delete: `posts/` folder and contents.

- [ ] **Step 1: Copy posts content and assets**

Run:
```bash
mkdir -p content/posts
cp -r posts/* content/posts/
```

- [ ] **Step 2: Rename all `.qmd` index files to `.md`**

Run:
```bash
find content/posts -name "index.qmd" -exec sh -c 'mv "$0" "${0%.qmd}.md"' {} \;
```

- [ ] **Step 3: Clean up Quarto callout blocks in posts**

Modify the files:
- [content/posts/bytewise-data-alignment/index.md](file:///home/achen/workspace/just-austin/content/posts/bytewise-data-alignment/index.md):
  Replace lines 68-70:
  ```markdown
  :::info
  :pencil2: A struct is always aligned to the largest type’s alignment requirements
  :::
  ```
  with:
  ```markdown
  > 📝 **Alignment Rule**
  > A struct is always aligned to the largest type’s alignment requirements
  ```

- [content/posts/bytewise-declaring-a-string/index.md](file:///home/achen/workspace/just-austin/content/posts/bytewise-declaring-a-string/index.md):
  Replace lines 17-20:
  ```markdown
  :::{.callout-tip title="Hint" collapse=true}
  1. The execution complains **`SEGFAULT`**.
  2. `GDB` tells the error happens at **line 7**.
  :::
  ```
  with:
  ```markdown
  > 💡 **Hint**
  > 1. The execution complains **`SEGFAULT`**.
  > 2. `GDB` tells the error happens at **line 7**.
  ```
  Replace lines 99-102:
  ```markdown
  :::{.callout-note title="Make use of the `SEGFAULT`"}
  We can make use of this feature to store a data that we don't want it to be mutated in a **read-only** segment. When someone try to modify the content, it raises `SEGFAULT`.
  :::
  ```
  with:
  ```markdown
  > 📝 **Make use of the SEGFAULT**
  > We can make use of this feature to store data that we don't want to be mutated in a **read-only** segment. When someone tries to modify the content, it raises `SEGFAULT`.
  ```

- [content/posts/bytewise-exit/index.md](file:///home/achen/workspace/just-austin/content/posts/bytewise-exit/index.md):
  Replace lines 61-63:
  ```markdown
  :::{.callout-note title="What Valgrind Defines Memory Leak"}
  **Valgrind** uses the stricter definition...
  :::
  ```
  with:
  ```markdown
  > 📝 **What Valgrind Defines as Memory Leak**
  > **Valgrind** uses the stricter definition...
  ```
  Replace lines 69-72:
  ```markdown
  :::{.callout-warning}
  Just to add, in the vast majority of cases the OS will free the memory...
  :::
  ```
  with:
  ```markdown
  > ⚠️ **Warning**
  > Just to add, in the vast majority of cases the OS will free the memory...
  ```

- [content/posts/bytewise-pragmas/index.md](file:///home/achen/workspace/just-austin/content/posts/bytewise-pragmas/index.md):
  No callouts to replace.

- [ ] **Step 4: Delete the old `posts` directory**

Run:
```bash
rm -rf posts
```

- [ ] **Step 5: Verify build works with posts**

Run:
```bash
hugo
```
Expected: Success with post pages compiled under `public/posts/`.

- [ ] **Step 6: Commit post migration**

Run:
```bash
git add content/posts
git rm -r posts
git commit -m ":truck: refactor(posts): migrate blog posts from Quarto to Hugo content directory"
```

---

### Task 4: Migrate About Page and Static Folder

**Files:**
- Create: `content/about.md`
- Create: `static/CNAME`
- Delete: `about.qmd`, `index.qmd`, `styles.css`, `CNAME`

- [ ] **Step 1: Create about page**

Create [content/about.md](file:///home/achen/workspace/just-austin/content/about.md) with this content:
```markdown
---
title: "About"
layout: "about"
---

I am Austin; A System Software Engineer by passion. Data Science Practitioner at heart. Avid reader. Essentialist. Nerd. Newbie Rustacean. Virgo. ♍
```

- [ ] **Step 2: Set up static folder and CNAME**

Run:
```bash
mkdir -p static
cp CNAME static/CNAME
```

- [ ] **Step 3: Copy profile image**

Run:
```bash
cp profile.jpg static/profile.jpg
```

- [ ] **Step 4: Delete root files**

Run:
```bash
rm about.qmd index.qmd styles.css CNAME profile.jpg
```

- [ ] **Step 5: Verify overall build works**

Run:
```bash
hugo
```
Expected: Site builds successfully with `public/about/` and `public/CNAME`.

- [ ] **Step 6: Commit page migration and cleanup**

Run:
```bash
git add content/about.md static/
git rm about.qmd index.qmd styles.css CNAME profile.jpg
git commit -m ":truck: refactor(pages): migrate about page and assets to Hugo static folder"
```

---

### Task 5: Add GitHub Actions Deployment Workflow

**Files:**
- Create: `.github/workflows/hugo-deploy.yml`

- [ ] **Step 1: Write deployment workflow**

Create [.github/workflows/hugo-deploy.yml](file:///home/achen/workspace/just-austin/.github/workflows/hugo-deploy.yml) with this content:
```yaml
name: Deploy Hugo Site to GitHub Pages

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true  # Fetch PaperMod submodule
          fetch-depth: 0    # Fetch all history for Git Info

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build site
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
          cname: www.sensorai.link
```

- [ ] **Step 2: Commit deployment workflow**

Run:
```bash
git add .github/workflows/hugo-deploy.yml
git commit -m ":ferris_wheel: ci(github): add Hugo deployment GitHub Actions workflow"
```
