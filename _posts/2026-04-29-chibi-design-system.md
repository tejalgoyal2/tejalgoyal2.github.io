---
title: "I Got Tired of My Apps Looking Like Every Other AI Project, So I Built a Design Language"
date: 2026-04-29
categories: [dev, vibe-coding, project]
---

Every vibe-coded app looks the same. You know the one — purple-indigo gradient background, three identical feature cards in a row, a glowing orb that serves no purpose, and Poppins. Always Poppins. I'd been building functional stuff with Claude for weeks, and the *logic* was great, but every time I looked at the output I thought... *this could be anyone's*.

That bothered me more than it should have. So instead of building another app, I spent a weekend building a design system. It's called **Chibi** — named after my pet sloth (a soft toy, to be clear, I don't have an exotic animal situation going on). And it's designed to look nothing like a soft toy.

## The problem with default AI aesthetics

Here's the thing nobody tells you about vibe coding: the code quality is genuinely impressive. The *design* quality is aggressively mid. And it's mid in a very specific, recognizable way.

I started noticing patterns in everything Claude and other models generated for me. Cold blacks and pure whites. Gradient buttons everywhere. Icons sitting inside little colored circles for no reason. That three-column feature grid — icon, bold title, two-line description, repeated three times — which has become the single most recognizable layout pattern in AI-generated interfaces. It's the equivalent of a stock photo handshake on a corporate website.

The apps *worked*. They just didn't feel like *mine*. If I'm going to build things, I want someone to look at them and think "oh, that's a Tejal thing" — not "oh, that's a Claude thing."

> The difference between a tool and a product is personality. A spreadsheet is a tool. A spreadsheet that makes you feel something when you open it... okay, that's still a spreadsheet. But you get the idea.

## How I found my aesthetic (with three different AIs)

I didn't start with a vision. I started with a vague dissatisfaction and a Perplexity session.

I knew I didn't want the generic AI look. I knew I wanted dark themes. Beyond that... nothing. So I did what any reasonable person does when they don't know what they want — I went looking at things until something made me stop scrolling.

I used [Perplexity](https://www.perplexity.ai/) to research design approaches for AI-built interfaces. What are people doing to make their projects stand out? What design systems exist? What tools does Claude actually have built-in? The research phase was genuinely useful — I learned about Claude's `frontend-design` skill (which I didn't know existed), design tokens, anti-pattern lists, and how people structure reusable style systems.

Then I brought the research into GPT 5.4 with thinking enabled and started a conversation that was less "tell me what to do" and more "help me figure out what I actually like." This is where it got interesting.

GPT ran me through an elimination process. Dark or light? Sharp corners or rounded? Dense or breathing room? Colorful or restrained? Serif or sans-serif? Which brand's visual vibe resonates — Linear, Stripe, Vercel, Apple, Notion?

I couldn't pick one. But I could *eliminate*.

Apple? Too plain. Raycast? No real design in my opinion. Vercel? Too dark and austere. Figma? Looks like a PowerPoint scroll. But Notion's use of black and white — yes. Linear's transitions and placement — yes. Stripe's polish — fine but not *it*.

Then it asked me for reference sites. And *that's* when I figured it out.

[Flyhyer.com](https://www.flyhyer.com/) — that plane that pushes forward, the boldness of the type, the confidence. *That.*

[Unseen.co](https://unseen.co/) — the serif-sans contrast, the cinematic warmth, the way the type alone creates drama. *Also that.*

And then GPT said something that made it click: "Your style in one sentence: Dark-warm cinematic minimalism — bold high-contrast serif display type, generous breathing room, near-black surfaces with off-white and one warm accent, smooth transitions, no clutter, feels expensive."

I read it twice. *That's it. That's the thing.*

## The font revelation

Let me rant about fonts for a second.

I kept telling GPT "I'm seeing some fonts where the lines get thin and thick in the corners, I think I like them." Very technical description, I know. Turns out what I was describing is a **high-contrast serif** — fonts where the strokes swell from dramatically thin hairlines to thick weight at the curves and corners. Think Cormorant Garamond, Playfair Display, Editorial New.

The moment I saw Cormorant Garamond at display scale on a dark background, I knew. It's architectural. It feels like someone carved it into the side of a building. Paired with Satoshi (a clean geometric sans-serif from [Fontshare](https://www.fontshare.com/)) for body text, the contrast between the two creates this... tension. Editorial meets engineering. Drama meets clarity.

Every single AI-generated site I'd seen used Space Grotesk, Inter, or Poppins. Picking a serif display font was, weirdly, a design statement by itself. It's the typographic equivalent of showing up to a hackathon in a blazer.

## From vibes to a system

Having an aesthetic is one thing. Making it *repeatable* is another.

I needed this to work across every future project without re-explaining my preferences each time. The answer was a **Claude Skill** — a structured file that lives in Claude's context and automatically enforces the design system whenever I ask for anything visual.

I took the full design direction from my Perplexity and GPT sessions and brought it to Claude to build the actual skill. This is where the division of labor made sense — GPT was great for the exploratory "who am I as a designer" conversation, but Claude's skill system is what makes the decisions *stick*.

The [skill](https://github.com/tejalgoyal2/tejalgoyal2.github.io) ended up as a three-file structure:

**`SKILL.md`** — the brain. Design philosophy, workflow rules, the quality test every project must pass before shipping. About 84 lines. Lean on purpose — if the instruction file is bloated, the model reads it slower and follows it worse. I learned *that* lesson the hard way with my [resume system](/posts/2026/04/03/ai-resume-system/).

**`references/tokens.md`** — the complete CSS variable system. Every color, every spacing value, every shadow, every font declaration. Dark mode and light mode. This is the source of truth — if a value isn't defined here, it doesn't exist in Chibi.

**`references/components.md`** — how buttons, cards, inputs, and the signature "heading moment" should look and behave. Motion patterns, easing curves, hover states.

**`references/anti-patterns.md`** — the blacklist. Gradient buttons? Banned. Purple glow orbs? Banned. Icons in colored circles? *Extremely* banned. Poppins? I never want to see it again. This file exists so Claude knows what *not* to do, which turns out to be just as important as knowing what to do.

## The Chibi system in a nutshell

The whole thing comes down to a few core principles:

**Typography is the design.** Before any color, image, or layout decision, the type should already be doing most of the work. One moment per page where Cormorant Garamond goes big — really big — and everything else steps back. That's the "Hyer Principle." One moment where you mix roman and italic serif for contrast — that's the "Unseen Studio Principle." After that, Satoshi handles everything else and stays out of the way.

**Color restraint is the accent.** The palette is warm near-blacks (`#0f0e0d`, not cold `#000000`), layered dark surfaces that get slightly lighter as they elevate, and off-white text (`#f0ede8`, not screaming `#ffffff`). The accent is a warm cream-gold (`#e8dcc8`) — and it only appears on the *single most important interactive element on screen*. Everything else signals interactivity through border weight changes and surface tone shifts. If everything is accented, nothing is.

**Space directs attention.** More space around the important thing. Less space between related things. Never the same padding on every section — that's how you get a page that reads like a list instead of a story.

**Motion breathes.** Elements fade in with a gentle 16px upward drift. Hover transitions take 180ms with a specific easing curve (`cubic-bezier(0.16, 1, 0.3, 1)` — fast in, gentle settle). No bouncing. No spinning. If an animation draws attention to itself, it's wrong.

## Default Claude vs. Chibi — same content, different universe

To see the difference, I had Claude generate two versions of the same page — a personal project landing page with identical content. One using its default `frontend-design` aesthetic. One using Chibi.

The default version came out with cold dark blues, a purple accent, floating gradient orbs in the background, and card hover effects that glowed. It looked fine. It looked like every other AI-generated portfolio page on the internet.

The Chibi version used warm near-black surfaces, Cormorant Garamond at hero scale with one italic word for drama, cream-gold accent on just the primary button, and cards that signal hover through a subtle border shift instead of a glow. It looked like it was *designed*. By a person. With opinions.

Same code complexity. Same content. Completely different feeling. That's what a design system buys you — not fancier technology, just *intentional* decisions applied consistently.

## The five-question test

Every Chibi project has to pass five questions before I'll ship it:

**The double-take test.** Would someone scrolling past stop and look twice? If no, something needs more confidence.

**The attribution test.** Could this belong to any other AI-generated project? If yes, it has failed. *This is the one that matters most.*

**The material test.** Does it feel like a premium physical object — a matte black notebook, a brushed metal surface? If it feels plasticky, revisit the palette.

**The breathing room test.** Remove one element. Does the page feel better? If yes, remove it permanently.

**The type test.** Cover all colors and images. Does the typography alone look intentional? If the page is ugly without color, the type hierarchy needs work.

## What I actually learned

This whole thing took a weekend. Not a month. Not a "design bootcamp." One Perplexity session, one long GPT conversation, and one Claude skill-building chat. The design system is done. It applies automatically to every project I build now.

The biggest lesson wasn't about design — it was about *using the right model for the right phase*. Perplexity was perfect for research. GPT with thinking was perfect for the exploratory "what do I even want" conversation — it asked the right questions and synthesized my messy preferences into a coherent direction. Claude was perfect for turning that direction into a structured, reusable system. No single model did the whole job. They were a relay team, and the baton passes were the interesting part.

The other lesson: knowing what you *don't* want is almost as useful as knowing what you do. I couldn't describe my ideal design language from scratch. But I could point at things and say "not that." Enough "not thats" eventually leave you standing on the one thing that's *yes*.

## What's next

I'm starting a software engineering co-op next week — which means the side project velocity is about to drop from "every few days" to "whenever I'm not already staring at code for eight hours." But the design system goes with me. Every personal project, every portfolio piece, every random weekend build — it all runs through Chibi now.

The [skill is on GitHub](https://github.com/tejalgoyal2) if you want to see how it's structured. And if you want to build your own — you should. Not because my aesthetic is right for you (it almost certainly isn't), but because having *an* aesthetic that's consistently yours is the difference between a portfolio and a pile of projects.

> Chibi — Named after a soft toy sloth. Designed to look nothing like a soft toy.

---

*Built with Perplexity, GPT 5.4 with thinking, and Claude · April 2026*

*Find me on [LinkedIn](https://linkedin.com/in/tejalgoyal) — I promise my messages are more concise than my blog posts.*
