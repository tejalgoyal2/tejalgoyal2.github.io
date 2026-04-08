---
title: "The Resume System Worked. So Naturally, I Broke It Trying to Make It Better."
date: 2026-04-07
categories: [dev, vibe-coding]
---

I wrote a whole post about [building an AI resume system](/posts/2026/04/02/ai-resume-system/) that took five versions to get right. Five versions. Weeks of iteration. A companion [how-to guide](/posts/2026/04/04/build-your-own-ai-resume-pipeline/) and everything. I was *done*. The system worked. Resumes came out clean, one page, no orphan lines. I could paste a job description and have a tailored `.docx` in under a minute.

I should have stopped there.

...I did not stop there.

## The problem I didn't know I had

Here's the thing about Claude Pro — you get a usage allowance that resets every five hours. And I'd been generating resumes without really paying attention to how much each one cost. Why would I? It worked. The output was good. Life was fine.

Then one afternoon I generated a single resume and watched *ten percent* of my five-hour allowance disappear.

Ten percent. For one resume. One page. Maybe eighty lines of actual output.

I checked the previous week's usage. The same skill had been costing 3-4% per run. Something had changed, and I hadn't noticed because I was too busy admiring my beautiful one-page resumes to look at the meter.

> It's like driving a car that gets great mileage and then one day you notice the fuel gauge dropping twice as fast. The car still drives fine. The destination is the same. But something under the hood is *wrong*, and now you can't un-see it.

## The investigation

I pulled up the chat transcript from the last generation and started counting tool calls. A "tool call" is every time the model reads a file, writes a file, runs a script, or executes a command. Each one costs tokens. Tokens cost usage.

The count: **thirty-five tool calls** for a single resume.

Thirty-five.

I stared at the transcript. The model was reading every reference file — some of them twice. It was reading the master resume, then reading the rules, then reading the master resume *again* because the rules referenced something it wanted to double-check. It was running the build script, hitting a validation error, fixing the error, running the script again, hitting a *different* validation error, fixing *that*, running it a third time...

Each read, each write, each script execution — a tool call. Each tool call — tokens. Each token — usage. Thirty-five of them stacking up like a bar tab you forgot to close.

The model was doing exactly what I'd told it to do. The problem wasn't the model. The problem was that my instructions sent it on a scavenger hunt across seven files before it could write a single line of output.

## Version 6: Fewer files, fewer reads

The first fix was straightforward — stop making the model read things it doesn't need.

I had rules duplicated across three files: the SKILL.md instructions, a separate rules reference document, and embedded inside comments in the master resume data files themselves. The model was reading all three, sometimes finding slightly different phrasings of the same rule, and spending tokens reconciling them.

Sound familiar? This was the same duplication problem I thought I'd solved in version 5. I had. For the *content*. But the meta-instructions — the "how to use this skill" preamble, the "remember to check character counts" reminders — had crept back in. Like weeds. You pull them and they grow back in a different file.

I gutted everything. One instruction file. One. The master resumes became pure data — no comments, no guidance, no "when tailoring, consider..." blocks. The reference file got slimmed to just a verb list. Nothing else.

Tool calls dropped from 35 to about 20. Better. Not great. The model was still running the validator, failing, fixing, and re-running multiple times.

## Version 6.1: The validate-then-build loop

The validation script and the build script were separate files. The workflow was: model writes data → model runs validator → validator prints errors → model fixes errors → model runs validator again → if clean, model runs build script → build script generates `.docx`.

Every run of the validator was a tool call. Every run of the build script was a tool call. A resume with three validation errors meant three validator runs plus one build run — four tool calls just for the "check and compile" phase. And the model had to read the validator output, interpret it, make changes, and try again each time.

The fix was merging them. One script: `validate_and_build.js`. It validates *and* builds in a single execution. If validation fails, it prints the errors and stops — no `.docx`. If validation passes, it builds immediately. One tool call for the whole cycle.

```
Before: validate → fix → validate → fix → validate → build  (6 calls)
After:  validate_and_build → fix → validate_and_build        (2 calls)
```

The validator itself got smarter too. Hard fails for the things that actually break resumes: bullets over 210 characters, em dashes (which render inconsistently in `.docx`), duplicate action verbs across bullets, and missing required fields. Soft warnings for the orphan danger zone — bullets between 106 and 189 characters that *might* wrap badly but aren't guaranteed to. The model sees the warnings but doesn't have to fix them to get a build.

This was the version where it finally clicked that validation errors aren't just quality checks — they're *token costs*. Every error the model has to fix is another round trip. Catching more issues on the first pass means fewer iterations means fewer tool calls means less usage. Quality and efficiency pointing in the same direction.

...which sounds obvious written down. But it took me two versions to figure out. Classic.

## Version 7: Eight tool calls

Version 7 was the cleanup pass. The SKILL.md got rewritten to be more explicit about the workflow — not "here are some guidelines" but "do these steps in this order, do not deviate."

The model's job became almost mechanical:

1. Read the job description (already in context — zero tool calls)
2. Read the right master resume (one tool call)
3. Write `resume_data.js` (one tool call)
4. Copy `validate_and_build.js` to working directory (one tool call)
5. Run it (one tool call)
6. If errors, fix and re-run (one-two more tool calls)
7. Copy final `.docx` to output (one tool call)

Total: **roughly eight tool calls**. Down from thirty-five. The usage went from 10% back to about 3%.

I was... unreasonably proud of this. Like "told multiple people about it at dinner" proud. Nobody cared. They were right not to care. But *I* cared. The numbers were beautiful.

## Version 7.1: The one-line patch

And then the very next resume I generated broke.

```
Error: Cannot find module './resume_data.js'
```

The build script couldn't find the data file. But the data file was *right there*. I could see it. In the directory. Existing. Being a file. Doing file things.

The problem: the model was running the script from a different working directory than where it wrote the data file. `validate_and_build.js` was looking for `./resume_data.js` relative to wherever `node` was invoked from — not relative to where the script lived.

The fix was one line:

```javascript
process.chdir("/home/claude");
```

At the top of the script. Before anything else runs. Forces Node to operate from the right directory regardless of where the model invokes it from.

One line. That's v7.1. That's the entire changelog. I almost didn't version-bump it. But a fix that prevents a guaranteed failure on certain invocation paths felt like it deserved a number.

## The part where I try to be philosophical about this

There's a version of this story where I say something wise about knowing when to stop. About how v5 was good enough and the rest was vanity. About how the real lesson is being happy with what you have.

...and honestly, there's truth in that. V5 worked. The resumes were good. If I'd stopped there, I'd have more free time and the same career outcomes.

But v7 uses a quarter of the tokens. The resumes build in one pass instead of three. The error messages are clearer. The whole system is *tighter* — and I understand it better because I tore it apart and rebuilt it. Twice.

So I don't know. Maybe the lesson is that optimization is its own kind of building. Or maybe the lesson is that I will always find a reason to keep tinkering when I should be applying to jobs. One of those. Probably both.

There goes my career as a motivational speaker.

## The numbers

| Version | Tool calls | Usage per resume | What changed |
|---------|-----------|-----------------|--------------|
| v4-v5 | ~35 | ~10% | Worked, but expensive |
| v6 | ~20 | ~6% | Deduplicated rules, fewer file reads |
| v6.1 | ~14 | ~4% | Merged validate + build scripts |
| v7 | ~8 | ~3% | Explicit workflow, minimal reads |
| v7.1 | ~8 | ~3% | One-line working directory fix |

If you want to see how the system was built from scratch, the [companion guide](/posts/2026/04/04/build-your-own-ai-resume-pipeline/) covers the architecture. This post is the sequel nobody asked for — the part where a working system gets faster for reasons that only matter to the person paying the token bill.

Which is me. So it matters to me.

---

Find me on [LinkedIn](https://linkedin.com/in/tejalgoyal) — especially if you're hiring. My resume was generated by a system that took eight versions to build, and it'll arrive in your inbox in under thirty seconds. The system, not me. I take longer.

*Optimized with Claude Skills and an unhealthy attention to tool call counts · April 2026*
