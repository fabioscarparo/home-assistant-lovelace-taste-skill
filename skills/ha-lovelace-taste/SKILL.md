---
name: ha-lovelace-taste
description: Build Home Assistant Lovelace custom cards, card editors, badges, card features, dialogs and other frontend components that look and behave like native Home Assistant UI, built on its design system tokens (colors, fonts, radii, spacing), its card API (setConfig, hass, getGridOptions, ha-form editors, customCards) and its conventions for languages, accessibility, light and dark themes. Use this skill whenever the user wants to create, restyle, extend, review or fix a Home Assistant card or dashboard component, a "custom:" card, a HACS dashboard plugin, a card served by an integration, a card editor, or a Lit or vanilla web component for dashboards, or asks to make something "look like Home Assistant", "match the HA design" or "feel native", even if they don't say "design system".
---

# Home Assistant cards that feel native

A good custom card is one a user can't tell apart from a core card. Home Assistant users switch
between light and dark themes, custom themes, languages, 12 and 24 hour clocks, masonry and
sections dashboards, phones and wall tablets. A card that hard-codes a color, a font or a pixel
width breaks on some of these, and users notice at once. So the rule behind everything below:
**take every visual decision from Home Assistant itself, and add your own design only on top of
its tokens.**

Read the reference files when you get to that part of the work:

- `references/design-tokens.md` - the tokens with their light and dark defaults, and ready-made
  CSS for the native patterns (tile header, badges, alerts, features, segmented control, select
  rows, charts, dialogs and bottom sheets)
- `references/card-api.md` - card and editor skeletons, re-rendering, registration that survives
  the frontend's registry swap, `hass` helpers, events, distribution through HACS or the
  integration itself
- `references/verification.md` - the mock `hass` demo page, scenarios from real data, the
  regression check between versions, README images in both themes, the release checklist

## Workflow

1. **Start from the data, not the mock-up.** Ask for (or find) the real entities: domain,
   `translation_key`, states, attributes, units, device class, which ones are disabled by default
   or `unavailable`. An export of the entities of one device, taken in a few states (off, ready,
   running, finished), is worth more than any description. If the integration has more than one
   version or fork, collect data from each: keys, state casing and option order often differ.
2. **Pick the closest native pattern and mirror it.** Before inventing a layout, look at what core
   already does for the same job: tile card (header, icon tinted by state), thermostat card (dial,
   minus and plus), entities card rows, badges, `ha-alert`, more-info dialogs, card features.
   Reuse their sizes, typography and behavior.
3. **Build with tokens only** (next section). Raw values appear only as fallbacks inside `var()`.
4. **Wire it the Home Assistant way**: formatting through `hass`, actions through standard
   services, more-info on tap, a visual editor with `ha-form`, entities found from the device.
5. **Verify** in light and dark, in every supported language, at phone width and in a sections
   grid, against real data, and against the previous release so nothing else changed.
6. **Ship** with a version, short release notes and images rendered from the real card.

## Design system essentials

Full list and copy-ready CSS in `references/design-tokens.md`. The parts that matter most:

- **Let `ha-card` own the surface.** Wrap the card in `<ha-card>`: it already applies the theme's
  background, border, radius and shadow (`--ha-card-background`, `--ha-card-border-*`,
  `--ha-card-border-radius`, `--ha-card-box-shadow`). Don't paint your own card background.
- **Text**: `--primary-text-color`, `--secondary-text-color`, `--disabled-text-color`, and
  `--text-primary-color` for text on an accent fill. Never `#000`, `#fff` or `rgba(0,0,0,.6)`.
- **Type scale**: `--ha-font-family-body`, `--ha-font-size-xs/s/m/l/xl/2xl/3xl`,
  `--ha-font-weight-normal/medium/bold`, `--ha-line-height-condensed/normal`. The tile card's name
  is size m (14px) at medium weight, its secondary line size s (12px) in the secondary color.
- **Spacing and radii**: `--ha-space-1` to `--ha-space-20` (steps of 4px) and
  `--ha-border-radius-sm/md/lg/xl/2xl/3xl/pill`, instead of your own numbers.
- **Colors**: the named palette `var(--{name}-color)` (red, pink, purple, deep-purple, indigo,
  blue, light-blue, cyan, teal, green, light-green, lime, yellow, amber, orange, deep-orange,
  brown, grey, blue-grey...), the semantic ones (`--info-color`, `--warning-color`,
  `--error-color`, `--success-color`) and the state ones (`--state-icon-color`,
  `--state-inactive-color`). Let the user pick the card color with the `ui_color` selector and
  turn names into `var(--{name}-color)`.
- **Tints instead of new colors**: core components put a layer of the color behind the content
  (`::before` with `opacity: 0.2` for tile icons and buttons, `0.12` for alerts);
  `color-mix(in srgb, var(--accent) 20%, transparent)` does the same where a layer doesn't fit.
  Both are right in light and dark, because they show whatever is behind them.
- **Neutral controls** use `--disabled-color` at 0.2 opacity, 42px high, like card features.
- **Component tokens**: honor `--ha-tile-icon-border-radius`, `--ha-card-features-border-radius`,
  `--ha-badge-border-radius`, `--ha-dialog-border-radius` and `--ha-dialog-surface-background`
  in the matching pattern, so a theme that restyles core cards restyles yours too.
- **Icons**: `<ha-icon icon="mdi:...">` sized with `--mdc-icon-size`. Check that every icon name
  exists in the MDI version Home Assistant ships (a missing name renders as an empty square).
- **Always give a fallback**, `var(--ha-font-size-m, 14px)`: older Home Assistant versions and
  some themes don't define the newer tokens.

Design on top, not instead: a custom dial, a hob map or a usage chart is fine, as long as its
strokes, fills, text and states come from the same tokens. Check it with a dark theme and with a
custom theme that changes the primary color.

## Light and dark, always both

Every card must look right in the light theme and in the dark theme from the first draft, not as a
fix later. Users switch themes by hand, by schedule or with the sun, and a card that only works in
one of them is broken for half of them.

- **The theme comes from Home Assistant, not from the operating system.** Don't use
  `prefers-color-scheme` inside a card: the user can run a dark Home Assistant on a light OS. Read
  colors from the theme tokens; if logic really needs to know, use `hass.themes.darkMode`.
- **No fixed colors for text, surfaces, lines or fills.** Every one of them comes from a token, so
  the theme changes them for you. The only fixed colors are fallbacks inside `var()`.
- **Tint with an opacity layer or `color-mix(..., transparent)`**, never by mixing with white or
  black: the tint then sits on the light or dark card background and keeps its meaning in both.
- **Defaults change with the theme**, not only the background: `--disabled-color` goes from
  `#bdbdbd` to `#464646`, `--energy-grid-return-color` changes too. Colors computed in
  JavaScript (chart shades read with `getComputedStyle`) must be recomputed when `hass.themes`
  changes.
- **Text on a filled shape** (a full accent button, a zone at high power) uses
  `--text-primary-color` or white, and the fill must be strong enough for it in both themes:
  set a minimum opacity rather than letting a pale fill carry white or dark text.
- **Shadows and scrims** stay neutral and soft (`rgba(0, 0, 0, 0.28)` and similar); they read in
  both themes. Borders use `--divider-color` or the card border tokens.
- **SVG** drawn by the card uses `currentColor` or the same tokens in `fill` and `stroke`, never
  literal colors.
- **Check both themes on every change**: in the demo, in screenshots and in the release images.
  README images ship as a pair inside `<picture>` with
  `<source media="(prefers-color-scheme: dark)">`, because GitHub follows the reader's theme.

## Behavior that feels native

- **Tap the header or a value to open more-info** (`hass-more-info` event): users expect to reach
  the entity, its history and its settings from anywhere on a card.
- **Use standard services** (`switch.toggle`, `select.select_option`, `number.set_value`,
  `button.press`), so actions show in the logbook and can be reproduced in automations.
- **Format with Home Assistant**: `hass.formatEntityState(stateObj)` for states and units;
  `Intl` formatters that follow `hass.locale.number_format` (decimal separator chosen by the
  user, not by the language), `hass.locale.time_format` (12, 24, language, system) and
  `hass.locale.time_zone`. Don't format numbers or dates by hand.
- **Optimistic UI with a timeout**: show the new value at once, mark it as pending, and drop the
  pending value when the state arrives or after a few seconds.
- **Debounce repeated taps** on minus and plus, then send one call, like the thermostat card.
- **Don't re-render what didn't change**: `hass` is replaced on every state change in the
  instance. Redraw only when the card's own entities, the registry, the language or the theme
  changed, and then patch the DOM rather than replacing it, so open selects, focus and
  animations survive.
- **Errors** go to a toast (`hass-notification` event) in the user's language.
- **Unavailable is not off**: show `unavailable` and `unknown` as such, hide features whose
  entities never report a value, and don't treat an entity left behind by an old version
  (`restored: true` in its attributes) as the real one.

## Finding entities

Don't ask users to type entity ids. Let them pick the device in the editor (device selector
filtered by integration and model), then read `hass.entities` (the entity registry) and keep the
entities of that device, recognized by their `translation_key`. Renamed entities keep working,
and the card works in every language. When versions of the integration name a key differently,
keep a small alias table; when an old and a new entity share a key, prefer the one that has a
state.

## Languages

Follow the user's language (`hass.locale.language`, then `hass.language`), with English for
anything else. Ship your own strings for what the integration doesn't translate (program names,
options), named like the vendor's own app in that language, and keep them in one table that can be
exported for translators. Program and option ids from different appliances can collide: scope
names by appliance type when the same id means different things.

## Layout and accessibility

- **Grid size**: implement `getGridOptions()` for sections dashboards (for example
  `{ columns: 12, min_columns: 6 }`) and `getCardSize()` for masonry.
- **Container queries** (`container-type: inline-size` on the card) instead of viewport media
  queries: the same card can be a third of a desktop or a full phone width.
- **Touch targets** of at least 40px, rows of 42px like card features.
- **Keyboard**: `role="button"`, `tabindex="0"`, Enter and Space on anything clickable;
  `aria-pressed` on toggles; `aria-label` on icon-only buttons; a `:focus-visible` ring in the
  accent color.
- **Dialogs**: a native `<dialog>` styled like Home Assistant's (580px wide, 24px radius), that
  becomes a bottom sheet on phones (core switches below 870px wide or 500px tall) with room for
  the safe area.
- **Reduced motion**: stop or shorten animations under `prefers-reduced-motion: reduce`, as core
  does.

## Changing an existing card

Treat every card that already ships as a contract: adding a card or a feature must not change the
others. Before proposing a commit, render every card, in every state and language, with the
published version and with the new one, and explain every difference (see
`references/verification.md`). Data from real users, including the other version of the
integration, belongs in that check. Shared code (base classes, string tables, alias tables,
helpers) is where accidental changes come from: give new behavior a hook that does nothing by
default, so the output of the other cards stays identical byte for byte.

Lessons from real appliances that a mock won't show:

- Durations in hours as floats: one minute is `0.0166666666666667 h`, which is 60.0000001 seconds.
  Round before comparing with thresholds.
- Options listed as text, so `min120` comes before `min30`: sort stepped values by their number.
- State casing differs between versions (`Run` and `run`): normalize before comparing.
- An entity can exist and stay `unavailable` forever because the appliance lists the feature but
  never reports it.
- Several entities can share one `translation_key` (zones, cavities): tell them apart by the number
  in the entity id or friendly name.

## Shipping

- **One ES module file**: hand-written vanilla JavaScript, or Lit and TypeScript bundled by
  esbuild. A `VERSION` constant and a console banner with it.
- **Register after the app has started.** Home Assistant swaps `window.customElements` for a
  polyfilled registry while it boots; an element defined before that is unknown to the
  dashboard ("Custom element doesn't exist"). Wait for `home-assistant` (or `hc-main`) to be
  defined, then define the card and editor if not defined yet. Push the card into
  `window.customCards` once, with `preview: true`, so it shows in the card picker.
- **Distribution**: a HACS plugin repository (`hacs.json` with the file name), or the card served
  by its own integration (static path plus `add_extra_js_url`, with a `?v=` that changes with the
  file), so users install one thing. Details in `references/card-api.md`.
- A release per version with short notes.
- README images rendered from the real card in light and dark, text drawn as paths, with demo
  data that contains nothing personal (names, IP addresses, serial numbers).
