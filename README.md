# Journey Notes Blog

A minimal, branded blog built with Jekyll and GitHub Pages.

## Setup Instructions

1. Create a new GitHub repository (e.g., `journey-notes`)
2. Copy all files from this `blog/` directory to your new repo
3. Edit `_config.yml` and replace `YOURUSER` with your GitHub username
4. Commit and push:
   ```bash
   git add .
   git commit -m "Initial blog setup"
   git push origin main
   ```
5. Enable GitHub Pages:
   - Go to your repo → Settings → Pages
   - Source: "Deploy from a branch"
   - Branch: `main`, folder: `/ (root)`
   - Save

Your site will be live at `https://YOURUSER.github.io/` in a few minutes.

## Adding New Posts

Create a new file in `_posts/` with the format:
- `YYYY-MM-DD-your-post-title.md`

Example:
```markdown
---
title: "Your Post Title"
date: 2025-12-23
---

Your content here in Markdown.
```

## Customization

- Edit `_config.yml` for site title, description, etc.
- Edit `assets/css/main.css` to adjust colors/styling
- Edit `_layouts/default.html` or `_layouts/post.html` to change structure


