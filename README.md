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

- **Today** — calorie ring, macro bars, a 30-item food list by category, custom items, today's log
- **Activity** — weekly plan per sport, session logging with MET-based burn estimates, load-vs-fuel warnings
- **Trends** — 28-day grid built from real logged days, streaks, weight line, phase picker
- **Goals** — daily targets with safety checks against public-health ranges

Calorie burn uses MET values from the Compendium of Physical Activities, scaled by body weight,
minutes and an effort factor. Maintenance is estimated as weight × 29 plus average daily
training burn — a rough heuristic, not a measured RMR.

## Notes if you want to change things

- Food database: the `FOODS` array near the top of the script.
- Sports and MET values: the `SPORTS` object.
- Phases and their calorie adjustments: `PHASES`.
- Onboarding tour copy: the `TOUR` array.
- Colours: the CSS custom properties in `:root`.

The storage key is `ct.v2`. If you change the shape of what's saved, bump it to `ct.v3` so
existing users don't load incompatible data.

Two mobile gotchas already handled, worth knowing if you edit the CSS: form fields are set to
16px because anything smaller makes iOS Safari zoom the page on focus, and every button has an
explicit colour because iOS renders unstyled button text in the system blue.

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
