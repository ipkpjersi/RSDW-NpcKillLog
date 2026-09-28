# NPC Kill Log - RuneScape: Dragonwilds

A UE4SS Lua mod that counts every NPC you kill, per character, and shows the counts in a
searchable window, OSRS kill count style:

```
Kebbit: 12
Giant Rat: 9
Garou King: 1
```

After each kill a message shows for a few seconds: `Your Kebbit kill count is now: 12.`

## Install

1. Install [UE4SS](https://github.com/UE4SS-RE/RE-UE4SS) for the game.
2. Copy this folder into `RSDragonwilds/Binaries/Win64/ue4ss/Mods/NpcKillLog`.
3. Add `NpcKillLog : 1` to `ue4ss/Mods/mods.txt`.

## Use

- Press Esc and pick **NPC KILL LOG** in the pause menu. Type in the box to search. PageUp /
  PageDown or the mouse wheel scroll. Esc or **CLOSE** closes it.
- **BOSS KILL MESSAGES** and **NPC KILL MESSAGES** in the window turn the kill messages on or off
  for bosses and for everything else. Kills are always counted.
- Settings are in `config.txt` (restart the game after editing). Name corrections go in
  `names.txt`.

Counts are saved to `%LOCALAPPDATA%\RSDragonwilds\Saved\NpcKillLog-<character>-<id>.txt`, one
`Name: count` line per NPC, readable and editable by hand while the game is closed.

## How it works

- **Kills** come from `AiAttackTicketsManager:OnAiDeath(ai)`, which the game runs once for every
  AI that dies, with the dying AI's `AIAudioManagerComponent:HandleDeath(ai)` as a backup; one
  death is never counted twice. `ProgressComponent:OnAIKilled` looks like the right event but is
  called straight from C++, so a Lua hook on it never runs.
- **Names** come from the game's own `ST_AI_Names` string tables (`Scripts/npc_names.lua`),
  matched by the creature's AI data row name or its class name. `names.txt` overrides any name.
- **Bosses** are worked out from the game's own setup: a boss health bar component, the
  miniboss base classes, or "Boss" / "Colossal" in the class name; dragon hatchlings are left
  out. `ExtraBosses` / `NotBosses` in `config.txt` fix any it gets wrong.
- **Window** is the game's own framed panel (`WBP_Panel`) opened from an entry added to the pause
  menu. Opening it resumes the game first, so the window is not hidden behind the pause menu.

## Known limits

- Neither death event says who made the kill, so every AI death is counted. That is exact in
  single player; in co-op, kills by other players count too.
- Co-op is untested.
