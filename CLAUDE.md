# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal blog built with [Eleventy](https://www.11ty.dev/) (version 3.x), a static site generator. The site is deployed to AWS S3 with CloudFront for CDN, costing less than $1/month. Blog posts are written in Markdown and transformed into HTML using Nunjucks templates.

## Development Commands

### Build and Serve
- `npm run build` or `npx eleventy` - Build the site to `_site/` directory
- `npm run serve` or `npm start` - Build and serve locally with hot reload (recommended for development)
- `npm run watch` - Build automatically when templates change (no server)
- `npm run debug` - Run Eleventy with debug output

### Node Version
- Requires Node.js 22.x (see `.nvmrc`)

## Architecture

### Content Structure
- **Posts**: Markdown files in `/posts/` directory
  - Each post has YAML frontmatter with: `title`, `date`, `tags`, `layout`
  - Posts automatically get tagged with "posts" via `/posts/posts.json`
  - Use `layout: layouts/post.njk` for blog posts
  - Posts support the `link_to` plugin for internal linking: `{% link_to "post-slug" %}`

### Template System
- **Template Engine**: Nunjucks (`.njk` files)
- **Layouts**: Located in `/_includes/layouts/`
  - `base.njk` - Base HTML structure, header, footer
  - `post.njk` - Blog post layout (extends base.njk), includes created/modified dates, tags, prev/next navigation
  - `home.njk` - Home page layout
- **Includes**: Located in `/_includes/`
  - `postslist.njk` - Reusable component for listing posts

### Site Configuration
- **Eleventy Config**: `.eleventy.js` in root directory
- **Site Metadata**: `/_data/metadata.json` contains:
  - Site title, URL, description
  - Feed configuration
  - Author information
- **Directory Structure** (defined in `.eleventy.js`):
  - Input: `.` (root)
  - Includes: `_includes`
  - Data: `_data`
  - Output: `_site`

### Key Eleventy Features Used
1. **Plugins**:
   - `@11ty/eleventy-plugin-rss` - RSS feed generation
   - `@11ty/eleventy-plugin-syntaxhighlight` - Code syntax highlighting
   - `@11ty/eleventy-navigation` - Site navigation
   - `eleventy-plugin-git-commit-date` - Shows last modified dates from Git
   - `eleventy-plugin-link_to` - Internal post linking

2. **Custom Filters** (defined in `.eleventy.js`):
   - `readableDate` - Format dates as "dd LLL yyyy"
   - `htmlDateString` - Format dates for HTML datetime attributes
   - `head` - Get first n elements of array
   - `min` - Return smallest number
   - `filterTagList` - Filter out system tags (all, nav, post, posts)

3. **Collections**:
   - `tagList` - Automatically generated from all post tags

4. **Markdown**:
   - Uses `markdown-it` with HTML support, auto-linking, and anchor plugin
   - Preprocessed with Nunjucks (`markdownTemplateEngine: "njk"`)

### Static Assets
- `/css/` - Stylesheets (copied to output via passthrough)
- `/img/` - Images (copied to output via passthrough)

### Deployment
- **CI/CD**: GitHub Actions workflow (`.github/workflows/main.yml`)
  - Triggers on push to `master` branch
  - Builds site with Eleventy
  - Uploads `_site/` to S3 bucket `juancri-web`
  - Invalidates CloudFront cache (distribution ID: E37VQK4W6O1P5V)
- **Netlify**: Alternative deployment configured in `netlify.toml` (publishes `_site/` directory)

## Working with Blog Posts

### Creating a New Post
1. Create a new `.md` file in `/posts/` directory
2. Add frontmatter:
```markdown
---
title: Your Post Title
date: YYYY-MM-DD
tags:
  - english (or spanish)
  - topic1
  - topic2
layout: layouts/post.njk
---

Your content here...
```

### Linking Between Posts
Use the `link_to` plugin instead of hardcoding URLs:
```
[link text]({% link_to "post-file-slug" %})
```

### Tags
- Common tags: `english`, `spanish`, `software`, `development`, `cloud`, `aws`, `ai`
- Tags are automatically collected and displayed on tag pages
- System tags (all, nav, post, posts) are filtered from display

## Important Notes

- The site uses bilingual content (English and Spanish)
- Posts show both creation date and last modified date (from Git history)
- Each post includes a link to its Git commit history on GitHub
- The base layout includes navigation generated from Eleventy Navigation plugin
- 404 page is served via BrowserSync middleware during local development
