---
title: "First Post: Setting Up This Blog with GitHub Skills"
date: 2026-04-01
categories: [meta, github]
---

This is my first blog post, and it exists because I was procrastinating on something else.

I was poking around GitHub looking for a way to set up a simple blog — something I could write in Markdown, push to a repo, and have it show up as an actual website. No frameworks, no hosting bills, no CMS. No twelve-step deployment pipeline that makes you question your career choices. I found the [GitHub Pages](https://skills.github.com/courses/github-pages/) course in [GitHub Skills](https://skills.github.com/) and figured I'd try it.

The course walks you through creating a Jekyll-powered blog hosted on GitHub Pages. It's one of those "follow along and push commits" tutorials where GitHub Actions run checks after each step and unlock the next one. Took maybe 30 minutes. By the end of it I had a working blog at `tejalgoyal2.github.io/skills-github-pages/` with a single test post.

That was the proof of concept. This blog you're reading now is the real version — same idea (Jekyll + GitHub Pages), but with a custom theme instead of the default Minima template, and living at the root of `tejalgoyal2.github.io` instead of tucked inside a project repo.

## Why bother with a blog

Honestly... I keep building things and then forgetting how they work three weeks later. Writing about what I build forces me to actually understand it — you can't explain something you only half-know. And if future-me is going to forget anyway, at least there'll be a record. Something I can look back on in six months and either think "oh right, that's how that worked" or cringe at how wrong I was.

Both outcomes are useful. One is more fun than the other.

I'm also working through [Anthropic's 101 course](https://www.anthropic.com/) and building side projects to learn. The plan is to post about projects as I build them, share things I learn the hard way, and occasionally write about whatever else comes to mind. If this becomes a habit, great. If it doesn't... well, at least one post exists. You're reading it.

## The stack

Nothing fancy:

- **Jekyll** for static site generation (Markdown in, HTML out)
- **GitHub Pages** for hosting (free, deploys on push)
- **GitHub Actions** for the build pipeline
- **Custom HTML/CSS layouts** instead of a remote theme (more control, fewer dependencies)

The whole blog is [open source](https://github.com/tejalgoyal2/tejalgoyal2.github.io) if you want to see how it's put together.

---

*First post done. Next up: a write-up about building an AI resume system that broke in five increasingly creative ways.*
