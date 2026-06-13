# Real-World Examples

Complete, copy-paste patterns for common name style use cases.

---

## 1. Apply on Bot Ready — Single Guild

Set a name style once when the bot starts up in a specific server.

Font `10` (Sinistre), effect `3` (Neon), white `16777215`:

```js
import { REST } from '@discordjs/rest';
import { Client, GatewayIntentBits } from 'discord.js';

const client = new Client({ intents: [GatewayIntentBits.Guilds] });
const rest   = new REST({ version: '10' }).setToken(process.env.TOKEN);

const GUILD_ID = '123456789012345678';

client.once('ready', async () => {
  try {
    await rest.patch(`/guilds/${GUILD_ID}/members/@me`, {
      body: {
        display_name_font_id: 10,
        display_name_effect_id: 3,
        display_name_colors: [16777215],
      },
    });
    console.log('Name style applied.');
  } catch (err) {
    console.error('Name style failed:', err?.rawError?.message ?? err.message);
  }
});

client.login(process.env.TOKEN);
```

---

## 2. Apply on Bot Ready — All Guilds

Apply the same style across every server the bot is in. Uses `setImmediate` so it doesn't block the ready event. Adds a 1.5 second delay between calls to avoid rate limiting.

Font `9` (Ribes), effect `3` (Neon), pink `16711935`:

```js
client.once('ready', () => {
  setImmediate(async () => {
    const guildIds = client.guilds.cache.map(g => g.id);

    for (const guildId of guildIds) {
      try {
        await rest.patch(`/guilds/${guildId}/members/@me`, {
          body: {
            display_name_font_id: 9,
            display_name_effect_id: 3,
            display_name_colors: [16711935],
          },
        });
      } catch (err) {
        console.warn(`Skipped ${guildId}:`, err?.rawError?.message ?? err.message);
      }

      await new Promise(r => setTimeout(r, 1500));
    }
  });
});
```

> **Note:** For large bots (hundreds of servers), apply in batches or only on `guildCreate` rather than startup to avoid rate limiting.

---

## 3. Apply on Guild Join

Set the style automatically whenever the bot joins a new server.

Font `7` (Neo-Castel), effect `2` (Gradient), blue `5865` to white `16777215`:

```js
client.on('guildCreate', async (guild) => {
  try {
    await rest.patch(`/guilds/${guild.id}/members/@me`, {
      body: {
        display_name_font_id: 7,
        display_name_effect_id: 2,
        display_name_colors: [5865, 16777215],
      },
    });
  } catch {}
});
```

---

## 4. Slash Command — Let Admins Set the Style

A `/namestyle` command that lets server admins pick font, effect, and color:

```js
import { SlashCommandBuilder, PermissionFlagsBits, MessageFlags } from 'discord.js';
import { REST } from '@discordjs/rest';

export default {
  data: new SlashCommandBuilder()
    .setName('namestyle')
    .setDescription("Set the bot's name style in this server")
    .setDefaultMemberPermissions(PermissionFlagsBits.ManageGuild)
    .addIntegerOption(o =>
      o.setName('font').setDescription('Font ID (1–12)').setRequired(true).setMinValue(1).setMaxValue(12)
    )
    .addIntegerOption(o =>
      o.setName('effect').setDescription('Effect ID (1–6)').setRequired(true).setMinValue(1).setMaxValue(6)
    )
    .addStringOption(o =>
      o.setName('color1').setDescription('Primary color hex (e.g. #ffffff)').setRequired(true)
    )
    .addStringOption(o =>
      o.setName('color2').setDescription('Secondary color hex (optional, for gradient/glow)').setRequired(false)
    ),

  async execute(interaction) {
    const fontId   = interaction.options.getInteger('font');
    const effectId = interaction.options.getInteger('effect');
    const hex1     = interaction.options.getString('color1');
    const hex2     = interaction.options.getString('color2');

    if (!/^#?[0-9a-fA-F]{6}$/.test(hex1)) {
      return interaction.reply({ content: `Invalid color: \`${hex1}\``, flags: MessageFlags.Ephemeral });
    }
    if (hex2 && !/^#?[0-9a-fA-F]{6}$/.test(hex2)) {
      return interaction.reply({ content: `Invalid secondary color: \`${hex2}\``, flags: MessageFlags.Ephemeral });
    }
    if (effectId === 2 && !hex2) {
      return interaction.reply({ content: 'Gradient effect requires a secondary color.', flags: MessageFlags.Ephemeral });
    }

    const colors = [parseInt(hex1.replace('#', ''), 16)];
    if (hex2) colors.push(parseInt(hex2.replace('#', ''), 16));

    const rest = new REST({ version: '10' }).setToken(interaction.client.token);

    try {
      await rest.patch(`/guilds/${interaction.guildId}/members/@me`, {
        body: {
          display_name_font_id: fontId,
          display_name_effect_id: effectId,
          display_name_colors: colors,
        },
      });
      await interaction.reply({ content: 'Name style updated!', flags: MessageFlags.Ephemeral });
    } catch (err) {
      const reason = err?.rawError?.message ?? err?.message ?? 'Unknown error';
      await interaction.reply({ content: `Failed: ${reason}`, flags: MessageFlags.Ephemeral });
    }
  },
};
```

---

## 5. Reset Command

```js
export default {
  data: new SlashCommandBuilder()
    .setName('resetnamestyle')
    .setDescription("Reset the bot's name style to default")
    .setDefaultMemberPermissions(PermissionFlagsBits.ManageGuild),

  async execute(interaction) {
    const rest = new REST({ version: '10' }).setToken(interaction.client.token);

    try {
      await rest.patch(`/guilds/${interaction.guildId}/members/@me`, {
        body: {
          display_name_font_id: null,
          display_name_effect_id: null,
          display_name_colors: null,
        },
      });
      await interaction.reply({ content: 'Name style reset to default.', flags: MessageFlags.Ephemeral });
    } catch (err) {
      const reason = err?.rawError?.message ?? err?.message ?? 'Unknown error';
      await interaction.reply({ content: `Failed: ${reason}`, flags: MessageFlags.Ephemeral });
    }
  },
};
```

---

## 6. Store Style in Database and Re-apply on Restart

Save the chosen style per server on apply, then re-patch every stored style on the `ready` event so it survives restarts:

```js
await db.guild.setNameStyle(guildId, { fontId, effectId, colors });

client.once('ready', () => {
  setImmediate(async () => {
    const styles = await db.guild.getAllNameStyles();

    for (const { guildId, fontId, effectId, colors } of styles) {
      try {
        await rest.patch(`/guilds/${guildId}/members/@me`, {
          body: {
            display_name_font_id: fontId,
            display_name_effect_id: effectId,
            display_name_colors: colors,
          },
        });
      } catch {}
      await new Promise(r => setTimeout(r, 1500));
    }
  });
});
```

---

## Preset Styles — Ready to Use

```js
const PRESETS = {
  'sinistre-neon-white':      { font_id: 10, effect_id: 3, colors: [16777215] },
  'ribes-neon-pink':          { font_id: 9,  effect_id: 3, colors: [16711935] },
  'neo-castel-gradient':      { font_id: 7,  effect_id: 2, colors: [5865, 16777215] },
  'pixelify-pop-purple':      { font_id: 8,  effect_id: 5, colors: [8388736] },
  'bangers-glow-pink-purple': { font_id: 1,  effect_id: 6, colors: [16711935, 8388736] },
  'cherry-toon-white':        { font_id: 3,  effect_id: 4, colors: [16777215] },
  'zilla-solid-blue':         { font_id: 12, effect_id: 1, colors: [5865] },
};

/**
 * Applies a named preset style to the bot's member record in a guild.
 * @param {REST} rest
 * @param {string} guildId
 * @param {string} presetKey - Key from the PRESETS object
 */
async function applyPreset(rest, guildId, presetKey) {
  const style = PRESETS[presetKey];
  if (!style) throw new Error(`Unknown preset: ${presetKey}`);

  await rest.patch(`/guilds/${guildId}/members/@me`, {
    body: {
      display_name_font_id: style.font_id,
      display_name_effect_id: style.effect_id,
      display_name_colors: style.colors,
    },
  });
}

await applyPreset(rest, guildId, 'sinistre-neon-white');
```
