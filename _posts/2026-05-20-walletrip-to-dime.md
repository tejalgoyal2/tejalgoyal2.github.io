---
title: "WalletRIP Had a Security Problem. I Didn't Fix It. Here's the Full Story."
date: 2026-05-20
categories: [dev, security, project]
---

In early 2025, a vulnerability in Next.js middleware was publicly disclosed that allowed attackers to bypass authentication entirely — regardless of what your auth logic said. You'd check for a valid session, it would pass, and a crafted request could skip the whole check. [CVE-2025-29927](https://www.picussecurity.com/resource/blog/cve-2025-29927-nextjs-middleware-bypass-vulnerability). It was bad, it got patched, and it was the kind of thing that makes you look at your own projects and think *"okay, what else is wrong here."*

WalletRIP was on Vercel. WalletRIP used Next.js middleware for auth. WalletRIP had... other problems too, independent of the CVE. I didn't fix any of it. I want to be honest about why, because "I was busy" is true but it's not the whole picture.

## What WalletRIP actually was

[WalletRIP](https://github.com/tejalgoyal2/WalletRIP) was my first real full-stack project. I built it in December 2025 in roughly two weeks, deployed it to Vercel, and used it daily. It worked. Natural language expense logging, Gemini parsing, Hinglish roasts, subscription detection — all the features I wanted. The code was messier than I'd write today, but it ran.

The security posture was... let's call it optimistic.

The biggest issue: the client was writing directly to Supabase. When you logged an expense, your browser made a Supabase insert call with the anon key. Row-level security meant you could only write to your own data — so it wasn't catastrophic — but there was no server-side validation. Whatever Gemini returned, the client trusted and inserted. Malformed amounts, unexpected categories, whatever. It went straight into the database.

The rate limiter was in-memory. I had a `Map` in module scope that tracked requests per IP. On a real serverless platform, every cold start creates a fresh process. Every fresh process has an empty Map. The rate limiter reset approximately constantly. It was not a rate limiter. It was a rate limiter-shaped comment in the code.

The `withRetry` function had an off-by-one error that made it run four attempts instead of three. Minor, but it's the kind of thing that tells you the code wasn't reviewed carefully.

`select('*')` everywhere. No column projection. Every fetch returned every field including any future ones — a pattern that leaks data you didn't mean to expose.

There was also an invite code validation endpoint with no rate limiting of its own. Five guesses a minute indefinitely. The invite code was short.

None of this was catastrophic for a personal app where I'm the only real user. But "probably fine for just me" is not a real security posture.

## Why I didn't fix it

Here's the honest answer: it was winter, I had a co-op lined up, I was watching Formula 1 and *The Boys*, and fixing security debt in a working personal project that only I use felt like exactly the kind of thing I could do later.

This is not the heroic "I identified the problems and immediately rewrote everything" story. I identified the problems and wrote them down in an audit report that I did not act on for several months.

The Vercel CVE was the thing that moved it from "later" to "now." Not because WalletRIP was actively exploited — it wasn't, as far as I know — but because looking at the CVE made me look at the rest of the code, and looking at the rest of the code made me realize I didn't want to spend time patching a codebase I wasn't proud of. The architecture was the problem. A patch wouldn't fix architecture.

So I rewrote it.

<!-- MEDIA SUGGESTION: The "this is fine" dog meme. Literally the correct image for this section. -->

## Dime vs WalletRIP: the actual differences

[Dime](https://dime.tgoyal.me) is the rewrite. Same concept, different everything under the hood.

**Writes go through the server.** The client never touches Supabase directly. Every mutation hits an API route that checks auth, validates with Zod, and then writes. This means actual server-side validation on every request — not "trust whatever Gemini returned."

**Rate limiting that works.** Upstash Redis, sliding window, persistent across Workers isolates. 20 requests per minute on AI routes, 5 per 15 minutes on invite validation. Numbers I actually thought about.

**TypeScript strict, Zod v4 everywhere.** No `any`. Shared schemas between client and server so the shape of data is checked at the boundary, not just assumed.

**Origin checking.** API routes validate the `Origin` header. Not bulletproof on its own but part of a defense-in-depth approach.

**Column projection.** `select('id, item_name, amount, category, type, date, emoji')` — not `select('*')`. Explicit about what comes back.

**Cloudflare Workers instead of Vercel.** This wasn't primarily a security decision — I wanted the edge performance and I wanted to learn the Workers deployment model. But it does mean the middleware CVE class of vulnerability is someone else's problem now.

The full technical walkthrough is in [WALKTHROUGH.md](https://github.com/tejalgoyal2/Dime/blob/main/WALKTHROUGH.md) in the Dime repo. Every error I hit during the rewrite, documented in order, including the `runtime = "edge"` trap on Cloudflare Workers that caused silent 500 errors with no stack trace for two hours.

## The Vercel middleware thing specifically

CVE-2025-29927 worked by manipulating the `x-middleware-subrequest` header. Next.js middleware uses this header internally to track subrequest depth and avoid infinite loops. An attacker could send a crafted request with this header set in a way that caused the middleware to skip its own execution — including auth checks.

The patch was straightforward (Vercel deployed it quickly), but the disclosure pattern was uncomfortable: the vulnerability was in the framework itself, not in application code. You could write correct authentication logic and still be vulnerable because the layer that ran your auth logic could be bypassed before your code even ran.

WalletRIP's auth wasn't particularly sophisticated — Supabase cookie sessions, checked in middleware, redirect to login if no session. Exactly the pattern the CVE targeted. Would it have been exploited? Probably not. It's a personal expense tracker with no interesting data. But "probably not" isn't the same as "couldn't have been."

The more useful frame: the CVE is a good reason to audit, not a reason to panic. If your middleware does auth and you were on an unpatched version, patch it and then look at everything else. That "look at everything else" step is where I found the actual problems in WalletRIP.

## What I'd tell someone in the same situation

You're going to build a v1 that has security problems. This is fine. The alternative is never shipping anything.

What you should do: write down the problems. Keep an audit document. Don't let "I'll fix it later" mean "I'll never think about it again." WalletRIP had an `AUDIT_REPORT.md` in the repo that documented exactly what was wrong. I wrote it. I just didn't act on it for a while. At least having it written meant I knew what I was looking at when I finally did.

Also: in-memory rate limiting on serverless is not rate limiting. I feel strongly about this.

The other thing: rewrites are sometimes the right call. Not always — "let's rewrite it" is a classic way to spend six months rediscovering why the original decisions were made. But when the architecture is the problem and you're not proud of the codebase, a clean start with clear constraints produces something better. Dime is better than WalletRIP. Not because I'm better at writing code now (though maybe), but because I started with a clearer picture of what I was building and why.

WalletRIP is still live. It'll stay live. It's a useful artifact of what v1 looks like — and occasionally I use it to remember that "it works" and "it's good" are not the same thing.

---

*The Dime intro post, if you haven't read it, is [here](/posts/2026/05/17/dime/). The full build walkthrough — every error, every fix, in order — is in the [repo](https://github.com/tejalgoyal2/Dime). · May 2026*

*[LinkedIn](https://linkedin.com/in/tejalgoyal)*
