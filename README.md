# tejalgoyal2.github.io

Personal blog — dev projects, vibe coding with Claude, cybersecurity, and things I'm learning along the way.

Live at [tejalgoyal2.github.io](https://tejalgoyal2.github.io)

## Stack

- **Jekyll** — static site generation (Markdown in, HTML out)
- **GitHub Pages** — hosting (free, deploys on push)
- **GitHub Actions** — build pipeline (automatic on push to `main`)
- **Custom HTML/CSS layouts** — no remote themes, full control

## Writing a new post

1. Copy `_templates/post-template.md` to `_posts/`
2. Rename it: `YYYY-MM-DD-your-post-slug.md`
3. Fill in the front matter (title, date, categories)
4. Write the post in Markdown
5. Commit and push — GitHub Actions handles the rest

### Categories

Pick 1-3 per post:

| Tag | Covers |
|-----|--------|
| `dev` | Code, tools, engineering |
| `macos` | macOS-specific projects |
| `vibe-coding` | AI-assisted development |
| `security` | Cybersecurity, IAM, IR |
| `meta` | Blog updates, personal |
| `life` | Non-technical |
| `github` | GitHub, git, open source |
| `project` | Project showcase / launch |

## Voice and style guide

These guidelines exist so that posts stay consistent whether written manually or with AI assistance.

### Tone

- **First person, always.** "I built this" not "the system was built."
- **Conversational but technical.** Write like you're explaining it to a smart friend over coffee — not a README, not a lecture.
- **Honest about failures.** What broke is more interesting than what worked. Don't sand down the rough edges.
- **Humor lives in the observations, not in forced jokes.** Dry wit, relatable frustration, occasional absurdity. If something was annoying, say it was annoying.

### Formatting

- Use `*italics*` for emphasis, internal monologue, or the moment something clicks.
- Use `**bold**` sparingly — for key terms on first introduction or genuine "this is the important part" moments.
- Blockquotes (`>`) for asides, analogies, or the thing you wish someone had told you.
- Keep paragraphs to 2-4 sentences. Walls of text lose people.
- Code blocks with language hints for syntax highlighting.
- Section headers (`##`) should be scannable — someone should get the story arc just from the headers.

### Structure

Posts generally follow this arc:

1. **Hook** — what you built and why, or what went wrong. The first paragraph should make someone want to keep reading.
2. **The journey** — what you tried, what broke, what you learned. This is the bulk of the post.
3. **The result** — what it looks like now, what you'd do differently.
4. **Sign-off** — one line. Tools used, date, or a teaser for the next post.

### Rules

- No emoji in body text unless the post is explicitly casual/personal.
- No security tool names in public posts (keep vendor references generic).
- No personal data (contact info, GPA, real resume content).
- Link to things — repos, courses, reference material. The blog should be a web, not an island.
- If using AI to draft a post, paste the template and this style guide. The output should be indistinguishable from a manually written post.

## Local development

```bash
bundle install
bundle exec jekyll serve
# → http://localhost:4000
```

## License

Content is personal. Code (layouts, CSS) is MIT.
