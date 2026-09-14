# Swipe Count

One HTML file. Photograph a stack, drag a finger across the parts, get a count
you can check and correct.

No AI, no API key, no network, no cost. Everything runs on the phone.

---

## Getting it on the iPhone

It needs a real web address — the browser blocks the work if you open the file
straight from the Files app.

**Try it in five minutes.** On the Mac, in the folder holding `index.html`:

```bash
python3 -m http.server 8000
```

Find the Mac's address (`ipconfig getifaddr en0`), then open
`http://192.168.x.x:8000` in Safari on the phone. Works fully, as long as the
Mac is awake and on the same Wi-Fi.

**Keep it.** Upload `index.html` to a new GitHub repository, then Settings →
Pages → deploy from `main`, folder `/ (root)`. Live at
`https://<you>.github.io/<repo>/` a minute later. Or drop it on greentools.tech.

Then in Safari: Share → **Add to Home Screen**. It launches full-screen with
its own icon.

---

## Keeping it up to date

GitHub Pages serves with a ten-minute cache and an iOS home-screen shortcut can
hold its copy much longer, so a freshly pushed build can sit unseen for hours.

The page handles this itself. At start-up it fetches its own URL with the cache
bypassed, reads the build stamp out of the text, and if the server has a newer
one it loads that instead. Only at start-up, before any counting has happened,
so there is never a count to lose — and only once per launch, so it cannot loop.

If you ever want to check by hand, the stamp is at the top of **Settings**.

## Using it

1. Photograph the stack edge.
2. **Swipe from the middle of the first part to the middle of the last**, square
   across the parts. Two fingers pans and zooms.
3. **Swipe again, somewhere else across the same parts.** And once more. Each
   swipe is another vote. Six is the most it keeps.
4. Green ticks are parts. Cyan ticks are the boundaries between them, drawn on
   every swipe that voted — seeing them line up across the swipes is the check.
5. Tap a tick to remove it, tap the line to add one.

**Why more than one swipe.** A single swipe cannot tell a boundary between two
parts from a scratch on one of them: both are a place where lightness changes
fast. Several swipes can. A boundary runs the whole way across the stack, so
every swipe crosses it. A scratch, a burr, a blurred patch is crossed by one.

Measured on a stack of 14 plates whose edges are genuinely blurred, four
hand-drawn lines gave **4, 12, 11 and 6** on their own and **14** together.

The extra swipes have to be yours. Generating them automatically, parallel to
the first, was tested and is *worse* than a single swipe — on two of six photos
the generated lines wandered off the parts and the count collapsed. Where they
fall is the information.

**The bar under the photo** reads like `3 swipes · agreement 0.54 · even spacing`.

- *agreement* is how well the swipes corroborate each other, 0 to 1. Below about
  0.3 they are not crossing the same run of parts.
- *clear gaps (20×)* means the gaps it kept were far deeper than anything it
  rejected. No assumption needed. A hand of fingers reads 20–80×.
- *even spacing* means it fell back to choosing the number of gaps that comes out
  most regular. That assumes the parts are alike and evenly stacked, it is the
  weakest step in the whole method, and it is fitted to seven photographs.

**Longer than one photo?** Count a section, press **Save section**, photograph
the next stretch. The running total carries across.

## Torch pair (experiment, build 6)

The lightning button opens a live camera. One press takes **two frames a third
of a second apart — one with the torch off, one with it on** — aligns them for
hand shake, and divides the difference by the sum. Surface colour cancels, so a
painted mark or a saw striation that reflects both frames identically drops out.

After the shot, a **Photo / Torch** toggle appears. Swipe on either and compare
the counts. Treat it as an experiment, not an answer.

**What it can and cannot do.** The torch sits about a centimetre from the lens,
which is the flattest lighting possible — the opposite of the raking light that
made the two-lamp simulation work. So there are no cast shadows to exploit. What
survives is the ambient light being removed, plus 1/r² falloff and the cosine of
the surface angle: near, square-on surfaces brighten most. Whether that is
enough on real steel is exactly what the toggle is for.

**Why a third of a second and not faster.** The torch ramps up and the camera
re-exposes. Shooting sooner catches the transition, which aligns badly and
differences worse. Hand shake over that time is handled by the alignment step.

**One detail worth knowing.** The alignment search is deliberately capped below
half the part spacing. On a repeating pattern an unbounded search matches the
wrong repeat with complete confidence — the same aliasing that runs through this
whole project. Measured on synthetic stripes: a ±12px search got 6 of 6 shifts
right, a ±32px search on a 44px period got 5 of 6, failing confidently.

Needs iOS 17.5 or later. If the phone won't give torch control, the app says so
before you press the shutter and warns afterwards that the difference is
meaningless.

---

## The two things that will bite you

**Swipe across the parts, not along them.** The app never guesses the direction,
and that's deliberate. An earlier version picked the direction with the
strongest repeating pattern and locked onto the saw striations on a cut face —
it reported 97 parts in a photo containing eight. Your swipe is what removes the
ambiguity. A swipe that grazes the parts at a shallow angle also fails: on one
hand photo a square swipe gave a 13× separation and a hand-shaped swipe drawn at
a shallow angle gave 2–6× and counts scattered between 3 and 6.

**Start on the first part, end on the last.** See above. This is the one rule
the arithmetic depends on.

## How much to trust it

Measured on seven hand-marked lines, with nothing in the code knowing the answer:

| photo | truth | one swipe |
|---|---|---|
| A fanned arc segments | 37 | 38 |
| B nested curved parts | 15 | 17 |
| C close-up stack | 14 | **14** |
| D bent parts, edge on | 26 | **26** |
| E four fingers | 4 | **4** |
| F six keyboard keys | 6 | 4 |

Total error 11 across six single swipes, against 51 for build 7 on the same
lines. And on the one stack photographed with four lines, **14 exactly**, where
the best single line gave 10.

Where that came from, in order of how much it was worth:

- **Score every row of the band separately and keep what nearly all rows agree
  on**, instead of averaging the band into one curve. An average blends a gap
  that runs the full width with a letter or a tool mark that does not.
- **Let the boundary be slanted.** A stack photographed off-square has edges that
  are not perpendicular to your swipe. Searching the slant took the fanned arc
  photo from 18 to 36.
- **Level the edge strength, not the brightness.** Build 6 divided the brightness
  by its local contrast and thereby erased the gaps. Dividing the *edge score* by
  its local average lets a faint boundary in a dim stretch compete with a strong
  one in a bright stretch, which is what these photos visibly need.
- **Several swipes, voting.**

Things that were tried and did not help: Laplacian-of-Gaussian, multi-scale
gradients, morphological top-hat, and phase congruency. Each won on one or two
photos and lost on the rest. Seven photographs cannot choose between them.

## Why this can't be automatic — and what changed

Build 6 ran a levelling step before counting: it subtracted the lighting
gradient and then **divided by the local contrast**, so that one threshold would
mean the same thing at the bright and the dark end of a stack. That division is
gone in build 7, and removing it is the whole change.

Measured on a photo of four fingers: with the gradient merely subtracted, the
three gaps between fingers had depths 142, 124 and 106, and the deepest thing in
the picture that was not a gap was 8. A 13x cliff, with no threshold to tune —
the cut can be placed anywhere from 10 to 100 and returns four.

Run the same profile through build 6's levelling and those six numbers become
3.79, 3.78, 3.64, 3.62, 3.48, 3.39. A skin wrinkle now weighs exactly as much as
the gap between two fingers. The one unambiguous signal in the photograph had
been destroyed before anything looked at it, which is why thirteen attempts to
recover it downstream all failed.

That is not a claim that the problem is solved. On the steel photos the cliff
is not there: the 13th deepest dip measured 22.9 and the 14th measured 22.4,
where the true count is 14. Stacked steel carries bevels, shear marks and
grinder striations that are as dark as the separations between the sheets, and
a hand does not. Whether a squarely drawn swipe on a real stack changes that is
the open question, and it is why both methods ship.

## Known limits

- **2200px is calibrated, not solved.** It was fitted to four photos; at 2600px
  the errors return. If your parts are much finer or coarser, that's the knob in
  Settings. The proper fix is to derive it from the measured spacing instead of
  a fixed number, and that needs more hand-counted photos.
- **Below about 5px per part it stops working**, and there's no warning for that
  in the app yet.
- **Uneven stacking** is the one thing no amount of processing fixed. Square the
  pile up if you can.
- Camera angle doesn't matter much — up to about 35° off-square made no
  difference in testing.
