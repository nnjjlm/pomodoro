# pomodoro

A working Pomodoro timer that reproduces your Affinity card exactly. The
checkerboards, the clock, the transport control and the progress bar aren't
drawn in a colour — they're a **mask**, and whatever image you load shows
through them.

---

## Using it

| Action | What it does |
|---|---|
| The ▶ / ‖ control on the card | Start / pause |
| `Space` or `Enter` | Same thing |
| `R` | Reset to the top |
| **Short Break** / **Long Break** | 5 / 15 minutes |
| A shortcut chip | Loads its duration **and starts straight away** |
| Drag the **custom Time** bar | 1–100 minutes |
| `←` `→` | Nudge by 1 minute — hold `Shift` for 5 |
| `I` | Flip the mask (see below) |
| Hover → ***Customise* Card** | Open the settings panel |
| `Esc` | Dismiss the alarm, or close the panel |

The wordmark sits in the top-left corner and the transport takes the space it
used to occupy, on the clock's own dot grid — so it's part of the same visual
family rather than a bolted-on button. It swells slightly on hover and dips when
pressed. The active break preset gets a hairline in its own accent colour — navy
for Short, brown for Long.

The progress bar grows left to right across the full width of the card. Three
soft tones play when the session ends, then it resets itself.

## Invert

Two ways to read the same mask:

- **Normal** — grey card, image showing *through* the shapes.
- **Inverted** — image fills the card, shapes *knocked out* of it in the card
  grey. The wordmark switches to grey too, so the whole knockout reads as one
  layer.

It's the same mask either way; only the compositing operator changes
(`destination-in` vs `destination-out`). The setting is remembered per browser.

## Shortcuts

The row under the card scrolls. Press **+** at the end of it to add a chip —
this drops the whole row into **edit state**, with the new chip's label already
selected.

While editing:

| | |
|---|---|
| The label | Type over it |
| The emoji | Click for a picker |
| The little dot | Click to cycle the colour |
| `25m` | Click to set the minutes |
| The white **−** | Remove the chip |
| **Done**, or `Esc` | Back to normal |

**Short Break** and **Long Break** are fixed — no − badge, not editable. Only
your own chips are.

Chip colours come from a ten-swatch palette and the **label colour is derived
from the background**, not stored: the chip's own hue pushed down to 20%
lightness if the background is light, up to 90% if it's dark. Greys fall back to
the page ink. So `#90A3FF` gets navy text and `#E2B166` gets brown, exactly as
in the mockup.

### The shape sequence

Each chip has one fully rounded cap and one square corner, and the orientation
**alternates** along the row — cap on the left, then the right, then the left.
Traced from the Affinity path: the cap radius is exactly half the row height
(18.5 of 37), and the opposite corner is a hard 0. The **+** button carries the
same idea diagonally: top-left and bottom-right rounded at 6, the other two
square.

When the chips outrun the space they scroll, and a 50-unit fade appears on
whichever side has more to show.

## The panel

Hover the card and click ***Customise* Card** in the top-right. A white sheet
drops down over the card — sized to exactly the card's 335 of the 401 design
units, so it fits even in a tightly cropped Notion embed. Close it with the
`×`, `Esc`, or a click anywhere outside.

Three columns:

- **Card** — invert, auto-darken, chime volume
- **Image** — choose or clear, with the current filename
- **Alarm** — choose or clear a sound, loop toggle, preview

## The alarm

When a session ends the alarm fires and **keeps going until you dismiss it**.
Dismiss by clicking anywhere on the card, pressing `Esc`, or pressing any key.
The transport turns into a stop square and the whole mask pulses gently while
it's ringing.

Drop any audio file on the card, or use **Choose sound** in the panel. With no
sound loaded it falls back to a built-in three-tone chime, repeating every
2.1 seconds. Turn **Loop until dismissed** off and it plays through once and
resets itself instead.

Sound files are usually too big for `localStorage`, so the audio is kept in
**IndexedDB** as a Blob — same browser, same origin, same caveat as the image.

Browsers block audio until you've interacted with the page. Starting the timer
counts as that interaction, so in practice the alarm will always be allowed to
play.

## Changing the image

Three ways, all equivalent:

- Open the panel and click **Choose file**
- Drag an image file onto the page
- Copy an image and press `Cmd V`

It's resized to 1400 px max, saved as JPEG and kept in `localStorage`, so it
survives reloads. **Clear** puts the built-in gradient back.

### Auto-darkening

The card is a light grey (`#EBEBEB`), so a bright photo would make the dots and
checkerboards vanish. On load, the image is sampled **only where it actually
meets the grey** — inside the shapes normally, in the surrounding area when
inverted — its relative luminance is measured, and just enough black is laid
over it to hold a 2.7:1 contrast ratio against the card. It's capped at 68% so
nothing ever goes flat black, and a dark image is left completely alone. This
runs once per image, not per frame.

If you ever want it more or less aggressive, the two numbers are `MIN_RATIO`
and `MAX_DIM` near the top of `computeDim()`. The panel's **Auto-darken** switch
turns it off entirely.

One caveat about the stored image: `localStorage` is per browser and per origin.
The image you set on `localhost` won't appear on the hosted copy, and Notion's
embed iframe keeps its own copy again. Set it once in each place.

---

## Hosting it (for the Notion embed)

Notion can only embed a **public URL** — it can't embed a local file. GitHub
Pages is the least painful free option.

1. Create a new public repo, e.g. `pomodoro`.
2. Upload `index.html` to the root. Nothing else is needed — it's one file.
3. Repo → **Settings** → **Pages** → Source: *Deploy from a branch*,
   Branch: `main`, folder: `/ (root)` → **Save**.
4. Wait about a minute. The URL will be
   `https://<your-username>.github.io/pomodoro/`.

Then in Notion:

1. Type `/embed` on the page → **Embed**.
2. Paste the URL → **Embed link**.
3. Drag the bottom edge of the block to size it. The card keeps a fixed
   1224 × 401 ratio, so make it wide and short — roughly a 3:1 block.

Notion embeds run in a sandboxed iframe. Clicks, the slider, drag-and-drop and
`localStorage` all work; the finish tones may not play until you've clicked
inside the embed once, because browsers block audio before a user gesture.

---

## Where the numbers came from

Nothing in the layout is eyeballed. Everything was pulled out of `mockupS.psd`
and the `.svg` exports and checked against them.

- **Card** 1222 × 335, corner radius 29.5
- **Checkerboards** — the actual vector paths from the Affinity export, embedded
  as `Path2D`. Not a regenerated grid.
- **Clock** — **Array**, halftoned onto a 10-row grid: pitch 10.5556, dot radius
  5.279, first dot centred at (152, 120). The bitmaps for all ten digits were
  sampled from the real Array font at exactly that cell size, and `0`, `1`, `2`
  and `:` come back matching the artwork dot for dot — which is how the grid was
  confirmed. They're baked in, so Array doesn't have to load at runtime.
- **Wordmark** — ink box at (36, 38), 201 wide.
- **Control row** — DOM, not canvas, so it can scroll and be edited. Sized in
  the same design units via a `--s` custom property (one unit in px), taken from
  the 1157-wide Affinity export scaled to the card's 1222: row height 37,
  chip padding 42, gap 20, label 16 bold, track 428 × 18.
- **Transport** — same grid, column 58.5, centred in the box the wordmark
  vacated and vertically centred on the clock. The stop square shares that grid.
- **Progress bar** — x 2, y 293.9, height 41.1, clipped by the card's
  bottom-left corner.

Fonts: **Zodiak Italic** (from Fontshare) for the *pomodoro* logo, *Short* and
*Long*; **Helvetica Neue** for *Break*, *Custom Timer* and the chip. Every text
run is sized by measuring it and matching the ink width from the original, so if
a font fails to load the fallback still lands in the right place instead of
shifting the layout.

## A note on the countdown

It runs off the wall clock (`Date.now()`), not accumulated frame deltas, and a
500 ms interval backs up the animation loop. A browser throttles
`requestAnimationFrame` in a background tab, so a delta-based timer would drift
badly the moment you switched away. This one stays correct and will still fire
while hidden.

## Editing it

Everything adjustable lives in the block at the top of the `<script>`:
`CARD`, `BAR`, `DOT`, `PLAY`, `POMO`, `PILL_A`, `PILL_B`, `TRACK`, `CHIP`, the
colour table `C`, and the `GLYPHS` bitmaps. Change a glyph by editing its rows
of `#` and `.` — ten rows, any width.
