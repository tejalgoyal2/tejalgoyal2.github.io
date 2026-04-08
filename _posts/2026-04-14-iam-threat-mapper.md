---
title: "I Built an Interactive Attack Path Visualizer Because Spreadsheets Aren't Scary Enough"
date: 2026-04-14
categories: [dev, security, project]
---

During my cybersecurity co-op I spent a lot of time staring at IAM configurations. Access policies, MFA enforcement gaps, privileged account sprawl, the whole identity stack. And the thing that kept bugging me wasn't any single misconfiguration — it was that nobody could *see* how they chained together.

A weak MFA policy on its own? Manageable. That same weak MFA policy connected to a privileged service account connected to a VPN gateway connected to production infrastructure? That's a breach path. And it's invisible in a spreadsheet.

So I built [IAM Threat Mapper](https://iam.tgoyal.me) — an interactive tool that visualizes identity-based attack chains and maps them to the MITRE ATT&CK framework. You can see how a SIM swap leads to MFA bypass leads to privilege escalation leads to data exfiltration... as a graph. With nodes. And edges. And colors that mean things.

It's the tool I wished I had at work. So I built it on the side.

## Why identity attacks specifically

Here's the thing about modern breaches — the interesting ones almost never start with some zero-day exploit or sophisticated malware. They start with *identity*.

The [MGM Resorts breach](https://www.cisa.gov/news-events/cybersecurity-advisories)? Started with a helpdesk social engineering call. Someone called the helpdesk, pretended to be an employee, got a password reset. From there: lateral movement, privilege escalation, ransomware. Billions in damages. The initial entry point was a *phone call*.

The Uber breach? A contractor's credentials were compromised. The attacker spammed MFA push notifications until the contractor approved one out of frustration — a technique called MFA fatigue. Once inside, the attacker found hardcoded credentials in a script and pivoted to everything.

SolarWinds? Supply chain compromise, sure. But the *lateral movement* inside victim networks was all identity-based. Compromised tokens, forged SAML assertions, privilege escalation through identity federation trust.

The pattern is always the same: gain an identity foothold, escalate privileges, move laterally. And the defenses that matter — MFA enforcement, privileged access management, conditional access policies, session controls — are all identity controls.

I was seeing this pattern every day in my co-op work. Investigating access anomalies, reviewing IAM policies, triaging alerts from the identity platform. The attack paths were real. The data was right there. But there was no good way to *show* someone — a manager, a stakeholder, a new team member — how a chain of small weaknesses becomes one big problem.

## The two halves of the tool

IAM Threat Mapper does two things, and they're designed to work together.

### Attack path visualizer

This is the flashy part. An interactive node graph — built with [Cytoscape.js](https://js.cytoscape.org/) — that shows how real-world identity attacks unfold step by step. Each node is a stage in the attack chain (initial access, credential theft, privilege escalation, lateral movement, impact). Each edge is the technique that connects them, mapped to a specific [MITRE ATT&CK](https://attack.mitre.org/) technique ID.

You can click on an attack scenario — SIM swap, OAuth token theft, helpdesk social engineering, MFA fatigue, federation abuse — and watch the path light up. Each node expands to show what happened, what MITRE technique applies, and what defensive control could have stopped it.

The goal isn't just "look, a pretty graph." It's to answer the question security teams actually care about: *where does this chain break if we fix one thing?* If you enforce phishing-resistant MFA, the MFA fatigue path dies at node two. If you implement privileged access management, the lateral movement path dies at node four. The graph makes the ROI of specific controls *visible*.

### IAM maturity assessment

The other half is a questionnaire — a structured self-assessment that walks through the major IAM control categories: authentication strength, privileged access management, lifecycle governance, monitoring and detection, conditional access, and a few others.

Each category has pointed questions. Not "do you have MFA?" but "is MFA enforced for *all* user types including service accounts and break-glass accounts?" Not "do you review access?" but "do you have automated access certification campaigns with defined recertification windows?"

The assessment generates a maturity score per category and an overall score. It's not meant to replace a proper audit — it's meant to give a team a quick snapshot of where they are and, more importantly, where the *gaps* are. The gaps are where the attack paths live.

> The two halves are connected by design. A low maturity score in "Authentication Strength" maps directly to the attack scenarios that exploit weak authentication. The assessment tells you *what's weak*. The visualizer tells you *what happens when it's exploited*. Together, they make the case for fixing it.

## The tech decisions

**React 19 with Cytoscape.js** for the graph visualization. I looked at D3 first — it's the standard for data visualization in JavaScript — but Cytoscape is purpose-built for network graphs. It handles node positioning, edge routing, zoom, pan, and layout algorithms out of the box. For a graph where the *relationships between nodes* are the whole point, Cytoscape was the right call. D3 would have meant building all of that from scratch.

**Tailwind CSS v3** for styling. I didn't want to fight CSS while also fighting graph layout algorithms. Tailwind let me move fast without making everything look like a Bootstrap template from 2019.

**No backend.** All logic runs client-side. All data is static JSON. The attack scenarios, the assessment questions, the MITRE mappings — it's all bundled into the React app. This was a deliberate choice: I wanted this to be something anyone could open in a browser without creating an account, without sending data to a server, without any privacy concerns. You're assessing your *security posture* — the tool that helps you do that shouldn't be collecting your answers.

**Cloudflare Pages** for hosting. Free, fast, deploys from a GitHub push. The whole app is a static site — no server-side rendering, no API calls, no database. Build it, push it, done.

**No external UI libraries.** Zero Bootstrap, zero Material UI, zero component libraries. Every element is hand-styled with Tailwind. This is partially stubbornness and partially because I wanted full control over how the graph and assessment panels interact visually. When your main feature is a complex interactive graph, you don't want a component library's opinions about layout fighting your opinions about layout.

## Building the attack scenarios

This was the part that took the most time and the most thought. The code was straightforward — React components rendering Cytoscape graphs. The *content* was the hard part.

Each attack scenario needed to be:

1. **Based on a real breach.** Not hypothetical. Not "imagine if someone..." Real incidents with public reporting. MGM, Uber, SolarWinds, and others where the attack chain is well-documented.

2. **Mapped to MITRE ATT&CK.** Every step in the chain needed a real technique ID — T1078 (Valid Accounts), T1556 (Modify Authentication Process), T1098 (Account Manipulation), and so on. This grounds the scenarios in an industry-standard framework, not my personal opinion about what's scary.

3. **Actionable.** Each node needed to show not just "what happened" but "what would have stopped this." If the answer to "how do we prevent this step" is "implement phishing-resistant MFA" — that's specific enough for someone to act on. If the answer is "be more secure" — that's useless.

Building this meant reading a *lot* of breach reports, CISA advisories, and threat intelligence writeups. Then distilling each incident into a clean chain of 5-7 nodes that tells the story without drowning in detail. It's a weird kind of editing — taking a complex, messy, real-world incident and turning it into a graph that a security analyst can absorb in thirty seconds.

I... genuinely enjoyed this part. Reading about how attackers chain identity weaknesses together is *fascinating* in the worst way. Like watching a heist movie where the vault is your company's production environment.

## What I learned building it

**Graph layout is a rabbit hole.** Cytoscape has multiple layout algorithms — breadthfirst, dagre, cose, grid. Each one arranges the same nodes differently. I spent an embarrassing amount of time switching between them, squinting at the screen, and going "no, that edge crosses that other edge and now it looks like step 3 leads to step 5 when it doesn't." Dagre (directed acyclic graph layout) ended up being the winner for attack paths because they're inherently directional — you move left to right through the kill chain.

**Assessment questions are harder to write than code.** A bad question gives you a useless answer. "Do you have identity monitoring?" — what does that even mean? Do you mean you have logs? Do you mean you have alerts? Do you mean someone *looks* at the alerts? Each question went through multiple rewrites to be specific enough to produce a meaningful maturity signal.

**Static data is underrated.** No backend means no auth, no database migrations, no API versioning, no server costs, no security surface to worry about. The entire app is a folder of HTML, CSS, and JavaScript. If I want to add a new attack scenario, I add a JSON object and push to GitHub. Fifteen seconds. If this had a backend, adding a scenario would mean a migration, a deployment, and probably a prayer.

**Building from work experience is a cheat code.** I didn't have to guess what IAM weaknesses matter — I'd been looking at them every day for months. The assessment categories came directly from the gaps I saw in real environments. The attack scenarios came from the incident reports I was reading as part of my actual job. The tool is better because I built it *while* doing the work it's about, not after.

## Try it

It's live at [iam.tgoyal.me](https://iam.tgoyal.me). The code is on [GitHub](https://github.com/tejalgoyal2/iam-threat-mapper). Click through an attack scenario, run the assessment, poke around.

If you're in security and you think a category is missing from the assessment, or an attack path could be more accurate, or the MITRE mapping is wrong somewhere — I want to hear about it. This is a living project. It gets better when people who know more than me tell me what I got wrong.

And if you're a student or early-career security person thinking about building a portfolio project... build the thing you wish existed at work. Not a tutorial project. Not a clone of something that already exists. The thing that would have made your Tuesday afternoon easier. That's the project that teaches you the most and looks the best on a resume.

Trust me. I built a [whole system](/posts/2026/04/03/ai-resume-system/) for generating those resumes. I know what looks good on them.

---

*Built with React, Cytoscape.js, and too many hours reading breach reports · [GitHub](https://github.com/tejalgoyal2/iam-threat-mapper) · April 2026*
