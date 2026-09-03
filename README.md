# Calorie Tracker

A calorie, macro and training tracker that installs to a phone's home screen and works offline.
No build step, no backend, no accounts. Everything a person logs stays in their own browser
(`localStorage`), so each user has their own private data and nothing is sent anywhere.

## Files

```
index.html              the whole app
manifest.json           makes it installable
sw.js                   offline caching
icon-192.png            home screen icon
icon-512.png            splash / store icon
icon-maskable-512.png   Android adaptive icon
icon-180.png            iOS home screen icon
README.md               this file
```

Keep them all in the same folder, at the root. `index.html` still works on its own if you
open it directly, it just won't be installable.

## Deploying

### Vercel

1. Put all the files at the **root** of a GitHub repo (not in a subfolder).
2. In Vercel: **Add New → Project → Import** your repo.
3. Framework Preset: **Other**. Leave build command and output directory empty.
4. Deploy.

If you get a 404, it's almost always one of: the file isn't named exactly `index.html`,
it's inside a subfolder, or Vercel picked a framework preset and looked for a build output
that doesn't exist.

### Netlify

Drag the folder onto the drop zone at app.netlify.com, or connect the repo and leave the
build command blank with publish directory `.`.

### GitHub Pages

Repo → Settings → Pages → Source: deploy from branch, `/` (root).

HTTPS matters: service workers only run over https or on localhost. All three hosts give
you https automatically.

## Installing it on a phone

**iPhone** — open the URL in Safari (it must be Safari, not Chrome), tap Share, then
"Add to Home Screen". It gets its own icon and opens full-screen with no address bar.

**Android** — open in Chrome. You'll get an "Install app" prompt, or use the ⋮ menu →
"Add to Home screen" / "Install app".

Once installed it works with no signal — the service worker caches the app itself, and the
data was always local anyway.

## Updating after you change something

Bump `CACHE` in `sw.js` (e.g. `calorie-tracker-v2`) whenever you change `index.html`.
Otherwise people who already installed it keep seeing the cached old version.

## What it does

- **Today** — calorie ring, macro bars, water tracking, a 120-item food database by category, saved custom items, today's log
- **Activity** — weekly plan per sport, session logging with MET-based burn estimates, load-vs-fuel warnings
- **Trends** — 28-day grid built from real logged days, streaks, weight line, phase picker
- **Goals** — daily targets with safety checks against public-health ranges

Calorie burn uses MET values from the Compendium of Physical Activities, scaled by body weight,
minutes and an effort factor. Resting burn uses the Mifflin-St Jeor equation (sex, age, height,
weight); maintenance is that × 1.25 plus average daily training burn. Neither is a measured RMR.

The 12-week projection on Trends is a model, not a forecast: energy balance gives total change
at ~7,700 kcal per kg, split between fat and lean by whether protein and training are adequate,
with muscle gain capped at 0.25 kg/week. Treat it as a direction.

## Notes if you want to change things

- Food database: the `FOODS` array near the top of the script.
- Sports and MET values: the `SPORTS` object.
- Phases and their calorie adjustments: `PHASES`.
- Onboarding tour copy: the `TOUR` array.
- Colours: the CSS custom properties in `:root`.

The storage key is `ct.v3`. It reads a `ct.v2` save if it finds one and backfills anything
new, so bumping the version doesn't wipe existing users. Object fields are merged rather than
replaced, which means adding a field to `goals` or `units` later keeps its default for people
who saved before it existed.

Units are a display layer only. Weight is always stored in kg and water always in ml, so
switching between kg/lb or glasses/cups/ml never alters logged data. A glass is 250 ml and a
cup is 240 ml (`WATER_UNITS`).

Haptics use `navigator.vibrate`, which Android and most desktop browsers support and iOS
Safari ignores entirely — there is no web API for iPhone haptics. On iOS the tick animation
carries the feedback on its own.

Two mobile gotchas already handled, worth knowing if you edit the CSS: form fields are set to
16px because anything smaller makes iOS Safari zoom the page on focus, and every button has an
explicit colour because iOS renders unstyled button text in the system blue.

## Layouts

One column on a phone in portrait. Two things change that:

- **Phone in landscape** (`orientation: landscape` and `max-height: 540px`) — barely 380px of
  height, so the column ran out of room instantly. It switches to two columns, tightens the
  vertical rhythm, shrinks the ring, and uses a shorter tab bar.
- **Tablet and up** — two columns from 700px, three from 1080px, with the shell widening to
  840px then 1120px.

The cards are direct children of `.page-body`, which is a flex column by default and a grid at
the wider breakpoints. That means adding a card needs no layout work, but it must stay a direct
child of the wrapper or it won't participate in the grid.

The tab bar reads its own padding and gap via `getComputedStyle` rather than hardcoding them,
so the drag maths follows whatever the current breakpoint sets.

## Themes

Light by default; Nocturne dark via the moon button in the header. It is an explicit choice —
there is deliberately no `prefers-color-scheme` rule. In dark mode the accent ramps invert in
meaning: the 200s become dark chip fills and the 800s the light text on them, so markup written
against those variables works in both themes without change. Text that sits on an accent fill
goes through `--on-accent` / `--on-accent-2`, because white on the amber button would be about
2:1 in dark mode.

## The glass layer

The tab bar, form fields, scrollbar thumb and tour sheet are frosted with `backdrop-filter`.
It's driven entirely by custom properties at the top of the stylesheet (`--glass-nav-bg`,
`--glass-blur`, `--glass-edge` and friends), so you can dial the whole effect up or down in
one place.

Those properties default to **opaque** values. The translucent versions are swapped in inside
an `@supports` block, so browsers without `backdrop-filter` get the solid design rather than
washed-out unreadable panels. A `prefers-reduced-transparency` query switches back to opaque
for anyone who's turned transparency off at the OS level.

For the blur to have anything to work on, the content scrolls *underneath* the tab bar rather
than stopping above it. That's why `#nav` is absolutely positioned and `#page` carries a
bottom padding of `var(--nav-h)`. If you change the bar's height, change `--nav-h` on `#shell`
to match or the last item in a list will sit under the bar.

## Not medical advice

The safety checks reflect general public-health guidance, not personalised advice. Anyone
setting aggressive targets should talk to a doctor or dietitian.
