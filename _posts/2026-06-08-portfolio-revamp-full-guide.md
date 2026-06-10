---
title: "Building tgoyal.me With Claude: The Full Guide, Every Prompt Included"
date: 2026-06-08
categories: [dev, vibe-coding, project]
---

The first time I gave Claude total creative freedom over my portfolio, it produced the most average website I have ever seen. The second time, I handed it three reference sites I genuinely love and it ignored all of them. The third attempt became [tgoyal.me](https://tgoyal.me), and the difference between the three attempts is the entire point of this post.

This is the real story with the real prompts, typos and meltdowns included, because that is the most useful thing I can hand you. Anyone can tell you what to build. Fewer people show you how they actually talked to the AI to get there, especially the parts where it went badly.

If you only want the method, the next section is the part to steal. Everything after it is what happens when you actually use it.

## The loop: one chat, two models

Everything ran inside the Claude Code desktop app, in one long chat, switching between two models and two modes. I barely touched regular Claude chat for this project.

The loop: plan with Opus in plan mode, switch to Sonnet with accept edits, let it build, look at the preview, go back to plan mode with Opus, repeat. Plan, code, plan, code, for weeks. Then deploy.

Why one chat instead of fresh sessions: the plan and the build need to share context. Opus writes the plan knowing the whole codebase, and Sonnet implements it with that same history sitting right above it. Switching the model is not starting over. It is handing the same conversation to a different brain.

The mechanics that took me a while to figure out:

**Make the plan do the thinking.** I told Opus, explicitly, to make the plan detailed enough that Sonnet does not need to think, just implement. The expensive reasoning happens once, up front, and every decision gets baked into the plan. Sonnet then executes a spec instead of inventing one, which also means it wanders off less.

**Comment on the plan instead of accepting it.** When Claude Code presents a plan, you can accept it or make changes first. If I accepted, Opus would start implementing immediately, on Opus, the expensive way. So I read the plan top to bottom and dropped a comment wherever something was off ("no, this is not what I had in mind"), added any general ideas to the prompt, and sent it back for another planning pass. Usually one or two rounds and the plan was right. Only then did I switch the model to Sonnet, flip to accept edits, and tell it to follow the plan. The model swap is the go signal.

**When you get disconnected, paste the plan back.** I got disconnected a lot. Just saying "continue" makes things fuzzy. What worked: copy the whole plan back into the chat and say "go on, keep working on this." Tedious. Also the reason the build never drifted.

**Which models, for the record.** Opus 4.8 with thinking on high for planning. Sonnet 4.6, also on high, for building. I did try Opus for implementation early on and dropped it fast: the output was not meaningfully better at executing an already detailed plan, and it ate my usage. Anthropic has a [guide on picking models](https://claude.com/resources/tutorials/choosing-the-right-claude-model) that lands in the same place. I am on the Pro plan, for context.

<!-- MEDIA SUGGESTION: a screenshot of Claude Code plan mode with one of your inline comments on a plan line. It makes "comment, don't accept" instantly concrete. -->

## Attempt one: total freedom, total slop

What I had before this all started was a perfectly decent portfolio. Dark lavender theme, a 3D particle field in the hero you could push around with your cursor, smooth scroll, animated sections. Professional and polished, and that was the ceiling. Every section was some variation of a card grid. Zero personality.

So I opened a fresh session and gave Claude the dream brief: full visual redesign, start from zero, pick a direction, design every section with a distinct identity.

It picked "soft brutalism." Clean, structured, legible, and completely soulless. My response, verbatim:

> "you know what, this lacks so much character. the old portfolio was way way ahead of leagues, with moving 3d cards, blog cards moving, the hero section, now it is more ai slop actually. [...] and this color itself screams ai slop. i am written by a shitty ai bot. the old lavender theme the glow everything was so so so so leagues ahead, i thought this would be useful but this is so shitty. can you do anything about this. not one element i like about this."

Then the rollback:

> "i want you to remove all these changes done. fully roll back to the state we were before. drop everything. and just put a md file documenting what you tried and it went wrong what i said i did not like etc."

Back to lavender. And the failure was mine, not the model's. "Design a portfolio" with no constraints produces the statistically most average portfolio, because the average is exactly what the model samples from when you give it nothing else. I had already learned a version of this lesson building [Chibi](/posts/2026/04/29/chibi-design-system/), where the whole idea was that a defined visual language exists so the AI is not guessing. I just had not connected it to this project yet.

## Attempt two: references everywhere, none used

New approach: give it taste through references. I had three sites I genuinely love. [buttermax.net](https://buttermax.net), with a magnetic cursor that deforms like fluid. [igloo.inc](https://igloo.inc), with its 3D scroll effects. [drumspirit.be](https://drumspirit.be), with fluid section morphing. Plus screen recordings of specific effects I wanted, plus some Spider-Verse comic energy. I sent all of it.

The result used none of it.

> "man i want to say, i am so so disappointed right now. i gave so many so many good ideas and references and (attaching the screenshots) this is what you came up with? there was not a single feature i think you used."

So I sent the prompt I had been holding back:

> "I am feeling like you are trying very hard to stick to the current code, the current repo or the current design. if this is difficult, i can say that start from scratch, use these amazing references, [...] search the web for award winning design. i believe we can match the level of a fully well designed award winning website. so YOU ARE FREE TO DO WHAT YOU WISH. GIVE ME YOUR BEST."

Remember that line, because it failed here and worked later, and the difference is entirely about when you say it.

This era also produced a throwaway practical note that became a permanent rule, after Claude ran the dev server and hung my MacBook so badly the battery was draining while plugged in:

> "also keep work and do not run the dev server after each change as i just saw, it hanged my macbook pro real bad."

`npm run dev` starts a server that never exits. Claude runs it, waits for it to finish, and it never finishes. Into the CLAUDE.md it went:

```text
NEVER run npm run dev in this project, it hangs the machine.
Use npm run build (it exits clean) then the preview tool.
```

Whatever your equivalent long-running command is, ban it on day one.

## The session where it clicked

After a few more sessions of improving-but-not-crystallising, I did the thing that finally worked. I opened a clean session, told Claude to ignore every earlier attempt, and asked it to only think, not build.

> "so we are now redesigning and not implementing. so think think and think. I have given you all the tools.... i started this new chat to not bring any old trash with this. [...] I do not care about the usage, i want the monument of a portfolio filled with character, animation, elements, a blessing to looks and interact with."

This is the plan-then-build split earning its keep. A planning-only session, Opus on high, no file edits allowed. Just thinking. And the concept that came back was the whole game.

THE PRESS. A living risograph broadsheet that prints itself as you scroll. Kinetic type that falls and slams into place. A registration crosshair cursor. Redaction bars that reveal like declassifying documents. A whole editorial, letterpress print metaphor.

I read the plan. I said do it. And the same words that produced mush in attempt two now produced the best work of the project, because this time there was a frame to be free inside of.

## Watching the build, dumping the feedback

Sonnet kept a visual preview running the whole time it worked, and I kept it open and watched. Not just the end result. I would glance at the page as it changed, skim the copy, and sometimes read the thinking to see what it was trying to do. The reviewer was me looking at the site, not the model grading itself.

When I caught problems, I did not interrupt. Sonnet sometimes ran 30 to 40 minutes on one turn, and stopping it mid-build is worse than waiting. I kept a separate note open and wrote ideas down as they came, because I would absolutely forget them otherwise. Then when the turn ended, I dumped everything in one message:

> "what established in 2017? i mean i was born in 2002? [...] the grab things is a bit phoney. [...] The project card expansion is breaking the red line going. fix that. [...] the contact or whatever section that was supposed to be after the blog is just a red screen. there is nothing."

Six problems in one message, each specific enough to act on. The copy claimed I was established in 2017. I would have been fifteen. The "grab things" was a physics simulation in the Skills section where you could pick up letters and throw them around. Fun for ten seconds, unreadable as a skills section, and it got cut along with a lot of other clever ideas: the fake newspaper masthead, a dispatch ticker, a halftone monogram, a vertical red connector line that jerked every time a card expanded. The site got better every time something was removed.

The other thing that needed correcting: the copy kept latching onto one part of my work and overweighting it, and it had started referring to me in the third person. "Tejal did this." It is my site. I should be talking. First person everywhere, and that rule stayed.

One more pass came from this prompt:

> "and this is going a bit too towards newspaper, i do not even have anything much to do with that. so rethink a bit."

The press concept was generating great decisions about typography, motion, and palette. But the literal newspaper props were making the theme the subject instead of me. The call we landed on: keep everything that came from a good design principle that happened to use the press metaphor, cut everything that required the visitor to understand the newspaper reference to make sense.

<!-- MEDIA SUGGESTION: a short clip or GIF of the kinetic headline characters dropping into place on the live site. This is the signature effect and no amount of prose replaces seeing it. -->

## The mud problem

The last big fight was color. I sent four screenshots of section transitions with this:

> "all these, and the dotted one especially, it is not looking good and looks unfinished, either change it to let it mix better and i think the main issue is the color, this whole mud color is not seeming good. the red cream etc looks great but this mud feels very weird."

I did not know what was technically wrong. I just knew it looked like mud. That was enough. The dark backgrounds were `oklch(20% 0.018 60)`: low lightness, near-zero chroma, yellowish hue. Against the warm cream and the press red, that near-neutral read as muddy brown. The fix was to make the dark ground the dark end of the red family instead. The press red sits at hue 27, so the new ground became `oklch(17% 0.060 27)`: same hue, modest chroma, very low lightness. A dark red ink. Now the whole page is one hue story, cream to dark red-ink to vivid red, and the dark sections read as the dark end of the same palette instead of a different theme dropped in.

Notice the wording in that prompt, though, because this is how I gave feedback the whole way through. Not "remove this." Instead: here is roughly what I want in its place, now you figure out how. When I did not like something, I described what I pictured instead of prescribing the fix. That keeps Claude from fixating on my exact suggestion while still giving it a direction to aim at.

<!-- MEDIA SUGGESTION: before/after screenshots of the mud seam vs the dark red-ink ground. The color story is hard to appreciate in words. -->

## The bug that only broke in production

One morning the Cloudflare Pages deploy started failing with `Could not resolve entry module "three"`. The cause: the Vite config was still splitting `three` and `framer-motion` into separate build chunks, except neither package was installed anymore. The dev server is lazy and never resolved them, so locally everything worked. The production build actually bundles everything, hit the phantom modules, and choked. The fix was deleting two lines.

The lesson that stuck: what runs locally is not what ships. Which leads to the last step.

## Ship it, then let people break it

Get it on a real domain early. The deployed site surfaces things the preview never does: real browser rendering, a real screen, the actual domain. And after staring at your own site for weeks you stop being able to see it, so I asked friends to just use it and tell me where it felt slow, confusing, or off.

Their notes plus mine went back to Sonnet in as much detail as I could manage, screenshots attached. One practical annoyance: Claude Code caps you at five images per message. I spent the slots on things a screenshot explains faster than words, and described everything else in text. Five is not a lot when half your site has something you want to point at.

## What I'd tell you to steal

1. **Write your taste constraints first, not last.** My CLAUDE.md says things like "Bold design choices over safe ones. The portfolio is a playground, not a LinkedIn profile. Safe is wrong here." Those lines went in after the first disaster. They should have gone in before it.
2. **"Do what you wish" only works inside a frame.** Said with no constraints, it produces the average of the internet. Said after a strong concept is locked, it produces the best work. Same words, different timing.
3. **Delegate execution, not taste.** Claude is excellent at implementing an animation you describe, finding color values for a relationship you define, researching options for a pattern you want. It cannot decide what your site should feel like. You bring that part every time.
4. **Describe what you see, suggest what you want, skip the diagnosis.** "This looks like mud" plus "make it mix better" got to the right fix faster than I could have. You do not need the technical root cause to give complete feedback.
5. **If you keep rolling back, fix the brief, not the implementation.** Both of my full rollbacks were brief problems. You cannot implement your way out of a wrong concept.
6. **Plan with the expensive model, build with the fast one, same chat.** And do not ask one model for the plan and the build in a single breath. That mistake cost me whole afternoons.

## Questions I actually get asked

**Do I need to know how to code?** You do not need to write the code, but you need to be able to judge what comes back and you need enough git to do a rollback without panicking. The job is taste and review. Claude does the typing.

**What did this cost?** A Claude Pro subscription, which I already had, and about $7 a year for the domain from Namecheap. Hosting was free. If you are a student, the [GitHub Student Pack](https://education.github.com/pack) bundles a lot of free dev tools, and [this list](https://jhaxce.github.io/student-perks/) collects most of what is out there.

**Where do I host it?** For a static site, GitHub Pages or Cloudflare Pages. Both are free and both redeploy automatically when you push to the repo. This blog runs on GitHub Pages; tgoyal.me runs on Cloudflare Pages. Point your domain at either and you are done.

**Is it safe to let an AI run commands on my machine?** Claude Code asks before running things, and you control how much it is allowed to do on its own. The real risks in my experience were boring ones: a long-running command that hangs the session (ban those in your CLAUDE.md), and secrets or personal info ending up in a repo that later goes public. Review what is in the repo before publishing it, and never paste API keys into the chat.

**What about the site itself being a security risk?** A static portfolio has no backend, no database, and no login. There is close to nothing to attack. This is one of the few projects where you can vibe code with a clear conscience.

**How long did it take?** A few weeks of evenings, and most of that time was me reviewing and giving feedback rather than waiting on the model. A simpler site would take days.

**Won't it look AI generated?** Only if you give it nothing to work with. That is what attempt one was. The slop is not in the tool, it is in the empty brief.

## Where it landed

As of June 2026, [tgoyal.me](https://tgoyal.me) is live and I am happy with it. Cream paper, dark red-ink, vivid press red. Characters that drop into place. 3D cards. A crawl that speeds up when you scroll fast. Torn paper edges between color sections. A registration cursor. An APPROVED stamp in the footer.

For the curious, the stack is React 19, Vite 7, and Tailwind v4 with OKLCH tokens throughout, GSAP and ScrollTrigger for motion, Lenis for scroll, all hanging off one shared ticker. The whole motion system lives in one folder: the kinetic headline, the cursor, the redaction bars, the torn seam, the crawl.

Still on the list: a proper light mode pass, a better mobile experience, and maybe an easter egg or two in Contact. The site does what I wanted. It has a point of view, and it is something you interact with rather than read through. If it makes one person think "I want a site like that," it did its job.

---

*Built over a few weeks with Claude, Opus for the hard thinking and Sonnet for the building. Source at [github.com/tejalgoyal2](https://github.com/tejalgoyal2). Find me on [LinkedIn](https://linkedin.com/in/tejalgoyal).*
