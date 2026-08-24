# Static Bloom

A private, interactive birthday website — built as a single self-contained `index.html`
(no build step, no dependencies). Runs as a full-viewport presentation: one slide on
screen at a time, advanced only by button (never by scroll — the page can't scroll at
all). Starts as a fake 404 error page, glitches into a pastel bloom reveal, then walks
through a gated, linear sequence: six candlelit letter slides, a photo gallery, a
spam-to-open gift box, a rigged spin wheel, a date picker, and a closing message.

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
   along with a rose-colored heart, confetti, and a chime.
3. **Letter — six slides** *(the first locked until the reveal above finishes)* — one
   line of message text and one photo per slide. Each slide lights its own candle a
   beat after it appears, then unlocks its own "continue →" once that settles; the
   sixth slide's continue hands off to the gallery instead of a seventh slide.
4. **Gallery** *(locked until the sixth letter slide's continue)* — a grid of flip
   cards, tap to reveal a remark on the back of each photo.
5. **Gift box** *(locked until "continue" on the gallery)* — starts as a grey
   silhouette, colors in a beat after the slide appears, then needs rapid clicking
   (with a decaying progress bar) to pry the lid open.
6. **Guesses form** — after the box opens, she can list as many guesses as she wants
   about what the gift is. Submitting opens her mail app with the list pre-addressed
   to you (see **Email limitations** below).
7. **Spin wheel** *(locked until the guesses form settles)* — six options, rigged to
   always land on "secret gift 🎁" (verified via direct rotation-math computation, not
   just visually).
8. **Date picker** — on claiming the gift, she picks a date/time. Confirming offers her
   a real downloadable `.ics` calendar invite, opens her mail app with the chosen time
   pre-addressed to you, and flies a little paper airplane across the screen.
9. **Ending** *(locked until the date is confirmed)* — closing message.

Every "continue" trigger (button, or an interaction like opening the gift box) is
**locked** — disabled/non-interactive — until the slide it belongs to has actually
finished its own entrance animation, not just the instant it appears. There's no
scrolling to guard against in the first place: `html,body{overflow:hidden}`, and each
slide is `position:fixed;inset:0` with only one ever unhidden (`hidden` attribute off,
`.entered`) at a time — `showSlide()` in the script hides every other slide before
revealing the target one. Advancing is buttons only, on a delay after each slide's own
entrance settles — never scroll-triggered.

## What's still needed (all clearly marked `[ in brackets ]` in the HTML)

- **Her name** — used in a few places (the ending title, etc.)
- **Your name** — signs the letter and the ending
- **The actual message text** — 4 of the 6 timeline entries are still bracketed prompts
  (an opening line, something specific you love about her, a shared memory, what you
  hope this year holds)
- **Real photos** — every photo slot (timeline, gallery, the main reveal card) is a
  placeholder gradient box with a `[ bracketed ]` label. They need to become real
  `<img>` tags (base64-embed them directly in the HTML, or host them somewhere and
  point `src` at the URL) — the photo lightbox's download button specifically checks
  for a real `<img>` and won't do anything until one exists.
- **Gallery remarks** — the one-line comment on the back of each flip card
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

## Continuing this in a new chat

Point Claude Code at this repo. The whole site is one file (`index.html`) — read it,
it's heavily commented at the points that matter (gating logic, noise-based
animation, the wheel's rigged rotation math). If you want the live Artifact link kept
in sync, tell Claude to republish `index.html` with `url:
https://claude.ai/code/artifact/425f0429-c6cd-4cda-b5b0-926021a3e569` rather than
publishing fresh (that flag is what makes it update the *same* link instead of
creating a new one).
