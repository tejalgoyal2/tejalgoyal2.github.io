---
title: "How I Built an AI Resume System That Actually Works"
date: 2026-04-02
categories: [dev, vibe-coding, project]
---

I apply to a lot of jobs. Like... *a lot*. And every single one wants a "tailored resume" — which, if you've ever tried doing that manually, means spending 45 minutes per application tweaking bullet points, reordering sections, and praying to whatever god handles `.docx` formatting that the margins don't implode when you open it on a different machine.

It felt like being on the Ferrari pit wall. Same strategy. Same execution. Same result. *Except somehow worse every time.*

So I built a system using Claude Skills that generates tailored resumes and cover letters from my actual experience data. You feed it a job description, it spits out a one-page `.docx` ready to submit. It took five versions to get right, and every single version broke in a new and creatively painful way.

This is the story of those five versions. If you're here for the how-to, there's a [companion guide](/posts/2026/04/04/build-your-own-ai-resume-pipeline/) for that. This one is just the mess.

## Version 1: The "just wing it" era

The first version was barely a system. I'd paste a job description into Claude and say "write me a resume for this." Claude would generate the entire `.docx` from scratch — layout, content, formatting, everything in one shot.

It worked *technically*. The way a paper airplane technically flies. The way Ferrari *technically* has a race strategy.

The problem was consistency. Every resume looked different. Sometimes the margins were off. Sometimes the font sizes changed between sections. Sometimes a bullet point that was *beautifully* phrased in one resume would come out completely different in the next, even though it was describing the same experience. The model was making thousands of micro-decisions every run — font choices, spacing, section ordering — and there was no guarantee any two runs would agree on literally anything.

It was like asking someone to repaint your living room from memory every week. Sure, it'll be blue. But *which* blue?

...who knows. Not the AI. Not me. Nobody.

## Version 2: Templates and the contamination disaster

The fix seemed obvious: stop letting the model make formatting decisions. I built a JavaScript build script that handled all the visual stuff — margins, fonts, spacing, section dividers — and the model's only job was to write a data file. Think of it as a form: the model fills in the blanks, the script builds the document.

This was a *massive* improvement. Every resume looked identical in structure. Same margins. Same font. Same section order. The model went from "architect, interior designer, and painter" to just "painter."

Great. Wonderful. I was feeling like a genius.

Then I added cover letters to the same skill.

And everything... went sideways.

The cover letters were supposed to sound like a human — warm, specific, conversational. The resumes were supposed to be tight, metric-driven bullet points. But because they lived in the same skill and ran in the same chat, the *styles bled into each other*. Resume bullets started reading like cover letter paragraphs. Cover letters started listing metrics like a spreadsheet having an identity crisis.

I call this **contamination**, and it was the sneakiest bug I've ever dealt with. Not a crash. Not an error message. Just... vibes slowly deteriorating.

I'd generate a resume, squint at it, think "hmm, this bullet feels wordy..." and move on. It wasn't until I compared outputs side-by-side that I saw the pattern: the model was *averaging* the two writing styles because both sets of rules were in its context window at the same time.

> It's like trying to write a formal email and a text to your best friend in the same sitting. Eventually "Dear Hiring Manager, lol anyway here's my experience" starts feeling normal. And that's... that's a problem.

## Version 3: The split that changed everything

The fix was simple and dramatic: two separate skills. One for resumes, one for cover letters. Different instruction files, different reference documents, different *chats*.

That last part — different chats — matters more than you'd think. Even with separate skills, if you generate a resume and then a cover letter in the same conversation, the model still has the resume output in its context window. It will unconsciously mirror that style. Like how you start picking up the accent of whoever you've been talking to for an hour.

Separate chats meant a clean context every time.

The quality difference was *immediate*. Resume bullets got tighter. Cover letters got more human. This was the single biggest improvement across all five versions, and it was purely architectural. I didn't change a single word of the actual content rules. Not one.

**The lesson:** when an AI's output quality degrades and you can't figure out why... check what *else* is in the context window. The problem might not be your prompt. It might be the ghost of a previous output haunting the current one.

Spooky? A little. True? Absolutely.

## Version 4: Structural surgery

With the contamination fixed, I could finally see the *content* problems clearly. Like cleaning a windshield and realizing the road has potholes.

The resume had a summary section at the top that was eating valuable space. The bullet distribution was all over the place — one job would have six bullets and another would have two, which made it look like I only worked at one of them. Projects were missing dates, which made them look like they could be from 2019 or last Tuesday.

I tore the whole content structure apart:

**Killed the summary section.** On a one-page resume, a summary is a luxury you cannot afford. Those 3-4 lines are better spent on another project or another bullet point that actually shows what you *did*. Every resume coach on the internet will fight me on this. I don't care. The math doesn't lie — real estate on a one-page resume is too expensive for a paragraph that basically says "I am good at things."

...we know. That's why there's a resume. Moving on.

**Balanced the bullets.** Set a target range per job so no single role dominates the page. Because nothing says "I peaked at my first internship" like giving it eight bullets while your most recent role gets two.

**Added project dates.** A project without a date is a project nobody trusts. Is this from last month or from when you were 14? The recruiter doesn't know, and they're not going to ask.

I also rewrote the skill instructions to be way more explicit. Turns out, if you write your rules the way you'd explain something to a very literal, very fast coworker who takes everything at face value — the output gets dramatically better. Vague rules produce vague output. *Shocking*, I know.

## Version 5: The optimization rabbit hole

This is where I went full Ferrari-pit-wall-overengineering on it. Overthinking every detail. Micro-optimizing spacing by fractions of an inch. Staring at character counts at 1 AM.

But honestly? Unlike Ferrari's strategies... this one actually paid off.

**The duplication problem.** I had the same rules appearing in three different files — the skill instructions, the rules reference document, and embedded inside the master resume data files themselves. The model was reading the same instruction three times, sometimes worded slightly differently, and occasionally deciding that version 2 of a rule was more correct than version 1. Classic "too many cooks" situation. The fix was aggressive: one single instruction file as the source of truth, master resumes stripped to pure data (zero rules, zero guidance), and the reference file slimmed to just a verb list.

**The orphan problem.** This one drove me *actually insane*. Not figuratively. I was losing sleep over single words on resume lines.

Here's the deal: a bullet point that's 105 characters long will wrap to two lines — but the second line will only have one or two words sitting there alone, wasting an entire line of vertical space. On a one-page resume, that orphan word is eating space that could hold actual content.

The fix was a character budget system baked into the instructions:

```
~100 characters = 1 line (safe)
~200 characters = 2 full lines (safe)
106-185 characters = DANGER ZONE (orphan territory)
```

The model checks its own bullet lengths before generating the final file. If a bullet lands in the danger zone, it rewrites — either trim it under 100 or expand it past 185. No orphans allowed.

Is this overkill? Maybe. Did it work? *Yes.* Orphan lines dropped from roughly 1-in-3 resumes to basically zero.

**The small stuff that adds up:** 0.4-inch margins instead of 0.5 (fits ~2 extra lines — don't underestimate this), navy section headers instead of plain black (scannable without looking "designed"), date alignment flush with the right margin, coursework labels that aren't accidentally bold because someone forgot to set a flag in the build script, and shorter output filenames so I can actually *find* them on my machine later.

The final skill instruction file is 233 lines. The master resume files are pure data — three of them, one per career cluster. The build script is locked and the model never touches it.

## The architecture (what actually ships)

Here's the final system:

```
resume-tailor/
  SKILL.md              <- all rules, workflow, character budgets (233 lines)
  references/
    resume_rules.md     <- verb list + bullet formula reference
  master_resume_*.md    <- pure experience data, zero rules (x3 clusters)
  resume_data_template.js  <- the blank form the model fills in
  resume_build.js       <- locked build script, model never touches this
```

The model's job is tiny and well-defined: read the job description, pick the right master resume cluster, select the most relevant bullets, tailor the wording, and fill in the data template. That's it. ~80 lines of output. The build script handles everything visual.

**Why `.docx` and not PDF?** Applicant tracking systems parse `.docx` more reliably. I tested this. A beautifully formatted PDF that a human can read perfectly will sometimes get mangled by an ATS. The `.docx` is boring but it passes through every system I've thrown it at.

**Why split the model choices?** Resumes are mechanical — select, tailor, format, fit to one page. Cover letters need depth — they need to connect experiences to a company's mission, sound human, tell a story. I use a faster, more precise model for resumes and a more capable one for cover letters. Different tools for different jobs. Like how Ferrari uses... actually, let's not go there. They don't use their tools well.

## What I'd tell someone starting from scratch

**Start with the output format, not the content.** I wasted weeks tweaking bullet language when the real problem was that every resume looked different. Lock down the visual format first (build script, template, margins), *then* worry about what goes inside. Foundation before furniture.

**Separate early.** If your system does two things with different *styles* — formal vs. conversational, concise vs. narrative — split them into separate skills and separate chats from day one. The contamination problem is real, it's subtle, and it will waste your time if you discover it late.

**Treat the AI like a very fast intern, not a mind reader.** Explicit, literal instructions with concrete examples beat clever prompting every single time. "Write a good bullet point" is useless. "Write a bullet point where you state what you did, how you did it, and the measurable result, keeping it under 100 characters" actually works.

**Your master data is sacred.** Keep it clean, keep it factual, keep rules *out* of it. The moment you start embedding instructions inside your data files, you've created a maintenance nightmare where updating one rule means finding it in six places. Ask me how I know.

...actually, don't. I've already told you. It was version 5. It was 1 AM. I was counting characters.

And if your formatting breaks at 2 AM and you can't figure out why — it's the margins. *It's always the margins.*

---

If you want to build something like this yourself, the step-by-step companion guide is [here](/posts/2026/04/04/build-your-own-ai-resume-pipeline/). And if you just want to talk about it — or if you have a job for me (please) — find me on [LinkedIn](https://linkedin.com/in/tejalgoyal). I'll be there, refreshing the page, pretending I'm not.

**Update:** I thought version 5 was the end. It was not. The optimization rabbit hole goes [deeper than I expected](/posts/2026/04/07/optimizing-ai-resume-system/).

*Built iteratively with Claude Skills across five versions and approximately 400 moments of "why does this look wrong" · April 2026*
