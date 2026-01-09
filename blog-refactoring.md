# Beckshome Blog Migration Plan
## Statiq.net → Hugo + PaperMod Theme

**Repository:** `github.com/thbst16/dotnet-statiq-beckshome-blog`  
**Target URL:** `blog.beckshome.com`  
**Deployment:** GitHub Pages with GitHub Actions  
**Estimated Time:** 4-6 hours for full migration

---

## Table of Contents

1. [Prerequisites](#phase-0-prerequisites)
2. [Project Setup](#phase-1-project-setup)
3. [Content Migration](#phase-2-content-migration)
4. [Theme Configuration](#phase-3-theme-configuration)
5. [Advanced Features](#phase-4-advanced-features)
6. [GitHub Actions Deployment](#phase-5-github-actions-deployment)
7. [Custom Domain Setup](#phase-6-custom-domain-setup)
8. [Testing & Validation](#phase-7-testing--validation)
9. [Cutover Strategy](#phase-8-cutover-strategy)
10. [Rollback Plan](#rollback-plan)
11. [Post-Migration Tasks](#post-migration-tasks)

---

## Phase 0: Prerequisites

### Install Hugo (Extended Version)

The extended version is required for SCSS processing used by PaperMod.

**macOS (Homebrew):**
```bash
brew install hugo
```

**Windows (Chocolatey):**
```bash
choco install hugo-extended
```

**Windows (Scoop):**
```bash
scoop install hugo-extended
```

**Linux (Snap):**
```bash
snap install hugo --channel=extended
```

**Verify installation:**
```bash
hugo version
# Should show: hugo v0.139.x+extended ...
# Minimum required: v0.112.4
```

### Required Tools
- [ ] Git (you have this)
- [ ] Hugo Extended v0.112.4+
- [ ] Text editor (VS Code recommended)
- [ ] Terminal access

---

## Phase 1: Project Setup

### Option A: In-Place Migration (Recommended)

This approach lets you keep your existing repo and migrate incrementally.

```bash
# Clone your existing repo (fresh copy for safety)
cd ~/projects
git clone https://github.com/thbst16/dotnet-statiq-beckshome-blog.git beckshome-hugo-migration
cd beckshome-hugo-migration

# Create a new branch for the migration
git checkout -b hugo-migration

# Create Hugo project structure alongside existing files
hugo new site . --force
```

### Option B: Fresh Repository

If you prefer a clean start:

```bash
# Create new Hugo site
hugo new site beckshome-blog
cd beckshome-blog
git init
```

### Install PaperMod Theme

```bash
# Add PaperMod as a git submodule (recommended for updates)
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# For future clones, initialize submodules
git submodule update --init --recursive
```

### Resulting Directory Structure

After setup, your repo should look like this:

```
beckshome-blog/
├── archetypes/
│   └── default.md           # Template for new posts
├── assets/                  # (empty, for custom SCSS)
├── content/
│   └── posts/               # Your blog posts go here
├── data/                    # (empty, for data files)
├── layouts/                 # Custom layout overrides
├── static/
│   └── images/              # Static assets (images, etc.)
├── themes/
│   └── PaperMod/            # Theme submodule
├── hugo.yaml                # Main configuration
├── .github/
│   └── workflows/
│       └── hugo.yaml        # GitHub Actions workflow
│
├── [OLD] input/             # Your existing Statiq content
├── [OLD] Program.cs         # Statiq configuration (remove later)
└── [OLD] *.csproj           # .NET project file (remove later)
```

---

## Phase 2: Content Migration

### 2.1 Understand Your Current Frontmatter

Typical Statiq.net frontmatter:
```yaml
---
title: "My Post Title"
published: 2024-01-15
tags: [technology, dotnet]
image: /assets/images/my-image.jpg
description: "Post description for SEO"
---
```

Hugo/PaperMod frontmatter:
```yaml
---
title: "My Post Title"
date: 2024-01-15
tags: ["technology", "dotnet"]
cover:
  image: "/images/my-image.jpg"
  alt: "Image description"
  caption: "Optional caption"
description: "Post description for SEO"
summary: "Short summary for list pages"
draft: false
showToc: true
TocOpen: false
---
```

### 2.2 Migration Script

Create this script to automate frontmatter conversion:

**migrate-posts.sh** (save in repo root):
```bash
#!/bin/bash

# Configuration
SOURCE_DIR="input/posts"      # Adjust to your Statiq posts location
DEST_DIR="content/posts"
IMAGE_SOURCE="input/assets/images"
IMAGE_DEST="static/images"

# Create destination directories
mkdir -p "$DEST_DIR"
mkdir -p "$IMAGE_DEST"

# Copy and transform markdown files
for file in "$SOURCE_DIR"/*.md; do
    if [ -f "$file" ]; then
        filename=$(basename "$file")
        echo "Processing: $filename"
        
        # Copy file
        cp "$file" "$DEST_DIR/$filename"
        
        # Transform frontmatter using sed
        # Change 'published:' to 'date:'
        sed -i '' 's/^published:/date:/' "$DEST_DIR/$filename"
        
        # Change 'image:' to 'cover.image:' format
        # This is a simple transform - complex cases may need manual adjustment
        sed -i '' 's|^image: /assets/images/|cover:\n  image: "/images/|' "$DEST_DIR/$filename"
        
        echo "  -> Copied to $DEST_DIR/$filename"
    fi
done

# Copy images
if [ -d "$IMAGE_SOURCE" ]; then
    cp -r "$IMAGE_SOURCE"/* "$IMAGE_DEST/"
    echo "Images copied to $IMAGE_DEST"
fi

echo "Migration complete! Review files in $DEST_DIR"
```

Make executable and run:
```bash
chmod +x migrate-posts.sh
./migrate-posts.sh
```

### 2.3 Manual Frontmatter Adjustments

For each post, verify and adjust:

| Statiq Field | Hugo/PaperMod Field | Notes |
|--------------|---------------------|-------|
| `published` | `date` | Same format (YYYY-MM-DD) |
| `title` | `title` | No change |
| `tags` | `tags` | Use array format: `["tag1", "tag2"]` |
| `image` | `cover.image` | Nested under `cover:` |
| `description` | `description` | No change |
| (new) | `summary` | Optional: short text for list pages |
| (new) | `draft` | Set to `false` for published posts |
| (new) | `showToc` | Enable table of contents |

### 2.4 Sample Converted Post

**Before (Statiq):**
```yaml
---
title: "Building Microservices with .NET"
published: 2024-03-15
tags: [dotnet, microservices, architecture]
image: /assets/images/microservices-diagram.png
description: "A guide to building microservices"
---

Post content here...
```

**After (Hugo/PaperMod):**
```yaml
---
title: "Building Microservices with .NET"
date: 2024-03-15
tags: ["dotnet", "microservices", "architecture"]
cover:
  image: "/images/microservices-diagram.png"
  alt: "Microservices architecture diagram"
  hidden: false
description: "A guide to building microservices"
summary: "Learn how to build scalable microservices using .NET"
draft: false
showToc: true
TocOpen: false
---

Post content here...
```

### 2.5 Move Static Assets

```bash
# Create static directories
mkdir -p static/images
mkdir -p static/css
mkdir -p static/js

# Copy images from Statiq structure
# Adjust paths based on your actual structure
cp -r input/assets/images/* static/images/

# If you have custom CSS/JS
cp -r input/assets/css/* static/css/ 2>/dev/null || true
cp -r input/assets/js/* static/js/ 2>/dev/null || true
```

---

## Phase 3: Theme Configuration

### 3.1 Create Main Configuration

Create **hugo.yaml** in the repo root:

```yaml
baseURL: "https://blog.beckshome.com/"
title: "Beckshome Blog"
paginate: 10
theme: "PaperMod"

# Build settings
enableRobotsTXT: true
buildDrafts: false
buildFuture: false
buildExpired: false

# Minification
minify:
  disableXML: true
  minifyOutput: true

# Language
languageCode: "en-us"
defaultContentLanguage: "en"

# Taxonomies
taxonomies:
  tag: tags
  category: categories

# Permalinks (maintain URL structure if needed)
permalinks:
  posts: "/:year/:month/:slug/"

# Menu
menu:
  main:
    - identifier: home
      name: Home
      url: /
      weight: 10
    - identifier: posts
      name: Posts
      url: /posts/
      weight: 20
    - identifier: archives
      name: Archives
      url: /archives/
      weight: 30
    - identifier: tags
      name: Tags
      url: /tags/
      weight: 40
    - identifier: search
      name: Search
      url: /search/
      weight: 50

# PaperMod specific parameters
params:
  # Environment
  env: production
  
  # Site metadata
  title: "Beckshome Blog"
  description: "Technology insights and development experiences"
  keywords: ["Blog", "Technology", "Development", "Architecture"]
  author: "Thomas"
  
  # Images for social sharing
  images: ["/images/site-cover.png"]
  
  # Date format
  DateFormat: "January 2, 2006"
  
  # Theme settings
  defaultTheme: auto  # auto, light, dark
  disableThemeToggle: false
  
  # Reading experience
  ShowReadingTime: true
  ShowShareButtons: true
  ShowPostNavLinks: true
  ShowBreadCrumbs: true
  ShowCodeCopyButtons: true
  ShowWordCount: false
  ShowRssButtonInSectionTermList: true
  UseHugoToc: true
  disableSpecial1stPost: false
  disableScrollToTop: false
  
  # Comments (configure if using Disqus, Giscus, etc.)
  comments: false
  
  # Meta
  hideMeta: false
  hideSummary: false
  
  # Table of contents
  showtoc: true
  tocopen: false
  
  # Assets/favicon
  assets:
    favicon: "/favicon.ico"
    favicon16x16: "/favicon-16x16.png"
    favicon32x32: "/favicon-32x32.png"
    apple_touch_icon: "/apple-touch-icon.png"
  
  # Label (top-left branding)
  label:
    text: "Beckshome"
    icon: "/apple-touch-icon.png"
    iconHeight: 35
  
  # Home page mode: regular, profile, or homeInfoParams
  homeInfoParams:
    Title: "Welcome to Beckshome Blog"
    Content: >
      Technology insights, development experiences, and architectural patterns.
      Exploring enterprise software, cloud architecture, and modern development practices.
  
  # Social icons (shown on home page)
  socialIcons:
    - name: github
      url: "https://github.com/thbst16"
    - name: linkedin
      url: "https://linkedin.com/in/yourprofile"
    - name: rss
      url: "/index.xml"
  
  # Cover image defaults
  cover:
    hidden: false
    hiddenInList: false
    hiddenInSingle: false
  
  # Edit post link (optional - links to GitHub)
  editPost:
    URL: "https://github.com/thbst16/dotnet-statiq-beckshome-blog/tree/main/content"
    Text: "Suggest Changes"
    appendFilePath: true
  
  # Search (Fuse.js)
  fuseOpts:
    isCaseSensitive: false
    shouldSort: true
    location: 0
    distance: 1000
    threshold: 0.4
    minMatchCharLength: 0
    keys: ["title", "permalink", "summary", "content"]

# Privacy settings
privacy:
  disqus:
    disable: true
  googleAnalytics:
    disable: true
  instagram:
    disable: true
  twitter:
    disable: true
  vimeo:
    disable: true
  youtube:
    disable: true

# Markup settings
markup:
  goldmark:
    renderer:
      unsafe: true  # Allow HTML in markdown
  highlight:
    noClasses: false
    codeFences: true
    guessSyntax: true
    lineNos: false
    style: dracula

# Output formats
outputs:
  home:
    - HTML
    - RSS
    - JSON  # Required for search
```

### 3.2 Create Required Pages

**Archives Page** - `content/archives.md`:
```yaml
---
title: "Archives"
layout: "archives"
url: "/archives/"
summary: "All posts"
---
```

**Search Page** - `content/search.md`:
```yaml
---
title: "Search"
layout: "search"
placeholder: "Search posts..."
---
```

**About Page** (optional) - `content/about.md`:
```yaml
---
title: "About"
url: "/about/"
summary: "About this blog"
hidemeta: true
---

Your about content here...
```

### 3.3 Custom Archetype

Update **archetypes/default.md** for new posts:

```yaml
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ .Date }}
draft: true
tags: []
cover:
  image: ""
  alt: ""
  caption: ""
description: ""
summary: ""
showToc: true
TocOpen: false
---

```

Create new posts with:
```bash
hugo new posts/my-new-post.md
```

---

## Phase 4: Advanced Features

### 4.1 Enable Search

Search is already configured in hugo.yaml. The JSON output enables Fuse.js search.

Verify `outputs` section includes JSON:
```yaml
outputs:
  home:
    - HTML
    - RSS
    - JSON
```

### 4.2 Comments (Optional)

**Option A: Giscus (GitHub Discussions)**

Add to `params` in hugo.yaml:
```yaml
params:
  comments: true
  giscus:
    repo: "thbst16/dotnet-statiq-beckshome-blog"
    repoID: "YOUR_REPO_ID"  # Get from giscus.app
    category: "Comments"
    categoryID: "YOUR_CATEGORY_ID"  # Get from giscus.app
    mapping: "pathname"
    reactionsEnabled: "1"
    emitMetadata: "0"
    inputPosition: "top"
    theme: "preferred_color_scheme"
    lang: "en"
```

Create `layouts/partials/comments.html`:
```html
{{- if site.Params.giscus }}
<script src="https://giscus.app/client.js"
    data-repo="{{ site.Params.giscus.repo }}"
    data-repo-id="{{ site.Params.giscus.repoID }}"
    data-category="{{ site.Params.giscus.category }}"
    data-category-id="{{ site.Params.giscus.categoryID }}"
    data-mapping="{{ site.Params.giscus.mapping }}"
    data-reactions-enabled="{{ site.Params.giscus.reactionsEnabled }}"
    data-emit-metadata="{{ site.Params.giscus.emitMetadata }}"
    data-input-position="{{ site.Params.giscus.inputPosition }}"
    data-theme="{{ site.Params.giscus.theme }}"
    data-lang="{{ site.Params.giscus.lang }}"
    crossorigin="anonymous"
    async>
</script>
{{- end }}
```

**Option B: Disqus**

Add to `params`:
```yaml
params:
  comments: true
  disqusShortname: "your-disqus-shortname"
```

### 4.3 Google Analytics (Optional)

Add to hugo.yaml root level:
```yaml
services:
  googleAnalytics:
    ID: "G-XXXXXXXXXX"
```

And update params:
```yaml
params:
  env: production  # Required for analytics to work
```

### 4.4 Custom CSS

Create `assets/css/extended/custom.css`:
```css
/* Custom styles that override PaperMod */

/* Example: Adjust code block font size */
.post-content pre code {
  font-size: 14px;
}

/* Example: Custom link color */
.post-content a {
  color: #0066cc;
}
```

---

## Phase 5: GitHub Actions Deployment

### 5.1 Create Workflow File

Create `.github/workflows/hugo.yaml`:

```yaml
name: Deploy Hugo site to GitHub Pages

on:
  # Runs on pushes to main branch
  push:
    branches:
      - main
      - hugo-migration  # Remove after migration complete
  
  # Allows manual trigger
  workflow_dispatch:

# Sets permissions for GITHUB_TOKEN
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default shell
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.139.0
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: ${{ env.HUGO_VERSION }}
          extended: true

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5

      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Detroit
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### 5.2 Enable GitHub Pages

1. Go to repository **Settings** → **Pages**
2. Under "Build and deployment":
   - Source: **GitHub Actions**
3. The workflow will automatically deploy on push

---

## Phase 6: Custom Domain Setup

### 6.1 DNS Configuration

Add this record to your DNS provider (where beckshome.com is managed):

| Type | Name | Value | TTL |
|------|------|-------|-----|
| CNAME | blog | thbst16.github.io. | 3600 |

### 6.2 GitHub Configuration

**Option A: Via GitHub UI**
1. Go to repository **Settings** → **Pages**
2. Under "Custom domain", enter: `blog.beckshome.com`
3. Click **Save**
4. Wait for DNS check to pass
5. Enable **Enforce HTTPS**

**Option B: Via CNAME File**

Create `static/CNAME`:
```
blog.beckshome.com
```

This file will be copied to `public/` during build and tells GitHub Pages your custom domain.

### 6.3 Verify Setup

```bash
# Check DNS propagation (may take up to 1 hour)
dig blog.beckshome.com +noall +answer

# Expected result:
# blog.beckshome.com.  3600  IN  CNAME  thbst16.github.io.
```

---

## Phase 7: Testing & Validation

### 7.1 Local Testing

```bash
# Build and serve locally with drafts
hugo server -D

# Build and serve (production mode)
hugo server --environment production

# Build only (output to ./public)
hugo --gc --minify
```

Visit http://localhost:1313 to preview.

### 7.2 Validation Checklist

**Content:**
- [ ] All posts migrated and accessible
- [ ] Images display correctly
- [ ] Code blocks render with syntax highlighting
- [ ] Tags and categories work
- [ ] Archives page shows all posts
- [ ] Search functionality works

**Navigation:**
- [ ] Menu links work
- [ ] Post pagination works
- [ ] Previous/Next post links work
- [ ] Breadcrumbs display correctly

**Theme:**
- [ ] Dark/Light mode toggle works
- [ ] Mobile responsive layout
- [ ] Table of contents generates
- [ ] Social share buttons work

**Technical:**
- [ ] RSS feed generates (`/index.xml`)
- [ ] Sitemap generates (`/sitemap.xml`)
- [ ] robots.txt exists
- [ ] No broken links (use a link checker)
- [ ] Page speed acceptable (Lighthouse)

### 7.3 Link Validation

```bash
# Install link checker
npm install -g broken-link-checker

# Check local server
hugo server &
blc http://localhost:1313 -ro

# Or use htmltest
# brew install htmltest
hugo --gc --minify
htmltest
```

---

## Phase 8: Cutover Strategy

### 8.1 Pre-Cutover Checklist

- [ ] All content migrated and tested
- [ ] GitHub Actions workflow passing
- [ ] DNS CNAME record ready (but not active if using same domain)
- [ ] Backup of current live site
- [ ] Rollback plan documented

### 8.2 Cutover Steps

**If migrating in the same repository:**

```bash
# 1. Merge hugo-migration branch to main
git checkout main
git merge hugo-migration

# 2. Remove old Statiq files (or move to archive branch)
git rm -r input/
git rm Program.cs
git rm *.csproj
git rm *.sln
git rm -r .config/ 2>/dev/null || true

# 3. Commit cleanup
git add .
git commit -m "Complete Hugo migration, remove Statiq files"

# 4. Push to trigger deployment
git push origin main
```

**If using a new repository:**

1. Update DNS CNAME to point to new repo's GitHub Pages
2. Wait for DNS propagation (up to 1 hour)
3. Verify HTTPS works

### 8.3 Post-Cutover Verification

- [ ] Site loads at blog.beckshome.com
- [ ] HTTPS working (green lock)
- [ ] All pages accessible
- [ ] No console errors
- [ ] Search works
- [ ] RSS feed valid

---

## Rollback Plan

### If Something Goes Wrong

**Option 1: Revert Git Commit**
```bash
# Find the last working commit
git log --oneline

# Revert to that commit
git revert HEAD
git push origin main
```

**Option 2: Restore from Branch**
```bash
# If you kept an archive branch
git checkout main
git reset --hard archive/statiq-backup
git push origin main --force
```

**Option 3: DNS Rollback**
- Point DNS back to previous hosting
- Or temporarily disable custom domain in GitHub Pages

---

## Post-Migration Tasks

### Immediate (Day 1)

- [ ] Verify all pages indexed by search engines
- [ ] Submit updated sitemap to Google Search Console
- [ ] Test from multiple devices/browsers
- [ ] Monitor GitHub Actions for any failures

### Short-term (Week 1)

- [ ] Set up Google Analytics (if desired)
- [ ] Configure comments system
- [ ] Add any missing redirects for old URLs
- [ ] Update any external links pointing to the blog

### Ongoing

- [ ] Update PaperMod theme periodically:
  ```bash
  cd themes/PaperMod
  git pull
  cd ../..
  git add themes/PaperMod
  git commit -m "Update PaperMod theme"
  ```

---

## Quick Reference Commands

```bash
# Create new post
hugo new posts/my-new-post.md

# Local development server
hugo server -D

# Build for production
hugo --gc --minify

# Update theme
git submodule update --remote themes/PaperMod

# Check Hugo version
hugo version

# List all content
hugo list all

# Check for problems
hugo --templateMetrics --templateMetricsHints
```

---

## Useful Resources

- [Hugo Documentation](https://gohugo.io/documentation/)
- [PaperMod Wiki](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [PaperMod Demo Site](https://adityatelange.github.io/hugo-PaperMod/)
- [PaperMod Example Site Source](https://github.com/adityatelange/hugo-PaperMod/tree/exampleSite)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)

---

## Support

If you encounter issues:

1. Check Hugo's verbose output: `hugo server -D --verbose`
2. Check GitHub Actions logs in the repository's Actions tab
3. Refer to [Hugo Discourse Forum](https://discourse.gohugo.io/)
4. Search [PaperMod Issues](https://github.com/adityatelange/hugo-PaperMod/issues)

---

*Migration plan created: January 9, 2026*