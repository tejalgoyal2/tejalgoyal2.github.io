# tejalgoyal2.github.io — Tejal's Blog

This is the source for **[tejalgoyal2.github.io](https://tejalgoyal2.github.io)** — my personal blog built with [Jekyll](https://jekyllrb.com/) and hosted on GitHub Pages.

## Writing a new post

1. Create a file in `_posts/` named `YYYY-MM-DD-your-post-title.md`.
2. Add front matter at the top:
   ```yaml
   ---
   title: "Your Post Title"
   date: YYYY-MM-DD
   categories: [tag1, tag2]
   ---
   ```
3. Write your post in Markdown below the front matter.
4. Commit and push to `main` — GitHub Actions will build and deploy automatically.

## Running locally

```bash
bundle install
bundle exec jekyll serve
```

Then open `http://localhost:4000`.

## Structure

```
_posts/      ← blog posts (Markdown)
about.md     ← About page
_config.yml  ← site title, theme, author, etc.
Gemfile      ← Ruby dependencies
.github/
  workflows/
    pages.yml  ← GitHub Actions build & deploy workflow
```

## Migrated content

Posts were consolidated here from two earlier project-pages blogs:
- `tejalgoyal2/skills-github-pages` (previously at `/skills-github-pages/`)
- `tejalgoyal2/OnTop` docs (previously at `/OnTop/`)