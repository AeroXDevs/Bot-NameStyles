# Discord.js Name Styles — Complete Guide

> Guide by **itsfizys** • Support Server: [discord.gg/aerox](https://discord.gg/aerox)

Name Styles let your bot display its server nickname with a custom **font**, **effect**, and **color(s)** — visible in the member list, chat, and profiles. They are scoped per-server, set via the Discord REST API, and require no special OAuth flow.

---

## How It Works

Name Styles are applied by patching the bot's own guild member record. You send three fields:

| Field | Type | What It Sets |
|-------|------|-------------|
| `display_name_font_id` | `number` | Which font to use (1–12) |
| `display_name_effect_id` | `number` | Which visual effect to apply (1–6) |
| `display_name_colors` | `number[]` | One or two colors as 24-bit integers |

The patch goes to **`/guilds/{guild_id}/members/@me`** — the bot's own member endpoint. This is guild-scoped: each server gets its own style independently.

---

## Quick Example

Applies font `10` (Sinistre), effect `3` (Neon), color white (`16777215` = `#ffffff`):

```js
import { REST } from '@discordjs/rest';

const rest = new REST({ version: '10' }).setToken(process.env.TOKEN);

await rest.patch(`/guilds/${guildId}/members/@me`, {
  body: {
    display_name_font_id: 10,
    display_name_effect_id: 3,
    display_name_colors: [16777215],
  },
});
```

---

## Resetting a Style

Pass `null` for all three fields to clear the name style:

```js
await rest.patch(`/guilds/${guildId}/members/@me`, {
  body: {
    display_name_font_id: null,
    display_name_effect_id: null,
    display_name_colors: null,
  },
});
```

---

## Table of Contents

| # | Guide | Description |
|---|-------|-------------|
| 1 | [Fonts](Fonts.md) | All 12 font IDs with previews and notes |
| 2 | [Effects](Effects.md) | All 6 effects — what they do and color requirements |
| 3 | [Colors](Colors.md) | How colors work — hex conversion, 1 vs 2 colors |
| 4 | [Implementation](Implementation.md) | Full API call breakdown and reset |
| 5 | [Real-World Examples](Real-World-Examples.md) | Copy-paste patterns |
| 6 | [Tips & Gotchas](Tips-And-Gotchas.md) | What works, what doesn't, common mistakes |

---

## Important Rules

- Name styles are **guild-scoped** — each server is independent
- Only use `PATCH /guilds/{guild_id}/members/@me` — global user endpoints do **not** work for bots
- Colors are **24-bit integers**, not hex strings — `#ffffff` → `16777215`
- Some effects require **two colors** (Gradient). Others work fine with one
- The bot needs the **Change Nickname** permission in the server
