# Fonts

Each font has a numeric ID you pass as `display_name_font_id`. There are 12 fonts available.

Click **Preview** to see what the font looks like on Google Fonts. Fonts marked *Discord custom* are proprietary to Discord and not available externally.

---

## Font Reference

| ID | Name | Label | Style | Preview |
|----|------|-------|-------|---------|
| `1` | Bangers | Bold Comic | Thick, comic-book lettering | [Preview](https://fonts.google.com/specimen/Bangers) |
| `2` | BioRhyme | Elegant Serif | Classic serif, refined look | [Preview](https://fonts.google.com/specimen/BioRhyme) |
| `3` | Cherry Bomb | Sakura | Playful bubble-style characters | [Preview](https://fonts.google.com/specimen/Cherry+Bomb+One) |
| `4` | Chicle | Jellybean | Rounded, soft and bubbly | [Preview](https://fonts.google.com/specimen/Chicle) |
| `5` | Compagnon | Display | Stylish mixed-weight display font | [Preview](https://velvetyne.fr/fonts/compagnon) |
| `6` | MuseoModerno | Modern | Clean, geometric, modern feel | [Preview](https://fonts.google.com/specimen/MuseoModerno) |
| `7` | Neo-Castel | Medieval | Gothic, dark medieval lettering | *Discord custom* |
| `8` | Pixelify Sans | 8Bit | Retro pixel/blocky characters | [Preview](https://fonts.google.com/specimen/Pixelify+Sans) |
| `9` | Ribes | Decorative | Expressive, decorative display | *Discord custom* |
| `10` | Sinistre | Vampyre | Dark, jagged, gothic elegant | *Discord custom* |
| `11` | Default (GG Sans) | Default | Standard Discord font — no change | *Discord custom* |
| `12` | Zilla Slab | Tempo | Modern slab-serif, balanced weight | [Preview](https://fonts.google.com/specimen/Zilla+Slab) |

---

## Usage

Sets font `10` (Sinistre), effect `1` (Solid), color white:

```js
await rest.patch(`/guilds/${guildId}/members/@me`, {
  body: {
    display_name_font_id: 10,
    display_name_effect_id: 1,
    display_name_colors: [16777215],
  },
});
```

---

## Pairing Suggestions

| Font ID | Pairs Well With Effect |
|---------|----------------------|
| `10` — Sinistre | `3` Neon — dramatic glow on dark font |
| `9` — Ribes | `3` Neon or `6` Glow — decorative with light |
| `7` — Neo-Castel | `2` Gradient — medieval with color blend |
| `8` — Pixelify Sans | `5` Pop — retro with colored shadow |
| `3` — Cherry Bomb | `4` Toon — soft font with stroke outline |
| `1` — Bangers | `6` Glow — bold comic with color glow |
| `12` — Zilla Slab | `1` Solid — clean and sharp |

---

## Notes

- Font ID `11` (Default / GG Sans) is Discord's standard font — passing it applies no visual font change, but the effect and colors still apply
- Font IDs are integers `1`–`12` — passing `0` or any out-of-range value returns a `400 Bad Request`
- There is no ID `13` or beyond — the list is fixed
