---
title: "My Resume Builder Thought 120% of a Page Was 'Slightly Over'"
date: 2026-06-03
categories: [dev, vibe-coding]
---

Back in April I wrote that my resume system [finally worked](/posts/2026/04/03/ai-resume-system/), and then almost immediately wrote a [second post about breaking it](/posts/2026/04/11/optimizing-ai-resume-system/) while trying to make it better. I would like to report personal growth since then. I cannot.

Because a few weeks later I opened it back up, and the very first resume it produced was a page and a half long. The build script looked at that page and a half, checked its little internal threshold, and decided this was "slight overflow."

A page and a half. *Slightly* over. Sure.

<!-- MEDIA SUGGESTION: a screenshot of a resume spilling three or four lines onto a second page, captioned "slight overflow." The gap between what the script claimed and what the page actually looked like is the whole joke. -->

## The lie was in the measurement

Here is the thing the old version never did: look at the page.

It estimated length from character counts. Add up the letters, compare to a number it had in its head, declare victory. The problem is that character count is a terrible way to predict how long a document is. A bullet at 240 characters and a bullet at 260 characters can be the difference between two lines and three, depending entirely on where the words happen to wrap. The script had no idea. It just counted letters and felt confident.

So "slight overflow" was never a measurement. It was a guess wearing the costume of a measurement.

The April version worked the way a stopped clock works. Right often enough that I stopped checking.

## So I made it actually look at the page

The fix was conceptually simple and annoyingly overdue. Build the resume into a real document, convert it to a PDF, and *measure how much of the page the content fills.*

Now the number means something. `0.97` means it is sitting at ninety-seven percent of one page, a little empty at the bottom. `1.06` means it spilled onto a second page by six percent. These come from the actual rendered document, not from vibes.

And once it can measure, it can fix itself in a single run. Under-full? It pads, pulling a bullet I had held back in a reserve list. Over-full? It trims, cutting the weakest bullet, starting with the oldest role and never touching my current one. It loops until it lands on one full page. No asking me. No second prompt. It just converges and hands me the result.

It went from confidently wrong to quietly correct. I will take it.

## The part where I lost sleep over single words

At this font and these margins, a bullet wraps onto a third line somewhere around 250 characters. Two lines is good. Three lines is a waste of space I do not have. So every bullet has to land under that ceiling, which meant I spent an embarrassing number of evenings deleting single words to claw back a line.

"Implemented" becomes "Built." "In order to" becomes "to." Whole sentences rearranged to save eight characters.

There is no version of me from five years ago who would believe that I once stayed up arguing with myself about whether one comma was worth a second line, on a document most people skim for nine seconds.

> Monaco is [this weekend](https://www.formula1.com/en/racing/2026), which means I should be doing literally anything to emotionally prepare for it. As I write this, [a Mercedes is running away with the championship](https://www.formula1.com/en/results/2026/drivers), and Ferrari is doing its traditional thing where it is fast enough to make me hope and never quite fast enough to win. Anyway. Resume margins. Where was I.

## It was also lying about me

While reading the output, I noticed the resume was weirdly *modest.* It kept saying things were "underway," that reviews were "in progress," that I was "actively" doing this and that.

Hedge words. On a resume. Where the entire job of a line is to say what you did, cleanly and without a disclaimer.

So I killed the hedges. "Underway" is not an accomplishment. Either I did the thing, in which case say it, or I did not, in which case it does not belong on the page.

## And then it got too confident

The opposite problem showed up too, and this one I am less proud of.

At some point the model, trying to make me sound impressive, turned "contributed to a published paper" into "co-authored a published paper." I contributed to one. I am not a named author on it. Those are different sentences, and the difference between them is the entire point.

So I wrote a rule into the system, in plain words: confidence is not inflation. Do not turn *helped build* into *built.* Do not turn *contributed* into *authored.* A resume can be sharp without claiming things that are not mine.

A resume that oversells is a problem you only discover later, in the interview, and by then it is sitting in the room wearing your face.

I would rather be accurately good than impressively fake.

## The cover letter got the same surgery

While I was in there, the cover letter side needed help too. It read like three paragraphs that had never met each other. Here is why your company. Here is my experience. Here is a polite goodbye. Three blocks, stacked.

So I rewrote how it builds them, so the whole thing reads like one person actually talking, each paragraph leading into the next, and so the opening sounds like I specifically want to work *there* and not like a template with the company name pasted in.

The test for whether a sentence belongs in a cover letter is simple. If you could lift it straight onto the resume, it is the wrong sentence.

## Where it landed

It fits one page. It measures that page itself, every single time, and pads or trims until it is full without spilling. It does not hedge and it does not oversell.

It cost, conservatively, three weeks of evenings and one genuine crisis about a comma. Worth it? I apply to a *lot* of jobs. Yes.

If you want the actual how-to, the one where I explain how to build a system like this for yourself instead of just complaining about mine, I wrote that [over here](/posts/2026/06/05/build-your-own-resume-skill/). The messy story lives in this post. The useful part lives in that one.

*Built with Claude, far too much LibreOffice, and the quiet dread of a Monaco weekend. Find me on [LinkedIn](https://linkedin.com/in/tejalgoyal).*
