# OnChat

An API framework for chat stream functionality, moderation, and personalization.

This project originally developed inside FilterPlus before being split into its own repository.

___

**Objective**

- Central event handler for `core.register_on_chat_message()`
- Provide a minimal set of built-in tools
- Maintain modular drop-in support

### Roadmap

- Refactor `init.lua` into the core chat event pipeline.
- Load chat processors as modular handlers instead of hard-coded logic.
- Expose a public API for external mods to register into the event chain.
- Preserve deterministic execution order between registered handlers.

**Included**

- Green-text message highlighted for players mentioned by name
- Override `/msg` to include features of this mod
- Support for nametag flair from external mods
- Exposed API for external mods
- Commands for players
- Commands for server staff

___

## Green-text Highlghting

Messages containing another player's name will be sent to the mentioned player as green text. If many players are mentioned, they will each receive green text.

___

**Override /msg**

Overrides the built-in `/msg` command to include:

  - Blocking messages from specified players
  - Shows the sender their own message after it is sent
  - Distinguish between outgoing and incoming messages

## Nametag Flair

Nametag default format is: `«PlayerName»`. Built-in support from mods if available: Ranks, Factions, Exp.

Ensure the `minetest.conf` has the setting enabled, and that the external mod exists in the global namespace:

```conf
# minetest.conf
filterplus_ranks = true
filterplus_factions = true
filterplus_exp = true
```

```lua
-- nametag_flair.lua
local factions_available = core.settings:get_bool("filterplus_factions") and
    core.global_exists("factions") == true
```

The external mod must provide at least the string for the nametag flair. If a color is also provided, make sure it is the second value returned.

For example, the Factions mod would have a callback defined within itself. FilterPlus will attempt to call the external mod, expecting to receive a hex color and a text string:

```lua
-- ensure compatibility with your mod
local get_player_faction = factions_available and factions.is_player_in or function() return nil end

-- expects a string, and hexadecimal color
local faction_name, faction_color = get_player_faction(name)

-- the resulting flair will appear as colored string in brackets, eg: [Faction]
local faction_tag = faction_name and "[" .. colorize((faction_color or "#FFFFFF"), faction_name) .. "]" or ""
```

The default ordering of flair tags is: `{Rank}[Faction](Exp)«PlayerName» `

___

## Chat Commands

**Blocking**

- Ignore all public and private messages from being delivered.

- Persistent between logins until unblocked or server shutdown.

`/block` or `/unblock`: Without parameter will show your list of blocked players.
  - Players with `staff` privilege are not able to block or be blocked. Using the block command with player name will list the player's block list.

`/block player_name` or `/unblock player_name`: Add or remove another player from your blocked-players list.

For server moderators, forcing a block is enabled until both players use /unblock.

`/forceblock player_name1 player_name2`: Add two players to each other's block list. Requires `mute` privilege.

`/forceunblock player_name1 player_name2`: Removes the given player names from each other's block list.

**Muting**

- Timeout player from using public chat.

- Timer stops when the player or alt leaves game, and resumes on their next login.

- Retroactively applied to alt-accounts using a 24-hour cache.

`/mute player_name minutes`: Defaults to 2 minutes if no parameter is provided, and 120 minutes maximum

`/unmute player_name`: To allow using public chat before timeout expires.


**Toggle Chat**

`/chat`

- Alternative to the "hide chat" feature built into Luanti client, and provides the ability for clients without.

- Toggle receiving messages sent to the public chat channel.

- Direct messages and other command-based messages are not affected.

- System notifications are still visible.

___

## Global Functions

Most of the functions are fitted to return with a boolean value, and may be added to the global table for access to external mods. These however would have to be added manually by editing the init.lua file until a better solution is implemented.

___

Copyright © 2026 monk https://github.com/monk-afk/
