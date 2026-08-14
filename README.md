# Date Night Roulette

One file — [`index.html`](index.html) — no build step, no dependencies, no server.
Spin the left wheel for a category, then the right wheel for an activity within it.

## Deploying

Pick whichever is least annoying:

| Option | How | Notes |
| --- | --- | --- |
| Claude Artifact | Already published from this file | Private by default; updates by republishing the same file |
| GitHub Pages | Push this repo, enable Pages on `main` | Free, permanent URL, works offline once loaded |
| Netlify Drop | Drag the folder onto app.netlify.com/drop | No account needed |
| No hosting at all | Open `index.html` from disk, or AirDrop it to a phone | Works over `file://` |

On an iPhone, open the URL in Safari and **Share → Add to Home Screen** for an app icon.

## Where the activity list lives

- The starter list is baked into `index.html` (the `STARTER` array).
- Edits made in **Edit the list** are saved to that browser's `localStorage`, per device.
- **Copy share link** encodes the whole catalog into the URL hash (`#c=…`). Open that link
  on another device and it adopts the list and saves it there. That's the sync mechanism —
  text yourself the link.

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

Drag a wheel to flick it — release velocity is measured from the drag and fed straight in.
`prefers-reduced-motion` shortens the spin by raising friction.

## Local preview

```bash
python3 -m http.server 8123
```

Then open http://localhost:8123. (`.claude/launch.json` wires this up for Claude Code's
preview pane.)
