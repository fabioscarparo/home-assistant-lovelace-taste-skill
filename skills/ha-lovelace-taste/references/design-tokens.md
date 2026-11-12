# Home Assistant design tokens for custom cards

Contents:
1. Where the tokens come from
2. Tokens by role, with light and dark defaults
3. Card defaults (`:host`)
4. Native patterns: tile header, big value, badges, alerts, features, segmented control,
   select rows, charts and tooltips, dialogs and bottom sheets
5. Light and dark checks

Values below are the frontend's own (`src/resources/theme/*.globals.ts`, `ha-tile-icon`,
`ha-badge`, `ha-alert`, `ha-control-button`, `hui-card-features`, `ha-dialog`,
`ha-adaptive-dialog` in home-assistant/frontend, `dev` branch, October 2026). They move between
releases: when exact parity matters, read the current source again.

## 1. Where the tokens come from

- **Classic tokens** (`--primary-text-color`, `--card-background-color`, `--primary-color`,
  `--{name}-color`...) are what themes override. Read these first: a user's custom theme
  changes them, and the defaults change with light and dark.
- **Core tokens** (`--ha-space-*`, `--ha-border-radius-*`, `--ha-font-*`, `--ha-line-height-*`,
  `--ha-animation-duration-*`, `--ha-box-shadow-*`) are the newer scale. Use them for sizes,
  always with a fallback, because older releases don't define them.
- **Semantic color tokens** (`--ha-color-text-primary`, `--ha-color-fill-primary-normal-resting`,
  `--ha-color-surface-default`...) are the newest layer; the classic tokens are now defined on top
  of them. Custom themes rarely set them, so a card that reads them can ignore a theme. Prefer the
  classic token; reach for a semantic one only when nothing classic fits, with a classic fallback.
- **Component tokens** (`--ha-card-*`, `--ha-tile-icon-border-radius`,
  `--ha-card-features-border-radius`, `--ha-badge-*`, `--ha-dialog-*`) let themes restyle one
  kind of component. Honor them in the matching custom pattern, so a theme that rounds tiles or
  badges rounds yours too.

Always write a fallback: `var(--ha-font-size-m, 14px)`.

## 2. Tokens by role

| Role | Tokens | Light / dark default |
|---|---|---|
| Card surface | `--ha-card-background`, `--card-background-color` | `#ffffff` / `#1c1c1c` |
| Card border, radius, shadow | `--ha-card-border-color` (else `--divider-color`), `--ha-card-border-width` (1px), `--ha-card-border-radius` (else `--ha-border-radius-lg`, 12px), `--ha-card-box-shadow` (none), `--ha-card-backdrop-filter` | `<ha-card>` applies them |
| Page | `--primary-background-color`, `--secondary-background-color` | `#fafafa`, `#e5e5e5` / `#111111`, `#282828` |
| Text | `--primary-text-color`, `--secondary-text-color`, `--disabled-text-color` | `#212121`, `#727272`, `#bdbdbd` / `#e1e1e1`, `#9b9b9b`, `#6f6f6f` |
| Text on a fill | `--text-primary-color` | `#ffffff` in both |
| Lines | `--divider-color`, `--outline-color`, `--outline-hover-color` | `rgba(0,0,0,.12)` / `rgba(225,225,225,.12)` |
| Neutral fill | `--disabled-color` | `#bdbdbd` / `#464646` |
| Accent | `--primary-color`, `--accent-color` (`#ff9800`) | theme dependent |
| Named palette | `--red-color` `#f44336`, `--pink-color`, `--purple-color`, `--deep-purple-color`, `--indigo-color`, `--blue-color` `#2196f3`, `--light-blue-color`, `--cyan-color`, `--teal-color`, `--green-color` `#4caf50`, `--light-green-color`, `--lime-color`, `--yellow-color`, `--amber-color` `#ffc107`, `--orange-color` `#ff9800`, `--deep-orange-color`, `--brown-color`, `--light-grey-color`, `--grey-color` `#9e9e9e`, `--dark-grey-color`, `--blue-grey-color`, `--black-color`, `--white-color` | what the `ui_color` selector returns |
| Semantic | `--info-color` `#039be5`, `--warning-color` `#ffa600`, `--error-color` `#db4437`, `--success-color` `#43a047` | same in both |
| RGB triplets | `--rgb-primary-color`, `--rgb-primary-text-color`, `--rgb-card-background-color`, `--rgb-{warning,error,success,info}-color` | for `rgba(var(--rgb-x), .5)` |
| State | `--state-icon-color` `#44739e`, `--state-active-color` (amber), `--state-inactive-color` (grey), `--state-unavailable-color`, `--state-{domain}-{state}-color` | tile icon and feature color |
| Energy | `--energy-grid-consumption-color` `#488fc2`, `--energy-grid-return-color` `#8353d1` / `#a280db`, `--energy-solar-color` `#ff9800`, `--energy-battery-in-color` `#f06292`, `--energy-battery-out-color` `#4db6ac`, `--energy-non-fossil-color`, `--energy-gas-color`, `--energy-water-color` `#00bcd4` | same as the Energy dashboard |
| Chart series | `--color-1` ... `--color-54` | the palette of HA's charts |
| Font | `--ha-font-family-body` (Roboto, Noto, sans-serif), `--ha-font-family-code` | |
| Font size | `--ha-font-size-xs` 10, `-s` 12, `-m` 14, `-l` 16, `-xl` 20, `-2xl` 24, `-3xl` 28, `-4xl` 32, `-5xl` 40 (px, times `--ha-font-size-scale`) | |
| Font weight | `--ha-font-weight-light` 300, `-normal` 400, `-medium` 500, `-bold` 700 | |
| Line height | `--ha-line-height-condensed` 1.2, `-normal` 1.6, `-expanded` 2 | |
| Spacing | `--ha-space-1` 4px ... `--ha-space-20` 80px, in steps of 4px | |
| Radius | `--ha-border-radius-sm` 4, `-md` 8, `-lg` 12, `-xl` 16, `-2xl` 20, `-3xl` 24, `-4xl` 28, `-pill` 9999px, `-circle` 50% | |
| Shadow | `--ha-box-shadow-s`, `-m`, `-l` | stronger in dark |
| Motion | `--ha-animation-duration-instant` 75ms, `-fast` 150ms, `-normal` 250ms, `-slow` 350ms | 1ms with reduced motion |
| Icons | `--mdc-icon-size` | size of `ha-icon` |
| Safe area | `--safe-area-inset-bottom` (and top, left, right) | bottom sheets on phones |

Turning a color config into CSS:

```js
const HA_COLORS = new Set(["primary", "accent", "red", "pink", "purple", "deep-purple", "indigo",
  "blue", "light-blue", "cyan", "teal", "green", "light-green", "lime", "yellow", "amber", "orange",
  "deep-orange", "brown", "light-grey", "grey", "dark-grey", "blue-grey", "black", "white", "disabled"]);
const colorVar = (color) => (HA_COLORS.has(color) ? `var(--${color}-color)` : color);
// in setConfig: this.style.setProperty("--accent", colorVar(config.color || "blue"));
```

## 3. Card defaults

```css
:host {
  display: block;
  --tile-color: var(--state-inactive-color, #9e9e9e);
  --radius: var(--ha-card-features-border-radius, var(--ha-border-radius-lg, 12px));
}
ha-card { height: 100%; overflow: hidden; }
.card {
  display: flex; flex-direction: column; gap: var(--ha-space-3, 12px);
  padding: var(--ha-space-3, 12px) var(--ha-space-4, 16px) var(--ha-space-4, 16px);
  container-type: inline-size;
  font-family: var(--ha-font-family-body, Roboto, Noto, sans-serif);
  color: var(--primary-text-color);
  -webkit-font-smoothing: antialiased;
}
button { font: inherit; color: inherit; background: none; border: none; margin: 0; padding: 0; cursor: pointer; }
button:disabled { cursor: not-allowed; }
svg { display: block; fill: currentColor; }
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 1ms !important; transition-duration: 1ms !important; }
}
```

## 4. Native patterns

### Tints: a layer, like the core components

Core components tint with a pseudo-element at a fixed opacity, so the color stays a plain token
and works with any color format a theme uses:

```css
.tinted { position: relative; overflow: hidden; }
.tinted::before {
  content: ""; position: absolute; inset: 0; border-radius: inherit; pointer-events: none;
  background: var(--tint); opacity: 0.2;
  transition: opacity 180ms ease-in-out;
}
.tinted > * { position: relative; }
```

The opacities core uses: tile icon 0.2 (0.35 on hover), control buttons and selects 0.2 (0.3 on
hover), alerts 0.12, badge hover 0.04, pressed 0.12. Where a pseudo-element doesn't fit (an
inline style, an SVG fill), `color-mix(in srgb, var(--tint) 20%, transparent)` gives the same
result. Never mix with white or black: the result would be right in one theme only.

### Tile header

```css
.tile { display: flex; align-items: center; gap: 10px; min-height: 64px; padding: 8px 16px 0; box-sizing: border-box; cursor: pointer; border-radius: var(--ha-card-border-radius, 12px); outline: none; }
.tile:focus-visible { box-shadow: inset 0 0 0 2px var(--tile-color); }
.tile-icon {
  position: relative; flex: none; width: 36px; height: 36px; overflow: hidden;
  display: flex; align-items: center; justify-content: center;
  border-radius: var(--ha-tile-icon-border-radius, var(--ha-border-radius-pill, 9999px));
  color: var(--tile-color); --mdc-icon-size: 24px;
}
.tile-icon::before { content: ""; position: absolute; inset: 0; background: var(--tile-color); opacity: 0.2; transition: opacity 180ms ease-in-out; }
.tile:hover .tile-icon::before { opacity: 0.35; }
.tile-icon > * { position: relative; }
.tile-info { flex: 1; min-width: 0; display: flex; flex-direction: column; justify-content: center; }
.primary, .secondary { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.primary { font-size: var(--ha-font-size-m, 14px); font-weight: var(--ha-font-weight-medium, 500); line-height: 20px; letter-spacing: 0.1px; color: var(--primary-text-color); }
.secondary { font-size: var(--ha-font-size-s, 12px); line-height: 16px; letter-spacing: 0.4px; color: var(--secondary-text-color); }
```

`--tile-color` is `--state-inactive-color` while off, the state or accent color while on or
running (core uses `--state-icon-color` or the domain's `--state-{domain}-{state}-color`). A
small status badge on the icon sits at `top: -3px; right: -3px`, 16px round.

### Big value

```css
.value { display: flex; align-items: baseline; gap: 4px; line-height: var(--ha-line-height-condensed, 1.2); }
.value .number { font-size: var(--ha-font-size-3xl, 28px); font-variant-numeric: tabular-nums; }
.value .unit { font-size: var(--ha-font-size-l, 16px); color: var(--secondary-text-color); }
```

### Badges, like `ha-badge`

```css
.badges { display: flex; flex-wrap: wrap; gap: 8px; }
.badge {
  display: inline-flex; align-items: center; gap: 8px; height: 36px; min-width: 36px; padding: 0 12px; box-sizing: border-box;
  border-radius: var(--ha-badge-border-radius, 18px);
  border: var(--ha-card-border-width, 1px) solid var(--ha-card-border-color, var(--divider-color, #e0e0e0));
  background: var(--ha-card-background, var(--card-background-color, #fff));
}
.badge ha-icon { --mdc-icon-size: 18px; margin-inline-start: -4px; color: var(--badge-color, var(--secondary-text-color)); }
.badge .label { font-size: var(--ha-font-size-xs, 10px); font-weight: 500; line-height: 10px; letter-spacing: 0.1px; color: var(--secondary-text-color); }
.badge .content { font-size: var(--ha-font-size-s, 12px); font-weight: 500; line-height: 1.2; letter-spacing: 0.1px; color: var(--primary-text-color); }
```

Inside a card, a lighter chip (32px, no background, same border) also reads as native.

### Alerts, like `ha-alert`

Use `<ha-alert alert-type="warning" title="...">` when it is loaded. Its look, for a custom one:

```css
.alert { position: relative; display: flex; padding: 12px; border-radius: var(--ha-border-radius-lg, 12px); overflow: hidden; }
.alert::before { content: ""; position: absolute; inset: 0; background: var(--alert-color); opacity: 0.12; }
.alert ha-icon { position: relative; flex: none; color: var(--alert-color); }
.alert .text { position: relative; margin-inline: 8px; line-height: normal; color: var(--primary-text-color); }
.alert .title { margin-top: 2px; font-weight: var(--ha-font-weight-bold, 700); }
.alert.info { --alert-color: var(--info-color, #039be5); }
.alert.warning { --alert-color: var(--warning-color, #ffa600); }
.alert.error { --alert-color: var(--error-color, #db4437); }
.alert.success { --alert-color: var(--success-color, #43a047); }
```

### Features: buttons, steppers, progress

Card features are 42px high, 12px apart, with `--feature-border-radius` (12px) and the neutral
fill `--disabled-color` at 0.2:

```css
.features { display: flex; flex-direction: column; gap: 12px; }
.buttons { display: flex; gap: 12px; }
.button {
  position: relative; overflow: hidden; z-index: 0; flex: 1 1 0; min-width: 0; height: 42px;
  display: flex; align-items: center; justify-content: center; gap: 8px; padding: 0 8px;
  border-radius: var(--radius); font-size: var(--ha-font-size-m, 14px); font-weight: var(--ha-font-weight-medium, 500);
  color: var(--primary-text-color); --mdc-icon-size: 20px; outline: none;
}
.button::before { content: ""; position: absolute; inset: 0; background: var(--disabled-color, #bdbdbd); opacity: 0.2; transition: opacity 180ms ease-in-out; pointer-events: none; }
.button:hover:not(:disabled)::before { opacity: 0.3; }
.button.active::before, .button.tonal::before { background: var(--tile-color); }
.button.primary::before { background: var(--tile-color); opacity: 1; }
.button.primary { color: var(--text-primary-color, #fff); }
.button:focus-visible { box-shadow: 0 0 0 2px var(--tile-color); }
.button:disabled { color: var(--disabled-text-color, #bdbdbd); }
.button > * { position: relative; }
.bar { display: flex; height: 42px; border-radius: var(--radius); overflow: hidden; }
.bar .done { background: var(--tile-color); }
.bar .rest { flex: 1; background: var(--tile-color); opacity: 0.2; }
```

A stepper (minus, value, plus) is a 42px row with the same fill. Long labels don't fit a stepper:
use a select row instead.

### Segmented control, like `ha-control-select`

```css
.segmented { position: relative; display: flex; gap: 4px; padding: 4px; height: 40px; box-sizing: border-box; border-radius: var(--ha-border-radius-lg, 12px); }
.segmented::before { content: ""; position: absolute; inset: 0; border-radius: inherit; background: var(--disabled-color, #bdbdbd); opacity: 0.2; }
.option { position: relative; flex: 1; min-width: 0; border-radius: calc(var(--ha-border-radius-lg, 12px) - 4px); font-size: var(--ha-font-size-m, 14px); font-weight: 500; color: var(--primary-text-color); }
.option::before { content: ""; position: absolute; inset: 0; border-radius: inherit; background: var(--tile-color); opacity: 0; transition: opacity 80ms ease-in-out; }
.option:hover::before { opacity: 0.2; }
.option.selected { color: var(--text-primary-color, #fff); }
.option.selected::before { opacity: 1; }
.option span { position: relative; }
```

### Select rows

A styled row with the real `<select>` laid over it at opacity 0: native picker on phones,
keyboard support for free. Give the hidden select `font-size: 16px`, or iOS zooms the page.

```css
.select { position: relative; overflow: hidden; height: 42px; display: flex; align-items: center; gap: 10px; padding: 6px 10px; box-sizing: border-box; border-radius: var(--radius); color: var(--primary-text-color); }
.select::before { content: ""; position: absolute; inset: 0; background: var(--disabled-color, #bdbdbd); opacity: 0.2; }
.select > * { position: relative; }
.select .label { font-size: var(--ha-font-size-s, 12px); letter-spacing: 0.4px; }
.select .current { font-size: var(--ha-font-size-m, 14px); }
.select select { position: absolute; inset: 0; width: 100%; height: 100%; margin: 0; opacity: 0; cursor: pointer; font-size: 16px; }
.select:focus-within { box-shadow: 0 0 0 2px var(--tile-color); }
```

When the list needs icons, descriptions or groups, open a dialog with the list instead.

### Charts and tooltips, like the energy cards

- Bars: fill at 50% opacity with a solid 1.5px border of the same color; grid lines
  `--divider-color`; labels 12px in `--primary-text-color`.
- Series colors: the energy tokens for energy, `--color-1`... for anything else. For shades of
  one token (bands of one meter), read it with `getComputedStyle(this).getPropertyValue(...)`
  and derive the shades in LAB, lighter in the dark theme (`hass.themes.darkMode`).
- Tooltip: card background, `1px solid var(--divider-color)`, radius 4px,
  `box-shadow: rgba(0, 0, 0, 0.2) 1px 2px 10px`, padding 8px 10px, 12px text, values medium
  weight with `tabular-nums`.

### Dialogs and bottom sheets, like `ha-dialog` and `ha-adaptive-dialog`

- Dialog: width `min(580px, 95vw)` (320px small, 1024px large), radius
  `var(--ha-dialog-border-radius, var(--ha-border-radius-3xl, 24px))`, background
  `var(--ha-dialog-surface-background, var(--card-background-color))`, content padding 24px,
  title 24px (`--ha-font-size-2xl`) at normal weight, open and close in 200ms.
- Scrim: dims the page (`--ha-dialog-scrim-backdrop-filter: brightness(68%)` in core; a plain
  `rgba(0, 0, 0, 0.32)` backdrop works too).
- Phones: the adaptive dialog becomes a bottom sheet below 870px wide or 500px tall, with
  `--ha-border-radius-2xl` (20px) top corners and padding for `--safe-area-inset-bottom`.

```css
dialog {
  width: min(580px, 95vw); max-height: min(80vh, 720px); padding: 0; border: none; overflow: hidden;
  border-radius: var(--ha-dialog-border-radius, var(--ha-border-radius-3xl, 24px));
  background: var(--ha-dialog-surface-background, var(--card-background-color, #fff));
  color: var(--primary-text-color); box-shadow: var(--ha-box-shadow-l, 0 16px 32px -6px rgba(0, 0, 0, 0.2));
  font-family: var(--ha-font-family-body, Roboto, Noto, sans-serif);
}
dialog::backdrop { background: rgba(0, 0, 0, 0.32); }
.dialog-title { font-size: var(--ha-font-size-2xl, 24px); line-height: 1.2; font-weight: var(--ha-font-weight-normal, 400); padding: 24px 24px 12px; }
.dialog-content { padding: 0 24px 24px; overflow: auto; }
@media (max-width: 870px), (max-height: 500px) {
  dialog { width: 100vw; max-width: 100vw; margin: auto 0 0; max-height: 90vh;
    border-radius: var(--ha-border-radius-2xl, 20px) var(--ha-border-radius-2xl, 20px) 0 0;
    padding-bottom: var(--safe-area-inset-bottom, 0px); }
}
```

A `<dialog>` inside the card's shadow root inherits the card's custom properties, so the accent
and the theme apply inside it.

## 5. Light and dark checks

Before calling a style done, look at it in both themes and ask:

- Is any color a literal outside a fallback? Replace it with a token.
- Does every tint use an opacity layer or a mix with `transparent`?
- Neutral fills come from `--disabled-color`, which is `#bdbdbd` in light and `#464646` in dark:
  does a control still stand out from the dark card?
- Does text on a filled shape stay readable at the lowest fill the shape can have?
- Do borders and dividers still show on the dark card background?
- Do SVG strokes and fills come from tokens or `currentColor`?
- Colors computed in JavaScript (chart shades) are recomputed when `hass.themes` changes?
- Are there README or store images for both themes?
