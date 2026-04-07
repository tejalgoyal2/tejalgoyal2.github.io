# Blog Writing Guide

This document describes the voice, tone, and style for Tejal's Blog. Use it when writing posts manually or when prompting any AI to generate blog content that sounds like it belongs on this site.

Copy-paste this guide (or the relevant sections) into your AI prompt along with the post template from `_templates/post-template.md`.

---

## Voice

The blog reads like someone explaining a project to a smart friend over coffee. First person, always. Technical but never lecturing. Honest about what broke — because what broke is more interesting than what worked.

The writer has a sense of humor but doesn't force it. Jokes come from observations, frustration, and absurdity — not from puns or "haha right??" energy. Think dry wit with occasional rants. Ferrari F1 references are fair game (and encouraged — the team provides endless material).

There's an underlying personality: someone who overthinks things, knows they overthink things, and writes about the overthinking anyway. Someone who genuinely loves building stuff but also sometimes wonders if opening a bakery would be simpler. The humor is self-aware without being self-deprecating.

## Tone rules

- **First person, always.** "I built this" not "the system was developed." Never passive voice for the writer's own actions.
- **Conversational.** Contractions, sentence fragments, trailing thoughts... all fine. If it sounds natural spoken aloud, it works on the page.
- **Technically honest.** Don't sand down the rough edges. If something was a bad decision, say it was a bad decision. If a workaround is ugly, call it ugly.
- **Humor lives in the observations.** Not in setups and punchlines. The funny part is usually the gap between expectation and reality. ("It worked *technically*. The way a paper airplane technically flies.")
- **Frustration is personality.** It's okay to say something drove you insane. It's okay to rant briefly. It's what makes the writing feel like a person, not a blog factory.

## Formatting that adds character

This blog uses formatting as voice, not just structure:

- ***Italics*** for emphasis, internal monologue, and the moment something clicks. "And then it hit me — *oh, that's why it was breaking.*"
- **Bold** for key terms on first introduction, or genuine "this is the important part" emphasis. Use sparingly — if everything is bold, nothing is.
- **Ellipsis (...)** for trailing thoughts, pauses, and "well... that didn't work." Creates rhythm and mimics natural speech patterns.
- **Em dashes (—)** for asides and parenthetical thoughts that are too important to parenthesize.
- **Blockquotes** for analogies, "the thing I wish someone had told me," and the occasional aside that deserves its own visual space.
- **Short paragraphs.** 2-4 sentences max. Walls of text lose people. If a paragraph is longer than 4 sentences, it's probably two paragraphs.
- **One-line paragraphs for emphasis.** Sometimes a thought deserves its own line.

Like this.

## Post structure

Every post follows this general arc:

1. **Hook** (1-2 paragraphs) — What you built and why, or what went wrong. This shows up as the excerpt on the homepage, so it needs to make someone want to keep reading. An analogy, a relatable frustration, or a surprising result all work.

2. **The journey** (bulk of the post) — What you tried, what broke, what you learned. Use `##` headers for major sections. The headers should be scannable — someone should get the story arc just from skimming them.

3. **The result** — What it looks like now. Architecture, numbers, outcome. This can be shorter than you think.

4. **Sign-off** (1-3 lines) — Tools used, date, LinkedIn link, or a teaser for the next post. Keep it light. A little humor here is good.

## Hard rules

These apply to every post, no exceptions:

- **No emoji** in body text. Formatting carries the emotion instead.
- **No security/vendor tool names** in public posts. Keep references generic ("the EDR platform" not a specific product). Claude and GitHub are fine to mention by name.
- **No personal data.** No contact info, no GPA, no real resume bullets in public posts.
- **No corporate speak.** Never "leverage," "utilize," "synergize," or "ecosystem." Say "use." Say "build." Say "broke."
- **No AI-sounding language.** Never "delve," "tapestry," "it's important to note," "in conclusion." If a sentence sounds like it came from a template, rewrite it.
- **Code blocks always have a language hint** for syntax highlighting.
- **Link to things.** Repos, courses, companion posts, reference material. The blog should be a web, not an island.
- **Always end with a way to reach you.** LinkedIn, at minimum.

## Example passages that capture the voice

**Good — has personality:**
> I tore the whole content structure apart. Killed the summary section. On a one-page resume, a summary is a luxury you cannot afford. Every resume coach on the internet will fight me on this. I don't care. The math doesn't lie.

**Good — frustration as character:**
> This one drove me *actually insane*. Not figuratively. I was losing sleep over single words on resume lines.

**Good — humor from observation:**
> It's like trying to write a formal email and a text to your best friend in the same sitting. Eventually "Dear Hiring Manager, lol anyway here's my experience" starts feeling normal. And that's... that's a problem.

**Good — trailing thoughts:**
> Sure, it'll be blue. But *which* blue? ...who knows. Not the AI. Not me. Nobody.

**Bad — too formal:**
> The system architecture was redesigned to separate resume generation from cover letter generation, resulting in improved output quality.

**Bad — forced humor:**
> LOL so then the margins broke AGAIN haha can you believe it?? Classic margins am I right 😂

**Bad — AI template voice:**
> It's important to note that the separation of concerns principle played a crucial role in improving the overall system architecture.

## Using this guide with AI

When prompting an AI to write a blog post:

1. Paste this entire guide as context
2. Paste the post template from `_templates/post-template.md`
3. Provide the topic, key details, and any specific points to cover
4. Ask it to write in first person matching the voice described above
5. Review the output and check against the hard rules
6. Read it aloud — if any sentence sounds like it was written by someone else, rewrite it

The goal: the output should be indistinguishable from a manually written post. If you can tell an AI wrote it... the AI didn't follow this guide closely enough.
