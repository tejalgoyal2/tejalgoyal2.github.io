---
title: "Nib: A Menu Bar Scratchpad, an Apple Rant, and the State of AI and Swift"
date: 2026-04-30
categories: [dev, macos, vibe-coding, project]
---

I had an itch to build something. Not for any grand reason — just that feeling where you want to make a thing and see if you can. The thing I landed on was a menu bar scratchpad for macOS. Simple premise: click an icon in your menu bar, a little window appears, you type, you close it. Your text is still there next time. That's it. I called it [Nib](https://github.com/tejalgoyal2/Nib).

It shipped. It works. The road to get there taught me something genuinely interesting about AI and Swift that I keep thinking about.

<!-- MEDIA SUGGESTION: "this is fine" dog-in-burning-room — right after the intro, sets the tone before the Swift section -->

## AI is actually bad at Swift, and I think I know why

I use Claude for basically everything code-related. Python? Excellent. JavaScript? Great. Obscure bash one-liners? Surprisingly confident. I assumed Swift would be in that same bucket.

It is not in that bucket.

The code it generated was *technically plausible*. It compiled. It looked like real Swift. And then at runtime it would just... not do the thing. No crash. No error. Sometimes a silent success that meant absolutely nothing happened. I've seen this before — if you read my [OnTop post](/posts/2026/04/07/building-ontop/), you know exactly the kind of thing I mean.

My theory: there just isn't that much Swift code in AI training data. Not compared to Python or JavaScript anyway. The open-source macOS app ecosystem is smaller than people think. Apple's own documentation is gorgeous and about 40% useful. The result is an AI that has *seen* enough Swift to write something that looks right, but not enough to know what actually works at runtime on a real Mac.

It's a real problem. I don't have a solution yet, but I'm thinking about it.

## A brief, necessary Apple rant

The autocorrect on macOS is still broken. I'm writing this and it just changed "SwiftUI" to "Swift Ui" for the fifth time.

I hope by the time you're reading this they've fixed it. I'm choosing to believe they will. That's what hope is, I think.

Apple makes beautiful hardware, ships genuinely good design direction — Liquid Glass in Tahoe looks incredible — and then wraps it in an OS that occasionally just disagrees with you for no discernible reason. I spent two hours on a Stack Overflow post from 2019 trying to figure out if it still applied to a global keyboard shortcut issue. It did not. I could keep going. I won't. But I could.

## What Nib actually does

Menu bar only — no Dock icon. Click the pen nib icon, a popover appears. Type. Close it. Come back. Your text is still there.

Beyond the basics: inline Markdown renders live (not in a preview pane — right there as you type), checkboxes you can click to toggle, word and character count in the toolbar, a pin button that floats Nib above full-screen apps and follows you across Desktop Spaces, one-click copy-all, and a formatting toolbar for when you'd rather click than type syntax. The background uses `ultraThinMaterial` so it picks up your wallpaper and system theme automatically.

No cloud. No accounts. No analytics. Your text is a file sitting quietly in Application Support.

## The Notion export that didn't make v1.0

I wanted Notion export — write something in Nib, hit a button, it lands in a database. The AI-generated integration was rough enough that I cut it and shipped without it.

*v1.0 without Notion export beats v0 with a broken one.* It's on the list for the next version, along with a global hotkey and settings panel.

## The product page

I built a site for Nib at [nib.tgoyal.me](https://nib.tgoyal.me) using [Chibi](https://github.com/tejalgoyal2/chibi-design-system) — the design system I wrote about [last week](/posts/2026/04/29/chibi-design-system/). Single HTML file, dark-warm, Cormorant Garamond, interactive app mockup built in HTML because screenshots get outdated. Happy with how it came out.

## Getting it

[DMG is in the releases.](https://github.com/tejalgoyal2/Nib/releases/tag/v1.0.0) macOS will give you the "unverified developer" warning. Right-click → Open → Open. One time, never again.

I'm not paying Apple ninety-nine dollars a year to skip a dialog box. That is my line.

---

*Built with Claude (doing its best with Swift), Xcode (doing whatever it wanted), and caffeine. Repo at [github.com/tejalgoyal2/Nib](https://github.com/tejalgoyal2/Nib).*

*[LinkedIn](https://linkedin.com/in/tejalgoyal)*
