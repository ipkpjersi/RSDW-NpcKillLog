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
- **Controller:** open the pause menu, move to **NPC KILL LOG** and press A. The kill log opens
  inside the pause menu, to the right of its list. Press Right to move into it, A to press
  **PREV PAGE** / **NEXT PAGE** (scroll), the two message switches or **CLOSE**, Left to go back
  to the pause menu, and B to close everything. There is no search box with a controller.
- **BOSS KILL MESSAGES** and **NPC KILL MESSAGES** in the window turn the kill messages on or off
  for bosses and for everything else. Kills are always counted.
- Settings are in `config.txt` (restart the game after editing). Name corrections go in
  `names.txt`.

Counts are saved to `%LOCALAPPDATA%\RSDragonwilds\Saved\NpcKillLog-<character>-<id>.txt`, one
`Name: count` line per NPC, readable and editable by hand while the game is closed.

## How it works

- **Kills** come from the Blueprint event `BP_DominionAICharacter_C:BP_OnDeath`, which runs once
  for every creature that dies. The hook is registered once a world is loaded (retried every 5 s
  until it works). A kill only counts when your character is in the creature's damage list
  (`ThreatSystem.DamageInstigators` on its AI controller), so other players' kills are left out;
  `OnlyMyKills = false` in `config.txt` counts every death instead. The first version's hooks,
  `AiAttackTicketsManager:OnAiDeath` and `AIAudioManagerComponent:HandleDeath`, are still in
  `main.lua`, commented out. `ProgressComponent:OnAIKilled` looks like the right event but is
  called straight from C++, so a Lua hook on it never runs.
- **Names** come from the game's own `ST_AI_Names` string tables (`Scripts/npc_names.lua`),
  matched by the creature's AI data row name or its class name. `names.txt` overrides any name.
- **Bosses** are worked out from the game's own setup: a boss health bar component, the
  miniboss base classes, or "Boss" / "Colossal" in the class name; dragon hatchlings are left
  out. `ExtraBosses` / `NotBosses` in `config.txt` fix any it gets wrong.
- **Window** is the game's own framed panel (`WBP_Panel`) opened from an entry added to the pause
  menu. Opening it resumes the game first, so the window is not hidden behind the pause menu.
- **Controller:** a pad's A presses the game's buttons from C++, so the click hook the mouse uses
  never sees it, and a window on top of the game never gets pad focus. So the pause menu entry is
  made selectable (A selects it, and a check four times a second notices), and the controller
  version of the panel is docked inside the pause menu with D-pad links into it and back. Mod Menu
  and Storage Search dock their controller panels in the same spot; the `PauseDock.owner` /
  `PauseDock.rev` shared variables make sure only one is open at a time. With many pause menu
  entries the game's list box is too short for the D-pad to reach them all, so it is grown to fit.

## Known limits

- A creature that only other players (or nothing, such as a fall) damaged is not counted. One you
  helped kill counts for you even if someone else landed the last hit.
- Co-op is untested, and the damage list may only be filled in on the host.
