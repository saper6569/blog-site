# Personal Engineering Blog

A minimal, professional Jekyll-based blog designed for GitHub Pages.

## Features

- Clean, minimal design focused on readability
- Responsive layout for desktop and mobile
- Blog posts written in Markdown
- Automatic blog post listing
- Project showcase page
- GitHub Pages compatible

## Setup for GitHub Pages

### Option 1: Username Repository (username.github.io)

1. Create a repository named `username.github.io` (replace `username` with your GitHub username)
2. Update `_config.yml`:
   ```yaml
   url: "https://username.github.io"
   baseurl: ""
   ```
3. Push this repository to GitHub
4. GitHub Pages will automatically build and deploy your site

### Option 2: Project Repository

1. Create a repository with any name
2. Update `_config.yml`:
   ```yaml
   url: "https://username.github.io"
   baseurl: "/repository-name"
   ```
3. Go to repository Settings → Pages
4. Select source branch (usually `main` or `master`)
5. GitHub Pages will build and deploy your site

## Local Development

1. Install Ruby and Jekyll:
   ```bash
   gem install bundler jekyll
   ```

2. Install dependencies:
   ```bash
   bundle install
   ```

3. Run the development server:
   ```bash
   bundle exec jekyll serve
   ```

4. Open `http://localhost:4000` in your browser

## Customization

1. **Site Information**: Edit `_config.yml` to update site title, description, author, and social links
2. **Styling**: Modify `assets/css/styles.css` to customize colors, fonts, and layout
3. **Content**: 
   - Edit `index.md`, `about.md`, and `projects.md` for main pages
   - Add blog posts to `_posts/` directory
   - Add images to `assets/images/`
   - Add downloadable files to `assets/files/`

## File Structure

```
.
├── _config.yml          # Jekyll configuration
├── _layouts/            # HTML layouts
│   ├── default.html    # Default page layout
│   └── post.html       # Blog post layout
├── _posts/             # Blog posts (Markdown files)
├── assets/             # Static assets
│   ├── css/           # Stylesheets
│   ├── images/        # Image files
│   └── files/         # Downloadable files
├── index.md           # Homepage
├── about.md           # About page
├── projects.md        # Projects page
└── blog.md           # Blog index page
```

## Adding Blog Posts

Create a new Markdown file in `_posts/` with the naming format:
```
YYYY-MM-DD-post-title.md
```

Include front matter:
```yaml
---
layout: post
title: "Your Post Title"
date: YYYY-MM-DD
tags: [tag1, tag2]
---
```

## License

This blog template is free to use and modify for personal or commercial projects.
