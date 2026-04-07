---
title: "How I Built an AI Resume System That Actually Works"
date: 2026-04-06
categories: [dev, vibe-coding, project]
---

I apply to a lot of jobs. Like, *a lot*. And every single one wants a "tailored resume" — which means I was spending 45 minutes per application tweaking bullet points, reordering sections, and praying the formatting didn't explode when I opened it on a different machine. It felt like being a Ferrari strategist: doing the same thing over and over and expecting a different result.

So I built a system using Claude Skills that generates tailored resumes and cover letters from my actual experience data. You give it a job description, it gives you a one-page `.docx` ready to submit. It took five versions to get right, and every version broke in a new and exciting way.

This is the story of those five versions.

## Version 1: The "just write it" era

The first version was barely a system. I'd paste a job description into Claude and say "write me a resume for this." Claude would generate the entire `.docx` from scratch — layout, content, formatting, everything.

It worked *technically*. The way a paper airplane technically flies.

The problem was consistency. Every resume looked different. Sometimes the margins were off. Sometimes the font sizes changed between sections. Sometimes a bullet point that was beautifully phrased in one resume would come out completely different in the next, even though it was describing the *same experience*. The model was making thousands of micro-decisions every single run — font choices, spacing, section ordering — and there was no guarantee any two runs would agree on anything.

It was like asking someone to repaint your living room from memory every week. Sure, it'll be blue. But *which* blue? Who knows.

## Version 2: Templates and the contamination problem

The fix seemed obvious: stop letting the model make formatting decisions. I built a JavaScript build script that handled all the visual stuff — margins, fonts, spacing, section dividers — and the model's only job was to write a data file. Think of it as a form: the model fills in the blanks, the script builds the document.

This was a *massive* improvement. Every resume now looked identical in structure. Same margins. Same font. Same section order. The model went from "architect, interior designer, and painter" to just "painter." Much better.

Then I added cover letters to the same skill. And everything went sideways.

The cover letters were supposed to sound like a human — warm, specific, conversational. The resumes were supposed to be tight, metric-driven bullet points. But because they lived in the same skill and ran in the same chat, the *styles bled into each other*. Resume bullets started reading like cover letter paragraphs. Cover letters started listing metrics like a spreadsheet.

I call this **contamination**, and it was subtle enough that I didn't notice it at first. I'd generate a resume, think "hmm, this bullet feels wordy," and move on. It wasn't until I compared outputs side-by-side that I saw the pattern: the model was averaging the two writing styles because both sets of rules were in its context window at the same time.

> It's like trying to write a formal email and a text to your best friend in the same sitting. Eventually "Dear Hiring Manager, lol anyway here's my experience" starts feeling normal.

## Version 3: The split

The fix was simple and dramatic: two separate skills. One for resumes, one for cover letters. Different SKILL.md files, different reference documents, different *chats*.

That last part matters. Even with separate skills, if you generate a resume and then a cover letter in the same conversation, the model still has the resume output in its context window. It will unconsciously mirror that style. Separate chats meant a clean context every time.

The quality difference was immediate. Resume bullets got tighter. Cover letters got more human. It was the single biggest improvement across all five versions, and it was purely architectural — I didn't change a single word of the actual content rules.

**Lesson learned:** when an AI's output quality degrades and you can't figure out why, check what *else* is in the context window. The problem might not be your prompt. It might be the ghost of a previous output haunting the current one.

## Version 4: Structural surgery

With the contamination fixed, I could finally see the *content* problems clearly. The resume had a summary section at the top that was eating valuable space. The bullet distribution was uneven — one job would have six bullets and another would have two, which made it look like I only worked at one of them. Projects were missing dates, which made them look like they could be from 2019 or last Tuesday.

I tore the whole content structure apart:

- **Killed the summary section.** On a one-page resume, a summary is a luxury you can't afford. Those 3-4 lines are better spent on another project or another bullet point that actually shows what you did. Every resume coach on the internet will fight me on this. I don't care. The math doesn't lie — real estate on a one-page resume is too expensive for a paragraph that basically says "I am good at things."

- **Balanced the bullets.** Set a target range per job so no single role dominates the page.

- **Added project dates.** A project without a date is a project nobody trusts.

I also rewrote the skill instructions to be more explicit for the model. Turns out, if you write your rules the way you'd explain something to a very literal, very fast coworker, the output gets dramatically better. Vague rules produce vague output. Who knew.

## Version 5: The optimization rabbit hole

This is where I went a little *Ferrari pit-wall overengineering* on it. But honestly? It was worth it.

**The duplication problem:** I had the same rules appearing in three different files — the SKILL.md, the rules reference document, and embedded inside the master resume data files themselves. The model was reading the same instruction three times, sometimes worded slightly differently, and occasionally deciding that version 2 of a rule was more correct than version 1. The fix was aggressive deduplication: one single SKILL.md as the source of truth, master resumes stripped down to pure data (zero rules, zero guidance), and the reference file slimmed to just a verb list.

**The orphan problem:** This one drove me *insane*. A bullet point that's 105 characters long will wrap to two lines — but the second line will only have one or two words sitting there alone, wasting an entire line of vertical space. On a one-page resume, that's a disaster. The fix was a character budget system baked into the skill instructions:

```
~100 characters = 1 line (safe)
~200 characters = 2 full lines (safe)
106-185 characters = DANGER ZONE (orphan territory)
```

The model checks its own bullet lengths before generating the final data file. If a bullet lands in the danger zone, it rewrites — either trim it under 100 or expand it past 185. No more orphans.

**The small stuff that adds up:** 0.4-inch margins instead of 0.5 (fits ~2 extra lines), navy section headers instead of plain black (scannable without looking "designed"), date alignment flush with the right margin, coursework labels that aren't accidentally bold, shorter output filenames so I can actually find them later.

The final SKILL.md is 233 lines. Under the 500-line recommended limit, single source of truth for everything. The master resume files are pure data — three of them, one per career cluster. The build script is locked and never touched by the model.

## The architecture (what actually ships)

Here's what the system looks like now:

```
resume-tailor/
  SKILL.md              ← all rules, workflow, character budgets (233 lines)
  references/
    resume_rules.md     ← verb list + XYZ formula (optional read)
  master_resume_*.md    ← pure experience data, zero rules (×3 clusters)
  resume_data_template.js  ← the blank form the model fills in
  resume_build.js       ← locked build script, model never touches this
```

The model's job is tiny and well-defined: read the job description, pick the right master resume cluster, select the most relevant bullets, tailor the wording, and fill in the data template. That's it. ~80 lines of output. The build script handles everything else.

**Why `.docx` and not PDF?** ATS systems parse `.docx` more reliably. I tested this. A beautifully formatted PDF that a human can read perfectly will sometimes get mangled by an applicant tracking system. The `.docx` is boring but it works everywhere.

**Why separate the model for resumes vs. cover letters?** Resumes are mechanical — select, tailor, format, fit to one page. That's a task for a fast, precise model. Cover letters need depth — they need to connect experiences to a company's mission, sound human, tell a story. That's a task for a model with more reasoning capacity. Different tools for different jobs.

## What I'd tell someone starting from scratch

**Start with the output format, not the content.** I wasted weeks tweaking bullet language when the real problem was that every resume looked different. Lock down the visual format first (build script, template, margins), then worry about what goes inside.

**Separate early.** If your system does two things that have different *styles* — formal vs. conversational, concise vs. narrative — split them into separate skills and separate chats from day one. The contamination problem is real and it's subtle.

**Treat the AI like a very fast intern, not a mind reader.** Explicit, literal instructions with concrete examples beat clever prompting every time. "Write a good bullet point" is useless. "Write a bullet point in XYZ format where X is what you did, Y is how you did it, and Z is the measurable result, keeping it under 100 characters for a single line" actually works.

**Your master data is sacred.** Keep it clean, keep it factual, keep rules *out* of it. The moment you start embedding instructions inside your data files, you've created a maintenance nightmare where updating one rule means finding it in six places.

And if your formatting breaks at 2 AM and you can't figure out why — it's the margins. It's always the margins.

---

*Built iteratively with Claude Skills across five versions and approximately 400 moments of "why does this look wrong" · April 2026*
