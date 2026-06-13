# Fonts

Each font has a numeric ID you pass as `display_name_font_id`. There are 12 fonts available.

---

## Font Previews

| ID | Preview |
|----|---------|
| `1` — Bangers | ![Bangers](assets/font-1-bangers.png) |
| `2` — BioRhyme | ![BioRhyme](assets/font-2-biorhyme.png) |
| `3` — Cherry Bomb | ![Cherry Bomb](assets/font-3-cherrybomb.png) |
| `4` — Chicle | ![Chicle](assets/font-4-chicle.png) |
| `5` — Compagnon | ![Compagnon](assets/font-5-compagnon.png) |
| `6` — MuseoModerno | ![MuseoModerno](assets/font-6-museomoderno.png) |
| `7` — Neo-Castel | ![Neo-Castel](assets/font-7-neocastel.png) |
| `8` — Pixelify Sans | ![Pixelify Sans](assets/font-8-pixelifysans.png) |
| `9` — Ribes | ![Ribes](assets/font-9-ribes.png) |
| `10` — Sinistre | ![Sinistre](assets/font-10-sinistre.png) |
| `11` — Default (GG Sans) | ![GG Sans](assets/font-11-ggsans.png) |
| `12` — Zilla Slab | ![Zilla Slab](assets/font-12-zillaslab.png) |

---

## Font Reference

| ID | Name | Label | Style |
|----|------|-------|-------|
| `1` | Bangers | Bold Comic | Thick, comic-book lettering |
| `2` | BioRhyme | Elegant Serif | Classic serif, refined look |
| `3` — Cherry Bomb | Sakura | Playful bubble-style characters |
| `4` | Chicle | Jellybean | Rounded, soft and bubbly |
| `5` | Compagnon | Display | Stylish mixed-weight display font |
| `6` | MuseoModerno | Modern | Clean, geometric, modern feel |
| `7` | Neo-Castel | Medieval | Gothic, dark medieval lettering |
| `8` | Pixelify Sans | 8Bit | Retro pixel/blocky characters |
| `9` | Ribes | Decorative | Expressive, decorative display |
| `10` | Sinistre | Vampyre | Dark, jagged, gothic elegant |
| `11` | Default (GG Sans) | Default | Standard Discord font — no change |
| `12` | Zilla Slab | Tempo | Modern slab-serif, balanced weight |

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
