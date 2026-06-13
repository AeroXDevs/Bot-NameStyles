# Effects

The effect controls how the color(s) are rendered on the name. Pass the effect's ID as `display_name_effect_id`.

---

## Effect Reference

| ID | Name | What It Does | Colors Needed |
|----|------|-------------|---------------|
| `1` | Solid | Flat single color fill | 1 |
| `2` | Gradient | Smooth blend between two colors, left to right | **2 required** |
| `3` | Neon | Glowing outline around the letters | 1 |
| `4` | Toon | Subtle gradient fill with a visible stroke/outline | 1 |
| `5` | Pop | Colored drop shadow behind the letters | 1 |
| `6` | Glow | Alternate soft outer glow (different from Neon) | 1 (2nd is accent) |

---

## Gradient — Two Colors Required

Effect `2` (Gradient) requires exactly **two colors** in `display_name_colors`. The first is the left/start color, the second is the right/end color.

Font `7` (Neo-Castel), effect `2` (Gradient), blue `5865` blending to white `16777215`:

```js
await rest.patch(`/guilds/${guildId}/members/@me`, {
  body: {
    display_name_font_id: 7,
    display_name_effect_id: 2,
    display_name_colors: [5865, 16777215],
  },
});
```

---

## Solid — Single Color

Font `12` (Zilla Slab), effect `1` (Solid), blue `5865`:

```js
await rest.patch(`/guilds/${guildId}/members/@me`, {
  body: {
    display_name_font_id: 12,
    display_name_effect_id: 1,
    display_name_colors: [5865],
  },
});
```

---

## Neon — Glowing Outline

Font `10` (Sinistre), effect `3` (Neon), white `16777215`:

```js
await rest.patch(`/guilds/${guildId}/members/@me`, {
  body: {
    display_name_font_id: 10,
    display_name_effect_id: 3,
    display_name_colors: [16777215],
  },
});
```

---

## Toon — Stroke Outline

Font `3` (Cherry Bomb), effect `4` (Toon), white `16777215`:

```js
await rest.patch(`/guilds/${guildId}/members/@me`, {
  body: {
    display_name_font_id: 3,
    display_name_effect_id: 4,
    display_name_colors: [16777215],
  },
});
```

---

## Pop — Drop Shadow

Font `8` (Pixelify Sans), effect `5` (Pop), purple `8388736`:

```js
await rest.patch(`/guilds/${guildId}/members/@me`, {
  body: {
    display_name_font_id: 8,
    display_name_effect_id: 5,
    display_name_colors: [8388736],
  },
});
```

---

## Glow — Soft Outer Glow

Font `1` (Bangers), effect `6` (Glow), pink `16711935` with purple `8388736` accent:

```js
await rest.patch(`/guilds/${guildId}/members/@me`, {
  body: {
    display_name_font_id: 1,
    display_name_effect_id: 6,
    display_name_colors: [16711935, 8388736],
  },
});
```

---

## Notes

- Effect IDs are integers `1`–`6` — anything outside this range returns `400 Bad Request`
- Gradient (`2`) **requires** two colors — passing only one will result in an error or fallback
- All other effects work with one color; passing a second is either ignored or used as an accent (Glow)
