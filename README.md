# htf-contentfeed

Public content files for the [How to Fish](https://store.steampowered.com/) mods by `0cahlo`. Everything here is
fetched at runtime over plain HTTPS from `raw.githubusercontent.com`, so a file can be updated without shipping a new
mod build. Nothing here is code, and nothing here is executed — the mods read these as data only.

This repository is public on purpose: the mods fetch anonymously, so a private repo would 404.

## Files

| File | Used by | What it is |
| --- | --- | --- |
| `buriedtreasures-messages.json` | BuriedTreasures (`Memos.MessagesUrl`) | The pool of short notes a Memo can carry. |
| `buriedtreasures.config` | BuriedTreasures (`CloudSettings`, fixed URL) | Setting values forced on every player. |

## `buriedtreasures.config`

```json
{
  "version": 1,
  "settings": {
    "TreasureChests.Reward": 2500,
    "Bottle.SpawnEnabled": true,
    "Memos.PlayerMessagesEnabled": false
  }
}
```

- A key is where the setting is declared in the mod's `Settings.cs`, `Class.Field` -- e.g. `Bottle.SpawnEnabled`, not
  the .cfg's `[Bottle Spawn] Enabled`. That name exists in every build, including Release builds that don't put the
  setting in the .cfg at all.
- A value is text, a number or `true`/`false`, written as the .cfg would write it; it's clamped to the setting's range.
  A value that doesn't parse is logged and ignored.
- **A listed value beats every player's own .cfg** (and the host's, which is what clients get). Only list what you
  mean to force on everyone; leave a key out to let players set it.
- `MessagesApi.SigningSecret` is the player-message service's signing secret; absent keeps player-written notes off.
  Anything here is public.
- The mod caches the last good download and loads it before anything reads a setting; a fresh download replaces it a
  few seconds into startup.

## `buriedtreasures-messages.json`

```json
{
  "version": 1,
  "messages": [
    "no thoughts, head empty",
    "%PLAYER_NAME% owes me $20 and thinks I forgot. I did not forget."
  ]
}
```

- `%PLAYER_NAME%` is replaced with a random in-game player's Steam name (each occurrence in one note gets the same name).
- Keep each note to a couple of short lines — anything longer is truncated to what the paper holds.
- This list **replaces** the pool baked into the mod, so removing a line here retires it.
- Only the host downloads it in practice: the host picks every note and sends the finished text to the lobby.
- The mod caches the last good download, so editing a note here reaches players on their next launch (a start with no
  network keeps the previous one).

Keep it valid JSON — a malformed file is ignored and the mod falls back to its cache, then to its built-in pool.
