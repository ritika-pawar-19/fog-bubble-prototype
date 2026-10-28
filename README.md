# Fog Bubble

A short game for store staff. It is **mandatory**: staff play it before the LMS opens.
It takes about a minute. Measured from the page opening to Enter LMS: a smooth round is 55 to 60 seconds
(about 70 for a slow, careful reader; about 40 for a player who taps through the feedback cards),
plus an 18 second demo on the first visit. A round full of slips (missed shoppers come back) takes about 1.5 minutes.
There is no Later button and no close button.

Six shoppers walk in, one at a time. Each one has a thought bubble, fogged up like a steamy mirror.
The player rubs the fog clear with a thumb (or the mouse), then taps one of four pictures to say where to take the shopper.
The catch: the shoppers' looks are drawn to fool you. The student in big headphones wants a fridge.
Dadi with her walking stick wants a laptop, but the laptop specialist is busy with another shopper (her bubble shows it).
A half-rubbed bubble can mislead too: two round dark shapes look like glasses until the headband shows.

It teaches the Level 1 deck's rule "Judge shoppers by how they look." (a DON'T), and "Listen first. Understand the need."
No sign-in. English only.

## What the player sees

1. **First time on this device only:** a 15 to 18 second demo plays by itself.
   A ghost hand rubs half the bubble (a flat screen: it looks like a TV), keeps rubbing (a keyboard: it is a laptop) and taps the laptop.
   Then a quick replay of a tap made too early, which goes wrong. Then a big "Your turn" button. The demo cannot be skipped the first time.
2. The round: 6 shoppers. The first one is always a plain "take them to a section" shopper (never Dadi or Mr Sethi).
   On the first one, small hints say "Rub the fog", then, once the fog is gone, "Tap where to go".
   In the first round the two twist pictures carry a 2-word caption: "Get team" under the bell, "Give time" under the hourglass.
   If the player stops for 6 seconds, a ghost hand shows the rub; once the fog is gone it glides over the four pictures instead (once).
   - **Right:** Rohan and the shopper walk off together (or Rohan steps back to give space, or a teammate waves from across the floor),
     and the deck line shows on a card.
   - **Wrong:** the fog blows away, the shopper shrugs and says what they wanted. If the tap matched their looks while the bubble was still
     mostly fogged, a red DON'T stamp shows "Judge shoppers by how they look." After a full read, a wrong tap shows that shopper's own deck
     line instead (Dadi's is "The laptop specialist is assisting another customer."). If the tap was Iris (the glasses case), she springs
     out and the shopper steps back: "Customer first. Product later." Then the right picture lights up green.
     That shopper comes back 1 walk-in later, and once more 2 walk-ins after that if they are missed again.
   - A tap anywhere on the game (or Space / Enter) moves on to the next card, or the next shopper, once a card has been up for 0.8 seconds.
3. The end card: score, stars, how many read right, quick bonuses (when there are any), reading time, one thing to remember, and **Enter LMS**. "Play again" is there too.
4. Enter LMS closes the gate and the LMS opens.

On later visits the demo is skipped. The (?) icon at the top plays it again while a shopper is walking in or the fog is up;
the player then goes back to that shopper (with fresh fog). It is not offered on the end card, so a curious tap never throws a finished round away.

Each round is a little different: the six shoppers come in a new order (never the same order twice in a row), some decoy pictures are dealt
afresh, and the four pictures are shuffled for every walk-in.

## Files

| File | What it is |
| --- | --- |
| `fog-bubble.html` | The game. One file (about 190 KB) with everything inside: styles, script, art and sound. |
| `demo.html` | A plain sample LMS page with the mandatory gate in front of it. Open it in a browser to try it. |
| `README.md` | This file. |
| `shots/` | Screenshots, for reference. |

Upload `fog-bubble.html` to a place your LMS can serve a web page. The same server as the LMS is best.

## Try the demo

Open `demo.html`. The gate covers the page at once. Play a round and tap Enter LMS: the gate fades away.
Reload the page: it goes straight in, because the round is done for today.

**Reset demo** (bottom right, marked "Demo only") clears today's pass and the game's saved flags,
so the gate and the first-time demo show again. It is only for testing. Do not copy it into the LMS.

It works best from a web server. It also works as a local file in Chrome.

## Add the gate to your LMS

The idea: when the page loads, cover it with the game. When the game says `enter`, remove the cover.

1. Wrap your LMS page in one element, for example `<div id="lms-root">`.
2. When the user still has to play, render the page with `<html class="fb-gated">` and the gate below, right after `<body>`.
   (Rendering the class on the server means the LMS never flashes before the gate.)
3. When the user does not have to play, leave all of this out. The game is then not loaded at all.

```html
<style>
  html.fb-gated, html.fb-gated body { overflow: hidden; height: 100%; }
  html.fb-gated body { position: fixed; left: 0; right: 0; top: 0; }
  html.fb-gated #lms-root { filter: blur(6px); pointer-events: none; }
  #fb-gate { position: fixed; inset: 0; z-index: 2147483000; display: flex; align-items: center;
             justify-content: center; background: rgba(16, 12, 52, .58); transition: opacity .4s; }
  #fb-gate.leaving { opacity: 0; }
  #fb-frame { width: 320px; height: 400px; border: 0; border-radius: 16px; background: #5B4BDB; }
  @media (max-width: 600px), (max-height: 440px) {
    #fb-frame { width: 100%; height: 100%; border-radius: 0; }   /* phones: full screen */
  }
</style>

<div id="fb-gate" role="dialog" aria-modal="true" aria-label="Play the floor game to enter the LMS">
  <iframe id="fb-frame" src="fog-bubble.html" title="Fog Bubble game" allow="autoplay"></iframe>
</div>

<!-- at the end of <body> -->
<script>
(function () {
  var gate = document.getElementById('fb-gate');
  var frame = document.getElementById('fb-frame');
  var lms = document.getElementById('lms-root');
  var html = document.documentElement;
  if (!gate) return;

  html.classList.add('fb-gated');
  lms.setAttribute('inert', '');      // nothing behind the gate can be clicked or tabbed to
  frame.focus();                      // so Space works at once

  function noEsc(e) { if (e.key === 'Escape') e.preventDefault(); }
  document.addEventListener('keydown', noEsc, true);

  window.addEventListener('message', function (e) {
    if (e.source !== frame.contentWindow) return;          // only messages from the game
    var d = e.data;
    if (!d || d.type !== 'fog-bubble') return;
    if (d.event === 'complete') {
      // Send it to your server here: d.score (0 to 100), d.right (0 to 6), d.stars (1 to 3), d.bonus, d.time, d.round.
      // The same round is sent again with d.repeat === true just before 'enter': keep one record per round.
    }
    if (d.event === 'enter') {
      document.removeEventListener('keydown', noEsc, true);
      lms.removeAttribute('inert');
      html.classList.remove('fb-gated');
      gate.classList.add('leaving');
      setTimeout(function () { gate.remove(); }, 400);     // also stops the game and its sound
    }
  });
})();
</script>
```

`demo.html` does the same and adds a few extras you can copy:
a line of text above and under the game on desktop, a focus loop that keeps the keyboard inside the gate,
the "once a day" memory in the browser, a small "Floor game done today" note on the LMS page, and the demo-only reset.

If the game is on a different web address than the LMS, also check `e.origin` against the game's address.

## How often to show it

The LMS decides. Pick one:

- **Every login.** Show the gate every time the user logs in.
- **Once a day.** Show the gate if the user has no `complete` recorded today. This is what `demo.html` does, to show the idea.
- **Once ever.** Show the gate until the user's first `complete` is recorded.

**Our advice for this version: once ever, or at most once a week.** There is one set of six shoppers. The order and pictures change,
but by the second or third day a player knows every bubble, and a daily mandatory round will feel like a chore.
Show it more often only when more shopper sets exist.

Decide it on the LMS server if you can. `demo.html` keeps the date in the browser (local storage) only to show the idea.
To try each one: `demo.html?gate=login`, `demo.html?gate=day`, `demo.html?gate=ever`.

## Sizes

- Smallest: 300 x 360 px.
- Best: 320 x 400 px, or larger in the same shape (4:5). It scales up cleanly, and the fog is drawn sharp for the screen it is on.
- Other shapes work too. The spare space is filled with the game's violet.
- Phones: full screen. `demo.html` switches to full screen below 600 px wide (or 440 px tall).

## URL options

Add these to the game's address, for example `fog-bubble.html?pass=60`.

| Option | What it does |
| --- | --- |
| `?pass=60` | Below 60% the end card offers only "Play again", with an encouraging line. Without it, any score can enter. Use with care: a high bar can keep staff out of work. |
| `?demo=always` | Always play the how-to-play demo first. |
| `?demo=never` | Never play the demo. |
| `?close=1` | Shows a close X that sends `close`. Only for a later, optional (not mandatory) use. Do not use it in the gate. The X gets its own corner slot; the walk-in dots move under the bonus and time chips. |
| `?shuffle=0` | For testing only: keeps the written shopper order and picture order, so a run can be repeated exactly. |

`demo.html` passes `?pass` and `?demo` on to the game, for testing: `demo.html?pass=60&demo=always`.

## Events (postMessage)

The game sends these messages to the page that holds it (`window.parent`):

| Message | When |
| --- | --- |
| `{type: 'fog-bubble', event: 'complete', score, stars, right, bonus, time, round}` | The round ended. `score` is 0 to 100. `stars` is 1 to 3. `right` is how many of the 6 shoppers were read right first time. `bonus` is the number of quick bonuses. `time` is the reading time in seconds. `round` counts the rounds played since the game opened (1, 2, ...). |
| `{type: 'fog-bubble', event: 'complete', ..., repeat: true}` | Sent again just before `enter`, for the same round (same `round` number). |
| `{type: 'fog-bubble', event: 'enter'}` | The player tapped Enter LMS. Remove the gate. It always comes after `complete`. |
| `{type: 'fog-bubble', event: 'close'}` | Only with `?close=1`. The gate ignores it. |

So `complete` can arrive twice for one round, and once more for each "Play again". If you save it on your server,
keep one record per `round` (or ignore messages with `repeat: true`).

How the score works:
- Each shopper read right first time is 1 point (6 in all). The score is points out of 6.
- A missed shopper comes back 1 walk-in later. Right then earns back half a point. Missed again, they come back once more,
  2 walk-ins later, for a quarter point. A round never goes past 12 walk-ins, and every missed shopper gets at least one return.
- A **quick bonus** (a lightning bolt) is a right first answer given after a full read: the bubble rubbed clear, then the tap
  within 3.5 seconds of the fog lifting. A half-fogged guess never earns it. Bonuses do not change the score.
  The end card shows the bonus chip only when there is at least one.
- **Reading time** counts only from each bubble appearing to the tap, so feedback cards do not count.
- Stars: 85% and up is 3 stars, 55% and up is 2, below that 1.

The messages carry no personal data. Check `e.source === iframe.contentWindow` so you only act on messages from the game.

## Score and SCORM

This version has no SCORM. To keep scores in the LMS, listen for `complete` and save it on your server.
The game also keeps the player's best score and "demo seen" in the browser's local storage.
It still works if storage is blocked; it just cannot remember that the demo was seen.

## How strict is the gate? (please read)

The gate runs in the browser. It stops normal use: there is no button, link or key that gets past it.
But someone who knows how can get past it, for example with the browser's developer tools
(remove the cover, or send a fake `enter` message).
Clearing browser storage does not help them: it only makes them play again.

For real enforcement, record the `complete` event on the LMS server, with the user and the time,
and let the server decide whether to show the gate. Even then, a determined user could fake the message.
Treat the game as a training step, not an exam.

Also:
- Esc does nothing while the gate is up. Inside the gate the browser's Back button leaves the page as usual
  (the game adds no history entry there); the gate shows again on the next visit until a round is done.
  Opened on its own as a pop-up (not in a frame), the game holds the first Back press so it is not closed by accident.
- Rubbing never scrolls or zooms the page, also inside an app's WebView.
- If the game file cannot load, the gate stays up. Keep a quick way to switch the gate off on the server.

## Offline and fonts

The game works without internet once the page is open. It asks Google Fonts for the Poppins and Figtree fonts.
The request does not hold the game up: if it fails or hangs (a slow or filtered store network), the game starts at once with
the device's own font and switches to Poppins if it arrives. Nothing else is loaded from outside.
All sounds are made in the browser (no audio files). Sound is off at the start; the speaker button turns it on.
With sound on, Android phones also give a tiny buzz on a full clear, a right tap and a wrong tap.
`demo.html` loads nothing from outside.

## Browser support

- Desktop: current Chrome, Edge, Firefox and Safari.
- Phones: Chrome on Android, Safari on iPhone.
- Internet Explorer is not supported.
- `inert` (which blocks the page behind the gate) needs Chrome 102, Safari 15.5 or Firefox 112 or newer.
  On older browsers the cover still blocks clicks.

Keyboard: hold Space (or Enter) to rub the fog clear. Tab reaches the four pictures; Enter or Space picks the focused one,
and the number keys 1 to 4 pick them too. After a pick, focus goes back to the bubble, so the next shopper needs no extra Tabs.
Space or Enter during the feedback cards moves on. A key that is still held down never picks by itself. Focus stays inside the gate.
The pictures have spoken labels ("Fridge section", "Give them time", "Get the team", "Show the glasses"), and screen readers
hear what the bubble shows as the fog clears. Reduced-motion settings are respected.

## Change the texts

Open `fog-bubble.html` in a text editor.
All the words are in **the CONTENT block at the top of the script**:
the six shoppers (what they say, which pictures they get, the deck line after the right move, and what screen readers hear),
the DON'T line, Archie's lines and the end-card lines, the hints and the demo captions.

- Change only the text inside the quotes. Keep the commas and brackets.
- Keep it short: shopper lines up to 6 words, cards up to about 13 words (`\n` starts a new line on a card; a longer deck line is split over two cards), demo captions up to 5 words.
- No exclamation marks.
- The card lines are the deck's exact words. Change them only if the deck changes.
- Our own wording needs the client's approval: what the shoppers say ("A fridge. For the hostel.", "I write a blog.",
  "For my work calls.", "Dropped it. Twice.", "Hosting the final.", "A laptop for college."), Archie's lines, the hints,
  and the first-round picture captions "Get team" and "Give time". "I'm just looking." is a shopper line from the deck.
- Also for the client's OK (a design choice, not wording): the lightning bonus rewards a quick right tap *after* the fog is
  rubbed clear (within 3.5 seconds), never a half-fogged guess. The spec asked for a bonus for "little rubbing"; we changed it
  because that rewarded the guessing the game warns against.

The two lines around the game on the demo page ("Before you start: one quick game" and "About a minute.
Your LMS opens when you finish.") are in `demo.html`, inside the gate.

Save, then open `demo.html` to check.
