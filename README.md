# beckshome-blog

![Uptime Robot ratio (7 days)](https://img.shields.io/uptimerobot/ratio/7/m792586859-9634d4aa6352cf540b960a54?logo=http)

A Hugo-powered blog hosted on GitHub Pages at https://thbst16.github.io/beckshome-blog/

This is the fifth iteration of my blog's hosting engine, having migrated from Das Blog → WordPress → Statiq.net → Hugo.

## Technology Stack

- **Static Site Generator**: [Hugo](https://gohugo.io/) (Extended v0.154.3+)
- **Theme**: [hugo-theme-cleanwhite](https://github.com/zhaohuabing/hugo-theme-cleanwhite) - Full-width hero images with clean typography
- **Search**: [Pagefind](https://pagefind.app/) - Client-side static search
- **Hosting**: GitHub Pages
- **CI/CD**: GitHub Actions

## Features

- Full-width hero images on homepage and posts
- Client-side search powered by Pagefind
- Tag-based content organization
- Archive page for chronological browsing
- RSS feed
- Mobile-responsive design
- Dark mode support (theme-provided)

## Local Development

### Prerequisites

- Hugo Extended (v0.154.3 or later)
- Git (for theme submodule)
- Node.js (for Pagefind, optional for local dev)

### Setup

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/thbst16/beckshome-blog.git
cd beckshome-blog

# Or if already cloned, initialize submodules
git submodule update --init --recursive

# Run local server
hugo server -D
```

The site will be available at http://localhost:1313/beckshome-blog/

### Building for Production

```bash
hugo --gc --minify
```

### Running Pagefind Locally

```bash
npx pagefind --site ./public
```

## Project Structure

```
beckshome-blog/
├── .github/workflows/    # GitHub Actions deployment
├── archetypes/           # Hugo content templates
├── content/
│   ├── post/             # Blog posts
│   ├── about/            # About page
│   ├── archive/          # Archive page
│   └── search/           # Search page
├── layouts/              # Theme overrides
│   ├── _default/         # Default layouts
│   ├── partials/         # Partial templates
│   └── taxonomy/         # Taxonomy layouts
├── static/
│   ├── css/              # Custom CSS
│   ├── images/           # Site images (hero, etc.)
│   └── img/              # Favicon and icons
├── themes/               # Hugo themes (submodule)
└── hugo.yaml             # Site configuration
```

## Deployment

The site automatically deploys to GitHub Pages when changes are pushed to the `main` branch. The GitHub Actions workflow:

1. Builds the Hugo site
2. Runs Pagefind to generate search indexes
3. Deploys to GitHub Pages

## Customizations

This site includes several layout overrides to ensure proper URL handling on GitHub Pages:

- `layouts/partials/nav.html` - Navigation with correct base URL handling
- `layouts/partials/sidebar.html` - Sidebar with fixed tag links
- `layouts/_default/search.html` - Pagefind search integration
- `static/css/custom.css` - Enhanced text contrast on hero images

## License

Content is copyright Thomas Beck. Theme is licensed under MIT.
