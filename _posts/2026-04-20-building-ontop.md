---
title: "Building OnTop: Three Broken APIs and One Very Clever Workaround"
date: 2026-04-20
categories: [dev, macos, vibe-coding]
---

I'm taking Anthropic's 101 course and one of the things they encourage is just *building stuff* to understand the tools. So I decided to build something I actually wanted: a utility to keep a window always on top of everything else on macOS. Terminal above Xcode. Reference doc above my editor. That kind of thing.

I found a project called [AlwaysOnTop](https://github.com/itsabhishekolkha/AlwaysOnTop) by @itsabhishekolkha and used it as a reference point. It works by using AppleScript to *activate* the target app every second. That's... a lot. It's slow, it steals focus, and it feels like a hack. Surely there's a proper API for this?

There isn't. But I didn't know that yet.

## What vibe coding actually feels like

I used [Claude Code](https://claude.ai/claude-code) for this entire project. I want to be honest about what that experience is: it's genuinely fast, it's genuinely useful, and it's not magic.

The way it worked: I described what I wanted ("always on top menu bar app, max 3 windows, priority levels, no dock icon, sound feedback"), Claude planned the architecture, wrote the code, and I reviewed and ran it. When things broke, we debugged together. When an approach didn't work, we pivoted.

The "vibe" part is real — you're moving faster than you could alone, especially if you're new to a platform. But the thinking is still yours. Claude doesn't know macOS's undocumented quirks from memory. It makes reasonable hypotheses, same as any engineer would. You still have to evaluate what it produces, test it, and push back when something's wrong.

What you get is a very capable pair programmer who types faster than anyone you've met.

## The API hunt

The core challenge: how do you keep a window you don't own above all other windows? On macOS, every window has a *level* in the global window stack. Normal app windows are at level 0. The menu bar is at 24. Screen savers are at 1000. There's a `kCGFloatingWindowLevel` (3) that's above normal apps but below the Dock. Set a window's level to 3 and it floats above everything.

The problem: the public API for setting window levels (`NSWindow.level`) only works for windows your process owns. For windows owned by other apps, there's nothing in the public SDK.

### Attempt 1: CGSSetWindowLevel

There's a private CoreGraphics function: `CGSSetWindowLevel`. You can call private APIs on macOS without a sandbox (we're distributing on GitHub, not the App Store). Swift's `@_silgen_name` lets you bind to any symbol by name:

```swift
@_silgen_name("CGSSetWindowLevel")
func CGSSetWindowLevel(_ connection: Int32, _ windowID: CGWindowID, _ level: Int32) -> Int32

let result = CGSSetWindowLevel(CGSMainConnectionID(), windowID, 3 /* floating */)
// result = 0  ← success!
```

It returned 0. Success! Except... the window didn't move. We added diagnostic logging: read the window's layer from `CGWindowListCopyWindowInfo` before and after the call.

```
layerBefore = 0
layerAfter  = 0
```

The WindowServer accepted the call, returned success, and silently did nothing. It turns out `CGSSetWindowLevel` validates that your CG connection *owns* the target window. If it doesn't, the call is silently ignored. This is not documented anywhere.

> Apple's sandboxing is deep. Not just at the process level — right down in the compositor. Even private APIs respect window ownership.

### Attempt 2: kAXRaiseAction

The Accessibility framework operates with elevated WindowServer trust — that's why you need to grant Accessibility permission to any app that does window manipulation. One of the things the AX framework can do is `kAXRaiseAction`: bring a foreign window to the global Z-front.

```swift
AXUIElementPerformAction(windowElement, kAXRaiseAction as CFString)
```

This worked! Across two monitors — if a Terminal was pinned on my external display and I switched to a different app on my laptop screen, the Terminal stayed in front.

Same screen: broken. When I clicked a different app on the same display, macOS brought that app's windows forward — and it finished doing so *after* our raise fired. We'd raise the pinned window, then macOS would put the newly active app on top. Race condition, and we always lost.

We tried a 120ms delayed second raise. It helped a little and felt awful. The fundamental problem: we were fighting the WindowServer for Z-order, and the WindowServer always wins.

### Attempt 3 (the one that works): Own your window

The realization: we can't change another app's window level. But we can create a window at any level we want — because we own it.

For each pinned window, we create a companion `NSWindow` — a borderless, click-through overlay at `kCGFloatingWindowLevel` — that shows a live capture of the original window. The original window keeps handling all input; the overlay just shows pixels.

```swift
// OverlayWindow.swift
final class OverlayWindow: NSWindow {
  init(windowID: CGWindowID, level: NSWindow.Level) {
    super.init(contentRect: .zero, styleMask: [.borderless],
               backing: .buffered, defer: true)
    self.level = level               // permanent — we own this window
    self.ignoresMouseEvents = true   // click-through to whatever's behind
    self.isOpaque = false
    self.backgroundColor = .clear
    self.collectionBehavior = [.canJoinAllSpaces, .stationary, .fullScreenAuxiliary]
    self.orderFrontRegardless()
  }
}
```

The level is set once. The WindowServer never overrides it because it's our window. When the user clicks another app, that app's windows come forward — but they go to level 0, and our overlay is at level 3. No race. No polling.

The live capture uses `CGWindowListCreateImageFromArray`. This API captures a window's backing buffer — the offscreen copy macOS keeps for every window regardless of whether it's visible. Even if the original window is completely hidden behind other apps, we can still read its pixels. We refresh at 10fps (100ms timer).

```swift
let cgImage = CGWindowListCreateImageFromArray(
  .null,             // no bounds restriction
  [windowID],        // just this one window
  .bestResolution    // Retina-quality
)
```

## What I learned about Claude Code

A few things stood out after building this:

**It's good at architecture.** The initial structure — PinnedWindowsStore as source of truth, WindowTracker for detection, PinningEngine (later OverlayWindowManager) for the mechanism, MenuBarController for the UI — came out clean. Better than what I'd have designed starting from scratch.

**It's honest about uncertainty.** When I asked "will CGSSetWindowLevel work cross-process?", Claude said it's a hypothesis worth trying, not a guarantee. That's the right answer. It didn't make up confidence it didn't have.

**Debugging is collaborative.** When the window levels weren't changing, Claude suggested adding diagnostic logging (reading `CGWindowListCopyWindowInfo` before and after the call). That's exactly what you'd do with a human pair programmer. We diagnosed the problem together.

**Platform knowledge matters.** Claude knows Swift well and macOS APIs at a broad level. But undocumented behaviors — like CGSSetWindowLevel's silent ownership check — aren't in any documentation, so they aren't in the training data either. You still have to run the code and reason about the results.

## The Apple API rant (brief version)

Every major OS has a "window always on top" concept. Windows has `HWND_TOPMOST`. Linux window managers have `_NET_WM_STATE_ABOVE`. macOS has... nothing. No public API for this. You can't do it through the sandbox. You can barely do it outside the sandbox.

The end result of our three-attempt journey is an app that:
- Requires Accessibility permission (to *detect* which window to pin)
- Requires Screen Recording permission (to *show* the pinned window)
- Doesn't actually change the pinned window's level at all
- Shows you a *picture* of the window at floating level instead

It's technically a lie. The "pinned" window is still at level 0. We're displaying a photo of it that's at level 3. But it works — and it works better than the alternatives.

If Apple ever adds a public `NSWindow.pinAboveAll()`, the entire `OverlayWindowManager.swift` becomes one line of code. Until then: we have 200 lines and a permission dialog.

## Try it yourself

The code is on GitHub: [tejalgoyal2/OnTop](https://github.com/tejalgoyal2/OnTop). The commit history tells the whole story — you can see the CGSSetWindowLevel attempt, the kAXRaiseAction detour, and the final overlay approach as separate commits.

If you're taking Anthropic's 101 course or just starting with Claude Code: build something you actually want. The constraints of a real project teach you more than any tutorial.

---

*Built with Claude Code (Anthropic) · Posted April 2026*
