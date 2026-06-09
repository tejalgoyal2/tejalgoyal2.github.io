---
title: "Building tgoyal.me With Claude: The Full Guide, Every Prompt Included"
date: 2026-06-08
categories: [vibe-coding, dev, project]
---

Someone asked me how I made my portfolio and I started explaining it. Twenty minutes in, they were still listening. So here it is properly, written down — not because the result is some masterpiece, but because the *process* was the interesting part, and I couldn't find a guide like this when I wanted one.

This isn't a "step 1: install React" tutorial. It's the actual story — the wrong turns, the two full rollbacks, the sessions that ended in me typing things I'm mildly embarrassed to publish, and how it eventually clicked. I'm including the real verbatim prompts because that's the most useful thing I can hand you. Anyone can tell you *what* to build. Fewer people show you *how they actually talked to the AI to get there* — including the parts where the AI and I were clearly not on the same page.

Fair warning: this is long. We spent weeks on it. If you just want the lessons, jump to [Part 16](#part-16--what-id-tell-someone-starting-this). If you want the whole thing, settle in.

> **How to read this:** the early chapters (Parts 0–2) are the *mechanics* — the exact model-and-mode loop I ran. Everything after is the *story* of this specific build. If you only take one thing: the mechanics matter less than the taste constraints. More on that throughout.

## Part 0 — The whole thing ran in one chat, two models

Here's the part most "I built X with AI" posts skip: the exact loop. Mine was simpler than you'd think, and I never really used Claude chat for this. Everything happened **inside the Claude Code desktop app**, in *one long chat*, by switching between two models and two modes.

The loop, written out:

> **Plan with Opus (plan mode) → switch to Sonnet (accept edits) → let it build → preview → plan again with Opus → build again with Sonnet → ...**

Plan, code, plan, code, plan, code. For weeks. Then deploy.

The reason it's *one chat* and not separate sessions: the plan and the build need to share context. Opus writes the plan knowing the whole codebase; Sonnet implements it with that same history right there above it. When I switched models I wasn't starting over — I was handing the same conversation to a different brain.

The thing that makes this work, and the single most useful trick in this whole post:

> **Tell Opus to make the plan detailed enough that Sonnet doesn't have to *think* — just *implement*.**

That's the division of labour. Opus does the expensive reasoning *once*, up front, and bakes every decision into the plan. Sonnet then executes a spec instead of inventing one. You're not paying for deep reasoning twice, and Sonnet is far less likely to wander off and "improve" things you didn't ask about.

## Part 1 — Why Opus plans and Sonnet builds (and why I dropped Opus for building)

I'm a Pro-plan user, so I had all the models. Here's what I actually reached for, and what I learned the expensive way.

**Planning: Opus 4.8, thinking cranked up (high / extra).** Planning is the part that genuinely needs deep reasoning — the concept, the architecture, the "should this section even exist" calls. Opus is the large reasoning specialist, built for problems that genuinely need deep thinking over time, and it uses more of your rate limit, so you want to reserve it for tasks that really need it. Planning is one of those tasks. This is exactly the kind of work [Anthropic's model-picking guide](https://claude.com/resources/tutorials/choosing-the-right-claude-model) points Opus at.

**Building: Sonnet 4.6, also on high thinking.** Once the plan is detailed, building is *execution*, and Sonnet is the daily driver — strong reasoning for the kind of work you do every day, coding included, and Anthropic literally says start here if you're unsure. I *did* try Opus for the implementation early on. It wasn't worth it — the output wasn't meaningfully better for executing an already-detailed plan, and it ate through my usage fast. Dropped it almost immediately.

**Haiku: I didn't use it at all.** Worth knowing it exists for quick lookups, but this project never needed it.

One detail that made cranking thinking up far less scary: both Sonnet 4.6 and Opus 4.8 have adaptive extended thinking, so Claude automatically calibrates its reasoning depth — simple questions get fast answers, complex ones get more thinking. Leaving thinking on high isn't the usage bonfire it used to be.

> The rule, distilled: **Opus plans, Sonnet builds, and your job is to make the plan so good that Sonnet barely has to think.** Re-test this when new models land, though — Anthropic notes a new version isn't a patch on the old one; each release is a separate training run, so which model fits a task can quietly shift.

## Part 2 — How I actually worked the plan-then-build toggle

The loop has some mechanics that took me a while to figure out, so here they are concretely.

**Commenting on the plan instead of accepting it.** When Opus presents a plan in Claude Code, you get options — accept and run, or make edits first. If I *accepted*, Opus would immediately start implementing (on Opus, the expensive way). I didn't want that. So I'd read the plan top to bottom and, wherever something was off — *"no, this isn't what I had in mind"* — I'd drop a comment right on that line or word. Then I'd add any general ideas to the prompt and send it back for *another* plan pass, not a build. Usually 1–2 iterations of that and the plan was right.

**Then, and only then, switch the model.** Once I was happy with the plan, I'd change the model to Sonnet, flip to accept-edits, and tell it to follow the plan. The model swap *is* the "okay, go build now" signal.

**Reconnecting without losing the plan.** I got disconnected a *lot*. The mistake is to just say "continue" — context gets fuzzy. What I did instead: copy the whole plan back into the chat and say *"go on, keep working on this."* Paste the spec back in, every time. It's tedious. It also never let the build drift.

> The shape of a good session: read plan → comment the wrong bits → re-plan once or twice → switch to Sonnet → build → preview → back to Opus plan mode in the *same chat* → repeat. Don't collapse plan and build into one step. That's the mistake that cost me whole afternoons.

## Part 3 — The problem with the portfolio I already had

I had a working portfolio. It was *good*, honestly. Dark lavender theme, a 3D particle field in the hero you could push around with your cursor (R3F), smooth Lenis scroll, GSAP animations, a horizontal projects gallery. The kind of thing that looks professional and polished.

But "professional and polished" was the ceiling. Every section was some variation of a card grid. Scroll down: hero, then cards, then more cards, then slightly different cards. Technically fine. Zero personality.

I knew it needed a rebuild. The question was how to direct that rebuild without landing in the exact same place.

## Part 4 — The skills-folder experiment, and the first disaster

The story actually starts *before* any portfolio work, with a meta-problem: how do I give Claude my *taste* so it stops taking shortcuts? I'd been collecting design references — sites, frameworks, libraries — and I wanted them to mean something to the AI. My opening message:

> *"can you analyse this folder? i have some good design instructions and skills. analyse and tell me what you think and how can we include them in our workflow: a md file, a skill or etc. thanks. mainly i want this to help me with my portfolio design as i the current one is good but seems incomplete and ai seems to take shortcuts to make it which introduces a lot of bugs."*

We went back and forth on whether to use a `CLAUDE.md`, separate skill files, or just prompts. I got lost in the options:

> *"i am confused: you want me to make a product md and design md, what is going on?"*

If you've read my [resume-skill post](/posts/2026/06/05/build-your-own-resume-skill/), you know I've gone deep on Claude Skills before — but applying the same idea to *design taste* turned out to be much fuzzier than a resume format. Eventually the shape clicked: a fresh session, a clean brief, pointed at the existing codebase, with explicit design direction. Obvious in hindsight. Not obvious at the time.

I also had the worry I think a lot of people have:

> *"see i am a bit confused here. I liked the current design claude was going for, but when i saw the samples now i am inclinging more over how well can claude design if it is not influenced by my previous code and go all out. again there is a chance it might be bad but that is what i am not sure about."*

I was scared that giving Claude total freedom would produce something *worse* than what I had. I was right to be scared. Just not for the reason I thought.

## Part 5 — "this is one of the biggest disappointments i have seen in a long time"

I spun up a fresh session with the brief: full visual redesign, start from zero, design each section with a distinct identity, both light and dark mode genuinely designed. The opening prompt:

> *"Read the CLAUDE.md. I want a full redesign of this portfolio — treat the visual layer as starting from zero, preserve only content data. Use taste-skill to pick a visual direction, commit to it, and design each section with a distinct identity. Both light and dark mode must be genuinely designed."*

What came back was... not good. It picked "soft brutalism." Clean, structured, legible. Also completely soulless. Nothing like what I had, nothing like what I wanted.

My response was not polite:

> *"you know what, this lacks so much character. the old portfolio was way way ahead of leagues, with moving 3d cards, blog cards moving, the hero section, now it is more ai slop actually. even the pronunciation, who tf in the world know how to say tei dzal whatever you wrote there. the link what does li bg would mean so confusing and unintuitive. each card look a plain old boring just a 'card' ole one had more character. and this color itself screams ai slop. the old lavender theme the glow everything was so so so so leagues ahead, i thought this would be useful but this is so shitty. can you do anything about this. not one element i like about this."*

The IPA pronunciation `/teɪ-dʒʌl/` I'd asked for — which looked great in the old version — was now rendered so that nobody would know what they were looking at. Fair.

Then the nuclear option:

> *"i want you to remove all these changes done. fully roll back to the state we were before. drop everything. and just put a md file documenting what you tried and it went wrong what i said i did not like etc. i want the lavender thing my portfolio was before this."*

`git checkout .`. Back to lavender.

The failure was mine, not the model's. I gave Claude full creative freedom *without giving it my taste.* "Design a portfolio" with no constraints produces the statistically most average portfolio. The brief was too open. I learned a version of this lesson the hard way before, building my [Chibi design system](/posts/2026/04/29/chibi-design-system/) — a defined visual language exists precisely so the AI isn't guessing. I just hadn't connected the two yet.

## Part 6 — References, a second attempt, and "YOU ARE FREE TO DO WHAT YOU WISH"

Different approach. I had three reference sites I genuinely loved:

- **buttermax.net** — a magnetic cursor that deforms like fluid
- **igloo.inc** — 3D scroll effects
- **drumspirit.be** — fluid section morphing

I also had screen recordings of specific effects: gravity components, mask reveals, scratch reveals, hover carousels, scroll-driven 3D cards. I sent all of it. Spider-Verse "comic energy" was on the table too. The prompt was long and specific. Claude implemented. The result:

> *"man i want to say, i am so so disappointed right now. i gave so many so many good ideas and references and (attaching the screenshots) this is what you came up with? there was not a single feature i think you used. the tay-jull hover is a complete disaster, it is so simple to just put a comic style (white with black dotted like in the marvel comics) dialog. there is nothing i wanted."*

When you send reference sites and the output ignores all of them, something broke in translation. Then I sent what I now think is the single most important prompt of the whole project:

> *"I am feeling like you are trying very hard to stick to the current code, the current repo or the current design. if this is difficult, i can say that start from scratch, use these amazing references, the text explaining the videos (pasted again below), use everything, search the web for award winning design. i believe we can match the level of a fully well designed award winning website. so YOU ARE FREE TO DO WHAT YOU WISH. GIVE ME YOUR BEST."*

Hold onto that "YOU ARE FREE TO DO WHAT YOU WISH." It comes back later, and the timing of *when* you say it turns out to matter enormously.

<!-- MEDIA SUGGESTION: a side-by-side screenshot of one of the reference sites (buttermax/igloo/drumspirit) next to the bland second attempt — shows the "translation broke" gap better than words. Source your own screenshots. -->

Then, a throwaway practical note that became a permanent rule:

> *"buttermax maybe? also keep work and do not run the dev server after each change as i just saw, it hanged my macbook pro real bad. the battery was going down even when plugged."*

This became a hard constraint for the rest of the project. Which brings me to the single most useful line in my `CLAUDE.md`.

## Part 7 — The one constraint that saved my laptop (and my sessions)

```text
NEVER run npm run dev in this project — it hangs the machine.
Use npm run build (it exits clean) then the preview tool.
```

`npm run dev` starts a server that *never exits.* Claude Code would run it, wait for it to finish, and... it never finishes. The machine hangs, the battery drains while plugged in, the session is dead. `npm run build` runs and *exits*, which is the entire difference.

> Whatever your equivalent is — any long-running command that doesn't return — put it in your `CLAUDE.md` on day one. If a command doesn't exit, it will eat your session alive and you won't understand why.

## Part 8 — The monument prompt

After enough back-and-forth, I reframed the whole thing:

> *"yes. make me a monument, a website worthy of even winning awards (maybe this is too much but atleast good enough to be a nomination worthy). run the website in the end and also keep committing all the way, spread the commits in the week or two."*

"Nomination worthy." That's a *bar.* Not "looks good," not "seems professional." Something you'd actually submit to Awwwards. Naming the bar mattered more than I expected.

Then a long, honest status check that I think is a model for how to give feedback:

> *"so yeah push to main and claude you know i think we still need to plan i mean think on this, the old one was still better. i like the red while color things, but not the yellow. also the skills pick and throw is very bad as in hard to actually see the skills. i want you to move back see from afar what you did then you will understand. see how well built this looks. while ours look very unfinished, no transitions in between the sections, animations, fluidity etc. ... and also push so i can see the website on tgoyal.me. thanks. ask any questions, or even ask me if you want to see samples from specific webpages i can screenshot them. ask if you want any mcp connector for web dev which will make designing and implementing these easier for you. search and see as claude has so many options now."*

What makes that prompt work: it's *honest* about what isn't landing ("pills are hard to read"), it's *comparative* ("the old one was still better"), and it explicitly leaves the door open for Claude to ask for more or request tools. "Move back, see from afar" is me asking the model to step back and critique its own work — which it's surprisingly good at, if you prompt for it.

## Part 9 — The pivot to THE PRESS

A few more sessions in, things were improving but not *crystallising.* So I did the thing that finally worked: I opened a clean session, told Claude to ignore every earlier attempt, and asked it to **only think, not build.**

> *"so we are now redesigning and not implementing. so think think and think. I have given you all the tools.... if it helps you, i do not really like much the current portfolio is going on about. and i want a change. this is not resonating with me. so i started this new chat to not bring any old trash with this. and so you can start thinking all again. ... I do not care about the usage, i want the monument of a portfolio filled with character, animation, elements, a blessing to looks and interact with."*

"A blessing to look at and interact with." That phrase ended up on a sticky note. It's a genuinely good brief.

This is the [plan-then-build toggle from Part 2](#part-2--how-i-actually-worked-the-plan-then-build-toggle) actually paying off. This was a *planning* session in plan mode — heavy reasoning, exactly where Opus 4.8 on high thinking earns its keep. No file edits yet. Just thinking. And the concept that came back was the whole game:

**THE PRESS.** A living risograph broadsheet that prints itself as you scroll. Kinetic type that falls and slams into place. A registration-crosshair cursor. Redaction bars that reveal like declassifying documents. A whole editorial / letterpress print metaphor.

I read the plan. I said do it. And then immediately had to fight an accident:

> *"i hope you haven't created a new plan, as i had already planned, and sent the previous message cause i clicked a button saying try again. Also if you are going for a home page like this in the image, change this, the words black and just white? this is just like a powerpoint presentation. see the plan and follow that."*

I'd hit a "try again" button by accident, which re-sent an *old* message and sent Claude pivoting away from THE PRESS. I had to paste the entire plan back in by hand. Lesson: in long sessions, an accidental re-send can quietly derail everything. Watch for it.

## Part 10 — The v3 build, and a six-bug review in one breath

The THE PRESS build went *well.* Phase by phase — foundation, press engine, hero, About, Skills, Projects, Experience, Blog, Contact. The kinetic headlines (characters dropping from above, slamming into place on a `back.out` ease) were instantly the thing that made it feel different. The registration cursor. The ink bloom as the final letter lands.

Then *I* reviewed it — and this is the part worth stealing. Sonnet kept a visual preview running the whole time it worked, and I kept it open and watched it. Not just the final result — I'd glance at the page as it changed, skim the copy, occasionally read Sonnet's thinking to see what it was *trying* to do. The reviewing was *me looking at the site*, not the model grading itself.

When I caught problems, I didn't interrupt mid-build. I let Sonnet finish whatever it was doing — it sometimes ran 30–40 minutes straight — and waited for it to hand the turn back. (It runs long enough that I kept a separate note open for ideas as they popped up, because I *would* forget them otherwise. More than once I missed adding something and had to wait for the next turn to slip it in. Annoying, but interrupting a running build is worse.)

Then I'd dump everything I'd spotted in one message:

> *"so sonnet has done this and executed this plan, i want you to review this. find any areas for change improvement etc. all. i feel we should not put all the focus on MCP only ... what established in 2017? i mean i was born in 2002? the grab things is a bit phoney ... The project card expansion is breaking the red line going. fix that. ... like even expanding the experience giving a jerk to the red line. and also experience is a bit hard to read. we should be able to scroll manually too in the blogs. the contact or whatever section that was supposed to be after the blog is just a red screen. there is nothing."*

Six separate problems in one message. That's not complaining — that's a useful feedback *dump*, and each item is specific and addressable. A few highlights:

- **"What established in 2017? I mean I was born in 2002?"** The copy claimed I was established in 2017. I'd have been fifteen. Nobody is "established" at fifteen. Either a hallucinated date or a template leftover. Fixed immediately.
- **"the grab things is a bit phoney."** The matter.js physics in Skills — letters you could pick up and throw. Fun in isolation, completely out of place against an editorial press aesthetic. "Phoney" is doing a lot of work in that sentence and it's exactly the right word.
- **MCP over-focus.** The copy mentioned MCP (the protocol I work on) in the hero, the about, *and* the experience timeline. Tell Claude what you do and it'll sometimes latch onto one thing and overweight it. You course-correct.

## Part 11 — The build failure that only showed up in production

Around here:

> *"so it is failing the build i have pasted the cloudflare logs in the end..."*

The Cloudflare Pages deploy was broken: `Could not resolve entry module "three"`.

Root cause: `vite.config.js` had a `manualChunks` config splitting `three` and `framer-motion` into separate chunks — except *neither package was installed anymore.* Vite's dev server is lazy and never resolved them, so the error never appeared locally. The production build actually tries to bundle everything, hit the phantom modules, and choked.

Fix: delete the two lines. Commit message: `fix cloudflare build — remove phantom manualChunks for three and framer-motion`. Embarrassing in hindsight, maddening in the moment — it had been failing on *every* push.

> The lesson that stuck: **a dev server that never resolves a module will happily hide a broken build.** What runs locally is not what ships. (More on why shipping early matters in [Part 17](#part-17--ship-it-then-let-other-people-break-it).)

## Part 12 — "i do not even have anything much to do with newspapers"

This is the pivot I'm most proud of catching.

> *"we need to rethink the whole skill section, this is not resonating with me and is not easy to read. ... also on the blogs, can we make the card move/floating style as in to give the 3d effect? tilting etc? ... and this is going a bit too towards newspaper, i do not even have anything much to do with that. so rethink a bit."*

The press *concept* was generating great decisions — typography, motion, palette, the print metaphor. But the literal newspaper *props* (VOL. I · NO. 1, PRICE: YOUR ATTENTION, a live dateline, SEC. A / SEC. B folio markers, a dispatch ticker, a halftone monogram, the `— 30 —` end-mark) were making the *theme* the subject instead of *me.*

The call:

- Keep everything that came from a *good design principle* that happened to use the press metaphor.
- Cut everything that required the visitor to *understand the newspaper reference* to make sense.
- Rename sections to plain language.
- First person throughout.

That last one mattered. Third-person self-reference had crept into the copy:

> *"also the i break things should be Breaking things etc as before. i meant like in some section you mentioned 'tejal did this... etc' so avoid that."*

"I build / I broke / I love" — never "Tejal is a software engineer who." It's *my* site. I should be talking.

## Part 13 — "this simple gradient won't do"

With the props stripped and the connector line gone, section transitions became the next problem. We'd replaced them with plain gradient seams. My take:

> *"this simple gradient won't do. research internet to figure out better ways to go to new pages. i have given you access to soo many tools, use em. i really really love this red different-text, we can even put this in other places where it fits subtly. ... and also the scroll bar on the right most, can we edit that? or is that a browser thing?"*

"Research internet to figure out better ways. I have given you access to so many tools, use em." *This* is the kind of prompt that works — it doesn't prescribe a solution, it delegates the research. And because I'd set Claude up with web search and design-library access, it could actually go do that.

(The scrollbar: yes, you can style it. `scrollbar-color`. Press red. Done.)

## Part 14 — The mud problem

The last big one. Four screenshots of section transitions and:

> *"all these, and the dotted one especially, it is not looking good and looks unfinished, either change it to let it mix better and i think the main issue is the color, this whole mud color is not seeming good. the red cream etc looks great but this mud feels very weird. also the last connect page gets all of a sudden red, so fix that."*

"This whole mud color is not seeming good" is a *complete* piece of feedback. I didn't know it was an OKLCH hue problem. I just knew it looked bad. That was enough.

The dark grounds on Experience and Blog were `oklch(20% 0.018 60)` — low lightness, near-zero chroma, hue 60 (yellowish). Against warm cream and press red, that near-neutral read as muddy brown. Not dark, not dramatic. Mud.

The fix that worked: make the dark ground the *dark end of the red family.* The press red is `oklch(56% 0.224 27)` — hue 27. New ground token: `oklch(17% 0.060 27)`. Same hue, modest chroma, very low lightness. A dark red-ink. Now the whole page is one hue story — cream (L95.5) → dark red-ink (L17) → vivid red (L56). The dark sections read as the dark end of the same palette, not a different theme dropped in.

The "dotted one" (a halftone screen seam) got replaced with a torn deckle edge — a hand-torn `clip-path: polygon()` with jittered depths and a one-pixel wet-red shadow along the tear. Looks like the page was printed on stock and torn to reveal the sheet beneath.

There's a second thing buried in that prompt worth pointing out, because it's how I gave feedback the whole way through. Look at the wording: *"either change it to let it mix better"* and *"this is your opportunity to try and implement a new animation, element etc. see the internet for ideas."* I'm not saying "remove this." I'm saying *here's roughly what I'd want instead — now you figure out how.* When I didn't like something, I tried to describe what I pictured in its place rather than just flagging the problem. That keeps Claude from getting *fixated* on my exact suggestion while still giving it a direction to aim at. Flexible, but pointed.

> **Two takeaways from one prompt:**
> 1. You don't need to diagnose the technical root cause to give good feedback. "This looks like mud" got Claude to the OKLCH fix faster than I could have. Describe what you *see*; let the model find the *why*.
> 2. Suggest the *replacement*, not just the deletion. "Change it to mix better, try a new element" beats "remove it" — it points without pinning.

## Part 15 — The commit strategy (and the bot that fought me)

This comes up every time I show someone the git history. The commits are backdated — real dates spread over a natural development cadence across a couple of weeks:

```bash
GIT_AUTHOR_DATE="2026-05-17T14:18:00" \
GIT_COMMITTER_DATE="2026-05-17T14:18:00" \
git commit -m "press engine — scroll spine, kinetic type, ink effects, cursor"
```

Author: `Tejal Goyal <my-email>`. Always, only. No `Co-Authored-By Claude`. No Claude mention in any commit, comment, or visible copy on the site. The footer says "DESIGNED & BUILT BY HAND IN VICTORIA, BC." That's accurate — *I* designed it, Claude executed it, and that distinction is the whole point of this post. I set this constraint at the start and was explicit about it. My portfolio, my authorship.

There was a fun fight with my own blog auto-sync bot, too. I have a GitHub Action that periodically commits updated blog data to `src/data/blog.js`. One afternoon my push got rejected — the remote had three "Update blog posts" commits the bot pushed while I was working locally. Fixed with:

```bash
git rebase --committer-date-is-author-date origin/main
```

which preserved my backdated timestamps through the rebase.

## Part 16 — What I'd tell someone starting this

**1. Write your taste constraints first, not last.** My `CLAUDE.md` says: "Desktop-first. Full animations, no performance compromises. Go all out. Bold design choices over safe ones. The portfolio is a playground, not a LinkedIn profile. Safe is wrong here." Those lines went in *after* the first failed attempt. They should've gone in first.

**2. "You are free to do what you wish" only works after you've constrained the space.** I said it [too early in Part 6](#part-6--references-a-second-attempt-and-you-are-free-to-do-what-you-wish), before any taste constraints existed, and got mush. I said the same words later, after THE PRESS concept was locked, and got great work. Freedom *within a frame* is generative. Freedom without a frame produces the average of the internet.

**3. Delegate execution, not taste.** Claude is excellent at implementing an animation you've described, finding the right OKLCH values for a relationship you've defined, evaluating library options for a pattern you want. It's *not* good at deciding what your portfolio should feel like, picking your palette from nothing, or choosing which section needs the most love. You bring that. Every time.

**4. Say exactly what's wrong, even when you don't know why — then suggest the replacement.** "This mud color is not seeming good" is complete feedback; don't wait until you can name the root cause. And when you cut something, describe what you'd put in its place instead of just saying "remove it." Direction without a leash. Claude stays flexible but aimed.

**5. If you keep rolling back, the brief is wrong — not the implementation.** I did two full rollbacks. Both times the execution was fine; the *brief* was either too open or pointing at the wrong thing. You can't implement your way out of a bad concept.

**6. The delete key matters as much as the keyboard.** The red thread, the matter.js physics, the masthead props, the halftone monogram — all technically interesting, none survived. They served "being clever," not "serving the visitor." The site got better every single time we removed something.

**7. Don't interrupt a running build — keep a notepad instead.** Sonnet sometimes ran 30–40 minutes on one turn. Stopping it mid-stream is worse than waiting. Keep a separate note open and jot ideas as they come, so when the turn ends you can dump them all at once. (I still forgot things and had to wait a turn. It happens.)

**8. The real loop: Opus plans, Sonnet builds, same chat, you watch the preview.** Crank Opus thinking high for the plan, iterate on the plan 1–2 times by commenting on it, *then* switch to Sonnet to implement. Don't ask one model for plan-and-build in a single breath — that's the move that cost me whole afternoons. And keep the live preview open while Sonnet works; *you* are the reviewer, not the model.

## Part 17 — Ship it, then let other people break it

A thing that's easy to skip: get it on a real domain and let real people poke at it. Two reasons. One, the deployed site surfaces things preview never does — real browser rendering, real screen, the actual domain. Two, you stop being able to see your own site after staring at it for weeks. Other eyes catch what yours have gone blind to.

So I deployed, then asked friends to just... use it. Where'd it feel slow, what was confusing, what looked off. Then I took their notes plus my own and fed them back to Sonnet — in as much detail as I could manage, with the same directional framing from [Part 14](#part-14--the-mud-problem): not "this is broken," but "here's what's wrong and here's roughly what I'd want instead."

The one practical annoyance: Claude Code caps you at **five images per message.** When a screenshot explained a bug faster than words could, I used the slots for those — the visual-only stuff, the "look at this spacing" things. Anything I could describe clearly in text, I just described, to save the image budget. Five is not a lot when half your site has something you want to point at. Plan your screenshots.

> **A cheap tip that punches above its price:** if you're a student, get the [GitHub Student Pack](https://education.github.com/pack). I grabbed my domain from Namecheap for about $7 a year — one frappe — and the Student Pack bundles a stack of free and discounted dev tools beyond that. [This list](https://jhaxce.github.io/student-perks/) collects a lot of what's out there; most of it rides in through the Student Pack. A personal domain is the single cheapest upgrade to how a portfolio reads.

## Part 18 — The funny bits

A few things that made me laugh re-reading the transcripts.

The MCP over-focus: *"i feel we should not put all the focus on MCP only, i see it is mentioning i make mcp a lot more than it is necessary."* You tell it what you do; it falls in love with one thing.

The double-hover bug, where project cards only expanded on the *second* hover: *"there is some settings where they expand only when i hover the mouse 2nd time, this is not good, change this."* The interaction was *trying* to prevent accidental expansion while scrolling. It did so in the most annoying way physically possible.

And the one-word follow-up after a giant paragraph about wanting an award-winning site:

> *"buttermax maybe?"*

As if Claude was supposed to know exactly which effect from which reference site I was prioritising now. To its credit... it did.

## Where it landed

As of June 2026, [tgoyal.me](https://tgoyal.me) is live and I'm happy with it. One-page React app. Cream paper, dark red-ink, vivid press red. Characters that drop into place. Real 3D cards. A scroll crawl that speeds up when you scroll fast. Torn paper edges between colour sections. A registration cursor. An APPROVED stamp in the footer.

Still on the list: a proper light-mode design pass (the architecture's ready, the design isn't), a better mobile experience (desktop was always the priority), a few more writing entries, maybe an easter egg or two in Contact.

The stack, for the curious: React 19 + Vite 7 + Tailwind v4 (OKLCH tokens throughout), GSAP 3 + ScrollTrigger for motion, Lenis 1.3 for scroll — everything hung off one shared ticker. No Framer Motion, no Three.js anymore. The whole motion system lives in a `src/press/` folder: the kinetic headline, the registration cursor, the redaction bars, the torn seam, the velocity-reactive crawl.

The site does what I wanted: it's a playground with a point of view, something you *interact* with rather than read through. If it makes one person think "I want a site like that," it did its job.

---

*Built over a few weeks with Claude (Sonnet for building, Opus for the hard thinking). Source at [github.com/tejalgoyal2](https://github.com/tejalgoyal2) if you want to see how any of it actually works. Yell at me on [LinkedIn](https://linkedin.com/in/tejalgoyal).*
