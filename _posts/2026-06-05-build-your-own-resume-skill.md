---
title: "How to Build a Resume Skill in Claude That Tailors Itself to Every Job"
date: 2026-06-05
categories: [dev, vibe-coding]
---

A few people have asked me some version of the same question: how do you make your resumes? The honest answer is that I do not make them one at a time anymore. I built a skill that does it, and I point it at a job description.

The follow-up is usually whether I can just share that skill. I cannot, and the reason is worth explaining, because it is also the reason this post exists. My skill is full of my resume. Every real bullet I have, my contact details, the specifics of what I have actually done. Handing it over would mean handing over all of that, and you would still be left with a resume that describes me instead of you.

So here is the method instead. Build your own, with your own content. It is not complicated, and the structure is the part that matters.

## What a skill actually is

A skill in Claude is a folder. Claude reads the instructions inside it and can run the code inside it. Mine has three parts and nothing more:

```text
my-resume-skill/
  SKILL.md          # the instructions Claude follows
  references/
    master.md       # every real bullet you have
  scripts/
    build.js        # generates the document and fits it to the page
```

The current steps for adding a skill to Claude live in [Anthropic's documentation](https://docs.claude.com), and that is the source of truth, so I am going to keep this post about the parts that are actually mine to explain: what goes in each file, and why.

## Step 1: Write your master resume first

Before any automation, write one document that contains everything true about you. Every role, every project, every bullet you could plausibly want to use, written out properly. This is your source of truth, and the most important file in the whole system.

The skill never invents anything. It only selects from this.

If you apply to more than one kind of role, keep more than one master. I keep separate ones for the different directions I apply in, because a bullet that is perfect for one kind of job is just noise on another. Either way, the raw material is real and already written. Tailoring becomes a selection problem, not a writing-from-scratch problem.

> The single most important rule of the whole system: the model selects and trims. It does not write new accomplishments. If it is not in your master, it does not go on the resume.

## Step 2: Write the instructions

The SKILL.md file is where you tell Claude how to behave. Mine covers a handful of things, all in plain language:

- **Select from the master, do not rewrite.** Pick the bullets that match the job. Leave the rest.
- **Keep a length budget.** One page. A fixed number of bullets, with a few extras held in reserve for when there is room.
- **Mirror the posting's language.** If the job says "incident response" and you have done incident response, use their words, because plenty of resumes are read by software before a person ever sees them. But do not stuff keywords. A human reads it next, and keyword soup is obvious and embarrassing.
- **Do not inflate.** This one matters far more than it sounds. Tell the model explicitly that confidence is not inflation: contributed is not authored, helped build is not built, supported is not led. I learned why that rule has to be written down the hard way, which is a story I told [in the other post](/posts/2026/06/03/resume-builder-slight-overflow/).

## Step 3: Make it fit the page by itself

This is the part most people skip, and it is the part that separates a resume system from a resume-shaped text generator.

A one-page resume has to actually be one page. Full, but not spilling onto a second. The obvious approach is to guess length from how many characters you have. Do not do this. Character count is a bad proxy, because whether a bullet takes two lines or three depends on the exact font, the margins, and where the words happen to wrap.

Instead, render the resume to a real document and measure it. My build script generates the document, converts it to a PDF with a headless office renderer, and measures how much of the page the content fills. That gives a real number: a little under one page, exactly one page, or spilled over.

Once you can measure, you can correct automatically. If it is under-full, pad by pulling a held-back bullet from your reserve list. If it is over-full, trim by cutting the weakest bullet, starting with the oldest role and never touching your current one. Loop until it lands on one full page. The point is that this happens inside a single run, without you refereeing every pass.

<!-- MEDIA SUGGESTION: a simple flow diagram - select bullets from master, build the document, measure page fill, then pad if under or trim if over, looping back until it is exactly one page. It makes the auto-fit loop concrete. -->

To do this you need a small build environment: a way to run the script, and a headless renderer to turn the document into something measurable. I use Node for the first and LibreOffice for the second, but the principle does not depend on the tools. Measure with whatever you have. Look at the real page, not a guess.

## Step 4: The rules that actually matter

Most of what makes the output good is a short list of rules I arrived at by getting them wrong first:

- **Bullets are one to two lines. Never three.** Find the character count where your layout wraps onto a third line and treat it as a hard ceiling. For my font and margins it sits around 250 characters. Yours will be different, so measure it once and write it down.
- **One page, full or slightly over, then trim by hand if you have to.** Empty space at the bottom looks unfinished. A second page with three lonely lines on it looks worse.
- **Select, do not rewrite.** I have said this. I am saying it again because it is the rule people break first.
- **Mirror the posting, do not stuff it.**
- **Never upgrade your own credit.**

## Step 5: Use it

Once the folder is built and added to Claude, the day-to-day is genuinely simple. Paste in a job description. The skill reads it, selects the bullets from your master that fit, mirrors the relevant language, builds the document, and fits it to the page. You read the result, fix anything that feels off, and send it.

The first build of the skill takes real effort. Every build after that takes about a minute. That trade is the entire reason the thing exists.

## On sharing, and why yours has to be yours

Back to where this started. I cannot give you mine, because mine is my resume, and you would not actually want it, because a tailored resume is only useful when it is built from true things about the person sending it. The value was never in my files.

It is in the structure: a real master document, a clear set of instructions, and a build that measures the page instead of guessing at it. Build that with your own history and you will have something better than anything I could hand you. A resume system that tells the truth about you, on one page, in under a minute.

If you want the messy version of how I got here, with the page-and-a-half disasters and the evening I lost to a single comma, that is [over here](/posts/2026/06/03/resume-builder-slight-overflow/). The original build is documented in [this post](/posts/2026/04/03/ai-resume-system/) and [this companion one](/posts/2026/04/05/build-your-own-ai-resume-pipeline/) if you want the back catalogue.

*Find me on [LinkedIn](https://linkedin.com/in/tejalgoyal).*
