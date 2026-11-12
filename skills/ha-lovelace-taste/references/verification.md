# Verifying a card

Contents:
1. A demo page with a mock `hass`
2. Scenarios from real data
3. Regression check: published version against working copy
4. Unit tests for the logic
5. Images for the README, light and dark
6. Browser gotchas
7. Checklist before a release

Keep these tools out of the published package (a `tools/` or `demo/` folder, excluded from the
release or the HACS file), and keep the data in them anonymous.

## 1. A demo page with a mock `hass`

The card needs only a few things from Home Assistant, so a static page can render the real
card file with no Home Assistant at all:

- **Stub elements**: `ha-card` (a block with the card tokens: background, border, radius,
  shadow), `ha-icon` (an SVG from `@mdi/js` looked up by name, sized by `--mdc-icon-size`),
  `ha-alert` if used, and an empty `home-assistant` element so the registration waits and then
  runs as it does in the app.
- **Theme variables** on `:root` for light and for dark, with Home Assistant's defaults (see
  `design-tokens.md`, section 2), switched by `?theme=dark`, and the page background set to
  `--primary-background-color`.
- **A `hass` object**:

  ```js
  const hass = {
    states, // entity_id -> { entity_id, state, attributes, last_changed, last_updated, context }
    entities, // entity_id -> { entity_id, device_id, platform, translation_key, entity_category }
    devices, // id -> { id, name, model, manufacturer, name_by_user: null }
    areas: {},
    language: lang,
    locale: { language: lang, number_format: "language", time_format: "language", time_zone: "local" },
    themes: { darkMode: theme === "dark" },
    config: { unit_system: { temperature: "°C" }, time_zone: "Europe/Rome", version: "2026.10.1" },
    user: { name: "Demo", is_admin: true },
    formatEntityState: (stateObj, state) => `${state ?? stateObj.state}${stateObj.attributes.unit_of_measurement ? ` ${stateObj.attributes.unit_of_measurement}` : ""}`,
    callService: async (domain, service, data) => { /* update states, then assign a new hass */ },
    callWS: async (msg) => { /* answer recorder/statistics_during_period from simulated data */ },
  };
  ```

  After every change, give the card a new object (`card.hass = { ...hass, states: { ...states } }`):
  the card compares by identity, as it does with the real frontend.
- **URL parameters** for `theme`, `lang` and `scenario`, so any state can be opened, shared and
  captured.

Make time fixed (`Date.now = () => FIXED`) so remaining times and "ends at" don't change
between runs.

## 2. Scenarios from real data

- Build the scenarios from real exports: the entities of one device (with `translation_key`,
  state, attributes, unit, device class, disabled or not), in each phase (off, ready, running,
  paused, finished, error, offline).
- Ask users who report an issue for the integration's diagnostics or an export of the
  device's entities; turn them into fixtures after removing names, serial numbers, addresses,
  tokens and keys.
- When the integration has versions or forks that name keys differently, keep a fixture for
  each, and render every scenario on each variant.
- Include the ugly cases: entities `unavailable` forever, entities left behind by an older
  version (`restored: true`), options sorted as text, float durations, missing units, a second
  device of the same kind.

## 3. Regression check: published version against working copy

Every card that already ships is a contract. Before proposing a commit:

1. Extract the published file next to the working copy (`git show HEAD:card.js > tools/card-head.js`,
   or the tag of the last release).
2. One page renders every card × scenario × variant × language × layout (full, compact, every
   dialog or sheet the card can open) with one version, chosen by `?v=old` or `?v=new`, and
   stores the rendered markup by name (`results[name] = shadowRoot.innerHTML`).
3. A second page loads both in iframes and lists every name whose markup differs, with the text
   around the first difference, plus errors from either side.
4. Explain every difference: it must be one of the intended changes. Anything else is a
   regression, even if it looks harmless.

Make the run deterministic: fixed time, awaited async work (statistics, dialogs that close with
an event that fires later and would clear the next one), no random ids.

## 4. Unit tests for the logic

Pure functions (energy flow split, time estimates, option sorting, state mapping) get unit tests
(`node --test`, or the bundler's test runner), separate from the rendering checks. They are
where unit and rounding mistakes hide.

## 5. Images for the README, light and dark

GitHub shows README images with `<img>`, which can't load web fonts: text in an SVG falls back
to the reader's font and the layout shifts. Ways that hold up:

- **Headless Chrome prints each scenario to a one-page PDF the size of the card**, then PyMuPDF
  converts the PDF to SVG with the text as paths, and SVGO shrinks it. Several cards can be
  stacked into one "wall" SVG per theme.
- **DOM to SVG in the page**: walk the rendered card (boxes with background, border, radius,
  shadow, `::before` layers, clipping, text, inline SVG) and draw text as glyph outlines with
  opentype.js from the same Roboto files the page uses.
- **PNG screenshots** at 2x, when vector output isn't needed.

For all of them: load Roboto with `display=block` so nothing is captured in a fallback font,
turn on reduced motion so animations stand still, print backgrounds exactly
(`print-color-adjust: exact`), mix scenarios and kinds of card in a wall so no two of the same
sit side by side, and publish a pair:

```html
<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/card-dark.svg">
  <img alt="The card in a few states" src="assets/card-light.svg">
</picture>
```

## 6. Browser gotchas

- A preview pane that is hidden or not laid out has a 0×0 viewport: `ResizeObserver` reports
  0, and width-based drawing stays empty. Give the page an explicit size, or check the width
  before measuring.
- `color-mix()` needs Chrome 111, Safari 16.2 and Firefox 113; older companion app WebViews may
  lack it. The `::before` opacity layer works everywhere.
- iOS zooms on focus into inputs and selects under 16px.
- Check icon names against the MDI version Home Assistant ships: a missing name renders an
  empty square.
- Container queries need `container-type` on an ancestor inside the shadow root.

## 7. Checklist before a release

- [ ] Light and dark default themes, plus one custom theme with another primary color.
- [ ] Every language the card ships; strings that the integration translates come from it.
- [ ] Phone width (about 360px), a third of a desktop, a sections grid at `min_columns`, and
      masonry.
- [ ] Every scenario, including offline, unavailable and the other integration variants.
- [ ] Keyboard: tab order, Enter and Space, a visible focus ring, labels on icon buttons.
- [ ] No console errors; no re-render storm (count renders over a minute of state updates).
- [ ] Regression check clean, or every difference explained.
- [ ] Version bumped in one place, console banner shows it, release notes short.
- [ ] Images regenerated in both themes if the look changed, with anonymous data.
