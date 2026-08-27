# Static Bloom

A private, interactive birthday website — built as a single self-contained `index.html`
(no build step, no dependencies). Runs as a full-viewport presentation: one slide on
screen at a time, advanced only by button or an explicit dismiss (space / tap
anywhere) — never by scroll, the page can't scroll at all. Starts as a fake 404 error
page, glitches into a pastel bloom reveal, then walks through a gated, linear sequence:
a cut-the-cake moment, six candlelit letter slides, a gacha-style photocard gallery, a
booster-pack card pull (using real custom Pokémon-style cards), a spam-to-open gift
box, a rigged spin wheel, a date picker, and a closing message.

**Live preview (Claude Artifact):** https://claude.ai/code/artifact/425f0429-c6cd-4cda-b5b0-926021a3e569

## How to view it

Just open `index.html` in a browser — it's fully self-contained (all CSS/JS inline, no
external requests, no build step). To serve it locally instead of via `file://`:

```bash
npx http-server . -p 5173
```

## The flow (in order — each stage is gated behind the previous one)

1. **404 page** — fake dark error screen. Clicking "try again" triggers the reveal.
2. **Bloom reveal** — glitch transition into a pastel pink/yellow scene: flowers bloom
   in behind the headline, the photo card blows in on a noise-driven "wind" path and
   settles, then a letter-"A" made of hundreds of tiny flowers blooms in beside it
   along with a rose-colored heart, confetti, and a chime. Once that settles, a
   "cut your cake 🎂" button unlocks in place (no separate slide yet).
3. **Cut the cake** *(locked until the reveal above finishes)* — swipe to blow out
   three candles one at a time, then drag across the cake to cut it anywhere she
   likes; the two pieces slide apart along whatever line she actually drew, a confetti
   burst and a synthesized "chomp" sound fire on the cut, and "continue" unlocks a beat
   later.
4. **Letter — six slides** — one line of message text and one photo per slide. Each
   slide lights its own candle a beat after it appears, then unlocks its own
   "continue →" once that settles; the sixth slide's continue hands off to the gallery.
5. **Gallery** *(locked until the sixth letter slide's continue)* — a grid of six
   Pokémon-card-styled photocards. Each rests showing its ornate "keepsake" back; the
   first tap on any card is a one-time dramatic gacha-style pull (zooms to
   screen-center, grows large, bursts with rays/sparkles/a glow) that reveals the
   photo side and then just **stays there**, held large and focused — nothing
   auto-returns it. A "touch anywhere to continue" hint appears while it's held;
   dismissing it (space, tapping anywhere, or tapping a different card) sends it back
   to its grid slot showing the mystery back again. Every tap after the first on that
   same card is a plain quick flip in place, no re-pull.
6. **Card pack** *(locked until "continue" on the gallery)* — a foil pack pinned to the
   bottom of the screen; tapping it pulls the next of 5 real custom Pokémon-style
   cards (her actual photos, made externally and embedded as base64 JPEGs) up out of
   the pack with the same "rises, bursts, then holds until manually dismissed" pattern
   as the gallery. Dismissed cards collect into a row up top; "continue" unlocks once
   all 5 are out.
7. **Gift box** *(locked until "continue" on the card pack)* — starts as a grey
   silhouette, colors in a beat after the slide appears, then needs rapid clicking
   (with a decaying progress bar) to pry the lid open — the ribbon and bow lift open
   together with the lid as one piece.
8. **Guesses form** — after the box opens, she can list as many guesses as she wants
   about what the gift is. Submitting opens her mail app with the list pre-addressed
   to you (see **Email limitations** below).
9. **Spin wheel** *(locked until the guesses form settles)* — six options, rigged to
   always land on "secret gift 🎁" (verified via direct rotation-math computation, not
   just visually).
10. **Date picker** — on claiming the gift, she picks a date/time. Confirming offers
    her a real downloadable `.ics` calendar invite, opens her mail app with the chosen
    time pre-addressed to you, and flies a little paper airplane across the screen.
11. **Ending** *(locked until the date is confirmed)* — closing message, framed with
    a couple of the same stickers used on the timeline/gift scenes.

Every "continue" trigger (button, or an interaction like opening the gift box) is
**locked** — disabled/non-interactive — until the slide it belongs to has actually
finished its own entrance animation, not just the instant it appears. There's no
scrolling to guard against in the first place: `html,body{overflow:hidden}`, and each
slide is `position:fixed;inset:0` with only one ever unhidden (`hidden` attribute off,
`.entered`) at a time — `showSlide()` in the script hides every other slide (via a
slow fade+blur+grow "leaving" animation, driven by the Web Animations API rather than
a CSS transition — the latter turned out to be unreliable when triggered from inside a
click handler) before revealing the target one. Advancing is buttons/dismiss-gestures
only, on a delay after each slide's own entrance settles — never scroll-triggered.

Every button on the site presses in and springs back like a real slow-rising squishy
toy (fast flatten on press, slow un-bouncy rise on release), paired with a
synthesized "squish" sound; text inputs get a separate "creamy" muted keyboard-thock
sound instead, tuned to feel deliberately different from the button squish.

## What's still needed (all clearly marked `[ in brackets ]` in the HTML)

- **Her name** — used in a few places (the ending title, etc.)
- **Your name** — signs the letter and the ending
- **The actual message text** — 4 of the 6 timeline entries are still bracketed prompts
  (an opening line, something specific you love about her, a shared memory, what you
  hope this year holds)
- **Real photos** — every photo slot (timeline, gallery, the main reveal card) is
  still a placeholder gradient box with a `[ bracketed ]` label. They need to become
  real `<img>` tags (base64-embed them directly in the HTML, or host them somewhere
  and point `src` at the URL) — the photo lightbox's download button specifically
  checks for a real `<img>` and won't do anything until one exists. (The card-pack
  slide is the one exception — its 5 cards already use real photos, base64-embedded.)
- **Gallery remarks** — the one-line comment on the back of each flip card, and the
  caption below each card once revealed
- **Recipient email** — `YOUR_EMAIL` near the top of the `<script>` is currently set to
  `rzqrdzn03@gmail.com`; change it if that's not where the guesses/date should land

## Known limitations (deliberate, not bugs)

- **No real background music/silent email API.** This was originally built and tested
  as a Claude Artifact, which sandboxes the page and blocks *all* outbound network
  requests except Google Fonts — so a service like EmailJS literally cannot send
  anything from inside it, no matter the credentials. The workaround used instead:
  `mailto:` links (open her own mail app, pre-filled, she hits send) and a real
  downloadable `.ics` calendar file. **If you host this file yourself** (Netlify,
  Vercel, GitHub Pages, etc.) instead of via the Artifact link, that sandbox goes away
  and a real client-side email API becomes usable if you want fully silent sending —
  just note that would need its own re-wiring, not a config flip.
- **All sound effects are synthesized** (Web Audio oscillators/noise), not audio
  files — the same sandbox blocks loading external audio.
- **Tulip/cake/bow/heart artwork** is based on Twemoji (CC-BY 4.0, credited in the
  footer) — real illustration assets pulled via browser fetch and recolored to match
  the palette.
- **The card-pack images are already fairly heavy.** The 5 source card PNGs were
  13-21MB each straight out of the card-maker tool; downscaled to 1000px on the long
  edge and re-saved as JPEG before embedding, which brought the full set to just over
  1MB combined — reasonable for a single file, but worth knowing if you swap in
  higher-res originals later, since embedding those directly would balloon the page.

## Continuing this in a new chat

Point Claude Code at this repo. The whole site is one file (`index.html`) — read it,
it's heavily commented at the points that matter (gating logic, noise-based
animation, the wheel's rigged rotation math). If you want the live Artifact link kept
in sync, tell Claude to republish `index.html` with `url:
https://claude.ai/code/artifact/425f0429-c6cd-4cda-b5b0-926021a3e569` rather than
publishing fresh (that flag is what makes it update the *same* link instead of
creating a new one).
