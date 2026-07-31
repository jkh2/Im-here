# I'm Here!

**Repo:** https://github.com/jkh2/Im-here
**Live demo:** https://jkh2.github.io/Im-here/ (landing page — choose "I'm traveling" to share, or "I'm finding someone" to look up a code)

A live location share for the moments you want someone to be able to find you — a road trip, a flight, a hike, picking up a kid from an event. No account, no app download, no ongoing tracking relationship. You share for a set window of time, on your terms, and then it's over.

## The idea

Turn it on before you go, share a short code with whoever you want to be able to find you, and turn it off when you're there. That's the whole product.

- **You control how long it lasts.** Pick anywhere from 15 minutes to 48 hours when you start. It stops automatically when the timer runs out, or the moment you tap "Stop sharing" — whichever comes first.
- **You control who can see you.** There's no public directory or search. The code is the only way in. Nobody can browse for people using this app; they can only look up a code they were personally given.
- **It doesn't linger.** Once a share ends, the location itself is gone — not just hidden from view. Reopening an expired link shows "this share has ended," nothing more.

## How it works

**If you're sharing your location:**
1. Open the app and tap "I'm traveling."
2. Pick how long to share (15 min – 48 hr) and how often to check in (1 min – 1 hr — more frequent drains your battery faster, less frequent is easier on it).
3. Optionally add a short note, like "Driving to Denver."
4. Tap "Start sharing." You'll get an 8-character code and a direct link — text either one to whoever you want to find you.
5. Tap "Stop sharing" any time you're done early.

**If you're looking for someone:**
1. Open the link they sent you (it takes you straight to the map), or open the app, tap "I'm finding someone," and type in the code they gave you.
2. You'll see their current location on a map, their trip note if they added one, how long ago they last checked in, and when the share is set to expire.
3. The map updates on its own as they check in — no refreshing needed.
4. Once the share ends, the same link just shows that it's over.

No sign-up on either end. Works the same in any browser, on any phone.

## Emergency SOS (fully local, no Supabase)

`sos.html` is a separate, self-contained page — it doesn't touch the sharing feature or the Supabase backend at all. Tap the button, or shake the phone three times, and after a 3-second cancel window it grabs your current location and opens your phone's own texting app with an SOS message pre-filled, addressed to contacts you've saved. You can add your name so recipients instantly know who it's from. Those contacts (and your name) live only in this browser's local storage on this device — never uploaded, never synced.

If GPS is slow, the SOS still goes out: after ~7 seconds it hands off a "I need help" message without a map pin rather than leaving you stuck on a loading screen. Triggers are guarded so a shake plus a tap (or a double-shake) can't stack multiple countdowns or double-send.

Worth knowing: no web page can actually send a text message on its own — this hands you off to your phone's native Messages app with everything pre-filled, and you tap Send there. That's a platform limit, not a shortcut we took.

Opening the page as `sos.html?auto=1` arms it immediately with no tap required, so it can be bound to a hardware shortcut — for example, on newer Samsung phones, Settings → Advanced features → Side key → Double press → Open app can be pointed straight at that URL.

**Voice control:** during the 3-second cancel window, saying "Send" fires immediately and "Cancel" aborts — no tap needed for our part of the flow. This can't extend to the actual text-sending step, though: no web page can press Send inside your phone's native texting app, by voice or otherwise, regardless of trigger method. For a genuinely zero-tap experience including the send itself, the real answer lives at the OS level, not in this app — on iPhone, Settings → Siri & Search → Automatically Send Messages lets you say "Hey Siri, text [contact], I need help, [location]" and have it actually go out with no confirmation step. Android's Google Assistant has similar hands-free sending. Worth knowing about and mentioning to people, even though it's outside what this page can control.

## Why the location isn't blurred

Since a location only ever shows up for someone who was deliberately handed the code, there's no need to fuzz or approximate it — you're already choosing exactly who gets to see it. What you're sharing is genuinely your exact position, for exactly as long as you decide.

## Battery behavior

Your location isn't tracked continuously in the background. Each check-in takes a single fresh GPS reading at the interval you chose, then goes back to sleep — a 1-hour interval really does only touch your phone's GPS once an hour, not continuously with the results just sent less often. You can change the interval mid-share without restarting anything.

## License

## License

### Free noncommercial use

The I’m Here! source code is source-available under the **PolyForm Noncommercial License 1.0.0**.

You may view, study, test, modify, and use the source code for personal, educational, research, experimental, hobby, and other noncommercial purposes, subject to the full terms contained in the `LICENSE` file.

This means you are welcome to:

* Use the project to learn how it works
* Experiment with the source code
* Modify it for personal or noncommercial purposes
* Study or demonstrate its design and functionality
* Contribute improvements or report issues

### Commercial use requires approval

The public license does not authorize commercial use.

You may not sell, commercially rebrand, white-label, host for payment, license, monetize, or incorporate this Software into a paid product or service without a separate written commercial agreement.

Commercial licenses are available only to partners approved in writing by the Licensor.

Commercial licensing inquiries:

**James Keith Harwood II**
**[jameskharwood2@gmail.com](mailto:jameskharwood2@gmail.com)**

### Branding

The source-code license does not grant permission to use the **I’m Here!** name, logo, visual identity, or other associated branding.

Modified versions must not claim or imply that they are official versions of I’m Here!, endorsed by the project, or produced by an approved partner without written authorization.

### Using the hosted app

The publicly hosted I’m Here! application is currently available for people to use without charge.

Using the hosted application does not grant any rights to commercially reproduce, rebrand, sell, or operate the underlying Software. Use of the hosted application may also be governed by separate Terms of Use and a Privacy Policy.

See `LICENSE` for the complete source-code license terms.

## Under the hood

Four static pages — `index.html` (landing), `share.html`, `find.html`, and `sos.html` — the sharing pair backed by a small Supabase project, the SOS page entirely local. No server to run, no build step; push all four files to any static host (GitHub Pages, Netlify, wherever) with `index.html` at the root so it's the front door people land on.

**Security model:** the database table behind this has no public read or write access at all. Every action — starting a share, checking in, ending a share, or looking one up — goes through a dedicated database function that requires the exact code. There's no way to list, scan, or browse every active share; the code isn't just a suggestion, it's the actual access control. Live updates travel over a private channel named after the code, so even the "live" part of live-tracking is only reachable by someone who already has it.

**Data lifecycle:** the timer is fixed the moment a share starts and doesn't reset on further check-ins. The moment a share expires or is manually ended, location lookups stop returning coordinates — not just visually marked as stale, but actually withheld. Records stick around for 48 hours after the last update (so a just-expired link can still say "this share has ended" instead of erroring out), then age out for good.

**Schema** — a single `shares` table:

| column | type |
|---|---|
| code | text, primary key |
| lat / lng | double precision |
| label | text, optional |
| status | text (`active` / `ended`) |
| expires_at | timestamptz |
| updated_at / created_at | timestamptz |
