# Tips & Gotchas

Common mistakes, limits, and things to know before you ship.

---

## Only the Guild-Scoped Endpoint Works for Bots

```
✅ PATCH /guilds/{guild_id}/members/@me
❌ PATCH /users/@me  — returns 403 for bot tokens
```

Name styles for bots are **guild-scoped only**. The global user endpoint does not accept style fields from bot tokens. Every style is per-server.

---

## Colors Are Integers, Not Hex Strings

**Wrong:**
```js
display_name_colors: ['#ffffff']
```

**Right:**
```js
display_name_colors: [16777215]
```

Convert with:
```js
parseInt('ffffff', 16)
```

---

## Gradient Requires Two Colors

Effect `2` (Gradient) **must** have two colors in the array or Discord returns `400 Bad Request`.

**Wrong:**
```js
{ display_name_effect_id: 2, display_name_colors: [16777215] }
```

**Right:**
```js
{ display_name_effect_id: 2, display_name_colors: [16777215, 5865] }
```

---

## IDs Are Out-of-Range = 400 Bad Request

Font IDs must be `1`–`12`. Effect IDs must be `1`–`6`. Passing `0`, `-1`, or any value outside these ranges returns `400 Bad Request` immediately.

**Wrong:**
```js
display_name_font_id: 0
display_name_font_id: 13
display_name_effect_id: 7
```

---

## Reset Uses null, Not 0

To clear a name style, pass `null` for all three fields — not `0` or an empty array.

**Wrong:**
```js
{ display_name_font_id: 0, display_name_colors: [] }
```

**Right:**
```js
{ display_name_font_id: null, display_name_effect_id: null, display_name_colors: null }
```

---

## Change Nickname Permission Is Required

The bot needs `Change Nickname` (`PermissionFlagsBits.ChangeNickname`) in the target server. Without it you get `403 Forbidden`:

```js
const me = guild.members.me;
if (!me.permissions.has(PermissionFlagsBits.ChangeNickname)) {
  return;
}
```

---

## Rate Limiting — Space Out Calls

If applying styles to multiple servers, add a delay between each call. Hammering all at once on a large bot will trigger Discord's rate limiter:

```js
for (const guildId of guildIds) {
  try {
    await rest.patch(`/guilds/${guildId}/members/@me`, { body: style });
  } catch {}
  await new Promise(r => setTimeout(r, 1500));
}
```

---

## Styles Don't Persist Automatically

Discord **does not re-apply** the style when the bot restarts or rejoins. If you want the style to survive restarts, save it to your database on apply and re-patch on the `ready` event.

---

## No GET Endpoint for Current Style

There is no official endpoint that returns the bot's current active name style for a guild. Track it yourself in your database if you need to display the current values to users.

---

## Quick Limits Reference

| Limit | Value |
|-------|-------|
| Font IDs | `1` – `12` |
| Effect IDs | `1` – `6` |
| Colors per call | `1` or `2` |
| Color value range | `0` – `16777215` |
| Colors for Gradient | **2 required** |
| Endpoint | `PATCH /guilds/{id}/members/@me` only |
