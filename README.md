<p align="center">
  <img src="assets/icon.svg" alt="Home Assistant Lovelace Taste Skill" width="80">
</p>

<h1 align="center">Home Assistant Lovelace Taste Skill</h1>

<p align="center">
  <strong>Build Lovelace cards that look, behave and feel like part of Home Assistant.</strong>
</p>

<p align="center">
  <a href="#claude-code-as-a-plugin"><img alt="Claude Code plugin" src="https://img.shields.io/badge/Claude_Code-Plugin-D97757?logo=claude&logoColor=fff"></a>
  <a href="#other-agents"><img alt="Agent Skills" src="https://img.shields.io/badge/Agent_Skills-SKILL.md-000"></a>
  <a href="https://www.home-assistant.io/"><img alt="Home Assistant" src="https://img.shields.io/badge/Home_Assistant-18BCF2?logo=homeassistant&logoColor=fff"></a>
  <a href="LICENSE"><img alt="MIT" src="https://img.shields.io/badge/License-MIT-333"></a>
</p>

An Agent Skill for Claude and other coding agents that support the
`SKILL.md` format. It helps agents build custom Home Assistant cards and frontend components
that follow the platform's visual language and engineering conventions.

The scope goes beyond styling. It covers the complete lifecycle of a card: design tokens and
native interaction patterns, the Lovelace card API and visual editor, entity discovery,
registration, localization, verification and distribution.

## Why this skill exists

AI-generated cards can be functional and still feel out of place. They may hard-code colors that
break in dark mode, format numbers and dates independently of the user's locale, require users to
enter entity IDs by hand, or re-render the entire card whenever any entity in Home Assistant
changes. Some fail even earlier: when an integration loads the card before Home Assistant has
finished initializing its custom element registry, the dashboard reports that the custom element
doesn't exist.

This skill addresses those issues through one guiding principle:

**Let Home Assistant make the visual decisions. Build on its design tokens, components and
interaction patterns instead of recreating them.**

It combines that design foundation with the implementation details and verification steps needed
to make cards reliable across themes, languages, devices, layouts and releases.

## Cards built with it

The projects below show the skill applied to different integrations and use cases. The README
images follow your GitHub light or dark theme, while the cards themselves adapt to the active
Home Assistant theme.

<table>
  <tr>
    <td width="50%" valign="top">
      <a href="https://github.com/fabioscarparo/HomeAssistant-EDistribuzione"><picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/fabioscarparo/HomeAssistant-EDistribuzione/main/docs/images/pod-card-dark.svg">
        <img width="100%" alt="E-Distribuzione POD card in the day, week, month and year views" src="https://raw.githubusercontent.com/fabioscarparo/HomeAssistant-EDistribuzione/main/docs/images/pod-card-light.svg">
      </picture></a>
      <br>
      <b><a href="https://github.com/fabioscarparo/HomeAssistant-EDistribuzione">E-Distribuzione</a></b><br>
      A POD card for the Italian electricity distribution operator's integration. It displays
      energy consumption and injection across day, week, month and year views, and ships with the
      integration itself, so there is nothing else to install.
    </td>
    <td width="50%" valign="top">
      <a href="https://github.com/fabioscarparo/bosch-homeconnect-cards"><picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/fabioscarparo/bosch-homeconnect-cards/main/images/wall-dark.svg">
        <img width="100%" alt="Washer, dryer, dishwasher, oven and induction hob cards" src="https://raw.githubusercontent.com/fabioscarparo/bosch-homeconnect-cards/main/images/wall-light.svg">
      </picture></a>
      <br>
      <b><a href="https://github.com/fabioscarparo/bosch-homeconnect-cards">Bosch Home Connect cards</a></b><br>
      A set of cards for Bosch and Siemens appliances connected through Home Connect Local,
      distributed as a HACS plugin. It includes dedicated interfaces for washers, dryers,
      dishwashers, ovens and induction hobs.
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <a href="https://github.com/fabioscarparo/ha-growatt-thor-cloud"><picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/fabioscarparo/ha-growatt-thor-cloud/main/assets/card/wall-dark.svg">
        <img width="100%" alt="THOR Wallbox card while charging, with the home battery, with a scheduled start and with a fault" src="https://raw.githubusercontent.com/fabioscarparo/ha-growatt-thor-cloud/main/assets/card/wall-light.svg">
      </picture></a>
      <br>
      <b><a href="https://github.com/fabioscarparo/ha-growatt-thor-cloud">Growatt THOR</a></b><br>
      A TypeScript and Lit card for the Growatt THOR EV charger integration. It brings together
      charging modes, home battery status, scheduled starts and fault states.
    </td>
    <td width="50%" valign="top">
      <a href="https://github.com/fabioscarparo/helty-home-assistant"><picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/fabioscarparo/helty-home-assistant/main/images/wall-dark.svg">
        <img width="100%" alt="Helty Flow40 card in several modes, with alarms and with the unit offline" src="https://raw.githubusercontent.com/fabioscarparo/helty-home-assistant/main/images/wall-light.svg">
      </picture></a>
      <br>
      <b><a href="https://github.com/fabioscarparo/helty-home-assistant">Helty Flow40</a></b><br>
      A card inspired by the ventilation unit's physical control panel, designed for both the
      project's ESPHome firmware and Home Assistant's Helty integration.
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <a href="https://github.com/k-the-hidden-hero/bticino_ha_extras/pull/12"><picture>
        <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/k-the-hidden-hero/bticino_ha_extras/858ed896f52ce407ca0b604c233b0a1b6cb91a47/docs/images/card-100x-dark.png">
        <img width="100%" alt="BTicino Classe 100X card with an incoming call, the answer and reject buttons, the door and the stair light" src="https://raw.githubusercontent.com/k-the-hidden-hero/bticino_ha_extras/858ed896f52ce407ca0b604c233b0a1b6cb91a47/docs/images/card-100x-ringing.png">
      </picture></a>
    </td>
    <td width="50%" valign="middle">
      <b><a href="https://github.com/k-the-hidden-hero/bticino_intercom">BTicino Intercom</a></b>, coming soon<br>
      A Classe 100X card for the BTicino Intercom integration, featuring WebRTC video and two-way
      audio, a call log and a door button. Its interface is designed to sit naturally alongside
      Home Assistant's native cards. The implementation is currently under review in the
      integration's <a href="https://github.com/k-the-hidden-hero/bticino_ha_extras/pull/12">companion repository</a>.
    </td>
  </tr>
</table>

## What the skill covers

The skill brings together the visual, behavioral and technical details involved in building and
maintaining a production-ready Lovelace card.

- **Design system and native patterns**: Home Assistant color, typography, spacing, radius,
  shadow and motion tokens, including light and dark defaults. Includes copy-ready CSS for
  tile headers, badges, alerts, card features, segmented controls, select rows, charts, dialogs
  and bottom sheets.
- **Theme support**: token-based colors and tints that work across light, dark and custom themes.
  It avoids operating-system color preferences and accounts for theme changes when colors are
  computed in JavaScript.
- **Card API and configuration**: card and editor skeletons, `getGridOptions` for sections
  dashboards, visual editors built with `ha-form` and selectors, Home Assistant formatting
  helpers, services, statistics and frontend events.
- **Efficient updates**: the card re-renders only when its own entities, the registry, the
  language or the theme change, and patches the DOM instead of replacing it, so focus, open
  selects and animations survive.
- **Native interaction patterns**: more-info dialogs, standard service calls, optimistic updates,
  debounced controls, toast notifications and explicit handling of `unavailable` and
  `unknown` states.
- **Device-based entity discovery**: users select a device instead of entering entity IDs.
  The card resolves its entities through the entity registry and `translation_key`, remaining
  resilient to renamed entities and different interface languages.
- **Localization and locale-aware formatting**: the user's language, number format, 12- or
  24-hour clock and time zone.
- **Responsive layout and accessibility**: container queries, touch target sizes, keyboard
  interaction, visible focus indicators and reduced-motion support.
- **Reliable registration and distribution**: custom element registration after Home Assistant
  initializes its registry, HACS packaging, or serving the card directly from its integration.
- **Verification and release workflow**: a demo page with a mock `hass` object, scenarios based
  on real integration data, regression checks against the published version, screenshots in both
  themes and a release checklist.

## Example prompts

- "Build a card for my washing machine integration showing the current program, remaining time
  and a start button."
- "This card looks wrong in the dark theme. Update it to use Home Assistant's design tokens."
- "Add a visual editor so users can select a device instead of entering entity IDs."
- "Review my custom card and identify what prevents it from feeling native."
- "My card reports 'Custom element doesn't exist' when the integration loads it. Find the cause."

## Install

### Claude Code, as a plugin

```
/plugin marketplace add fabioscarparo/home-assistant-lovelace-taste-skill
/plugin install ha-lovelace-taste@ha-lovelace-taste
```

Update the marketplace to fetch new versions:

```
/plugin marketplace update ha-lovelace-taste
```

### Claude Code, manually

Copy the skill directory into your personal skills folder, or into `.claude/skills/` within a
project:

```bash
git clone https://github.com/fabioscarparo/home-assistant-lovelace-taste-skill.git
cp -R home-assistant-lovelace-taste-skill/skills/ha-lovelace-taste ~/.claude/skills/
```

### Claude apps

Download `ha-lovelace-taste.zip` from the
[latest release](https://github.com/fabioscarparo/home-assistant-lovelace-taste-skill/releases/latest)
and upload it from the Skills section in Claude's settings.

### Other agents

Any coding agent that supports the Agent Skills format can use the
`skills/ha-lovelace-taste` directory. To install it with the
[skills CLI](https://github.com/vercel-labs/skills), run:

```bash
npx skills add fabioscarparo/home-assistant-lovelace-taste-skill
```

## Repository structure

```
skills/ha-lovelace-taste/
  SKILL.md               workflow, design rules, behavior, languages, layout, shipping
  references/
    design-tokens.md     tokens with light and dark defaults, CSS for native patterns
    card-api.md          card and editor APIs, registration, hass helpers, events, distribution
    verification.md      mock hass demo, real-data scenarios, regression checks, images, checklist
```

The agent starts with `SKILL.md` whenever a task involves a Home Assistant card. It then
consults the relevant reference file as the work moves into design, implementation or
verification. This keeps the main workflow focused while preserving the implementation details
when they are needed.

## Keeping up with Home Assistant

Token values and component details are derived from the Home Assistant frontend source
(`dev` branch, October 2026). The frontend evolves continuously, so values and component
behavior may change between releases. When exact parity matters, verify the current source
before relying on a token or implementation detail.

Found a discrepancy or a new native pattern? Open an issue or pull request and include a link
to the relevant frontend source file.

## Disclaimer

This is an independent project and is not affiliated with, endorsed by or sponsored by Home
Assistant or the Open Home Foundation. Home Assistant is a trademark of the Open Home Foundation.

## License

[MIT](LICENSE)
