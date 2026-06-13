# Implementation

Everything you need to apply, update, and reset name styles in a real bot.

---

## Setup

```js
import { REST } from '@discordjs/rest';

const rest = new REST({ version: '10' }).setToken(process.env.TOKEN);
```

Inside a slash command handler, use the client's token directly:

```js
const rest = new REST({ version: '10' }).setToken(client.token);
```

---

## Applying a Name Style

```js
/**
 * Applies a name style to the bot's member record in a specific guild.
 * @param {REST} rest - Configured @discordjs/rest instance
 * @param {string} guildId - Target guild ID
 * @param {number} fontId - Font ID (1–12)
 * @param {number} effectId - Effect ID (1–6)
 * @param {number[]} colors - Array of 1 or 2 24-bit color integers
 */
async function applyNameStyle(rest, guildId, fontId, effectId, colors) {
  await rest.patch(`/guilds/${guildId}/members/@me`, {
    body: {
      display_name_font_id: fontId,
      display_name_effect_id: effectId,
      display_name_colors: colors,
    },
  });
}

await applyNameStyle(rest, '123456789012345678', 10, 3, [16777215]);
```

---

## Resetting a Name Style

Pass `null` to all three fields to clear the style:

```js
/**
 * Clears the bot's name style in a specific guild, reverting to default.
 * @param {REST} rest - Configured @discordjs/rest instance
 * @param {string} guildId - Target guild ID
 */
async function resetNameStyle(rest, guildId) {
  await rest.patch(`/guilds/${guildId}/members/@me`, {
    body: {
      display_name_font_id: null,
      display_name_effect_id: null,
      display_name_colors: null,
    },
  });
}
```

---

## Hex to Integer Helper

Colors must be integers. Strip the `#` and parse as base-16:

```js
/**
 * Converts a hex color string to a 24-bit integer.
 * @param {string} hex - e.g. "#ffffff" or "ffffff"
 * @returns {number} Integer in range 0–16777215
 */
function hexToInt(hex) {
  return parseInt(hex.replace(/^#/, ''), 16);
}

const colors   = [hexToInt('#FF00FF')];
const gradient = [hexToInt('#FF00FF'), hexToInt('#800080')];
```

---

## Input Validation

Validate before calling the API to avoid unnecessary `400` errors:

```js
const FONT_IDS   = new Set([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]);
const EFFECT_IDS = new Set([1, 2, 3, 4, 5, 6]);

/**
 * Returns true if the value is a valid 6-digit hex color string.
 * Accepts with or without a leading #.
 * @param {string} hex
 * @returns {boolean}
 */
function isValidHex(hex) {
  return /^#?[0-9a-fA-F]{6}$/.test(hex.trim());
}

/**
 * Returns true if the value is a valid 24-bit color integer.
 * @param {number} n
 * @returns {boolean}
 */
function isValidColor(n) {
  return Number.isInteger(n) && n >= 0 && n <= 0xFFFFFF;
}

/**
 * Throws if any name style field is out of range or malformed.
 * @param {number} fontId
 * @param {number} effectId
 * @param {number[]} colors
 */
function validateNameStyle(fontId, effectId, colors) {
  if (!FONT_IDS.has(fontId))
    throw new Error(`Invalid font ID: ${fontId}`);
  if (!EFFECT_IDS.has(effectId))
    throw new Error(`Invalid effect ID: ${effectId}`);
  if (!Array.isArray(colors) || colors.length < 1 || colors.length > 2)
    throw new Error('colors must be an array of 1 or 2 integers');
  if (!colors.every(isValidColor))
    throw new Error('Each color must be an integer between 0 and 16777215');
  if (effectId === 2 && colors.length < 2)
    throw new Error('Gradient effect requires 2 colors');
}
```

---

## Complete Apply Function with Validation

Accepts hex strings for colors, converts and validates before patching:

```js
import { REST } from '@discordjs/rest';

const FONT_IDS   = new Set([1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]);
const EFFECT_IDS = new Set([1, 2, 3, 4, 5, 6]);

/**
 * @param {string} hex
 * @returns {number}
 */
function hexToInt(hex) {
  return parseInt(hex.replace(/^#/, ''), 16);
}

/**
 * @param {number} n
 * @returns {boolean}
 */
function isValidColor(n) {
  return Number.isInteger(n) && n >= 0 && n <= 0xFFFFFF;
}

/**
 * Validates and applies a name style to the bot's guild member record.
 * @param {REST} rest
 * @param {string} guildId
 * @param {{ fontId: number, effectId: number, hexColors: string[] }} options
 */
async function applyNameStyle(rest, guildId, { fontId, effectId, hexColors }) {
  if (!FONT_IDS.has(fontId))
    throw new Error(`Invalid font ID: ${fontId}`);
  if (!EFFECT_IDS.has(effectId))
    throw new Error(`Invalid effect ID: ${effectId}`);

  const colors = hexColors.map(hexToInt);

  if (colors.length < 1 || colors.length > 2)
    throw new Error('Provide 1 or 2 colors');
  if (!colors.every(isValidColor))
    throw new Error('Invalid color value');
  if (effectId === 2 && colors.length < 2)
    throw new Error('Gradient requires 2 colors');

  await rest.patch(`/guilds/${guildId}/members/@me`, {
    body: {
      display_name_font_id: fontId,
      display_name_effect_id: effectId,
      display_name_colors: colors,
    },
  });
}

const rest = new REST({ version: '10' }).setToken(process.env.TOKEN);

await applyNameStyle(rest, guildId, {
  fontId: 10,
  effectId: 3,
  hexColors: ['#ffffff'],
});
```

---

## Permissions Required

The bot must have **Change Nickname** permission in the target server:

```js
const me = guild.members.me;
if (!me.permissions.has(PermissionFlagsBits.ChangeNickname)) {
  return;
}
```

---

## Error Handling

Common status codes returned by Discord on this endpoint:

| Status | Cause |
|--------|-------|
| `400` | Invalid field value — bad font/effect ID or out-of-range color |
| `403` | Bot is missing Change Nickname permission in the guild |
| `429` | Rate limited — back off and retry |

```js
try {
  await rest.patch(`/guilds/${guildId}/members/@me`, {
    body: {
      display_name_font_id: fontId,
      display_name_effect_id: effectId,
      display_name_colors: colors,
    },
  });
} catch (err) {
  const reason = err?.rawError?.message ?? err?.message ?? 'Unknown error';
  console.error('Name style failed:', reason);
}
```

---

## Notes

- The endpoint is `PATCH /guilds/{guild_id}/members/@me` — **guild-scoped only**
- `PATCH /users/@me` does **not** support name style fields for bot tokens
- Each server stores its own style independently
- There is no GET endpoint that returns the current active style — track it in your database if needed
