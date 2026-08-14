# Date Night Roulette

**Live: https://analyzeplatypus.github.io/date-roulette/**

One file — [`index.html`](index.html) — no build step, no dependencies, no server.
Spin the left wheel for a category, then the right wheel for an activity within it.

On an iPhone, open the live URL in Safari and **Share → Add to Home Screen** for an
app icon that launches it like a native app.

## Deploying

GitHub Pages serves `index.html` straight off `main` — no Actions workflow, no build.
Push and it's live within about a minute:

```bash
git push
```

### One gotcha when pushing

This repo lives under the **AnalyzePlatypus** GitHub account, but the SSH key on this
machine authenticates as **Michoels**. So `origin` is an HTTPS remote that authenticates
through the `gh` credential helper, which uses whichever `gh` account is *active*.

Cloning and fetching work either way, because the repo is public. Only **pushing** needs
write access, and as Michoels it fails with `Permission to AnalyzePlatypus/date-roulette
denied to Michoels` (403). Switch accounts first:

```bash
gh auth switch --user AnalyzePlatypus
```

To avoid the dance permanently, either add Michoels as a collaborator on the repo and
switch `origin` back to SSH, or add this machine's SSH key to the AnalyzePlatypus account.

### Other options

| Option | How | Notes |
| --- | --- | --- |
| Claude Artifact | Republish `index.html` from a conversation | Private by default; handy as a preview while iterating |
| Netlify / Cloudflare Pages | Connect the repo | Deploys from a *private* repo for free if you ever want the list unlisted |
| No hosting at all | Open `index.html` from disk, or AirDrop it to a phone | Works over `file://` |

## Where the activity list lives

- The starter list is baked into `index.html` (the `STARTER` array): 8 categories, 52
  activities, each with a one-or-two-sentence instruction shown only on the result card.
- Edits made in **Edit the list** are saved to that browser's `localStorage`, per device.
- **Copy share link** encodes the catalog into the URL hash (`#c=…`). Open that link on
  another device and it adopts the list and saves it there. That's the sync mechanism —
  text yourself the link.

The share link omits any instruction still matching its built-in, and `sanitizeItem`
backfills it by activity name on the way in. Without that, all 52 instructions ride along
and the URL passes 10,000 characters — long enough for messaging apps to truncate, which
would fail silently. Names-only is ~2,100 characters; only wording you actually changed
adds to it. A bonus: an older names-only link picks up the current instructions for free.

Because there's no server, a stranger who finds the page can only change their *own*
copy. Nothing they do reaches your list. The tradeoff: the starter list baked into the
file is readable by anyone who can open the page, so treat it as public, not secret.

## Editing the list in the file

To change the defaults permanently rather than per-browser, edit `STARTER` near the top
of the `<script>` block:

```js
const STARTER = [
  { name: "Board Games", items: ["Codenames", "Azul"] },
  // …
];
```

Bump `KEY` (`dateRoulette.catalog.v2`) if you want existing browsers to pick up new
defaults instead of their saved copy.

## How the wheels work

Not a CSS `rotate` with an eased timing function — the wheel is integrated on canvas at
240 Hz:

- **Viscous drag** proportional to angular velocity, plus **Coulomb friction** (a constant
  bearing torque), which is what makes it stop rather than asymptote.
- A **detent torque**, `DETENT * sin(n * u)`, models the pegs on the rim pulling the wheel
  toward a wedge center. It's why the wheel clacks past pegs at speed, crawls at the end,
  and always settles centered under the flapper instead of straddling a line.
- The **flapper** deflects by peg proximity and direction of travel, and each peg passing
  it fires a click.
- The wheel is lettered radially with **no upright-flip**, deliberately. Flipping labels to
  stay upright snaps them 180° as a wedge crosses the pointer, which reads as a glitch; the
  cost is that lettering on one side is inverted, exactly like a physical prize wheel.

### Deciding the winner early

The result is committed the moment physics makes it inevitable, not when the wheel stops.
The wheel sits in one detent well, and reaching a neighbouring wedge means climbing the peg
barrier. With detent torque `D*sin(n*u)` the potential is `U(u) = (D/n)(1 - cos(n*u))`, so
the barrier is `2D/n`. Friction only ever removes energy, so once total energy (kinetic
plus potential) drops below that barrier the wheel can never leave the well — and every
angle within a well maps to the same wedge index. `Wheel.decided()` tests exactly that.

Measured over eight spins across the full `omega0` range, the winner always matched the
wedge the wheel came to rest on, and the result landed 2.7s earlier on average (up to
3.6s). The bigger win is variance: it used to appear anywhere between 3.9s and 8.0s
depending on where the wheel got trapped, and now it's consistently 3.7–4.4s.

Drag a wheel to flick it — release velocity is measured from the drag and fed straight in.
`prefers-reduced-motion` shortens the spin by raising friction.

## Local preview

```bash
python3 -m http.server 8123
```

Then open http://localhost:8123. (`.claude/launch.json` wires this up for Claude Code's
preview pane.)
