---
title: "I Built My Own Sudoku App Because an Ad Interrupted My Winning Streak"
date: 2026-04-30
categories: [vibe-coding, project, life]
---

I play Sudoku on my commute. Every day. It's the one thing that keeps me from doom-scrolling through LinkedIn job posts or reading about whatever Ferrari did wrong this weekend. The app I was using worked fine — until it didn't.

At some point, the ads went from "mildly annoying banner at the bottom" to "full-screen video ad between every single game." Mid-streak. Mid-flow. Right when you finish a puzzle and want to ride that momentum into the next one — *bam*, thirty seconds of some mobile game you'll never download. The kind of ad placement that makes you wonder if someone on the product team actively hates their users.

## The math was simple

The app wanted a subscription to remove ads. For Sudoku. A game that's been free in newspapers since before I was born. I'm a student. I'm not paying a monthly fee to fill numbers into a 9x9 grid.

So the options were: find another app (they all have ads), pay up (no), or build my own.

*...obviously I built my own.*

## The Perplexity research phase

Before writing a single line of code, I went to Perplexity — same workflow I used when I was [researching design systems](/posts/2026/04/29/chibi-design-system/). I wanted to understand how people actually build good Sudoku apps. Not just "here's a grid," but the puzzle generation algorithms, the UX patterns that make a Sudoku app *feel* right, difficulty calibration, the whole thing.

The research covered puzzle generation approaches, what makes a valid Sudoku (exactly one solution — turns out a lot of cheap apps don't even guarantee this), how difficulty levels actually work at an algorithmic level, and what features people care about most. Timer, streaks, pencil marks, undo — the basics that every decent Sudoku app needs to not feel broken.

Once I had a solid understanding of the landscape, I took all of that to Claude and started building.

## A web app that pretends to be a real app

Here's the thing — I can't publish an iOS app. Apple charges $99/year for a developer account, and I don't live in the EU where sideloading is an option. So the move was a **Progressive Web App**. You build a website, add the right meta tags and a service worker, and when you "Add to Home Screen" on iPhone, it looks and behaves like a native app. Full screen, its own icon, no Safari chrome visible.

The stack ended up being React, Vite, and Cloudflare Pages for hosting. I already had my domain (`tgoyal.me`) on Cloudflare, so pointing `sudoku.tgoyal.me` at a Pages project was trivial. No backend. No database. No accounts. Everything — stats, streaks, settings — lives in `localStorage` on the device.

Free. Forever.

## What it actually does

The app generates puzzles algorithmically with four difficulty levels: Easy, Medium, Hard, and Expert. Every puzzle is guaranteed to have exactly one solution. It tracks your streak (consecutive days played), your average completion time, and your best time per difficulty.

It has the features that matter: pencil marks for candidates, undo, a mistake counter, a timer. Light and dark mode, because I play on the train and sometimes the lighting changes. The whole thing is responsive and optimized for iPhone — which is where I use it 95% of the time.

<!-- MEDIA SUGGESTION: A screenshot or side-by-side of the app in light and dark mode on an iPhone would work well here — shows the actual product. -->

## The part where it just... worked

I keep waiting for the part of this post where I tell you about the three-day debugging nightmare or the API that returned garbage. But honestly? This one went smooth. The puzzle generation algorithm worked on the first real attempt. The PWA setup with Vite's plugin was mostly plug-and-play. Cloudflare Pages deployed in under a minute.

The hardest part was probably getting iOS-specific quirks right — safe area insets, preventing pull-to-refresh, making sure the viewport didn't zoom when you tap an input. The kind of stuff that isn't hard, just *annoying* and poorly documented.

## This is the first one, not the last one

Here's the bigger idea. That Sudoku app I was paying for? It's not the only tool on my phone that's "free with obnoxious ads." I use a clipboard manager on my MacBook — basically a notepad that lives in the menu bar. Same deal. Free tier, but with ads wedged into the interface.

I'm already rebuilding it. My own aesthetic, my own keyboard shortcuts, the features *I* actually want. And when it's done, I'll put it out there. Not to make money. Just because if I stumbled onto someone else's clean, ad-free version of a tool I needed, I'd be grateful.

That's kind of the play now. Every time an app I rely on starts shoving ads into the experience, I'll just... build my own. It's more fun than being annoyed, and I end up with something better anyway.

You can play it right now: [sudoku.tgoyal.me](https://sudoku.tgoyal.me)

---

*Built with React, Vite, and Cloudflare Pages. Researched with Perplexity. Developed with Claude. Zero ads, zero subscriptions, zero regrets.*

*Find me on [LinkedIn](https://linkedin.com/in/tejalgoyal).*
