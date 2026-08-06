# Limbonia — Mod Instructions

Reference for making Limbonia mods for Limbus Company.

A mod is a folder with a `mod.json` in it. That one file declares everything the mod
does — the art, the names, the voices, the sounds, the animations, the effects, the
status effects. A mod can also carry a Lua script, which is how it *behaves* rather
than how it looks.

These documents explain every field and every moment, in plain terms, with the traps
called out where they exist.

---

## The documents

| | |
|---|---|
| **[mods info.md](mods%20info.md)** | Every `mod.json` field. Who the mod replaces, art, names, skills, voices, sound, animations and their gameplay beats, camera, movement, visual effects, custom status effects. Start here. |

Each one follows the same shape: a heading per block, a real example you can copy, a
table of every field with its type, and a paragraph wherever a field does something
you would not guess from its name.

---

## Which one you need

**Everything a mod shows** — a drawing, a name, a portrait, a voice line, a sound on a
swing, an effect on a hit, a status effect and its icon — is declared in `mod.json`
and nothing else. You do not need a script for any of it.

**A script is for things the game has to ask about.** It runs at moments during a
fight — a blow about to land, a unit about to die, a turn ending — and it can change
a number, refuse something, add to a total, or put a line on screen. If you find
yourself wanting "when X happens, do Y", that is a script. If you want "this looks
like Z", that is `mod.json`.

---

## Two things worth knowing before you read either

**A mod replaces a character the game already has.** There is no way to add a
thirteenth sinner. You pick someone in the game, and your work plays instead of
theirs.

**One wrong value never breaks the game.** Every part of `mod.json` is read
defensively: a misspelled key, a number where text belongs, a file that is not there —
each one is reported against your mod by name and skipped, and the rest of the mod
loads. When something does not appear, the mod's own list of problems is the first
place to look, not the last.

---

## The script function list is not in here, on purpose

`scripts info.md` explains the moments, the payloads and the rules, but it does **not**
list the callable functions and their arguments. That list is produced by the game
itself, so it is never out of date and never disagrees with the build you are running.

Get it from the companion's **Scripts** panel, which reads it live and offers the same
text as autocomplete while you type. A written copy would drift, and a drifted API
reference is worse than none.

---

<!-- Adding a document: put the .md file in this folder and add one row to the table
     above. Keep the format the existing two use -- heading, example, field table,
     prose only where a field genuinely traps someone. Filenames with spaces need
     %20 in the link. -->
