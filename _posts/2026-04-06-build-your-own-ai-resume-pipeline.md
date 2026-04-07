---
title: "Build Your Own AI Resume Pipeline with Claude Skills"
date: 2026-04-06
categories: [dev, vibe-coding, project]
---

This is the companion post to [How I Built an AI Resume System That Actually Works](/posts/2026/04/06/ai-resume-system/). That post tells the story. This one tells you *how to build it yourself*.

The system generates a tailored one-page resume from a master data file and a job description, outputting a properly formatted `.docx` every time. It runs entirely inside Claude Skills — no external APIs, no local scripts, no dependencies beyond what the skill environment provides.

I'll walk through the architecture, show sanitized examples of every file, and explain the decisions that actually matter. If you want the "why" behind each design choice, read the story post. This one is pure "how."

## The architecture

```
resume-tailor/
  SKILL.md                  ← single source of truth for all rules
  references/
    resume_rules.md         ← verb list, XYZ formula reference
  master_resume_cyber.md    ← experience data (cluster 1)
  master_resume_data.md     ← experience data (cluster 2)
  master_resume_dev.md      ← experience data (cluster 3)
  resume_data_template.js   ← blank form the model fills in
  resume_build.js           ← locked build script (never model-edited)
```

Three types of files, three jobs:

1. **SKILL.md** — the brain. Every rule, every guardrail, every workflow step. The model reads this and nothing else for decision-making.
2. **Master resumes** — the memory. Pure experience data. Zero instructions, zero formatting guidance.
3. **Build script + template** — the hands. Handles all visual formatting. The model writes data, the script builds the document.

The critical design principle: **the model only writes a data file (~80 lines). It never touches formatting.**

## Step 1: Build the locked formatting script

This is the foundation. Get this right first, before you write a single rule or organize a single bullet point.

The build script is a Node.js file that uses the `docx` library to construct a Word document from a JavaScript data file. Here's a sanitized skeleton:

```javascript
// resume_build.js — locked, model never modifies this
const { Document, Packer, Paragraph, TextRun } = require("docx");
const fs = require("fs");

// ─── IMPORT THE DATA FILE THE MODEL WRITES ───
const {
  NAME, PHONE, EMAIL, LINKEDIN, GITHUB, PORTFOLIO,
  EDUCATION, EXPERIENCE, PROJECTS, SKILLS, CERTIFICATIONS
} = require("./resume_data.js");

// ─── CONSTANTS (tweak these, then never touch them again) ───
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

**Why 0.4-inch margins?** Standard is 0.5 inches (720 twips). Dropping to 0.4 inches (576 twips) gives you roughly 2 extra lines of vertical space and 3 more characters per line. Below 0.4 starts looking cramped on print. This is the sweet spot.

**Why `.docx` over PDF?** ATS parsing. Word documents get parsed more accurately by applicant tracking systems than PDFs, even beautifully formatted ones.

**Why a locked script?** If the model can modify formatting, it *will* — especially when it's trying to fit content onto one page. It'll shrink fonts, reduce margins, remove spacing. Locking the script means the model has to solve the one-page problem through *content selection*, not formatting hacks. That produces better resumes.

## Step 2: Create the data template

This is the blank form the model fills in. It's a `.js` file with `module.exports` that the build script imports:

```javascript
// resume_data_template.js — the model copies this and fills it in
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
      degree: "MEng Applied Data Science",
      school: "University of Somewhere",
      location: "City, Province",
      date: "Expected Apr 2026",
      gpa: "3.9/4.0",
      coursework: "Relevant Course 1; Course 2; Course 3; Course 4"
    }
  ],

  EXPERIENCE: [
    {
      title: "Security Analyst Co-op",
      company: "Big Company Inc.",
      location: "City, Province",
      date: "Jan 2026 – Apr 2026",
      bullets: [
        "Did something measurable that resulted in a 40% improvement in a thing",
        "Built another thing using specific technologies for a specific purpose"
      ]
    }
  ],

  PROJECTS: [
    {
      name: "Cool Project Name",
      date: "Mar 2026",
      bullets: [
        "What it does and why it matters, with a number attached"
      ]
    }
  ],

  SKILLS: [
    { category: "Category Name", items: "Tool 1, Tool 2, Tool 3" }
  ],

  CERTIFICATIONS: "Your Cert (Issuer, Year)"
};
```

The model's entire job is: copy this template, fill in the values based on the job description and master resume, and output it as `resume_data.js`. That's ~80 lines of work. Clean, bounded, hard to mess up.

## Step 3: Build your master resume data files

This is where most people go wrong. Your master resume should be a **data warehouse**, not an instruction manual.

Here's what a master resume file should look like:

```markdown
# Master Resume — Cybersecurity Cluster

## EDUCATION
### MEng Applied Data Science | Expected Apr 2026
University of Somewhere, City, Province — GPA: X.XX/X.0

All courses: Course A; Course B; Course C; Course D; ...
Cyber-relevant picks: Course A; Course C; Course F

### BEng Electronics and Computer Engineering | 2024
Another University, City, Country — GPA: X.XX/10.0

## WORK EXPERIENCE
### Security Analyst Co-op | Jan 2026 – Apr 2026
Big Company, City, Province

1. [IAM/Access] Investigated and resolved 200+ access anomalies across
   enterprise identity platform, reducing privilege escalation risk
2. [Detection/SIEM] Built 15 detection queries for shadow IT discovery,
   identifying 47 unsanctioned applications across 1,200+ endpoints
3. [IR/Tic] Led triage on 85+ security incidents using EDR platform,
   achieving 30-minute average response time
...

### Research Assistant | May 2025 – Aug 2025
University Lab, City, Province

1. [ML/Detection] Developed anomaly detection pipeline processing
   2.3M network flow records with 94.7% precision
...
```

**Notice what's *not* here:** no instructions about how to select bullets, no formatting rules, no guidance about character limits, no "when tailoring, consider..." blocks. All of that lives in SKILL.md. The master resume is just *facts*.

Each bullet has a tag like `[IAM/Access]` or `[Detection/SIEM]` — these help the model quickly match bullets to job requirements. They don't appear in the final resume.

**Why three master resumes?** If you're applying across different domains (say, security, data science, and software development), the relevant experience and project selection is completely different for each. Three data files means the model picks the right cluster and starts with a pre-filtered pool instead of sorting through everything.

## Step 4: Write the SKILL.md (the hard part)

This is the brain of the system. It contains:

1. **The workflow** — step-by-step what the model does when triggered
2. **Content rules** — how to write bullets, select experience, handle skills
3. **Character budgets** — the orphan prevention system
4. **Guardrails** — what the model must *not* do

Here's a sanitized version of the structure:

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
- XYZ format: "Accomplished [X] by doing [Y], resulting in [Z]"
- Lead with strong action verbs (detected, built, reduced, led)
- Include at least one metric per bullet where possible
- Echo JD terminology where it naturally fits real experience
- Do NOT fabricate metrics or experience

## 4. Selection Logic
- Match bullet tags to JD requirements
- Prioritize recent experience
- Balance bullets across roles (no single job dominates)
- If a JD emphasizes a skill area, weight those bullets higher

## 5. Character Budget (Orphan Prevention)
At Calibri 10pt with 0.4" margins on US Letter:
  ~100 chars = 1 line (SAFE)
  ~200 chars = 2 full lines (SAFE)
  106-185 chars = ORPHAN DANGER ZONE

If a bullet lands in 106-185: rewrite it.
Either trim under 100 or expand past 185. No exceptions.

## 6. Guardrails
- Output ONLY resume_data.js — never modify resume_build.js
- Never adjust margins, fonts, or spacing
- Never add a summary section
- Never fabricate experience, metrics, or skills
- Keep total output to ~80 lines
```

**The character budget is the most valuable thing in this file.** Before I added it, roughly 1 in 3 resumes had orphan lines — a single word wrapping to a second line, wasting an entire line of vertical space. After adding it, orphans dropped to near zero because the model self-checks before outputting.

**233 lines.** That's what the production SKILL.md is. Well under the 500-line recommended limit for Claude Skills. If you're approaching 300+, you probably have duplicated rules — audit for repetition.

## Step 5: Cover letters (separate skill, separate chat)

If you also want cover letters, build a completely separate skill for them:

```
cover-letter/
  SKILL.md
  references/
    coverletter_content.md
    coverletter_rules.md
  coverletter_build.js
  coverletter_data_template.js
```

**Critical:** always generate cover letters in a *different chat* than resumes. If both outputs exist in the same conversation, the model will average the styles. Resume bullets get wordy. Cover letter paragraphs get clipped. I learned this the hard way across about 20 contaminated outputs before I figured out what was happening.

The cover letter skill is intentionally designed for a model with more reasoning depth. Resumes are mechanical — pick bullets, tailor wording, fit to page. Cover letters need to connect your experience to a company's mission, tell a cohesive narrative, and *sound like you*. Different cognitive tasks, potentially different model tiers.

## Step 6: Test and iterate

Generate 5 resumes for different job descriptions. For each one, check:

1. **Does it fit on one page?** If not, you have too many bullets or your character budgets are wrong.
2. **Are there orphan lines?** Check every bullet. A word sitting alone on line 2 means the character budget didn't catch it (or the model ignored it — tighten the instruction).
3. **Does it *read* like a resume?** Not a cover letter, not a job description echo. Tight, metric-driven, action-verb-led bullets.
4. **Is the skills section relevant?** It should reflect the JD's tech stack, not just a dump of everything you know.
5. **Are the dates aligned?** Right-aligned dates should sit flush with the section divider lines, not floating in the middle of nowhere.

If something breaks, fix it in SKILL.md. Not in the master resume. Not in the build script. The SKILL.md is your single source of truth — if a rule exists anywhere else, delete it and point to SKILL.md instead.

## The result

The system generates a tailored, one-page `.docx` in under 30 seconds. The formatting is identical every time. The content varies per job description. I went from 45 minutes per application to about 2 minutes (paste JD → review output → submit).

It's not magic. It's just good separation of concerns: data lives in data files, rules live in one place, formatting is locked. The model does *one small thing well* instead of trying to do everything.

Build the foundation (script + template), fill in your data (master resumes), write the rules (SKILL.md), and iterate. Five versions taught me that the architecture matters more than the prompt.

---

*Built with Claude Skills · April 2026*
