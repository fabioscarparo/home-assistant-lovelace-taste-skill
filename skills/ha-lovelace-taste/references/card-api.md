# Card API, editor, registration and distribution

Contents:
1. Choosing the stack
2. Card skeleton (vanilla custom element)
3. Re-rendering only when needed
4. Visual editor with `ha-form`
5. Registration that survives Home Assistant's registry swap
6. `hass` helpers: formatting, services, statistics, navigation
7. Events the frontend listens to
8. Distribution: HACS plugin or served by the integration

## 1. Choosing the stack

Both work; pick by who maintains the card:

- **One plain JavaScript file**, a vanilla custom element with a shadow root and a small DOM
  diff. No build step, easy to read in a HACS install, nothing to update.
- **Lit and TypeScript**, bundled by esbuild into one ES module (`format: "esm"`,
  `target: "es2021"`, minified, the version injected with `define`). Better typing and
  templating for a large card; commit the built file where it is served from.

Don't import Home Assistant's own internal modules: they are not a public API. Use the
elements the frontend already defines (`ha-card`, `ha-icon`, `ha-alert`, `ha-form`,
`ha-icon-button`), the `hass` object and events.

## 2. Card skeleton

```js
const CARD_TYPE = "my-device-card";
const EDITOR_TYPE = "my-device-card-editor";
const VERSION = "1.0.0";
const DOMAIN = "my_integration";

class MyDeviceCard extends HTMLElement {
  static getConfigElement() {
    return document.createElement(EDITOR_TYPE);
  }

  // Shown in the card picker: point it at the first device of the integration
  static getStubConfig(hass) {
    const entry = Object.values(hass.entities).find((e) => e.platform === DOMAIN && e.device_id);
    return entry ? { device_id: entry.device_id } : {};
  }

  constructor() {
    super();
    this.attachShadow({ mode: "open" });
  }

  setConfig(config) {
    if (!config) throw new Error("Invalid configuration");
    this._config = { ...config };
    this._render();
  }

  set hass(hass) {
    const old = this._hass;
    this._hass = hass;
    if (!old || this._changed(old, hass)) this._render();
  }

  get hass() {
    return this._hass;
  }

  connectedCallback() {
    // Width-dependent drawing (charts, flows): measure the element, not the viewport
    this._ro ??= new ResizeObserver(() => this._render());
    this._ro.observe(this);
    // Back on the dashboard: the data may have changed while detached
    if (this._detached) this._render();
    this._detached = false;
  }

  disconnectedCallback() {
    this._ro?.disconnect();
    this._detached = true;
  }

  getCardSize() {
    return 5; // masonry rows of about 50px
  }

  getGridOptions() {
    return { columns: 12, min_columns: 6 }; // sections: 12 is full width
  }
}
```

Entities of the configured device, by `translation_key`, so renamed entities keep working:

```js
function resolveEntities(hass, deviceId) {
  const ids = new Map();
  for (const entry of Object.values(hass.entities)) {
    if (entry.platform !== DOMAIN || entry.device_id !== deviceId || !entry.translation_key) continue;
    ids.set(`${entry.entity_id.split(".")[0]}.${entry.translation_key}`, entry.entity_id);
  }
  return ids;
}
```

Rebuild this map only when `hass.entities` or the device changes (keep the source objects and
compare by identity). When several entities share a key, keep the one whose state isn't
`unavailable` and isn't `restored: true`.

States to treat as no value: `unavailable`, `unknown`, `""`. A numeric helper:

```js
const numericState = (stateObj) => {
  if (!stateObj || stateObj.state === "") return undefined;
  const value = Number(stateObj.state);
  return Number.isFinite(value) ? value : undefined;
};
```

Normalize units before comparing (`W`, `kW`, `MW`; `min`, `h`), and round floats before
comparing them with thresholds.

## 3. Re-rendering only when needed

`hass` is replaced on every state change in the whole instance, many times per second. Redraw
only when something the card shows changed:

```js
_changed(old, hass) {
  if (old.entities !== hass.entities || old.locale !== hass.locale || old.language !== hass.language || old.themes !== hass.themes) return true;
  return this._watched().some((id) => old.states[id] !== hass.states[id]);
}
```

`_watched()` lists the device's entities plus any extra entities from the config. In Lit, put
the same test in `shouldUpdate`. Then render without destroying what didn't change: compare
the new markup with the last one and patch only the changed nodes, so focus, open selects,
hover and animations survive.

When the card writes markup with `innerHTML`, escape every value from states, attributes and
config:

```js
const escapeHtml = (s) =>
  String(s).replace(/[&<>"']/g, (c) => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" })[c]);
```

## 4. Visual editor with `ha-form`

`ha-form` is only defined once a core card editor has loaded. Load one first:

```js
async function loadHaForm() {
  if (customElements.get("ha-form")) return;
  const helpers = await window.loadCardHelpers?.();
  const card = await helpers?.createCardElement({ type: "entities", entities: [] });
  await card?.constructor?.getConfigElement?.();
}
```

The card's own strings, in the user's language with English as fallback (`t` is used by the
editor below and by the card):

```js
const STRINGS = {
  en: { editor_device_id: "Device", helper_device_id: "The appliance to show", editor_show_controls: "Show controls" },
  it: { editor_device_id: "Dispositivo", helper_device_id: "L'elettrodomestico da mostrare", editor_show_controls: "Mostra i comandi" },
};
const t = (hass, key, fallback = key) => {
  const lang = hass?.locale?.language || hass?.language || "en";
  const table = STRINGS[lang] || STRINGS[lang.split("-")[0]] || {};
  return table[key] ?? STRINGS.en[key] ?? fallback;
};
```

```js
const DEFAULTS = { show_controls: true };

class MyDeviceCardEditor extends HTMLElement {
  setConfig(config) {
    this._config = config;
    this._render();
  }

  set hass(hass) {
    this._hass = hass;
    this._render();
  }

  connectedCallback() {
    loadHaForm().then(() => customElements.whenDefined("ha-form")).then(() => this._render());
  }

  _render() {
    if (!this._hass || !this._config || !customElements.get("ha-form")) return;
    if (!this._form) {
      this._form = document.createElement("ha-form");
      this._form.addEventListener("value-changed", (ev) => {
        const config = { ...ev.detail.value };
        // Keep the YAML short: drop defaults and empty values
        for (const [key, value] of Object.entries(DEFAULTS)) if (config[key] === value) delete config[key];
        for (const key of Object.keys(config)) if (config[key] === "" || config[key] === undefined) delete config[key];
        this.dispatchEvent(new CustomEvent("config-changed", { detail: { config }, bubbles: true, composed: true }));
      });
      this.appendChild(this._form);
    }
    this._form.hass = this._hass;
    this._form.data = { ...DEFAULTS, ...this._config };
    this._form.schema = SCHEMA;
    this._form.computeLabel = (schema) => t(this._hass, `editor_${schema.name}`);
    this._form.computeHelper = (schema) => t(this._hass, `helper_${schema.name}`, null);
  }
}

const SCHEMA = [
  { name: "device_id", required: true, selector: { device: { filter: { integration: DOMAIN } } } },
  { type: "grid", name: "", schema: [
    { name: "name", selector: { text: {} } },
    { name: "icon", selector: { icon: {} } },
  ] },
  { name: "color", selector: { ui_color: { include_none: false } } },
  { name: "power_entity", selector: { entity: { filter: { domain: "sensor", device_class: "power" } } } },
  { name: "period", selector: { select: { mode: "dropdown", options: [{ value: "day", label: "Day" }] } } },
  { name: "content", type: "expandable", flatten: true, title: "Content", iconPath: mdiFormatListBulleted /* an @mdi/js path */, schema: [
    { name: "show_controls", selector: { boolean: {} } },
  ] },
];
```

- The device selector filtered by integration (and `model` or `manufacturer` if needed)
  replaces asking for entity ids.
- `expandable` with `flatten: true` groups options without nesting them in the YAML.
- Labels and helpers come from the card's own strings in the user's language.
- `getStubConfig` and the editor's defaults must agree, or the preview and the saved card
  differ.

## 5. Registration that survives Home Assistant's registry swap

In browsers without scoped custom element registries, Home Assistant replaces
`window.customElements` with a polyfilled registry while it starts, and only knows elements
defined through it. A module that loads early (an extra module from an integration, a fast
cache) and defines its element at once lands in the old registry: the dashboard then shows
"Custom element doesn't exist". Define the elements once the app has started, reading
`window.customElements` at that moment:

```js
const define = () => {
  const registry = window.customElements;
  if (!registry.get(CARD_TYPE)) registry.define(CARD_TYPE, MyDeviceCard);
  if (!registry.get(EDITOR_TYPE)) registry.define(EDITOR_TYPE, MyDeviceCardEditor);
};

if (window.customElements.get("home-assistant") || window.customElements.get("hc-main")) {
  define();
} else {
  Promise.race([
    window.customElements.whenDefined("home-assistant"),
    window.customElements.whenDefined("hc-main"), // casting
    new Promise((resolve) => setTimeout(resolve, 30000)), // safety net
  ]).then(define);
}

window.customCards = window.customCards || [];
if (!window.customCards.some((c) => c.type === CARD_TYPE)) {
  window.customCards.push({
    type: CARD_TYPE,
    name: "My device",
    description: "Status and controls of my device.",
    preview: true,
    documentationURL: "https://github.com/owner/repo#card",
  });
}

console.info(`%c MY-DEVICE-CARD %c ${VERSION} `, "color: #fff; background: #03a9f4; font-weight: 500", "");
```

A demo page outside Home Assistant has no `<home-assistant>`: define a stub of it, or let the
safety net fire.

## 6. `hass` helpers

Formatting, in this order of preference:

- `hass.formatEntityState(stateObj)` for a state with its unit and translation;
  `hass.formatEntityAttributeValue(stateObj, attribute)` and
  `hass.formatEntityAttributeName(stateObj, attribute)` for attributes.
- Numbers with `Intl.NumberFormat`, following `hass.locale.number_format`:

  ```js
  function numberLocale(hass) {
    switch (hass.locale?.number_format) {
      case "comma_decimal": return "en-US";
      case "decimal_comma": return "de";
      case "space_comma": return "fr";
      case "quote_decimal": return "de-CH";
      case "system": return undefined;
      default: return hass.locale?.language || hass.language; // "language"
    }
  }
  const formatNumber = (hass, value, digits) => new Intl.NumberFormat(numberLocale(hass), {
    minimumFractionDigits: digits, maximumFractionDigits: digits, useGrouping: hass.locale?.number_format !== "none",
  }).format(value);
  ```

- Times with `Intl.DateTimeFormat`, `hour12` from `hass.locale.time_format` (`"12"`, `"24"`,
  or `"language"` and `"system"`, which follow the locale), and the time zone from
  `hass.locale.time_zone` (`"server"` means `hass.config.time_zone`).
- Durations from the integration in seconds or hours as floats: round to the minute first.

Services, with errors shown to the user:

```js
async _call(domain, service, data) {
  try {
    await this._hass.callService(domain, service, data);
  } catch (err) {
    this.dispatchEvent(new CustomEvent("hass-notification", { detail: { message: err?.message ?? String(err) }, bubbles: true, composed: true }));
  }
}
```

Long-term statistics (what the Energy dashboard reads):

```js
const ids = await hass.callWS({ type: "recorder/list_statistic_ids", statistic_type: "sum" });
const stats = await hass.callWS({
  type: "recorder/statistics_during_period",
  start_time: start.toISOString(), end_time: end.toISOString(),
  statistic_ids: [statisticId], period: "day", // 5minute, hour, day, week, month
  types: ["change"], units: { energy: "kWh" },
});
```

Cache the answers per period and drop late answers when the user moved on (a request counter).

Other things the card can read: `hass.themes.darkMode`, `hass.user.is_admin` (hide admin-only
actions, or say to ask an admin), `hass.config.version` (feature checks), `hass.devices`,
`hass.areas`.

Per-viewer preferences (a hint dismissed, the last tab) go in `localStorage` inside
`try`/`catch`: private mode and the companion app can block it.

## 7. Events the frontend listens to

All with `bubbles: true, composed: true`:

| Event | Detail | Effect |
|---|---|---|
| `hass-more-info` | `{ entityId }` | opens the entity's more-info dialog |
| `hass-notification` | `{ message }` | toast at the bottom |
| `config-changed` | `{ config }` | from the editor: the new config |
| `ll-custom` | anything | for cards that talk to a parent card |

Navigation without a reload, like the frontend's own helper:

```js
const navigate = (path) => {
  history.pushState(null, "", path);
  window.dispatchEvent(new CustomEvent("location-changed", { detail: { replace: false } }));
};
// My Home Assistant redirects resolve to the right page on every version:
navigate("/_my_redirect/config_flow_start?domain=my_integration");
navigate("/_my_redirect/developer_call_service?service=my_integration.refresh");
```

## 8. Distribution

### HACS plugin (a separate repository)

- `hacs.json` with `name` and `filename` (the JS file at the root or in `dist/`), and
  `render_readme` if wanted.
- Users add it from HACS; HACS adds the dashboard resource.
- A GitHub release per version, with short notes.

### Served by the integration (no second install)

Put the built file in `custom_components/<domain>/frontend/`, serve it and add it to every
dashboard from `async_setup`, once per start:

```python
from pathlib import Path
import hashlib

CARD_FILE = Path(__file__).parent / "frontend" / "my-device-card.js"
CARD_URL = f"/{DOMAIN}_static/my-device-card.js"

async def async_register_card(hass: HomeAssistant) -> None:
    if hass.data.get(f"{DOMAIN}_card") or hass.http is None or "frontend" not in hass.config.components:
        return  # no web server or frontend (tests, headless): the card must never block setup
    from homeassistant.components.frontend import add_extra_js_url
    from homeassistant.components.http import StaticPathConfig

    digest = await hass.async_add_executor_job(lambda: hashlib.sha256(CARD_FILE.read_bytes()).hexdigest()[:12])
    await hass.http.async_register_static_paths([StaticPathConfig(CARD_URL, str(CARD_FILE), cache_headers=True)])
    add_extra_js_url(hass, f"{CARD_URL}?v={digest}")
    hass.data[f"{DOMAIN}_card"] = True
```

- `"after_dependencies": ["frontend", "http"]` in `manifest.json`, and the imports inside the
  function, so the integration still loads without them.
- `cache_headers=True` caches the file for a month: the `?v=` query must change with the file.
  A hash of the content is safer than the version, which doesn't change on every commit.
- Extra modules load early: the registration in section 5 is required here.
