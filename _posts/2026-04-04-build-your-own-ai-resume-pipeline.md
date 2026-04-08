---
title: "Build Your Own AI Resume Pipeline with Claude Skills"
date: 2026-04-04
categories: [dev, vibe-coding, project]
---

This is the companion post to [How I Built an AI Resume System That Actually Works](/posts/2026/04/02/ai-resume-system/). That one tells the story — the five versions, the failures, the 1 AM character counting. This one tells you *how to actually build it*.

The system generates a tailored one-page resume from a master data file and a job description, outputting a clean `.docx` every time. It runs entirely inside Claude Skills — no external APIs, no local scripts, no dependencies beyond what the skill environment provides.

If something below seems like overkill... it probably felt that way when I built it too. But every "overkill" decision exists because a lazier version broke in production. So here we are.

## The architecture

```
resume-tailor/
  SKILL.md                  <- single source of truth for all rules
  references/
    resume_rules.md         <- verb list, bullet formula reference
  master_resume_cyber.md    <- experience data (cluster 1)
  master_resume_data.md     <- experience data (cluster 2)
  master_resume_dev.md      <- experience data (cluster 3)
  resume_data_template.js   <- blank form the model fills in
  resume_build.js           <- locked build script (never model-edited)
```

Three types of files, three jobs:

1. **SKILL.md** — the brain. Every rule, every guardrail, every workflow step lives here and *only* here.
2. **Master resumes** — the memory. Pure experience data. Zero instructions. Zero formatting guidance. Just facts.
3. **Build script + template** — the hands. Handles all visual formatting. The model writes data, the script builds the document.

The critical principle: **the model only writes a data file (~80 lines). It never touches formatting.** This is non-negotiable. The moment you let the model adjust margins to "make things fit," you've lost control of your output. It's like letting the intern redesign the slide deck at 4:59 PM — technically they *can*, but should they?

No. The answer is no.

## Step 1: Build the locked formatting script

Start here. Not with rules. Not with bullet points. Here.

I wasted weeks fine-tuning content when the real problem was inconsistent formatting. Get the visual foundation locked *first*, then worry about what goes inside. Foundation before furniture. Always.

The build script is a Node.js file that uses the `docx` library to construct a Word document from a JavaScript data file. Here's the skeleton:

```javascript
// resume_build.js — LOCKED. The model never modifies this. Ever.
const { Document, Packer, Paragraph, TextRun } = require("docx");
const fs = require("fs");

// Import the data file the model writes
const {
  NAME, PHONE, EMAIL, LINKEDIN, GITHUB, PORTFOLIO,
  EDUCATION, EXPERIENCE, PROJECTS, SKILLS, CERTIFICATIONS
} = require("./resume_data.js");

// ─── CONSTANTS (tweak once, then leave alone) ───
const FONT = "Calibri";
const FONT_SIZE_BODY = 20;        // 10pt
const FONT_SIZE_NAME = 26;        // 13pt
const MARGIN = 576;               // 0.4 inches in twips
const HEADER_COLOR = "1B3A5C";    // dark navy for section titles

// ... builder functions for each section ...
// buildHeader(), buildEducation(), buildExperience(), etc.

const doc = new Document({
  sections: [{
    properties: {
      page: {
        size: { width: 12240, height: 15840 },  // US Letter
        margin: { top: MARGIN, right: MARGIN, bottom: MARGIN, left: MARGIN },
      },
    },
    children: [
      ...buildHeader(),
      ...buildEducation(),
      ...buildExperience(),
      ...buildProjects(),
      ...buildSkillsAndCerts(),
    ],
  }],
});

Packer.toBuffer(doc).then(buf => {
  fs.writeFileSync(`Company-Role-YourInitials.docx`, buf);
});
```

A few decisions that matter here:

**0.4-inch margins.** Standard is 0.5 inches. Dropping to 0.4 gives you roughly 2 extra lines of vertical space and 3 more characters per line width. Below 0.4 starts looking cramped. This is the sweet spot — I tested it obsessively.

**`.docx` over PDF.** ATS systems parse Word documents more reliably. I go into why [in the previous post](/posts/2026/04/02/ai-resume-system/) — short version: a beautifully formatted PDF will sometimes get mangled by an ATS. The `.docx` is boring, but it survives every system I've tested.

**A locked script.** If the model *can* modify formatting, it *will*. Especially when trying to fit content onto one page — it'll shrink fonts, squeeze margins, remove spacing. Locking the script forces the model to solve the one-page problem through *content selection*, which produces better resumes. Constraints breed creativity... or in this case, better bullet points.

## Step 2: Create the data template

This is the blank form. The model copies it, fills in the values, and saves it as `resume_data.js`. The build script imports it.

```javascript
// resume_data_template.js
module.exports = {
  COMPANY: "Target Company",
  ROLE: "Target Role",

  NAME: "Your Name",
  PHONE: "555-000-0000",
  EMAIL: "you@email.com",
  LINKEDIN: "linkedin.com/in/you",
  GITHUB: "github.com/you",
  PORTFOLIO: "yoursite.com",

  EDUCATION: [
    {
      degree: "MEng Something Useful",
      school: "University of Somewhere",
      location: "City, Province",
      date: "Expected Apr 2026",
      gpa: "3.9/4.0",
      coursework: "Course 1; Course 2; Course 3; Course 4"
    }
  ],

  EXPERIENCE: [
    {
      title: "Job Title That Sounds Important",
      company: "Big Company Inc.",
      location: "City, Province",
      date: "Jan 2026 – Apr 2026",
      bullets: [
        "Did something measurable that resulted in a 40% improvement",
        "Built a thing using specific methods for a specific purpose"
      ]
    }
  ],

  PROJECTS: [
    {
      name: "Cool Project (The One That Actually Works)",
      date: "Mar 2026",
      bullets: [
        "What it does and why anyone should care, with a number attached"
      ]
    }
  ],

  SKILLS: [
    { category: "Category Name", items: "Skill 1, Skill 2, Skill 3" }
  ],

  CERTIFICATIONS: "Your Cert (Issuer, Year)"
};
```

That's ~80 lines. That's all the model writes. Clean, bounded, hard to mess up. The model goes from "write me an entire formatted document" to "fill in this form." Huge difference.

## Step 3: Build your master resume data files

This is where most people go wrong, and where I *definitely* went wrong for the first three versions.

Your master resume should be a **data warehouse**. Not an instruction manual. Not a style guide. Not a place to put "remember to use strong action verbs!" — that's what SKILL.md is for.

Here's what a clean master resume file looks like:

```markdown
# Master Resume — Cybersecurity Cluster

## EDUCATION
### MEng Applied Data Science | Expected Apr 2026
University of Somewhere, City, Province — GPA: X.XX/X.0

All courses: Course A; Course B; Course C; Course D; ...
Cyber-relevant picks: Course A; Course C; Course F

## WORK EXPERIENCE
### Security Analyst Co-op | Jan 2026 – Apr 2026
Big Company, City, Province

1. [IAM/Access] Investigated and resolved 200+ access anomalies
   across enterprise identity platform, cutting privilege risk
2. [Detection] Built 15 detection queries for shadow IT discovery,
   flagging 47 unsanctioned apps across 1,200+ endpoints
3. [IR] Triaged 85+ security incidents with 30-min avg response
...
```

Notice what's *not* here: instructions. Rules. Formatting guidance. "When tailoring, consider..." blocks. Nothing. Just facts, numbers, and tags.

Each bullet has a tag like `[IAM/Access]` or `[Detection]` — these help the model quickly match bullets to job requirements. They don't appear in the final resume. Think of them as filing labels, not content.

**Why three master resumes?** If you apply across different domains (security, data science, software dev), the relevant experience is completely different for each. Three data files means the model picks the right cluster first, then selects from a pre-filtered pool instead of sorting through everything. It's faster and more accurate.

## Step 4: Write the SKILL.md

The hard part. The part that took five versions. The part where you'll be tempted to put rules in other files "just this once" and I'm telling you right now... don't.

Here's the structure:

```markdown
---
name: resume-tailor
description: Tailor a one-page resume from a master resume and
  a job description. Trigger when user pastes a JD or asks for
  a resume. Do NOT use for cover letters.
---

# Resume Tailor

## 1. Resume Structure
Sections in this exact order, no exceptions:
1. Header
2. Education
3. Work Experience
4. Projects (2-3, each with dates)
5. Technical Skills & Certifications

There is NO summary section.

## 2. Workflow
Step 1: Identify the target role from the JD
Step 2: Pick the right master resume cluster
Step 3: Select bullets that match JD requirements
Step 4: Tailor bullet wording — echo JD terminology naturally
Step 5: Select relevant coursework (4-6 per degree)
Step 6: Build the skills section from JD keywords + real skills
Step 7: Character-check every bullet (see §5)
Step 8: Write resume_data.js
Step 9: Run resume_build.js

## 3. Bullet Writing Rules
- Format: "Accomplished [X] by doing [Y], resulting in [Z]"
- Lead with strong action verbs (detected, built, reduced, led)
- At least one metric per bullet where possible
- Echo JD terminology where it naturally fits real experience
- NEVER fabricate metrics or experience

## 4. Selection Logic
- Match bullet tags to JD requirements
- Prioritize recent experience over older roles
- Balance bullets across roles (no single job dominates)
- If JD emphasizes a skill area, weight those bullets higher

## 5. Character Budget (Orphan Prevention)
At Calibri 10pt with 0.4" margins on US Letter:
  ~100 chars = 1 line (SAFE)
  ~200 chars = 2 full lines (SAFE)
  106-185 chars = ORPHAN DANGER ZONE

If a bullet lands in 106-185: REWRITE IT.
Either trim under 100 or expand past 185. No exceptions.

## 6. Guardrails
- Output ONLY resume_data.js — never modify resume_build.js
- Never adjust margins, fonts, or spacing
- Never add a summary section
- Never fabricate experience, metrics, or skills
```

**The character budget is the single most valuable thing in this file.** Before adding it, roughly 1 in 3 resumes had orphan lines. After? Basically zero. The model self-checks before outputting. It works *shockingly* well.

Keep this file under 500 lines. If you're pushing past 300, you probably have duplicated rules — audit for repetition. My production version is 233 lines. Lean and mean.

## Step 5: Cover letters (separate skill, separate chat)

If you also want cover letters, build a **completely separate skill**:

```
cover-letter/
  SKILL.md
  references/
    coverletter_content.md
    coverletter_rules.md
  coverletter_build.js
  coverletter_data_template.js
```

Two things that are critical here:

**Always generate cover letters in a different chat than resumes.** If both outputs exist in the same conversation, the model will average the styles. Resume bullets get wordy. Cover letter paragraphs get clipped. I call this *contamination* — and I wrote about it at length [in the story post](/posts/2026/04/02/ai-resume-system/) because it was the sneakiest bug I've ever dealt with. The fix is simple: separate chats. Every time.

**Consider using a different model tier.** Resumes are mechanical — pick bullets, tailor wording, fit to page. A fast, precise model handles this well. Cover letters need to connect your experience to a company's mission, tell a cohesive narrative, and sound like *you*. That's a harder cognitive task. I use a more capable model for cover letters and it makes a noticeable difference.

## Step 6: Test and iterate

Generate 5 resumes for different job descriptions. For each one, check:

**Does it fit on one page?** If not, too many bullets or your character budgets need adjusting.

**Any orphan lines?** A word sitting alone on line 2 means either the character budget didn't catch it or the model ignored the instruction. Tighten the wording in SKILL.md.

**Does it read like a resume?** Not a cover letter. Not a JD echo. Tight, metric-driven, action-verb-led bullets. If bullets are getting wordy, check for contamination (is a cover letter output in the same chat?).

**Is the skills section relevant?** It should reflect the JD's requirements, not a dump of everything you've ever touched.

**Are dates aligned?** Right-aligned dates should sit flush with the section dividers. If they're floating in the middle of nowhere, that's a build script issue.

If something breaks — and something will break, I promise — fix it in SKILL.md. Not in the master resume. Not in the build script. SKILL.md is your single source of truth. If a rule exists anywhere else, delete it and point to SKILL.md.

## The result

The system generates a tailored, one-page `.docx` in under 30 seconds. The formatting is identical every time. The content varies per job description. I went from 45 minutes per application to about 2 minutes — paste the JD, review the output, submit.

Is it perfect? No. Sometimes a bullet needs a manual tweak. Sometimes the coursework selection is slightly off. But it's *consistent*, and consistency is what I never had when doing this manually.

It's not magic. It's just good separation of concerns: data lives in data files, rules live in one place, formatting is locked. The model does *one small thing well* instead of trying to do everything at once.

Build the foundation first. Fill in your data. Write the rules. Iterate. And if you find yourself counting characters at 1 AM...

Welcome to the club. We have snacks. And regrets.

---

Have questions? Want to argue about resume formatting? Considering hiring me? (Please consider hiring me.) Find me on [LinkedIn](https://linkedin.com/in/tejalgoyal) — I'm always happy to chat about this stuff.

**Update:** I thought this system was done. It was not. The sequel — where I obsess over token usage and tool call counts — is [here](/posts/2026/04/07/optimizing-ai-resume-system/).

*Built with Claude Skills · April 2026*
