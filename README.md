# Limbonia — Mod Instructions

Reference for making Limbonia mods for Limbus Company.

A mod is a folder with a `mod.json` in it. That one file declares everything the mod
does — the art, the names, the voices, the sounds, the animations, the effects, the
cut-ins, the status effects. A mod can also carry a Lua script, which is how it
*behaves* rather than how it looks.

---

## What is here

| | |
|---|---|
| **[mods info.md](mods%20info.md)** | Every `mod.json` field. Who the mod replaces, art, names, skills, voices, sound, animations and their gameplay beats, camera, movement, visual effects, cut-ins, forms, custom status effects. It is the whole reference — start there and stay there. |
| **[scripts info.md](scripts%20info.md)** | The guide to writing a mod's Lua script: the four ways to register a handler and what each one's answer means, which character a handler is about, coins, criticals, SP, status effects, and the traps that produce a script which loads cleanly and does nothing. Read it after `mods info.md`. |
| **[LimboniaModTemplate.zip](LimboniaModTemplate.zip)** | The Unity project that builds an animation bundle and writes `mod.json` for you. You need it only if your mod ships drawings, animations or effect prefabs of its own; its own README is inside. |

`mods info.md` follows one shape throughout: a heading per block, a real example you can
copy, a table of every field with its type, and a paragraph wherever a field does
something you would not guess from its name. Read the table first and the paragraphs
only when a field surprises you.

`scripts info.md` is prose and worked examples rather than a field table, because a
script is not a list of keys — the hard part is knowing what a handler is allowed to
answer and when it runs, and neither of those is something a table can say.

---

## mod.json, or a script

**Everything a mod shows** — a drawing, a name, a portrait, a voice line, a sound on a
swing, an effect on a hit, a full-screen cut-in, a status effect and its icon — is
declared in `mod.json` and nothing else. You do not need a script for any of it.

**A script is for things the game has to ask about.** It runs at moments during a
fight — a blow about to land, a unit about to die, a turn ending — and it can change
a number, refuse something, add to a total, or put a line on screen. If you find
yourself wanting "when X happens, do Y", that is a script. If you want "this looks
like Z", that is `mod.json`.

Once you have decided it is a script, [scripts info.md](scripts%20info.md) is the one
to read.

---

## Two things worth knowing before you start

**A mod replaces a character the game already has.** There is no way to add a
thirteenth sinner. You pick someone in the game, and your work plays instead of
theirs.

**One wrong value never breaks the game.** Every part of `mod.json` is read
defensively: a misspelled key, a number where text belongs, a file that is not there —
each one is reported against your mod by name and skipped, and the rest of the mod
loads. When something does not appear, the mod's own list of problems is the first
place to look, not the last.

---

## Whether this build runs scripts at all

Scripting is a build-time decision, so it is either compiled into the copy of Limbonia
you are running or it is not there at all. The way to tell is the companion: when
scripts are in, there is a **Scripts** panel; when they are not, there is no panel,
because the commands behind it do not exist either.

On a build without them, a mod that ships a `.lua` still loads — its art, names,
voices, sounds, animations and status effects all work exactly as they would — and the
mod's own list of problems carries one sentence saying the script will never be read.
Nothing about a script fails silently; it is simply absent. Check for the panel before
you write one.

---

## The script function list is not in here, on purpose

`scripts info.md` teaches scripting and shows real calls in worked examples, but
**nothing in this folder enumerates the callable script functions, their arguments, or
the moments they can run at.** Nobody writes that list by hand: the game prints its own,
so it describes a build that exists.

You meet it in two places. The script editor offers it as autocomplete while you type,
read live out of the game you have running. And the whole of it, written out and marked
up with which moments nothing in the game actually delivers, is published: **Open the
reference** on the companion's **Scripts** panel, or **Reference** in the script editor's
toolbar, opens it in your browser. A copy written by hand here would drift, and a drifted
API reference is worse than none.

**If you would rather write in VS Code**, the same panel will set a mod's folder up for
it — every name offered as you type, each one explained on hover, and a line under
anything the moment you are writing does not hand over. It is written from the build you
are running, for the same reason as everything above. See *Writing it in VS Code
instead* in [scripts info.md](scripts%20info.md).

That is the line to hold when adding to `scripts info.md`: an example that names a call
in context is fine and is how anybody learns this, and a table of every call is not.

---

<!-- Adding a document: put the .md file in this folder and add one row to the table
     above. Keep the format mods info.md uses -- heading, example, field table,
     prose only where a field genuinely traps someone. Filenames with spaces need
     %20 in the link. Anything above that talks about "the documents" in the plural
     has to be checked against the table when a row is added or removed; the plural
     already outlived a document once, which is why nothing above says it now. -->
