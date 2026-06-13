# Colors

Colors in name styles are passed as **24-bit integers** — not hex strings. The `display_name_colors` field takes an array of 1 or 2 numbers.

---

## Hex to Integer Conversion

Discord colors are standard RGB hex parsed as a decimal integer.

`hexToInt` strips the leading `#` and parses the remaining 6 hex digits as base-16:

```js
/**
 * Converts a hex color string to a 24-bit integer.
 * @param {string} hex - e.g. "#ffffff" or "ffffff"
 * @returns {number} Integer in range 0–16777215
 */
function hexToInt(hex) {
  return parseInt(hex.replace(/^#/, ''), 16);
}
```

```
hexToInt('#ffffff')  →  16777215
hexToInt('#5865F2')  →  5792242
hexToInt('#000000')  →  0
```

`intToHex` is the reverse — useful for displaying stored values back to users:

```js
/**
 * Converts a 24-bit integer back to a hex color string.
 * @param {number} int - Integer in range 0–16777215
 * @returns {string} Uppercase hex string e.g. "#FFFFFF"
 */
function intToHex(int) {
  return '#' + int.toString(16).toUpperCase().padStart(6, '0');
}
```

```
intToHex(16777215)  →  #FFFFFF
intToHex(5865)      →  #0016E9
```

---

## Common Color Reference

| Name | Hex | Integer |
|------|-----|---------|
| White | `#FFFFFF` | `16777215` |
| Black | `#000000` | `0` |
| Discord Blurple | `#5865F2` | `5792242` |
| Discord Green | `#57F287` | `5701255` |
| Discord Red | `#ED4245` | `15548997` |
| Discord Yellow | `#FEE75C` | `16646492` |
| Pink | `#FF00FF` | `16711935` |
| Purple | `#800080` | `8388736` |
| Blue (Discord Presence) | `#0016E9` | `5865` |
| Gold | `#FFD700` | `16766720` |
| Cyan | `#00FFFF` | `65535` |

---

## One Color vs Two Colors

Most effects only need one color:

```js
display_name_colors: [16777215]
```

Gradient (`effect_id: 2`) requires two — the blend goes left → right:

```js
display_name_colors: [16711935, 8388736]
```

Glow (`effect_id: 6`) accepts a second color as an accent:

```js
display_name_colors: [16711935, 8388736]
```

---

## Validating Colors

Colors must be in the range `0` to `16777215` (`0x000000` to `0xFFFFFF`):

```js
/**
 * Returns true if the value is a valid 24-bit color integer.
 * @param {number} value
 * @returns {boolean}
 */
function isValidColor(value) {
  return Number.isInteger(value) && value >= 0 && value <= 0xFFFFFF;
}

/**
 * Returns true if the string is a valid 6-digit hex color.
 * Accepts with or without leading #.
 * @param {string} hex
 * @returns {boolean}
 */
function isValidHex(hex) {
  return /^#?[0-9a-fA-F]{6}$/.test(hex.trim());
}
```

---

## Full Example — Hex Input to API Call

Takes two hex strings, converts them to integers, and applies a Glow style:

```js
/**
 * Converts a hex color string to a 24-bit integer.
 * @param {string} hex
 * @returns {number}
 */
function hexToInt(hex) {
  return parseInt(hex.replace(/^#/, ''), 16);
}

const color1 = hexToInt('#FF00FF');
const color2 = hexToInt('#800080');

await rest.patch(`/guilds/${guildId}/members/@me`, {
  body: {
    display_name_font_id: 1,
    display_name_effect_id: 6,
    display_name_colors: [color1, color2],
  },
});
```

---

## Notes

- Colors are `number[]` — not strings, not hex, not RGB objects
- Minimum 1 color, maximum 2 colors in the array
- Passing an empty array or more than 2 returns `400 Bad Request`
- `display_name_colors: null` clears the color when resetting the style
