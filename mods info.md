# THE FOLDER
- what a mod is on disk, and where each kind of file goes.

A mod is one folder under the game's `Mods` folder, and its name is the mod's name everywhere. Inside it, one file is
required and everything else is filing:

```
Mods/
  YourMod/
    mod.json          the only required file - everything below is named by it
    cover.png         the picture on this mod's card in the companion. Optional
    scripts/          the .lua your `script` key names
    art/              portraits, character art, skill cards, cut-in pictures
    icons/            the small square pictures for status effects
    audio/            sounds a beat plays, and anything your `audio` list names
    voices/           voice recordings
    motions/          the .bundle Unity built, and its .manifest
```

**Nothing here is enforced, and nothing here is magic except `audio`.** Every path in mod.json is read relative to
the mod's own folder, so `"icon": "pictures/mine/bleed.png"` works exactly as well as `"icon": "icons/bleed.png"`, and
a file sitting beside mod.json with the manifest naming it plainly works too. These are the folders the app puts
files in when you import one, and the folders the Unity build writes into -- so a mod built by the toolkit and a mod
assembled by hand end up looking the same.

**`audio` is the exception, and it is worth knowing.** Every `.wav` and `.ogg` sitting DIRECTLY in that folder is
registered as a sound your mod owns, named `<your mod>/<the file name without its ending>`, whether you declared it
in `audio[]` or not. That name is the only way a beat on a motion can ask for a sound -- a cue's `clip` is a name and
never a path -- so a sound in `audio/hit.wav` can be played by a beat and the same file in the mod's root cannot.
Files in folders INSIDE `audio` are not picked up; it is one level only.

**Recordings are kept out of `audio` on purpose.** A mod with thirty voice lines would otherwise add thirty names to
every sound list in the app, none of which a beat has any use for. Put them in `voices` -- `voices[].file` is an
ordinary path and reads from anywhere inside the mod.

**A path may never leave the mod's folder.** `cover` and `script` say so outright: a full path off your own disk, a
drive letter, a leading slash or a `..` is refused by name and reported, because a mod is a folder somebody hands to
somebody else and a path to your own machine arrives pointing at nothing.

**Three things in a mod folder are not part of the mod.** `.limbonia/`, `.vscode/` and `.luarc.json` are written by
**Set up `<your mod>` for VS Code**, under *Write it in VS Code instead* on the Scripts screen. They describe the
build of the game you have right now, and the game never opens any of them. Delete the lot and the mod works exactly
as before. **Share this mod…** leaves them out. If you have edited `.luarc.json` yourself, pressing the button again
keeps your version and only adds to it.

**All three sit at the TOP of the mod folder, and that is why VS Code is opened on the mod folder** rather than on
the `scripts` folder your script is probably in. An editor looks for those settings at the root of whatever it was
opened on and nowhere else, so a window opened on `scripts` -- or on the script by itself -- finds none of them and
quietly offers you nothing, with no message to say why. Open the mod folder with **File → Open Folder**, then reach
your script from the sidebar. They cannot be moved down to meet the expectation either: a mod may equally keep its
script beside mod.json, and a settings file inside `scripts` would do nothing at all for that one.

**Moving a file does not move the name that points at it.** Every path in mod.json is written by hand or by the app,
and neither follows a file you drag into a folder afterwards -- so the commonest way a finished mod stops working is
that the file is perfectly safe one level down while the manifest goes on naming where it used to be. Both places
that report a name pointing at nothing now look for the file before saying it is gone: the **Behaviour script**
field offers to point the mod file at it in one press, and **Share this mod…** says where each missing file turned
out to be. Nothing is rewritten unless you press something, nothing on disk is moved, and the match is the file name
and nothing cleverer -- a file you RENAMED is not found, because guessing which one you meant is worse than saying
nothing.

# SHARING
- **Share this mod…**, at the bottom of the Overview tab on the Mod Author screen.

Pick where to save it and you get one zip holding everything the mod needs and nothing else. Whoever you send it to
drops it into their own `Limbus Company/Mods/` folder as it is -- the archive opens as a folder with your mod's name
on it, so nothing is scattered over what they already have installed.

**It packs what your mod ASKS FOR, not what happens to be in the folder.** mod.json goes in, then every file it
names -- your script, your pictures, your sounds, your recordings, your bundle, your cover -- and then the two things
the game reads without being told: every `.wav` and `.ogg` sitting directly in `audio/` (see THE FOLDER), and the
`.manifest` sitting beside a bundle if Unity wrote one. That is the whole rule, and it is that way round on purpose:
a list of things to LEAVE OUT would ship whatever nobody thought of, on the day somebody invents a new kind of
clutter, silently. It also means a file is packed because something in mod.json NAMES it, not because it sits under
a folder this document happens to list -- filing is a convenience, and the packager reads the manifest.

**What it leaves behind, and it tells you which is which:**

| Left out | Why |
|----------|-----|
| `.limbonia/`, `.vscode/`, `.luarc.json` | the editor pack. Written for the game build on YOUR machine, read by nothing
| `mod.json.<date>-<time>.bak` | the dated copies taken whenever your mod file is rewritten. Your undo, not their mod
| `Thumbs.db`, `desktop.ini`, `.DS_Store`, `.git/` and the like | your computer's own leftovers
| a zip already sitting in the folder | a previous export, not part of this one
| anything else nothing points at | **read this one.** Either it is no longer needed, or a name in mod.json is spelled differently from the name on disk

**A file mod.json names that is not there stops the first press and asks.** That is the one fault worth interrupting
for, because it is invisible from the other end: the mod loads, reports nothing to the person who downloaded it, and
the part that needed the file simply does not happen. You are shown each one -- the value, and where in the file to
find it -- and you can go and fix them, or press **Share it anyway** if you meant it. Nothing is written until you
answer, and the archive is never half-made: it is built beside the file you chose and only becomes it once whole.

**A path to somewhere on your own computer is reported the same way** and never travels. It is the mistake that works
perfectly on the machine it was written on and on no other, which is exactly why the packager is where you find out.

**One place you cannot save it is inside the mod it is packaging**, and that is refused with the reason rather than
producing an archive that contains most of itself. Anywhere else is fine.

# UNIT
- int - top level
```json
"appearance": 10916 // Thumb Nursefather Rodion
```

# NAMES
- names[] - top level. One entry per thing renamed: who it is about, then the wording.

```json
"names": [
    {
      "oneLineTitle": "",
      "title": "Doro",
      "name": "Doro",
	  "characterName": "Doro",
      "desc": "5",
      "acquisitionMethod": "6",
      "skinItemTitle": "7",
      "skinItemDesc": "8"
    },
	{"keyword": "LIMBUS_COMPANY", "traitKeyword": "Nikke"},
	{"keyword": "LIMBUS_COMPANY_LCB", "traitKeyword": "Meme"}
  ]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| personalityId | int  | WHICH identity. Aliases appearance, unitId. Omit to use the one at the top of the file
| characterId   | int  | WHICH sinner (1-12). Only for `characterName`
| enemyId       | int  | WHICH enemy
| keyword       | text | WHICH trait keyword, in the game's own capitals - `LIMBUS_COMPANY`. Only for `traitKeyword`
| name          | text | the identity's own name, or the enemy's
| title         | text | the identity's title, and the SECOND half of the big line on the battle name tag
| oneLineTitle  | text | the FIRST half of that same line
| desc          | text | the identity's description, or the enemy's
| acquisitionMethod | text | how the identity is obtained, on its information screen
| skinItemTitle | text | the name of this identity's outfit item
| skinItemDesc  | text | the description of that item
| characterName | text | the SINNER's name. See below
| traitKeyword  | text | one of the small tags in the Trait Keywords panel. See below
| traitKeywordTitle | text | the "Trait Keywords" heading over those tags. There is one for the whole game

Every text field is a plain string (every language) or `{ "EN": "...", "KR": "...", "JP": "..." }`, and a named
language always beats the plain form -- the same rule as SKILLS.

**The big name in battle is not `name`.** The large line over a character in a fight is `oneLineTitle` followed by
`title`; `name` is not part of it. Set only `name` and the most visible name in the game does not change, with
everything else looking correct. The smaller line under it is the sinner's name, `characterName`.

**`characterName` renames that sinner EVERYWHERE.** On every identity they have, and in story dialogue, mirror
dungeon logs, the store and battle results. Use it when you mean to rename the person rather than one of their
identities.

**A trait keyword belongs to everyone who carries it.** `traitKeyword` renames one shared piece of text, so it
changes for EVERY character with that trait and not only the one your entry is about. It reads like a local change
because the entry sits under a character in the file, and it is not one.

An id or a keyword the game has no entry for changes nothing and breaks nothing. Whether the game knows an id can
only be answered once its text tables have loaded, so it is not a scan-time fault: the Mod Author screen lists them
under "The game has no entry for these" once the game is running.

# SKILLS
- skills[] - top level. One entry per skill: the wording and the picture live together.

```json
"skills": [
  { "skill": 1, "name": "Sundered Guard", "icon": "art/skill1.png" },
  { "skill": 2, "name": { "EN": "Bloom", "KR": "..." }, "desc": "On hit, inflict 2 Bleed" },
  { "skillId": 1010104, "flavor": "...", "icon": "art/skill4.png" }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| skill    | int    | WHICH skill, counting from 1 — the same "Skill 1 / 2 / 3" the game shows. `"skill": 0` is refused and reported. Needs a character: `appearance` here or once at the top. Aliases skillSlot, slot
| skillId  | int    | that one skill outright. Needs no character, and works before the game has loaded
| textId   | int    | the shared wording row outright. For when you already know you want the shared one
| name     | text   | the skill's name — battle skill card, the skill list on the information screen, every tooltip that names it
| desc     | text   | the skill's description. See the two warnings below
| flavor   | text   | the skill's flavour line
| icon     | path   | 256x256 — the picture on the skill card. See "where it shows up" below

Any text field is a plain string (every language) or `{ "EN": "...", "KR": "...", "JP": "..." }`. A named language
always beats the plain form. Every uptie level of the skill is renamed, so it does not change at +2 / +4.

**Two things about `desc`.** Most of the game's own sinner skills leave it EMPTY and put the effect text on the
individual coins instead — that per-coin text cannot be changed yet, so setting `desc` on such a skill adds a line
rather than replacing the one you are reading. And the game fills a description in from a template, so `{` and `}`
are placeholders: write `{{` and `}}` for a literal brace. `name` and `flavor` have neither problem.

**Where the icon shows up, and where it does not.** It is replaced on the battle skill cards — the card you drag onto
a target, the icon over a unit, and the queued-action slot. Other screens, including the skill list on a character's
information screen, load their copy in a way that cannot be redirected, so those keep the game's own picture. The
name and description change everywhere.

**E.G.O. skills**: name and description work; the picture comes from the E.G.O. artwork rather than a skill icon,
so `icon` will not appear. The mod list says so against the entry.

Several skills sometimes share one wording row. Renaming it renames all of them — the mod list names which.

# PORTRAIT
- art[] - top level
```json 

"art": [
  {
    "profile":  "art/profile.png",
    "info":     "art/info.png",
    "cg":       "art/illust.png" 
  },

  { "awakened": true, "cg": "art/illust_awakened.png" },
  { "characterId": 9,       "icon":    "art/face.png" },
  { "portraitName": "SomeEnemy", "portrait": "art/bust.png" },

  { "buff": "Laceration", "buffIcon": "art/bleed.png" },
  { "skill": 1,           "icon":     "art/skill1.png" }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| personalityId | int    | who. appearance is an alias and accepts the full name for ("10916_Rodion_Thumb_FatherAppearance")
|characterId   | int    | sinner, for icon only. Alias sinnerId. Derived from personalityId if omitted - (id/100)%100
|portraitName  | string | for portrait only - enemies/NPCs are addressed by name, not id
|awakened      | bool   | which state. Alias gacksung. Omit to cover both; an exact-state entry beats a catch-all regardless of order
|profile       | path   | 256x256 - battle portrait and every small unit icon
|info          | path   | 537x827 - tall card bust
|cg            | path   | 1920x1080 - flat illustration (the Spine overlay is suppressed for you)
|portrait      | path   | 493x434 - enemy/NPC bust
|icon          | path   | square - the sinner logo. Changes it on every identity of that sinner, plus name tags, passive slots, battle results, store
|buff          | int/string | a STATUS EFFECT, not a character. A number is the effect's id; a word is the GAME's own key, not the name on screen - Bleed is `Laceration`, Tremor `Vibration`, Rupture `Burst`. See BUFFS. Aliases buffName, statusEffect, buffId, buffID
|buffIcon      | path   | the icon in the status bar and its tooltip
|buffTypoIcon  | path   | the small symbol beside a floating damage number. Falls back to buffIcon if you leave it out
|skill         | int    | a SKILL icon, counting from 1. skillId addresses one outright. Uses the `icon` key

**Two entries here are not about a character at all.** A `buff` entry addresses a status effect and a `skill` entry
addresses a skill, so neither takes the mod's top-level `appearance` — they are deliberately left out of it, because
an effect belongs to the game rather than to your character. Everything else in `art[]` needs to know who it is for.

# voices[] — top level
```json 
"voices": [
  {"situation": "battle_startstage", "file": "voice/start.ogg" },
  {"situation": "battle_kill", "files": ["voice/k1.ogg","voice/k2.ogg"] },
  {"situation": "battle_select", "variant": 2, "file": "voice/sel2.ogg" },
  {"situation": "*", "mute": true },
  {"voiceId": "battle_select_hysteric_10312_1", "file": "voice/hyst.ogg" },
  {
    "situation": "battle_kill",
    "text": "Let's go.",
    "file": "sounds/entry.ogg",
    "bubbleSeconds": 2.5
  }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| personalityId | int          | aliases appearance, unitId
| voiceId       | string       | the full id, e.g. battle_kill_10916_1 - highest precedence, ignores every other selector
| situation     | string       | "*" or omitted = any
| variant       | int          | the trailing number. Battle cues always use 1 
| file / files  | path / array | files picks one, avoiding an immediate repeat 
| mute          | bool         | silence. The only way to get it - a custom character with no recordings should be silent, not speak in its stand-in's voice 
| volume        | number       | clamped to 4.0 max
| text          | text         | a speech bubble over the character when this cue fires. A plain string, or `{ "EN": "...", "KR": "...", "JP": "..." }` like every other text field
| bubbleSeconds | number       | how long that bubble stays up, 0 to 60. Left out, it follows the length of the sound

***

|    situations      |          when it fires      |
|--------------------|-----------------------------|
| battle_startstage  | Entering a fight            |
| battle_select      | Picked for a turn           |
| battle_endcommand  | Turn confirmed              |
| battle_break       | This unit is staggered      |
| battle_enemy_break | This unit staggers an enemy |
| battle_kill        | This unit defeats an enemy  |
| battle_dead        | This unit is defeated       |
| battle_add         | Extra / multi line          |
| battle_special     | Scripted one-off lines      |
| battle_defeat      |   when battle ends and you lose|
| battleentry        | when entering a battle after a formation confirm |
|   any or *         | match every situation       |

**That table is not a limit.** The hook sits inside the game's own sound lookup, so ANY situation the game asks for
works -- a cue added by a game update can be written straight into `situation` without waiting for a new Limbonia.
The list is what this build has been seen to use, nothing more, and a name it does not recognise is passed through
rather than refused.

**A situation nobody records is a FREE SLOT, not a broken option.** The game asks for the cue whatever character is
standing there, so the fact that almost no vanilla character has a recording for one is a reason to take it, not a
reason to avoid it. Read the coverage the other way round from the way it looks.

**To find out what a line you just heard is called, turn on voice logging.** The switch is beside the voice list on
the Mod Author screen; it writes every cue the game asks for to `Limbonia.log`, marked `[VoiceLog]` -- every
character's, not only yours. `ok=0` on a line is the game saying it has no recording for that cue at all. Turn it
off again when you are done: it writes a line per voice.

# audio[] - top level, and cues[] - per motion asset
```json
"audio": [
  { "id": "slash", "file": "audio/slash.wav", "volume": 1.2, "name": "Sword slash" }
],
"motions": [{ "appearance": "…", "bundle": "…", "assets": [
  { "name": "s1", "motion": "S1", "index": 0,
    "cues": [ { "time": 0.35, "clip": "mymod/slash", "volume": 1.4 } ] }
]}]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| audio[] |    []    |                                                                                  
| id      | string | required                                                                         
| file    | string | required, relative to the mod folder. WAV or OGG only - decided by the EXTENSION, not by the bytes 
| volume  | number | optional, max 4.0                                                                
| name    | string | optional label, defaults to id

**Nothing sniffs the file's contents.** The extension alone decides how the reference is read; whether the bytes
actually play is FMOD's call later, at load time. So a renamed mp3 passes the scan and then fails with "FMOD could
not decode it" when it is first asked for. Use a plain PCM `.wav` or a real `.ogg`.

**You can export one of the game's sounds and edit it.** Pick a sound, press export, and Limbonia decodes it
straight out of the game's own sound bank into a `.wav` **inside your mod's `audio` folder**. Nothing has to be
playing and you do not have to be in a fight - it reads the file on disk, so it works at the title screen.

That gives you the whole round trip in one place: export, open the file in whatever you edit sound with, save it
back where it already is, and name it in a `clip`. It is now a sound of your own and everything on this page
applies to it.

A few sounds cannot be exported, and each says which it is: a name the game does not have; a sound Limbonia could
not find in any of the banks it read, so there is nothing for it to decode; or one whose bank it has not read yet,
which fixes itself in a moment. The middle one is about Limbonia's own search and not about the game - if a sound
refuses to export but plays when you audition it, report it. And sometimes the game names a sound's audio
differently from the event that plays it - when that happens you are shown what is in that bank and you pick one,
rather than being told no.

**You can slow a sound down or speed it up.** `"speed": 0.5` makes it take twice as long, `2` makes it take half.
It works on a sound of your own and on one of the game's, on a motion cue, on a `sounds[]` beat, on a status
effect's cue (`soundSpeed` there, because that entry names its sound with `sound`), and on the audition button.
From 0.05 to 10 - the same range the visual effects' `speed` uses.

**It changes the pitch too, and that is not a bug.** Playing a recording slower means reading it slower, so it also
comes out lower - the familiar "slowed down" sound. A voice becomes a growl and a sword swing becomes a boom. That
is usually exactly what people want when they ask for this, but it is worth knowing before you put `0.3` on a line
of dialogue. Limbonia does not stretch time while holding the pitch: doing that needs a pitch-shifting effect that
smears the sharp edge off impacts, which are the very sounds this is most often reached for.

**`stop` is not moved by it.** An end time is a moment on a real clock, so a sound played at half speed still stops
when you told it to - it has simply got half as far through itself. If you slow a sound down and want all of it,
raise `stop` as well or leave it out. Same rule as the visual effects' `seconds`.

**Sound needs no Unity.** A `.wav` or `.ogg` sitting in the mod folder, named in `audio[]`, is a complete mod on its
own: no bundle, no Unity project, no version to match. That is true of everything outside `motions` -- `names`,
`art`, `voices`, `buffs`, `targetGroup` and `script` all work from files and text alone.

**Mod sound comes out on its own track.** Every sound a mod plays -- a cue on an animation, a replaced character
voice line, a status effect's sound, a preview -- goes to one track that belongs to mods and nothing else. The
game's own **voice**, **effects** and **music** sliders do not touch it. A player who turns character voices down
does not turn your mod down with them.

The one control over it is **Mod sound volume**. It sets the level of every installed mod's sound at once and is
remembered between sessions. The game's **overall** volume still applies on top, and muting the game mutes mod
sound with it -- a mod that kept playing after the player silenced the game would be a bug.

**It is in two places, and it is one level.** There is a slider on the Mods screen, and there is a **Limbonia
Sounds** row in the game's own audio options, under Master / BGM / Effects / Voice, with the same slider and the
same mute button as the four above it. They are two views of one number and cannot disagree. Moving the one in
the app reaches the game at once; moving the one in the game reaches the app when you press **Confirm** or leave
that screen, which is the moment the game saves its own rows too. The row runs full travel from silent to twice as
loud, so "as you intended" is the middle of it, and the slider on the Mods screen has the same range.

On the game's screen it also behaves like the rows beside it: you hear the change as you drag, **Confirm** keeps
it and **Cancel** puts it back. Nothing of ours goes into the game's own settings file -- that file has no room
for it -- so the player's real audio settings cannot be disturbed by this, and the level is saved with the rest of
Limbonia's.

If a game update ever reshapes that screen, the row simply is not there; the Mods screen slider is unaffected and
so is every sound.

**Three levels multiply, in this order:**

| Level | Set by | Where |
|-------|--------|-------|
| the sound's own | you | a cue's `volume`, or failing that the clip's `volume` in `audio[]`. 1.0 = the recording as it is |
| mod sound volume | the player | the Mods screen, or **Limbonia Sounds** in the game's audio options. 1.0 = as you intended, up to 2.0 |
| the game's overall volume | the player | the game's own options. Muting the game is zero |

So `"volume": 1.5` on a cue means "half again as loud as this recording *within the mod track*", not "louder than
the game". Mix your sounds against each other with the per-sound numbers and leave the rest to the player -- if
every one of your sounds needs the same boost, the recordings are quiet and `audio[]`'s own `volume` is the place
to fix it once.

**Telling a level problem from a loading problem, because a level problem is silent.** A sound whose FILE is wrong
is named on the mod's card when the mod is read: a path pointing at nothing, an extension that is not `.wav` or
`.ogg`, an empty file. A sound whose file is there and whose BYTES will not play is a different fault, found only
when something first asks for it -- so it appears on the card DURING a fight rather than when the mod loads, and it
says which sound and what is wrong with it. That holds wherever the sound was declared: a cue on an animation, a
beat on the character's own animation, a voice line, a status effect's own sound. A sound that loaded and then
played at nothing says *nothing at all, anywhere*: it is a working sound at zero volume, which is indistinguishable
from a working sound. So when you hear nothing:

1. Preview the sound in the editor (**Hear it**). It plays through the same track at the same levels, so hearing
   it there clears the file AND every level at once -- what is left is the cue not firing.
2. If the preview is silent too, it is a level. **Mod sound volume** on the Mods screen, then the game's overall
   volume, then whether the game is muted.
3. Then the numbers you wrote. A `volume` of `0` is silence and is honoured exactly as written, on the cue and in
   `audio[]` alike.
4. Only then the file. Read the mod's card -- every load failure is named there, by sound, whether it was found
   when the mod was read or when the fight first asked for it. If the card says nothing, the file is fine and the
   silence is coming from somewhere in 1-3.

**A sound plays to the end of the FILE, not to the end of the move.** A long sample is still going after the attack
that started it, which is not a fault and is the commonest thing reported as one. Leaving the fight stops every one
of them at once. To cut one short, give the cue a `stop` -- the **Stops at** box in the animation editor -- and
optionally a `fade`. Both are refused on a hit-anchored cue; see above.

| Field     |  Type  |  Notes  |
|-----------|--------|---------|
| cues[]    |   []     |                                              
| time      | number | seconds, on that coin's clock. Not read when the cue is anchored to the hit
| clip      | string | <modfolder>/<id> - note the namespace prefix 
| volume    | number | optional, overrides the clip's own
| speed     | number | how fast it plays. 1 (the default) is untouched, 0.5 is half speed and twice as long, 2 is double. From 0.05 to 10. Slower is also LOWER in pitch - see below. Does not move `stop`
| stop      | number | when it stops, on the same clock as `time`. Omitted = plays to its own end
| fade      | number | the ramp down to silence, ENDING at `stop`. Omitted = a short declick. Means nothing without `stop`
| playsWhen | text   | "time" (default) or "hit" - see below
| hitRepeat | text   | "impact" (default, once per hit) or "beat" (once per damage beat). Only with "hit"

**A cue can wait for the hit instead of a time.** `"playsWhen": "hit"` fires the sound every time that coin's damage
actually lands, rather than at a moment you picked. Use it for impacts: a `time` is on the animation's clock and the
hit is on the fight's, and the moment you retime the coin -- with `timing`, or with a `phases` set -- the two come
apart and your impact sound drifts. It also works on a coin you supplied no animation for. On a multi-hit beat you get
one per impact by default, which is what the game's own impact flourish does; `"hitRepeat": "beat"` gives you one per
damage beat instead. Four or more overlapping copies of one sample is a wall of noise rather than four hits, and the
mod list says so when your coin declares how many hits it makes.

Two things stop working when you anchor a cue to the hit, and both are reported rather than guessed at: `time` is
never read (write one and you are told it does nothing), and `stop`/`fade` are refused outright and the sound plays
all the way through -- an end time is a moment on the animation's clock, and a hit-anchored sound is not on it. To cut
a sound short, leave it on `"playsWhen": "time"`. A sound anchored to the hit on a POSE motion can never be heard,
because a pose never lands damage; that is reported too.

**You can use the GAME's own sounds as well as your own.** Everywhere a mod names a sound -- a `cues[]` entry on a
motion, a `sounds[]` beat, a status effect's own cue -- you can write one of the game's instead of a file of yours:

```json
"cues": [ { "time": 0.35, "clip": "event:/SFX/Battle/shield_break" } ]
```

**It is the same key.** `clip` takes either spelling and you never choose between two of them: a name beginning
`event:/` is one of the game's, anything else is yours. Nothing about the rest of the entry changes.

**The list is browsable and it is all of it.** Limbonia reads every sound the game has -- 56,237 of them -- straight
out of its audio engine, and each one carries a **category** taken from its own path: `event:/SFX/Battle/…` is SFX,
`event:/Voice/…` is Voice, and so on. The picker offers the categories with their counts, so you pick a category
first and then look inside it. The list is complete the moment the game has started; there is nothing to wait for
and nothing saved to disk.

The names are the sound engine's own folder paths. They are readable, but they are engineers' names rather than
titles somebody wrote for players, so browse and listen rather than trying to guess one.

**Voice lines are in the list, and naming one is not the same as replacing one.** The Voice category is the largest
by far -- 45,057 of the total. Naming one here PLAYS the game's own recording, wherever you put it. `voices[]`, a
different section entirely, puts a recording of YOURS in place of a character's line. Both are useful; they are
different acts.

**Some sounds say who plays them.** While Limbonia is reading the game's characters it also notes which sounds each
character's own animations fire, so a row can read **"Ryoshu (Spider Bud) - S3"** instead of just a folder and a
name. About one sound in six carries an attribution like that - it is not most of the catalogue and is not meant to
be. What it covers is almost entirely the battle sounds, which are the ones with nothing else to go on: a voice
line has the character's number written into its own name, and a sword impact has nothing at all.

A sound used by several characters shows the first one and says how many others also use it.

**Limbonia reads the game's sound banks once, in the background.** It works through them a few at a time whenever
you are not in a fight - never during one - to learn which bank holds which sound. It costs a few seconds in total
and the answer is saved next to the game, so it happens **once**: every session after that it is there from the
first second. If the game is patched it notices and reads them again by itself. Until it has reached a particular
bank, a sound from that bank says so and asks you to try again in a moment.

A bank that will not open on the first attempt is **tried again**, several times, spaced out - it is usually just
the game still starting up. If one still refuses after all of that, Limbonia says so rather than pretending the
sounds inside it do not exist.

**Three things a name alone cannot tell you, and Limbonia says all three.** Press the audition button beside the
picker and it plays there and then, and tells you what it turned out to be:

* **How long it runs**, when it has a fixed length.
* **Whether it loops.** A looping sound has no end of its own - so the audition button becomes a **toggle** for
  those: press it again to stop it. (A one-shot has already finished by the time you could press it twice.)
* **Whether it is positioned or flat.** A flat sound fills the screen rather than coming from anywhere.

**There is a stop button, and it stops two different amounts.** By default it silences everything **Limbonia**
started - your own sounds, and any looping game sound you left auditioning. That is exact, because Limbonia knows
what it started, and it never touches the game's own audio. The bigger version stops **everything currently
sounding**, the game's music and ambience included; new sounds play normally afterwards, but music that was already
running stays stopped until the game next changes track. Reach for the first one first.

**And three ways a declared one can fail, each of which lands on your mod's card rather than being silence:**

* **A name the game does not have.** Reported when the mod is read.
* **A sound whose part of the game is not loaded yet.** The list names every sound in the game, including ones the
  current fight has not loaded. Limbonia **fetches** those: it knows which of the game's sound banks each sound
  lives in, loads that one bank, and plays it. The first time you ask, you get "loading it -- try again in a
  moment", exactly as you do when you borrow an effect from a character whose artwork is not on your machine yet.
  A large bank takes a second the first time; the announcer's voice bank is the biggest at about 100MB, and a
  wait is not a hang. Every sound the game actually ships is usable this way.
* **A sound Limbonia cannot find a bank for.** The list of names comes from the game's audio project, and once
  every bank has been read a handful of those names turn out not to be in any of them. Limbonia then has nothing
  it could load to play that sound, and it says so instead of asking you to wait for something that is never
  coming. It says it about *itself*, not about the game: the game reaches its own sounds by its own means, and
  Limbonia is in no position to tell you what those are. If a sound refuses to export but plays perfectly when
  you audition it, that is worth reporting - it means the bank list has a hole in it.
* **A looping sound is refused outright.** You may audition one, but you may not declare one. The game hands its
  sounds out and immediately forgets them, so there would be nothing left for Limbonia to switch a loop off with --
  and a sound that runs for the rest of the session is worse than one that never plays.

**Two things a declared game sound does not do.** It has no position -- it plays centred and at full level, the way
the game plays a menu sound -- so putting one on a beat that belongs to a character does not make it come from
them. And `stop`/`fade` do not apply: the game plays its own sounds to their end and keeps nothing that could cut
one short. Ship the sound as a file of your own if you need to end it early. Both are reported rather than ignored.

**They play at the game's own sound level, not the mod one.** A sound of yours rides Limbonia's own audio track so
that the game's category sliders cannot silence it. One of the GAME's sounds is the opposite case: it already
belongs to a category, and it obeys the game's own sliders exactly as it does when the game plays it. That is
deliberate -- it *is* one of the game's sounds.

# THE TOP LEVEL
- the root of mod.json. Everything else in this document lives inside one of these.

```json
{
  "enabled": true,
  "appearance": "10101_YiSang_BaseAppearance",

  "art":     [],
  "names":   [],
  "skills":  [],
  "voices":  [],
  "audio":   [],
  "motions": [],
  "sidecar": [],
  "buffs":   [],
  "script":  "behaviour.lua"
}
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| enabled | bool  | false switches the WHOLE folder off - art, names, voices, sounds, animations, everything. A real yes/no, not "false" in quotes
| appearance | int or text | who this mod is about, said ONCE. Aliases personalityId, unitId, characterId, id - consulted in that order, the first one carrying a number wins
| renderer | text | how your animations are drawn. See below
| sidecar | [] | the older way to write a `motions` entry. See below
| script | text | the name of a .lua file in this folder. See SCRIPT
| cover  | path | a picture for this mod's card in the companion. The GAME never reads it. See below

**Say who once.** Every section that addresses a character -- `names`, `voices`, `art`, `skills`, `motions`,
`sidecar`, `buffs`, `script` -- falls back to the top-level declaration when its own entry names nobody. An entry that DOES name somebody
always wins, so a mod about one character can still make one section an exception. This is the same fact written once
instead of once per section, and forgetting one section is the failure it exists to remove.

Written as a bare number (`10101`), it matches every appearance of that identity -- the base one and each of its
skins -- which is what a whole-character mod wants. Written as a full appearance name
(`"10101_YiSang_BaseAppearance"`), it matches that one and no other. Written as a SINNER number (1-12), it is refused
and reported: a sinner has many identities and a number that low cannot say which you mean, so the rest of the file is
left with no character to fall back on. A string that does not start with digits is ignored rather than misread, so
`"id": "my-mod-guid"` is safe to keep for your own bookkeeping.

**Every number this document asks you for is printed in the app, beside the name, in every build.** Identity and
appearance numbers, skill numbers, status-effect numbers and their real keys, item numbers -- they are in the
pickers and the lists, so nothing here has to be guessed or hunted for elsewhere. That holds for the copy you hand
somebody else exactly as much as for the one you build with: a manifest addresses the game by these numbers, so
they are never hidden.

**One wrong value never kills the game.** Every section is read defensively and reports rather than throws: a number
where text was expected, a quoted `"false"` where a yes/no was expected, a time that is not a time -- each of them is
skipped, the rest of the file still loads, and the sentence appears on the mod's card in the Mods panel. If mod.json
will not parse at all (a missing comma, bracket or quote) the whole folder is skipped and told so. Read that card
first: almost everything in this document that can go wrong says so there, in words, before you ever start a fight.

**`renderer` has one value and you do not have to write it.** Your animations are drawn alongside the character's
own by a second renderer, which is what `"renderer": "sidecar"` names -- and it is the default, so a `motions` entry
that says nothing gets it. It is the only value the key takes, and any other word is refused by name when the mod
loads rather than acted on: a typo must never move a character onto a way of drawing you did not ask for. The key
may sit at the top of the file or inside a single `motions` entry, and the inner one wins.

**`sidecar[]` is the older spelling of `motions[]` and you should not write a new one.** It takes the same settings
with `clips` in place of `assets`, and `coin` in place of `index`. A `motions` entry is read by every section of this
document; a `sidecar` entry is read only by the part that draws, so it covers the picture -- `timing`, `loop`/`onEnd`,
`sync`, `form` and `becomes` -- and nothing else. `cues`, `speech`, `vfx`, `camera`, `move`, `picture` and the
authored beat set are simply not looked for there. Write `motions`: a `motions` entry is drawn by the same renderer,
so nothing about the picture changes.

**`bundleId` is written FOR you, and the game does not read it.** It is an 8-character hex id the Unity build puts
there so the bundle it produces keeps the same internal name across rebuilds -- leave it alone, and never write one
by hand. It is not reported as a mistake and it does nothing if you add one yourself. The mod's own name is its
FOLDER name, everywhere -- there is no `author`, `version` or `description` key either.

**`characters` names the OTHER characters a mod is about.** Who a mod is about is `appearance`, here and on each
section, and that is the one the game reads; a mod made in the app is given one the moment you pick a character.
`characters` is a plain list of further appearance ids beside it, for a mod that covers more than one character and
ships no animation for the extras -- the Animation Timing screen opens an editing target for each name in it, so
camera work, sounds and movement can be layered on a character you supply no artwork for. Nothing in the game reads
it, so it is never reported as a mistake and it changes nothing on its own.

**`cover` changes nothing in the GAME and something in the app.** It names a picture inside the mod's own folder -- `cover.png`, or
`art/cover.png` -- and it is drawn on the mod's card in the companion, cropped to fit. It must be a MOD-RELATIVE
path: a full path off your own machine is refused by name and reported, because a mod is a folder somebody hands to
somebody else. Left out, the card draws a picture from the mod's name instead, so nothing looks broken. Written as
`""` it means a deliberate no-picture; deleting the key does not, because a top-level key missing from a save is
carried back in from the file on disk.

**Saving mod.json is not the same as the game re-reading it.** The writer re-reads the animation side only, so a
save that changed pictures, wording, skills, voice lines, sounds or status effects is on disk and not yet in force.
The Mod Author screen asks for the whole-folder re-read itself and says whether it landed; **Reload mods** on the
Installed screen is the same re-read asked for outright, and is what to press after a Unity build or any edit made
outside the app. It picks up a rebuilt bundle without restarting the game, and it can refuse -- a modded animation
mid-play is not a safe moment to let go of an archive -- in which case it says so and you press it again.

**Switching a mod off does not stop all of it at the same instant.** Its art, voices and sounds are fetched when the
game asks for them, so those stop at once (re-open the screen you are looking at to clear what is already drawn).
Its animations have already been written into a character that may be on screen, and taking those back out from
under a running battle is not safe, so they clear at the NEXT battle.

# MOTIONS
- motions[] - top level. One entry per character: one appearance, one bundle, and everything in it.

```json
"motions": [
  {
    "appearance": "10101_YiSang_BaseAppearance",
    "bundle": "motions/yisang.bundle",
    "coverage": "wholeCharacter",
    "fallback": "Idle",
    "assets": [
      { "name": "s1_0",   "type": "TimelineAsset", "motion": "S1",   "index": 0 },
      { "name": "s1_1",   "type": "TimelineAsset", "motion": "S1",   "index": 1 },
      { "name": "idle",   "type": "TimelineAsset", "motion": "Idle", "index": -1, "loop": true },
      { "name": "AuraFx", "type": "GameObject" }
    ],
    "alwaysOn": [ { "asset": "AuraFx", "y": 0.2, "scale": 1.5, "sortingOffset": -1 } ]
  }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| appearance | text | which character. Omit to use the one at the top of the file
| bundle     | path | required unless the only things the entry carries are `gameEffects` and `targetGroup`. Relative to the mod folder; the .bundle Unity built
| assets     | []   | required with a bundle. Everything you want out of that bundle - see ASSETS
| coverage   | text | "wholeCharacter" (the default here) or "perMotion". See below
| fallback   | text | which of your motions to play for one you did not supply. Defaults to Idle, then Default
| alwaysOn   | []   | effects that sit on the character for the whole battle - see ALWAYSON
| gameEffects| []   | the GAME's own effects, played by name. Needs no bundle at all - see GAMEEFFECTS
| targetGroup| []   | where the units you are hitting stand - see TARGETGROUP
| renderer   | text | overrides the top-level one for this character only
| parent     | text | scalePivot (default) / appearance / animPivot / renderer - what your drawing hangs off
| spritePath | text | where the sprites load from. Defaults to "auto", the character's own
| sortingOffset | int | added to the character's drawing order, -1000..1000. 0 = exactly where the original drew
| sync       | text | auto (default) / game / own - whose clock your animation runs on
| hide       | {}   | main / parts / spine / effects - which of the character's own renderers to switch off. main defaults on, the rest off
| disableSpine | bool | ask the game not to draw its Spine skeleton at all. Default true
| disableTrail | bool | the same for its motion trail. Default true

**A bundle has to be built with Unity 6000.3.12f1, and the game will not accept one from any other version.** That
one version is the whole requirement -- there is nothing separate to match, because the render pipeline ships
inside that editor. The Limbonia Mod Template (`LimboniaModTemplate.zip`, beside this document) is already set up
against it, so an effect or a shader authored in there draws the same way in the game as it does in the editor.
Nothing else in a mod needs Unity at all; see `audio[]`.

**A bundle's contents cannot be listed, so everything must be declared.** Nothing can enumerate what is inside a
Unity bundle at runtime -- the only thing that can be loaded out of it is a name somebody wrote down. That is why
`assets` is required and why every prefab, texture and animation needs a line of its own, spelled exactly as it is
named in the bundle. A name that is not declared can never be found, and Limbonia says so when it reads the file
rather than in the middle of a fight.

**`coverage` decides what happens to the motions you did NOT supply.** In a `motions` entry it defaults to
`wholeCharacter`: the game's own artwork is never shown, and any motion you did not draw plays your `fallback`
instead. That is what replacing a character means. `"coverage": "perMotion"` is the other choice -- your animations
play where you supplied them and the character's own play everywhere else. A misspelling falls back to `perMotion`
and is reported, deliberately: the destructive one of the two must never be what a typo lands on. If you ask for
`wholeCharacter` and supply no Idle or Default and name no `fallback`, you are told, and the character's own artwork
is used for the gaps.

**A `timing` block belongs to ONE animation.** Written next to `bundle`, it does nothing at all -- and numbers
sitting in a file doing nothing are worse than numbers that are wrong, because you have no way to tell. Move it
inside the `assets` entry it was meant for. This is reported by name.

# ASSETS
- assets[] - inside a motions entry. One entry per thing you want out of the bundle.

```json
"assets": [
  { "name": "s2_0", "type": "TimelineAsset", "motion": "S2", "index": 0,
    "timing": { "damage": 0.62, "hit": 0.80, "end": 1.10 } },

  { "name": "s2_1", "type": "TimelineAsset", "motion": "S2", "index": 1,
    "hideDefaultImpact": true, "showDamageCounter": true },

  { "name": "guard", "type": "TimelineAsset", "motion": "Guard", "index": -1, "loop": true },

  { "name": "SlashFx", "type": "GameObject" }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| name   | text | required. The asset's name in the bundle, exactly
| type   | text | TimelineAsset (default) / GameObject / Texture2D / Material / AnimationClip / TextAsset
| motion | text | which motion this animation is for - S1, S2, Idle, Guard... An unknown name is refused
| index  | int  | WHICH COIN of that motion, from 0. -1 broadcasts one animation to every coin. Default 0
| loop   | bool | should it repeat. Only means anything on a pose - a skill animation is a one-shot by definition
| timing | {}   | damage / hit / end, in seconds - moves the game's own three beats. See below
| totalDuration | number | the switch that turns on authored beats. See GAMEPLAY BEATS
| hideDefaultImpact | bool | switch off the game's own slash/pierce/blunt burst for this whole coin
| showDamageCounter | bool | the running damage total. Only on an attack; on a pose it is reported and ignored
| fps / frames | number / int | how the animation was built. Written by the Unity build; only used to turn a `frame` in a PICTURE change into a moment
| cues   | [] | sounds on this coin - see the audio section above
| speech | [] | speech bubbles on this coin - see SPEECH
| vfx    | [] | effects on this coin - see VFX
| picture| [] | swap the drawing partway through the coin - see PICTURE
| form   | text | which set of drawings this animation belongs to. Left out = the character as you normally draw it. See FORMS
| becomes| [] | turn the character into another of its forms partway through the coin - see FORMS
| cutIns | [] | the full-screen splash, with your own picture, at a moment on this coin - see CUTIN
| camera | {} | shake / zoom (steady, or a shape you draw) / rotate - see CAMERA
| move   | [] or {} | the character stepping about - see MOVE
| sync   | text | auto / game / own, for this one animation

**Coin animations are a SEQUENCE, not variants.** A skill holds one animation per coin and the game asks for them in
order: S1 at index 0, then index 1, and so on. Coin 1 continues from where coin 0 finished -- they are not
interchangeable. Author them as a sequence and declare one per coin, in order. Supplying fewer than the skill has is
allowed and the uncovered coins keep the original animation, but the join can visibly jump, which is why the mod list
reports coverage per skill. How many coins a skill throws is gameplay data, not something an animation set changes:
supply more than the skill has and the extras are simply never played. `index: -1` is only right for a
coin-agnostic animation that starts and ends in the same pose; on an ordinary multi-coin skill it makes every coin
restart the lead-in, which reads as a stutter.

**`timing` moves the game's own beats; it does not create any.** Three numbers, in seconds from the start of that
coin: `damage` (when the hit lands), `hit` (when the coin may advance) and `end` (when the action is finished).
Anything you leave out keeps the time the game gave it. They are used exactly as written and NOT reordered for you, so
declaring them out of order -- the hit after the hand-over, say -- is legal JSON and impossible gameplay, and the
action may never finish. You are warned; nothing is repaired. `timing` is ignored entirely on a pose, and ignored
entirely on any coin that declares `totalDuration`, both of which are reported.

**Anything with no `motion` is loaded but never played by the fight.** That is correct for a prefab, a texture or a
material. It is also how you declare an animation that only a PICTURE change reaches -- though a `TimelineAsset` with
no `motion` is reported as unused, so expect that sentence on the card.

## Supplying an animation switches off the character's own camera work and movement for that coin

Give a coin an animation of yours and the character's own **presentation** for that coin stops happening. There is
nothing to write and nothing to switch on: supplying the animation is the switch. What stops:

- its camera shakes
- its steps and lunges towards the enemy
- the moments it turns to face a different way
- the moments it is told to keep following its target as the target moves
- its effect-speed ramps -- the slow-motion and speed-up written into the original
- the moments it hides and shows the battle HUD
- the cues that animate its own attached decorations, the pieces of its art that move separately from the body

Those were authored to dress the drawing the character normally makes, and you have replaced that drawing. Left in,
they are why a modded character walks to a spot your animation never goes to, and why the camera shakes on a frame
with nothing hitting in it. Use CAMERA and MOVE to put back whatever you want, where your animation wants it.

**What never stops, on any coin.** The hit, the stagger, the hand-over to the next coin and the end of the action all
fire exactly as the game schedules them, and so does anything else that is part of the fight rather than part of the
show. **Clash and duel presentation is on that side of the line too** -- the lock, the trade of blows and the way the
game stages a clash are not treated as dressing and are never stood aside, so a replaced character still clashes
looking like the game intended. That is not a setting -- there is no way to switch a hand-over off, and a fight that
never hands over is a fight you have to close the game to leave. Move them with `timing`, or author your own set with
`totalDuration`; those are the two doors, and both of them always produce a hand-over.

Anything the game grows later lands on this side by default: only the specific things listed above as dressing are
ever stood aside, so a beat added by a game update goes on playing rather than silently disappearing under a mod.

**Two things it cannot reach, and you WILL notice them.** The original animation's **built-in sounds** and the
**camera zooms and turns written into it** are not beats -- they are part of the animation itself, played by the
same machinery that draws it -- so nothing can stand them aside and they keep playing under your animation. Do not
confuse that camera turn with the character's own turn-to-face, which is a beat and DOES stand aside. For the
sounds, add your own with `cues` and treat the original's as a floor you cannot remove. For the camera, a
`camera.zoom` or `camera.rotate` of your own takes over from whatever the camera was doing, which is as close to
cancelling one as this gets.

**It follows your animation, not your declaration.** The rule applies while your animation is actually on screen for
that coin. A coin you did not supply keeps the character's own everything. A coin whose animation failed to load
keeps it too -- the character draws itself, so its own camera work is the right thing to play. And a `fallback`
standing in for a motion you never drew is not you supplying an animation for that motion, so the character's own
presentation plays there as usual.

**A coin of yours that declares no beats of its own will be quiet, and that is correct.** Supply the animation and
write no CAMERA, no MOVE and no `cues`, and the coin plays your drawing with no camera shake, no step towards the
enemy and no sounds of your own -- only the hit, the hand-over, the end, and whatever sounds and zoom the original
animation carries in itself. Nothing is inherited from the animation you replaced, because "keep the original's
shake but with my drawing" is a judgement about YOUR animation that only you can make: the original's shake is at
the original's impact frame, and yours is somewhere else. **If a replaced coin looks flat, that is what happened.**
Start with CAMERA `shake` at your impact frame and a MOVE step into the enemy; those two are most of what the
character's own animation was doing for you. The Animation Timing editor shows the character's own beats on their
own lanes, and on a coin your mod draws it strikes through the ones that will not play -- so you can still read the
time and the settings the game used and take them. Select one and **Copy it into my mod** puts an identical beat of
your own at the same moment, which you can then move. The game's beat is never edited: it is there to be read and
copied, and supplying the animation is what stands it aside.

**On a coin with `totalDuration`, none of this arises, and none of YOUR beats are touched.** That coin's beats are
built entirely from what you wrote and the original animation is not played at all, so there is nothing of the
character's to stand aside. Your own movement phases, your own hits and your own hand-over run exactly as written --
the rule above is about the character's beats, and on that coin the character has none.

# GAMEPLAY BEATS
- inside one assets entry. `totalDuration` is the switch; without it none of this is read.

```json
{ "name": "s3_0", "type": "TimelineAsset", "motion": "S3", "index": 0,
  "totalDuration": 2.4,
  "times": "seconds",
  "phases": [
    { "type": "ToTarget", "start": 0.15, "duration": 0.35, "arriveRadius": 1.2 },
    { "type": "GiveDamage", "start": 0.9, "damageRatio": 0.4,
      "multiHit": 3, "multiHitDuration": 0.25,
      "sturn": { "sturnType": "KNOCKBACK", "sturnDir": "DIR_TOTARGET", "forcePower": 6, "randomPower": 2 } },
    { "type": "GiveDamage", "start": 1.4, "damageRatio": 0.6, "isLastAttack": true },
    { "type": "Relative", "start": 1.7, "duration": 0.3, "x": -2.0 }
  ],
  "hitCheckers": [ { "time": 1.9, "isNextMotionCoinDelay": 0.1 } ],
  "endCheck": 2.2 }
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| totalDuration | number | how long this coin is, in seconds. Present = you author the beats yourself. Absent = nothing below is read
| times   | text   | "seconds" (default) or "fraction". See below
| phases  | []     | the hits and the movement. An EMPTY list means "this coin lands no hit"; leaving it out means "work it out for me"
| hitCheckers | [] | the hand-over to the next coin. AT MOST ONE
| endCheck | number | when the action is finished
| shapeTrack | bool | an escape hatch. Leave it alone unless something told you to set it

Every phase takes `type`, `start` (seconds from the start of the coin), and `steps` (repeat this phase that many
times, up to 64). `type` is GiveDamage (also spelled damage or hit), ToTarget, ToTargetWide or Relative. `MoveEnemy`
is accepted so a file written for the other motion format still loads, but it is reported: nothing in this game moves
the TARGET from a motion beat, and it is read as an ordinary Relative move, which moves the attacker.

| GiveDamage |  Type  |  Notes  |
|------------|--------|---------|
| damageRatio | number | how much of the coin's damage this beat delivers, 0..10. Alias ratio. Default 1
| multiHit    | int  | split it into this many impacts, 1..32. Above 32 is limited and reported
| multiHitDuration | number | how long to spread them over, 0..10s. Without it they arrive back to back and read as ONE hit
| afterDelay  | number | a pause after the beat, 0..10s
| isUpAttack  | bool | launches the target upward
| isLastAttack | bool | this is the coin's final blow
| isNonDamageMotion | bool | the beat plays but deals nothing
| isNonRotateSkinPivot | bool | do not turn the target's pivot
| sturn       | {}   | the knock -- see below

`sturn` takes `sturnType` (NONE, STURN, KNOCKBACK, AIRBORNE, HOLDING), `sturnDir` (NONE, DIR_ACTOR, DIR_TARGET,
DIR_TOTARGET, DIR_TOACTOR), `sturnTiming` (NONE, FIRST, LAST, ALL), `forcePower` and `randomPower` (0..500, both
default 5), `airborneAngle` and `targetRotateAngle` (-360..360) and `isRotateTarget`. A word this game does not have
is reported and the default kept. A `randomPower` larger than `forcePower` sends SOME impacts the opposite way,
because the game adds a roll between minus and plus the random amount to the force -- that is reported, not clamped,
in case you want it.

A movement phase takes the same keys a MOVE entry does, below, with `type` in place of `mode`. `x`/`y`/`z` may sit
directly on the phase or inside a `move` object; both are read, so a declaration pasted from the other format works.

**`times: "fraction"` exists only so a file written for the other motion format can be pasted in.** It makes every
number in `phases`, `hitCheckers`, `endCheck` and `vfx` a fraction of `totalDuration` instead of seconds. It does NOT
reach `cues`, `speech` or `picture`, which are always seconds. Unless you are porting something, leave it out.

**A coin may have exactly ONE hand-over beat.** The game advances the coin counter on every single one it receives --
it does not even check `isCanNextMotion` -- so a second is a second advance, and the fight either skips a coin or
never finishes. Extras are refused and reported; only the first is used. A `hitCheckers` entry may be a bare number
(the time) or an object taking `time`, `nextEffectSpeed` (0..10, default 1), `isNextMotionCoinDelay` (alias
`coinDelay`, 0..10s) and `isCanNextMotion`.

**Anything after the hand-over never happens.** The coin advances the instant that beat fires and the animation stops
there, so a hit placed later is simply never reached. You are told which beat and at what time; nothing is moved for
you, because moving somebody's hit is a bigger liberty than telling them it will not land.

**These belong to an attack.** A pose -- Idle, Guard, Move, Damaged, Default -- has no skill in flight to deliver
them to, so an authored set on one is dropped whole and reported, and the animation still plays as a pose. If you
added `totalDuration` to get an effect playing on a pose, use ALWAYSON instead: it needs no animation at all.

# CAMERA
- camera{} - inside one assets entry. Three lists, all optional, none of them synthesised.

```json
"camera": {
  "shake":  [ { "time": 0.9, "duration": 0.3, "strength": 0.6 }, 1.4 ],
  "zoom":   [ { "time": 0.2, "size": -3.0, "duration": 0.4, "between": 0.5 },
              { "time": 1.1, "duration": 0.4, "points": [ { "time": 0.07, "depth": -1.4 },
                                                          { "time": 0.17, "depth": 0.35 },
                                                          { "time": 0.27, "depth": -0.6 },
                                                          { "time": 0.40, "depth": 0 } ] } ],
  "rotate": [ { "time": 0.2, "z": 8, "duration": 0.5, "speed": 0.05 } ]
}
```

The second zoom above is a **shape**: instead of gliding to one framing, the camera goes where you put it -- in
hard, back out past where it started, in again less far, then settles. That is a punch, and it is four numbers
placed rather than a setting turned on. It is still a zoom in every other respect: same list, same `between`,
same `focusSpeed`, same one-at-a-time rule.
| shake  |  Type  |  Notes  |
|--------|--------|---------|
| time   | number | required. A bare number in the list means "at this time, everything else default"
| duration | number | 0.01..10. Default 0.25
| strength | number | 0..20. Default 0.35
| vibrato  | int | how many shudders, 1..200. Default 10
| randomness | number | degrees of scatter, 0..360. Default 90
| fadeOut  | bool | ease it out rather than cutting. Default true
| emitOnce | bool | one shove instead of a rattle

| zoom   |  Type  |  Notes  |
|--------|--------|---------|
| time   | number | required
| size   | number | HOW MUCH CLOSER. NEGATIVE ZOOMS IN. -12..12; the game's own clips live in 0..-6. Default -2. With `points`, this is worked out from the deepest of them and anything you write here is ignored
| duration | number | how long the move to that framing takes, 0.01..6. Default 0.3
| between | number | WHAT to frame: 0 = the acting character, 1 = its target, 0.5 = both. Default 0
| focusSpeed | number | how quickly the camera chases that point, 0.01..2. Never 0. Default 0.2
| axisY  | number | vertical bias, -5..5. The game leaves this at 0 on every clip
| ease   | int | 1..40. Default 6
| points | [] | THE SHAPE. Where the camera pushes and pulls, instead of moving one way. Left out = an ordinary zoom. Up to 32 of them

| zoom point |  Type  |  Notes  |
|--------|--------|---------|
| time   | number | required. Seconds from the start of THIS ZOOM, not of the coin. Two points must be at least 0.02s apart
| depth  | number | how far the camera is at that moment, in the same units and sign as `size` -- NEGATIVE IS CLOSER. A POSITIVE depth pulls the camera further out than where it started, which is the recoil of a punch. Default 0, which is back at the resting framing

| rotate |  Type  |  Notes  |
|--------|--------|---------|
| time   | number | required
| x, y, z | number | degrees off the camera's default, -360..360. y and z are mirrored by the acting character's facing, so one number reads the same for a sinner and for an enemy
| duration | number | 0.01..6. Default 0.3
| speed  | number | 0..2. Default 0.05
| ease   | int | 1..40. Default 6

**Nothing here is ever invented for you.** These are cosmetic, so leaving one out means "don't", never "pick
something for me". A time that is not a usable number skips that entry and says so.

**Never write `"ease": 0`.** Zero is DOTween's "unset", which sends the tween onto a curve rather than a preset. It
is clamped to 1 rather than passed on, but do not rely on that -- write a real one. `points` is how you ask for a
curve; there is no reason to reach for `ease` to get one.

**A zoom always starts from where the camera already is**, so a key at time 0 and depth 0 is added to your shape for
you. You do not write it, and you should not: without it the camera would jump to your first depth on the very first
frame. Where the shape ENDS is yours -- a last point at full depth leaves the camera pushed in, and a last point at
depth 0 brings it back to the resting framing.

**How fast a shape can go, and why nothing is clamped.** Two neighbouring points are a rate: a push at 0.07s and a
pull at 0.17s is five in-and-out moves a second. The camera smooths its own movement, so a slow shape comes through
almost whole and a fast one is ironed flat before it reaches the screen. Around **five to eight a second** is where a
push reads as an impact; past **twelve** there is nothing left of it. Points that close together are pointed out to
you and then left exactly where you put them -- they are yours, and nothing here is going to redraw your shape
behind your back. If a punch feels weak, make it **deeper**, not quicker: roughly **-0.3 to -1.5** is the punch
range, well short of the -2 to -6 a sustained push-in uses. Speeding it up is the one thing that will not help.

**Sharp and soft come from spacing.** Every point is a smooth turning place, so what makes a hit feel sharp is how
close it sits to the point before it, and what makes the recovery feel soft is how far it sits from the one after.
A push at 0.05s falling away to 0.30s snaps in and drifts out; the same two moments evenly spaced feels like a swell.

**A shaped zoom is still one zoom.** The camera runs a single zoom at a time, so one overlapping an ordinary zoom
does not layer -- the later one takes the camera over from wherever the first had got to. That is useful when you
mean it and confusing when you do not.

# MOVE
- move[] - inside one assets entry. The character stepping about during the coin.

```json
"move": [
  { "mode": "toTarget", "time": 0.1, "duration": 0.3, "arriveRadius": 1.2, "facing": "auto", "refreshDir": true },
  { "mode": "relative", "time": 1.6, "duration": 0.3, "x": -2.5, "y": 0 },
  { "mode": "toTargetWide", "time": 0.1, "duration": 0.3, "x": 1.2, "y": 0.4, "z": 0, "overrideTarget": -1 }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| time   | number | required. A bare number in the list is the time and nothing else
| mode   | text | "toTarget" (default), "relative" or "toTargetWide"
| duration | number | 0.01..10s. Default 0.2
| arriveRadius | number | how far short of the target to stop, in character-widths. Default 0. The game's own clips sit near 1.2, and nothing holds you to that -- but past the gap between the two, the stop lands BEHIND where you started and the character walks backwards
| includeTargetRadius | bool | add the target's body radius to that distance. Default OFF - on, your one number stops being the answer
| includeAttackerRadius | bool | the same for your own body. Default OFF
| chaseY | bool | follow the target's height as well
| refreshDir | bool | let the game turn the character. Default true
| facing | text | "auto" (default), "keep", "left" or "right"
| x, y, z | number | a fixed offset. RELATIVE and TOTARGETWIDE only - see below
| overrideTarget | int | which unit to aim at, -1..15. -1 is the middle of all of them
| yWorldZero | bool | measure height from the ground rather than from the character
| ease | int | 1..40. Default 6. Never 0

The older shape `"move": { "toTarget": { ... }, "relative": { ... } }` is still read and quietly migrated, so nothing
already written stops working -- but it caps you at one of each, which the game never did. Prefer the list.

**"auto" with `refreshDir` off never turns the character, and that is the one combination that means nothing
happens.** The game only turns a character when the facing is left for it to work out AND `refreshDir` is on; with
either missing, neither branch runs. Turning is what lets a step land on the other side as the fight moves, so as
written every coin ends in the same place. You are told; nothing is changed for you. An offset (`relative`) move is
exempt -- it has no facing argument at all, so `refreshDir` off is a real answer there.

**`x`/`y`/`z` on a plain `toTarget` reach nothing.** A step towards the target is aimed with a single stopping
distance and takes no offset, so those numbers sit in the file doing nothing. Use `"mode": "relative"` to move by a
fixed offset, or `"mode": "toTargetWide"`, which measures the stopping distance separately on each axis and does read
all three. Reported, never repaired -- the two modes that DO read an offset put the character in different places.

**Pair every approach with a return.** The game's own animations do. Declaring only steps towards the target leaves
the character standing wherever the last coin dropped it, and you are warned when a coin has an approach and no
offset move afterwards.

# VFX
- vfx[] - inside one assets entry. Effects on this coin: prefabs your mod ships, or the character's own replayed.

```json
"vfx": [
  { "asset": "SlashFx", "time": 0.85, "duration": 0.4, "x": 1.2, "y": 0.3, "sortingOffset": 2 },
  { "asset": "BurstFx", "playsWhen": "hit", "hitRepeat": "beat", "playsOn": "target", "disableAfter": 0.8 },
  { "effect": "S2/0/EffectTrack/Slash_01", "time": 0.4 },
  3
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| asset  | text | a prefab YOUR mod ships, by the name your `assets` declares it under with "type": "GameObject"
| effect | text | one of the CHARACTER's own effects, by its catalogue key. Pick it from the list in the editor with the character on screen
| index  | int  | the same effect by its position in that catalogue. A fallback for when the name stops matching
| time   | number | seconds from the start of the coin. Negative = wherever the character plays it itself
| duration | number | how long its slot lasts. Omitted = the effect's own clip length
| enabled | bool | false leaves the entry in the file and switches it off
| once   | bool | let it run to its own end instead of being switched off when its slot finishes
| hide   | bool | switch the effect OFF at this point rather than on
| activeTime | number | passed straight to the game's own field of that name
| playsWhen | text | "time" (default) or "hit" - see below
| hitRepeat | text | "impact" (default, once per hit) or "beat" (once per damage beat)
| hideDefaultImpact | bool | hide the game's own impact burst while this coin plays
| x, y, z | number | where it sits, relative to the character's origin. Y is up. -1000..1000
| disableAfter | number | how long the instance stays up before switching itself off. Never left at 0
| sortingOffset | int | added to the character's drawing order. Default 1 (just in front). Negative puts it behind
| playsOn | text | "self" (default) or "target"
| targetIndex | int | WHICH unit you are hitting, from 0. -1 is the middle of all of them

A bare number in the list is an effect index at its own original time -- that is the other motion format's whole
syntax for this, so pasting one in works.

**Say which effect ONCE.** An entry that names both an `asset` of your own and one of the character's own is refused
rather than resolved, because those are two different effects and guessing is exactly what this section is arranged
not to do. An entry that names neither is refused too. Naming both `effect` and `index` is allowed -- the name wins
and the number is the fallback -- but you are told. Addressing an effect by `index` ALONE is reported every time:
effect numbers move whenever the game changes that character's animations, and then your number quietly means
something else.

**Placement belongs to a prefab of your own.** `x`/`y`/`z`, `disableAfter`, `sortingOffset`, `playsOn` and
`targetIndex` apply only to an `asset` entry. On an entry that replays one of the character's OWN effects the game
places, orders and ends it itself, so those settings are read, accepted and never used -- which is reported, and the
targeting ones are dropped so the two do not fight over the object every frame.

**`"playsWhen": "hit"` needs a prefab of your own.** Same key and same two words as a sound cue, and for the same
reason: a `time` is on the animation's clock and the hit is on the fight's, and a coin that gets retimed pulls the two
apart. It also works on a coin with no animation of yours at all. But the character's OWN effects are driven by the
game off a clip, and driving one from both sides would have them fighting -- so an on-hit entry without an `asset` is
refused, told so, and left playing at its time. Setting a `time` on an on-hit entry is reported: the hit decides when
it plays. `hitRepeat` defaults to "impact" because that is what the game's own impact effect does -- a ten-hit flurry
really does draw ten of them.

**Effects need an animation to sit on.** A `vfx` list on anything that is not a `TimelineAsset` with a `motion` is
read and never used, and reported. Thirty-two effects on one coin is allowed and warned about -- each one is a track
of its own.

**The `"from": "unity"` line is not yours.** The Unity build tags the entries IT placed from the Timeline window so a
rebuild rewrites those and leaves the ones you typed by hand alone. Limbonia ignores it entirely.

# ALWAYSON
- alwaysOn[] - inside a motions entry, beside assets. An aura, a glow, a trail: something simply THERE.

```json
"alwaysOn": [
  "AuraFx",
  { "asset": "EmberFx", "x": 0.0, "y": 0.4, "z": 0, "scale": 1.25, "sortingOffset": -1 }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| asset  | text | required. The prefab, by the name your `assets` declares it under with "type": "GameObject". A bare string in the list is this and nothing else
| x, y, z | number | where it sits, relative to the character's origin. Y is up. -1000..1000
| scale  | number | a multiplier on the size you built it at. Must be above 0; capped at 100
| sortingOffset | int | DEFAULT -1, one step behind the character, which is where an aura belongs. 1 puts it in front
| enabled | bool | false leaves it in the file and switches it off

**This is not a `vfx` entry with a long time on it, and it cannot be.** Everything in `vfx` is a clip on a timeline
and needs an animation of yours to sit on. An aura has no coin to hang off -- and the obvious place to put one, the
character's Idle, is a POSE, which is not allowed to author beats at all. So this uses none of that machinery: the
prefab is copied under the character, placed, scaled and switched on for the whole battle. It is also the answer when
you find yourself putting `totalDuration` on an Idle to get an effect playing.

**Do not write `"sortingOffset": 0`.** It is legal and it is honoured, but zero is the character's own drawing
number -- shared with its sprite and with the second renderer's -- and nothing decides which of three things tied on
one number is drawn first. The effect then appears on some frames and not others for reasons nothing on screen
explains. Use -1 or 1. This is why the default is -1 and not 0.

Sixteen always-on effects on one character is allowed and warned about; every one of them is a copy of a prefab
running for the whole battle.

# GAMEEFFECTS
- gameEffects[] - inside a motions entry, beside alwaysOn. One of the GAME's own effects, played by name.

```json
"gameEffects": [
  "BUFF_BLEEDING",
  { "label": "EFFECT_BURN", "playsWhen": "hit", "seconds": 0.8, "scale": 1.4, "layer": "skin" },
  { "label": "EFFECT_BLOOD", "playsWhen": "while", "whileEffect": "Laceration" },
  { "label": "EFFECT_FIRE", "playsWhen": "time", "motion": "S2", "coin": 0, "time": 0.35, "seconds": 1.2 }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| label  | text | required. The game's own name for the effect. Exact - it is a dictionary lookup. A bare string in the list is this and nothing else
| playsWhen | text | "always" (default) - there for the whole fight. "while" - for as long as the character has a status effect you name. "hit" - when this character's damage lands. "time" - at a moment on one of this character's animations
| motion | text | "time" only, and required there. Which animation the moment is on - S1, S2, S3, Guard, Idle...
| coin   | int  | "time" only. WHICH COIN of that motion, from 0. -1, the default, means every coin of it
| time   | number | "time" only. Seconds from the start of that coin, the same clock a sound cue uses. Past 60 is read as a typo, moved back and reported
| hitRepeat | text | "impact" (the default) or "beat". On a "hit" entry: once per hit, or once per damage beat. On a "time" entry: follow the beat's hits, or play once
| seconds | number | "hit" and "time". How long it stays up before Limbonia switches it off - a LIFETIME, not a speed: it does not make the effect play slower or stretch it out. Above 0, capped at 30. NOTHING ELSE EVER ENDS IT. **Leave it out and the effect is held until it has finished**, at whatever `speed` you set. Write one and it is a wall clock, so at half speed an effect needs twice the `seconds` to finish
| scale  | number | a multiplier on the size the game would give it. Above 0, capped at 100. Default 1
| speed  | number | how fast it plays, as a multiplier on its own rate. 0.5 is half speed, 2 is double. From 0.05 to 10, default 1. Not every effect answers - the picker says which. Does NOT change `seconds`
| centred | bool | at the character's middle instead of at their feet. Default false. Alias centered
| layer  | text | "default" (the character's effect pivot), "back" (its back pivot) or "skin" (drawn and masked with the character's own art)
| whileEffect | text | "while" only, and required there. The status effect that holds it up, by name. NOT `effect` - that key means something else in a `vfx` entry
| whileStacks | int | "while" only. How much of the status effect is needed. Floored at 1, which is the default and means "any at all"
| playsOn | text | whose character it plays on: "self" (default, the character the entry belongs to) or "target" (the one being hit). Only with `"playsWhen": "hit"` or `"time"`
| targetIndex | int | which of the units being hit, from 0. Default 0. Above the number being hit it is clamped to the last one and reported. Must name a character - unlike the `vfx` list, -1 is not accepted here
| x, y | number | where on the character to put it, in units of the character's own HEIGHT. 1 = one character tall, y up, x their right. 0 (the default) is where the game puts it. Added to `centred` rather than replacing it
| z | number | depth against the scenery, -20 to 20, default 0. Negative pulls it clear so nothing can cut into it; positive lets the background cut into it on purpose. It does not move the effect - see below
| colour | text | what colour to play it in - a hex colour, `#ff4444`, `#f44`, or with an alpha as `#ff4444cc`. The `#` is optional. Spelled `color` too. Leave it out for the effect's own colour
| borrowFrom | text | only for an effect that belongs to ANOTHER character. The character name the effect list prints beside it, such as `SD_Personality/10113_YiSang_DerSchutzeAppearance.prefab`. Leave it out for everything else, including an effect belonging to somebody already in the fight
| row    | int  | WRITTEN BY THE EDITOR, not by you. Which line of the Animation Timing timeline the bar is drawn on. The game never reads it and it changes nothing about how the effect plays. Leave it out and the editor decides. `vfx` entries take it too
| group  | text | WRITTEN BY THE EDITOR, not by you. The name of the fold this entry was put in, so several bars can be collapsed into one. The game never reads it. `vfx` entries and sound cues take it too
| enabled | bool | false leaves it in the file and switches it off

**This is the only way to put an effect on a character without shipping one.** You are not supplying art: you are
naming an effect the game already has, and the game makes it, parents it to your character, positions it, sizes it
and draws it. So a `motions` entry carrying nothing but `appearance` and `gameEffects` is a complete, working mod
with no Unity project, no `.bundle` and nothing on disk but mod.json. (It is not the only bundleless part of a mod
-- `targetGroup` and everything outside `motions` need no bundle either. It is the only one that draws a particle
effect.)

**You cannot guess a name, and you are not expected to.** The Animation Timing editor has a picker fed from the
game's own catalogue - it works at the title screen and in the lobby, and it is the whole catalogue there rather
than a part of it. It opens on the whole list and scrolls, so it can be read through as well as searched, and
**See what the game has** beside it fetches that list from the game and says how many there are. Beside the picker
is a **Try it** button, which plays the effect on a character standing in a battle right now and saves nothing. The
Mods panel lists what a mod already declares, as chips beside its always-on effects, but does not edit them.

**Kinds of entry, and the picker marks which is which.** Most effects the game keeps in its own data and can
make anywhere: any fight, any character, nothing prepared. A smaller number belong to a particular battle list -
one piece of the game's content - and only play in fights that use that content. The picker says so on the entry.
Pick one of those and press **Try it** in a fight that is not using it and Limbonia will fetch the list for you and
tell you to press it again in a moment; if it still does nothing, that effect does not travel and the answer is to
pick one the list marks as available anywhere. The same is true in play: a named effect that belongs to a battle
list is fetched the first time your mod asks for it in a fight, and reported on your mod's card if it will not come.

**The characters bring thousands more, and you can use those too.** The names above are the ones the game keeps
in a shared list, and there are only a couple of hundred of them. Every character also carries their own effects -
their auras, their glows, the flourishes their skills throw - and there are several thousand of those. They are in
the same picker, marked with the character they belong to, and they are written into `label` exactly like any
other name. Three things are true of them and of nothing else in this list:

* **They play on the character they belong to with nothing else written down.** If the effect belongs to somebody
  in the fight, just name it.
* **To put one on a different character, add `borrowFrom`** with the character name the picker prints beside the
  effect. Limbonia loads that character's artwork, copies the one effect onto yours, places it and draws it in the
  right order, and destroys the copy when the fight ends or you remove the entry. Nothing about the character it
  came from changes.
* **The picker fills them in for you.** Pick a row and both halves are written down - the effect's name and, when
  it belongs to somebody else, the character it came from.

**Every character in the game is read once, in the background.** The first time you run Limbonia it works through
the game's character list a few at a time - never blocking the game, never during a fight, and never downloading
anything - and writes down which effects each one has. The picker fills in as it goes and says how far along it
is. The answer is saved next to the game, so it happens **once**: every session after that the whole list is there
from the first second. If the game is patched, Limbonia notices that its content changed and reads it again by
itself - so the list can never go on offering effects that no longer exist.

**A multi-hit beat plays the effect once per hit.** If the damage beat your effect sits on lands five times,
the effect plays five times, spaced the way the game spaces the hits - so it reads as five hits rather than one
effect sitting over five damage numbers. You do not write the count anywhere: it is read from the beat's own
`multiHit` and `multiHitDuration`, so changing the beat changes the effect with it and the two can never disagree.

A beat that hits once plays the effect once, which is what it always did. If you want a single play on a beat that
hits several times, set `"hitRepeat": "beat"`.

**You can play them on the character being hit.** Add `"playsOn": "target"` and the effect is built on whoever
the coin reaches instead of on your own character - their pivots, their sorting, their size. `targetIndex` picks
which one when an attack reaches several, counting from 0.

This works with `"playsWhen": "hit"` and `"playsWhen": "time"`. A timed beat resolves the same character the blow
will land on, so you can put an effect on the victim part-way through the swing rather than only at the moment of
impact. It does **not** work with `"always"` or `"while"`: at those moments nothing is being hit, so there is no
target - Limbonia says so on your mod's card when you save rather than playing it in the wrong place.

If the moment arrives and nothing is being hit, the effect does not play and your mod's card says why. It is never
quietly played on your own character instead.

**Careful with `"layer": "skin"` on a target.** `skin` draws the effect masked with the character's own artwork -
and on a target that means masked with *theirs*. Anything outside their silhouette is cut away, so a large effect
borrowed onto a small enemy can be masked down to almost nothing. `skin` is also the one layer the game does not
give its own drawing order to. If an effect on the target looks missing or clipped, try `"default"` first.

**You can move them around the character.** `"x": 0.4, "y": 0.8` puts the effect up and to the right of them.
The numbers are in **character heights**, not pixels and not Unity units - `1` is one character tall - so the same
values land in the same place on a sinner and on a boss three times the size. `y` is up, `x` is their right, and
`0` is wherever the game would have put it, so an entry that sets none behaves exactly as before. About `0.2` to
`0.5` reaches a hand or a shoulder, `1` is over their head, and past `2` you are well clear of them.

`z` is **depth against the scenery**, and it does not move the effect. The fight is drawn flat, so pushing an
effect away cannot make it look further off - what it changes is whether the background **cuts into it**:

* **Negative** pulls it toward you. Nothing in the scene can cut into it. Use this if an effect looks chopped.
* **Zero** is normal.
* **Positive** pushes it back, and the scenery starts cutting pieces out of it - more the further you go. That is
  worth having on purpose: an effect rising out of the ground, or half-hidden behind something. It is also what is
  happening if an effect looks chopped in half by accident.

`z` is not how you put an effect behind the character. That is `layer`: `"back"`.

If you set `centred` as well, the offset is measured from the character's middle instead of their feet - the two
work together rather than one overriding the other.

**Effects do not all start in the same place.** Most begin at the character's feet, so `y: 0.5` puts them at head
height; some begin at the middle already, so the same `0.5` puts them half a body higher again. The offset always
adds to wherever that effect starts, so the quickest way to find your bearings on an unfamiliar effect is to press
**Try it** at `0,0` once and look.

**To judge two things against each other, rehearse them.** **Try it** plays one effect once, which is the wrong
tool for the only question that matters when you have several: do these two, 200 milliseconds apart, read as one
blow or as two? By the time you have pressed the second button the first is over. So mark what you want to compare
and Limbonia plays the whole set, on its real spacing, over and over, until you stop it.

**Sounds are part of it.** Mark a sound beat and it joins the same loop on the same clock, so a sound marked 200
milliseconds after an effect is heard 200 milliseconds after it is seen. That is the question most worth asking -
whether the noise and the flash land together - and it is why sounds and effects rehearse as one set rather than as
two things you start separately. A sound of your own or one of the game's works the same way.

Four things about it are worth knowing:

* **The earliest one is time zero.** If your effects sit at 3.2 and 3.4 seconds into a coin, the rehearsal plays
  them straight away and 200 milliseconds apart. You are judging the gap, not the wait.
* **A pass lasts as long as the set needs.** Limbonia works it out from the effects themselves - including the ones
  with no length of their own, which run to their real end at whatever speed you set. You can ask for a longer loop
  if you want a pause between passes; ask for a shorter one than the set takes and it is lengthened and you are
  told why, because a loop that restarts an effect still on screen hides the very thing you are looking at.
* **A looping game sound cannot be rehearsed.** It is refused when you mark it, with the reason, rather than
  starting something the loop could never switch off. Everything else plays to its own end and needs nothing;
  laps are made long enough that a sound never restarts on top of itself.
* **It stops by itself, and it goes quiet as well as still.** Closing the editor, leaving the battle, saving a
  change to those effects, the character going down, or ten minutes of silence all end it - and anything it was
  playing is silenced with it. Pressing **Try it** on one of the effects takes that one out of
  the set and plays it on its own, because that button has always meant "this one, once, now".

**Limbonia knows how long each effect runs, and tells you.** This is the number nothing else could give you: an
effect plays, and there is no way to see when it *ended*. Beside every entry that plays on a hit or at a moment,
the editor says how long that effect takes to finish **at the speed you set** - so `seconds` stops being a guess.
Press **Try it** and the answer comes back in the sentence under the button as well.

**Leave `seconds` out and it is held until it is done.** This is the default for a new entry and it is what most
entries want:

```json
{ "label": "FX_Fire_Burst", "playsWhen": "hit", "speed": 0.4 }
```

That plays the effect at four tenths speed and holds it for as long as it needs, which is two and a half times its
normal run. You do not work that out and you do not write it down. Change the speed and the length follows.

**Three answers, and they are not the same.** What the editor can say about an effect's length depends on the
effect:

* **a number** - it finishes, and that is how long it takes. Most effects.
* **it loops** - it never finishes. About one in five. There is no length to run to, so `seconds` is the only thing
  that will ever take it off; leave it out and you get one second. The editor says so on the entry, and so does
  your mod's card when you save.
* **not read yet** - nobody has measured it. Every one of the game's shared names is in this state until the first
  time you play it, because the copy the game makes does not exist until then. Press **Try it** once and it is
  measured. This is *not* the same as "it loops", and the editor never shows one as the other.

**Your mod's card tells you if a length cuts an effect off.** If you wrote a `seconds` that stops the effect well
before it is finished, the card says so when you save, with both numbers. It only speaks when the effect is
*visibly* cut - a tenth of a second short is not worth a sentence, and a problems list that cries wolf is one
nobody reads.

**Thirty seconds is the ceiling, even for a computed length.** If an effect slowed right down would take longer
than that to finish, it is switched off at thirty and your mod's card says why. That cap is what makes it
impossible for a coin cut short to leave something burning for the rest of the fight, and a number Limbonia worked
out for you does not get to be the thing that breaches it.

**You can colour them.** Add `"colour": "#44aaff"` to an entry and the effect plays in that colour. These effects
are drawn as a shape multiplied by a colour, which is how the game colours them in the first place, so an effect
drawn in grey becomes exactly the colour you ask for.

**Not every effect will change colour, and the list tells you which.** Some have their colour painted into the
artwork, or shift colour as they play. Those take your colour as a *shading* of what they already were - asking
for red on an orange flame gives a deeper orange, not red. And a few have nothing on them that takes a colour at
all. Beside every effect the picker says which of the three it is:

* **takes a colour** - it will play in the colour you ask for.
* **colour shades it** - it has a colour of its own and yours tints it.
* **no colour** - there is nothing on it to colour.
* **colour not known yet** - the background read has not reached it. It is not a "no"; ask again once the list has
  finished filling in, or just try it.

The marking is worked out for the effects that belong to characters. For the game's shared names - the couple of
hundred with plain names - it cannot be known in advance, so the list says **try it**: they do take a colour, and
whether it lands as a repaint or a shading is a matter of looking.

**You can play them faster or slower.** Add `"speed": 0.4` and the effect runs at four tenths of the rate it was
made to run at; `2` runs it at double. `1`, the default, is untouched. It sits beside `scale` and reads the same
way: one multiplies how big the effect is, the other how fast it is.

**`speed` and `seconds` are different things and they do not adjust for each other.** `seconds` is how long
Limbonia leaves the effect switched on, and it is measured on a wall clock whatever the speed is. So an effect at
half speed needs **twice** the `seconds` you would have written at full speed. This is deliberate: each key means
one thing, and rescaling a number you wrote would be a surprise for anybody who set both on purpose.

**The way to avoid that arithmetic is not to do it: leave `seconds` out.** Then Limbonia reads how long the effect
actually takes, divides by the speed you set, and holds it exactly that long. Nothing you wrote is reinterpreted -
there was nothing there to reinterpret. That is the recommended way to write a slowed effect, and it keeps working
if you change the speed later.

**Not every effect can be slowed down, and the list says which.** An effect answers to `speed` if it is made of
particles or is driven by an animation - which is most of them, and is why the picker does not repeat it on every
row. What it *does* mark is the exception: a character's effect that is neither shows **cannot be slowed**, and a
`speed` on that one does nothing. The entry's own Speed control says the whole answer for the effect you picked,
including "not looked at yet" when the background read has not reached it - which is not a "no".

The game's shared names - the couple of hundred with plain names - are not read in advance, so they always say not
looked at yet. Nearly all of them are particle effects and answer perfectly well; try one and see.

**On a character's OWN effect the game can take the speed back.** Every effect a character was built with is held
by the game's own speed control, which it drives for hit-stops, duel markers and deaths - and each of those writes
every one of those effects back to the game's speed, not to yours. So a slow effect on your own character may
snap back to normal when the fight does one of those things. **A borrowed effect is not affected**: it was made
after the character was built, so nothing but your entry ever touches its speed. If a slow-motion moment has to
hold through a hit-stop, borrow the effect rather than naming your own character's.

**Colour works on every effect you can play** - a character's own, a borrowed one, and the game's shared names
alike. The game gives each character its own copy of an effect rather than passing one object around, so your
colour only ever lands on the copy your character is wearing; nobody else's looks different, and the effect goes
back to its own colour the moment your entry stops.

**Borrowing needs that character's artwork already on your machine.** Limbonia will not start a download because a
mod file mentioned a name, and the background read skips characters whose art has never been downloaded - so they
are simply not in the list. If you name one anyway you get a sentence saying so on your mod's card; play one fight
the character appears in and it will be.

**Browse as many characters as you like.** A character's artwork is held only while something is actually using
it - an effect standing on somebody, an entry that will borrow it again, or the one you are listening to right
now. The moment nothing is, it is given back, so trying twenty characters in a row costs twenty brief loads rather
than twenty held at once. Everything borrowed is given back when the fight ends regardless. There is still a
ceiling on how many can be held *at the same time* - thirty - but reaching it means thirty characters' effects
genuinely in use at once, not thirty you happened to look at.

**Effects the game drives from a status effect are named by NUMBER.** A few of the shared effects are the ones the
game raises for its own status effects, and those carry no name at all - the game addresses them by the status
effect's number. The picker shows them with the effect they play and gives you the number to write in `label`, so
a `label` that is all digits is one of these. They behave like any other entry in every other way.

**What is still NOT in this list.** Characters whose artwork has never been downloaded to your machine are not
read, so their effects are not offered - playing one fight they appear in fixes that permanently. And the exact
FRAME the game fires one of its own flourishes on is still the game's: you can play an effect at a moment you
choose with `"playsWhen": "time"`, but you cannot ask for "whenever the game would have played this one".

**Two keys in your file are not yours: `row` and `group`.** The Animation Timing editor lets you drag a bar onto
whichever line of the timeline you want, and fold several bars into one you can open - and it remembers both by
writing them into the entry. That is the whole of what they mean. They say nothing about when anything plays, what
it plays on, or in what order anything happens, and Limbonia throws them away as soon as it has read them - the
game never sees either. `row` appears on `gameEffects` and `vfx` entries. `group` appears on **everything you can
place on the timeline** - effects, the character's own effects, sounds, speech bubbles, cut-ins, camera shakes,
zooms, turns, movements, picture changes, form changes and hits. A folded hit still shows where it lands: the
group's own bar carries a mark at the moment damage arrives, so the thing every other beat is timed against stays
visible even while the fold is shut.

**Folding a hit is the one fold with a consequence.** Everything else you can put in a group is presentation - it
decides what is seen and heard, not what happens. A hit is not: it is when damage actually lands. So moving a fold
that contains one moves the hit with it, and the attack really does connect at a different moment. That is usually
exactly what you want when you fold a whole attack and slide it, but it is worth knowing before you do it.

**A fold's name belongs to its coin.** "Flurry" on S2 coin 1 and "Flurry" on S3 are two different folds and never
merge, however alike the names look. Names are not global, and the editor is what keeps them apart - Limbonia does
not check it and does not need to.

If you have never dragged or folded a bar you will never see either key, and deleting them all by hand does nothing
worse than letting the editor lay everything out unfolded again.

**Why this is not a `vfx` entry with a third source.** A `vfx` entry lives inside an `assets` declaration, so it
needs a `TimelineAsset` in a bundle before you may even name an effect - which is exactly the dependency this
removes. And every knob a `vfx` entry has belongs to machinery this does not use: `duration`, `once`,
`hide` and `activeTime` describe a clip on an animation of yours; `x`, `y`, `z`, `disableAfter`, `sortingOffset`,
`playsOn` and `targetIndex` describe a prefab of yours that Limbonia drives. Here the game owns everything. Writing
any of those keys on a `gameEffects` entry is REPORTED and ignored rather than silently accepted. Note that `effect` is on that list: in a `vfx` entry it means one of the character's OWN visual effects, and it is not how you name a status effect here - that key is `whileEffect`. A `time` is the
one thing both lists want, and it is not on that list -- but here it is written with the animation and coin beside
it, because this entry does not sit on a coin's declaration the way a `vfx` entry does. That is the next part.

**A moment on a coin, and it still needs nothing of yours.** `"playsWhen": "time"` switches the effect on `time`
seconds into coin `coin` of `motion`, on the character's own playback clock -- the same clock a sound cue, a speech
bubble and a camera shake are placed against. It works on a coin you supplied NO animation for: the game says
which motion and coin every character is playing whether or not a mod is involved. So "the game's own fire burst
0.35 seconds into this character's S2" is a complete mod with nothing on disk but mod.json.

**A timed entry has to name a moment that exists, and a bad one is dropped rather than turned into an always-on.**
No `motion`, a `motion` that is not one of this character's animations, or a `time` that is not a number of
seconds: each is refused by name and the entry does not play. That is deliberate -- quietly falling back to "for
the whole fight" would leave you watching an effect that never goes off, with nothing anywhere saying why. Leave
`coin` out to put it on every coin of that motion.

**An address on an entry that does not play at a moment does nothing, and you are told.** `motion`, `coin` and
`time` on an `"always"` or `"hit"` entry are read only so they can be reported: it is exactly what somebody writes
when they meant `"playsWhen": "time"` and forgot to say so, and it is the difference between one sentence and an
afternoon.

**One name is one effect per character, at one moment.** The game keeps a single copy of each label on each
character. Naming the same label twice in the SAME place is two switches fighting over one object, so the second
is dropped and you are told. Naming it at DIFFERENT moments is fine and is the ordinary thing to write across a
multi-coin skill -- asking again for an effect that is already standing re-triggers it, which is what a second
placement means.

**An effect that follows a status effect.** `"playsWhen": "while"` keeps the effect up for exactly as long as
the character has the status effect named in `whileEffect`, and takes it off again when they lose it. Nothing
else is needed: no moment, no length, no animation and no bundle. *While Ishmael is Bleeding, she smoulders* is
one entry.

```json
"gameEffects": [
  { "label": "EFFECT_BLOOD", "playsWhen": "while", "whileEffect": "Laceration" },
  { "label": "EFFECT_FIRE", "playsWhen": "while", "whileEffect": "Burn", "whileStacks": 5 }
]
```

`whileEffect` is a status effect by name - the game's own name for it, or one your own mod adds. Capitals and
spaces do not matter; the spelling does. `whileStacks` is how much of it is needed before the effect appears,
and leaving it out means "as soon as they have any at all". An entry set to `"while"` that does not name an
effect is refused by name and dropped, for the same reason a bad `"time"` entry is: an aura you meant to
follow Bleed, silently glowing all fight, is worse than one that does not appear and says why.

**It is checked, not announced, and that is what makes it right.** Limbonia asks twice a second whether the
character has the effect right now, rather than watching for the moment it is applied. That sounds like the
long way round and is the only one that works: a round is worked out in one burst the instant you confirm it and
then played back over the next half-minute, so "the moment Bleed was applied" happens up to half a minute before
the blow you are watching. Asking about the state instead means the effect is right at every instant by
construction - and it comes off correctly however the status effect leaves, whether it ran out, was cleansed,
was despelled, or the wave ended. What you give up is precision finer than about half a second on the switch,
which on something that stays up is not something anybody can see.

**`seconds` does nothing on one of these**, and you are told if you write one. It ends when the status effect
does, and by no other route.

**Naming a status effect the game has never heard of** is reported against the mod once, the first time the
check runs, rather than every half-second. Until the game has finished reading its own effect list - a few
seconds after launch - a name that has not resolved yet is left alone rather than reported, so an early miss is
never mistaken for your spelling.

**`seconds` is a lifetime, not a speed.** It says how long the effect is left on screen before Limbonia switches
it off. It does not slow the effect down or stretch it out - the effect plays at its own rate the whole time, and
`seconds` only decides when it stops. A short-lived effect with a long `seconds` finishes and sits there; a long
one with a short `seconds` is cut off part way.

**What ends it, and why `seconds` is not on the animation's clock.** A `"hit"` or `"time"` effect is switched off
`seconds` after the moment that started it - and if Limbonia did not do that, nothing would, because the game's
own switch is a plain on/off with no timer. That countdown runs on a real clock rather than on the animation's,
deliberately: a coin that is interrupted, a character that dies mid-swing, a wave that ends, and the animation's
clock simply stops where it is -- so an "off" scheduled at a moment on that clock is an off that never arrives,
and the effect burns for the rest of the fight. The cost is that a paused or loading fight keeps counting, which
at the length a burst lives at nobody can see. An `"always"` effect ends with the fight instead: it hangs off the
character, so it goes when the character does. A coin ending does nothing to any of them. Deleting the entry, or
switching the mod off, takes the effect off the character on the next save rather than at the next battle.

**A character that has died refuses new effects.** That is the game's own rule and Limbonia keeps it, so an effect
asked for after the character is down simply does not appear - and it is not reported as a fault.

**Telling a wrong name from an effect you cannot see.** If the name resolves nowhere the game makes nothing at
all, and this mod's card says *the game has no effect called "..."*. If the card says nothing, the name is right
and the effect drew nothing where it was put - try `"layer": "skin"` or a larger `scale`.

# CUTIN
- cutIns[] - inside one assets entry. The full-screen splash, with your own picture, at a moment you choose on
  one coin.

```json
"cutIns": [
  { "time":    0.35,
    "image":   "art/awaken.png",
    "style":   "awaken",
    "facing":  "right",
    "color":   [3.0, 0.6, 0.2],
    "seconds": 2.0,
    "once":    true }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| time   | number | required in practice. Seconds from the start of this coin, the same clock a sound cue uses. Default 0
| image  | path   | required. A PNG in your mod's folder. There is nothing else to put on screen, so without one the cut-in is dropped and you are told
| style  | text   | "awaken" (default), "erosion", "unstable", "stable", "upgrade". Decides where the picture slides in from and how far it is zoomed
| facing | text   | "right" (default) or "left". The same picture, flipped
| color  | []     | the tint, as [red, green, blue]. NOT 0-to-1 - see below. A fourth number is alpha
| seconds| number | how long before it comes off the screen, 0.3..10. Default 2. NOTHING ELSE EVER ENDS IT
| once   | bool   | true (default) = once per BATTLE. false = every time this coin plays. See below
| enabled| bool   | false leaves it in the file and switches it off

**It is a beat, and it goes where you put it.** `cutIns` sits beside `cues`, `speech`, `camera` and `move` on an
`assets` entry - one coin of one skill - and it comes due on the character's own playback clock, which is the
animation you are watching.

**The picture needs no Unity, but the coin does.** The splash, the slide, the blur behind it and the tint are all
the game's, and the only thing you supply is one PNG read straight off disk. What you cannot do is put one on a
coin you supply no animation for: a beat lives on an `assets` entry, and an `assets` entry needs the bundle its
animation comes out of.

**`once` means once per BATTLE.** A beat already fires at most once per play of the coin it sits on - that is true
of every beat here - so that is not what the word is for. What it answers is the next question out: does the splash
happen again the next time this character uses this skill? On (the default), once a fight. Off, every time that
coin plays. Two cut-ins on two different coins are two separate promises; one does not use up the other's.

**Only one splash at a time.** The game has a single cut-in canvas, so a cut-in raised while another is still up
takes the first one down rather than queueing behind it. Putting two on one coin a tenth of a second apart is one
slide restarting, not two cut-ins.

**The colour is not 0-to-1.** The game's own cut-ins write values as high as **5** into this property - it is an
emissive tint, not a paint colour - so `[1, 1, 1]` is a fairly dim white and `[3, 0.6, 0.2]` is a bright orange.
Anybody who has set a colour anywhere else gets this wrong first time. The list has to be exactly three numbers or
exactly four; any other length is dropped whole and the splash draws untinted. Each number is capped at 16, which is
well past anything the game itself writes.

**What `style` actually changes.** Two things, both visible. It picks which of the game's own cut-in settings are
read, which is where the picture slides in from, where it settles and how far it is zoomed. And `unstable` and
`stable` additionally play the game's overclock overlay on top of the splash; `erosion` uses the same geometry
without it. If the game has no settings for the style you asked for, the mod's card says so and no splash goes up.

**`style` and `facing` are matched letter for letter, and a word that is not one of the listed ones is taken as the
default in silence.** `"Awaken"` is not `"awaken"`, and anything that is not exactly `"left"` means right. These two
are the only keys here with no sentence on the mod's card when they are wrong, so check the spelling before you go
looking for a reason the splash came in from the wrong side.

**What takes it down.** Limbonia, and only Limbonia. The game's own flow closes a cut-in explicitly from a place a
mod never reaches, so nothing would ever close yours - a splash left up is indistinguishable from a broken game.
`seconds` is therefore clamped rather than trusted: below 0.3 it is a flicker (the slide-in alone takes a full
second) and above 10 a mistake looks exactly like the game having frozen, so a number outside that range is pulled
to whichever end it is nearer, and something that is not a number at all becomes the default 2. It also comes down
when the battle ends, when the mods are reloaded, whenever any beat is edited or deleted and saved, and when a
second cut-in goes up.

**Authoring it.** The Animation Timing editor has a **Cut-in** lane, under Form and above the camera lanes.
Double-click the lane to place one, drag the bar to move it and drag its right-hand edge to set `seconds` - the bar
is the splash, so its width is how long it is up. The inspector card underneath carries the picture, the style, the
facing, the tint and `once`.

**Trying it.** That card has a **Show it now** button, which raises the splash over whatever battle is running, and
a **Take it off** beside it. It shows the beat **on screen**, not
the one you last saved, so only the PNG has to be on disk already. If the splash appeared but showed nothing, the
picture is transparent or is not the file you meant; if nothing appeared at all, there was no battle to put it
over; and if this mod's card says the picture could not be read, it is not a PNG or not where the path says.

**Telling "it never fired" from "the picture did not load".** These are the two failures that look identical from
the outside, so they are reported differently. A picture that could not be read is named on this mod's card, by
file, before the fight ever starts - the splash goes up and shows nothing, or does not go up at all, and the card
says which. A beat that never came due says nothing anywhere, because nothing went wrong: check that the time is
before the coin actually ends (the editor greys a beat that sits past it), and that the coin you put it on is one
the skill really throws.

# SPEECH
- speech[] - inside one assets entry. What the character SAYS on this coin.

```json
"speech": [
  { "time": 0.2, "text": "Stand aside.", "seconds": 1.8 },
  { "time": 1.6, "text": "<color=#ff5555>Burn.</color>" }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| time   | number | seconds from the start of the coin, the same clock a sound cue uses. Default 0
| text   | text | required. What the character says
| seconds | number | how long the bubble stays up, 0.05..60. Alias duration. Default 2

The text is genuinely arbitrary -- it does not go through any voice table, so it is not restricted to lines the
character has. It lands in a TextMeshPro label, so `<b>bold</b>` and `<color=#ff0000>colour</color>` really work; a
`<` with no matching `>` swallows the rest of the line, and you are warned whenever the text contains one. Tabs and
control characters are stripped and reported; line breaks are fine. A bubble grows with its text, so a long one
covers the battlefield -- past the limit it is shortened and you are told to split it into two bubbles at two times.

**Unlike an effect, this needs nothing from the animation.** A bubble is not a clip on a timeline, so it does not
require the coin to declare `totalDuration` and it works on every coin and on both ways of drawing. The default of
2 seconds is deliberately shorter than the game's own 6, which outlasts most attack coins and would leave the line
hanging over the next action.

# PICTURE
- picture[] - inside one assets entry. Swap which animation is DRAWN, partway through one coin.

```json
"picture": [
  { "time": 0.6, "asset": "s3_hold", "frame": 4, "onEnd": "loop", "until": 1.2 },
  { "time": 1.2, "hold": true },
  { "time": 1.5, "asset": "" }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| time   | number | required, seconds from the start of the attack, 0..3600
| asset  | text | which animation to show. An EMPTY name means "back to this coin's own", which is how you change back
| start  | number | where to start inside that animation, in seconds
| frame  | int  | the same thing as a drawing number, counted from 1. Needs the mod's `fps`, so build in Unity first
| hold   | bool | freeze rather than play
| until  | number | when this change stops being in force. Omitted = until the next change
| onEnd  | text | "hold" (default) or "loop". "stop" is refused - add another change instead
| enabled | bool | false switches this one change off

**Times here are ALWAYS seconds, even on a fraction coin.** `times: "fraction"` does not reach this list. A picture
change has no counterpart in the other motion format, so nothing can ever be pasted in that expresses one as a
fraction -- while writing the obvious `"time": 6.0` on a 22-second fraction coin would put it at 132 seconds.

**A frame number needs the animation's frame rate, and it is never guessed.** The Unity build writes `fps` and
`frames` onto each `assets` entry; without them a frame number cannot be turned into a moment and you are told to
rebuild. Twelve a second is the toolkit's default, but any one animation may have been built at another rate, and
picking the common one would put the picture on a neighbouring drawing with nothing saying so. Asking for a frame past
the end uses the last one and reports it.

**Two clips cannot have a picture change: a pose, and one that owns its own schedule.** A pose is played through the
Animator and has no attack clock to hang a moment on. A clip that carries a `timing` block, or `"sync": "own"`, is the
clip everything else on the attack is timed against -- re-anchoring it partway through would wind that clock backwards
and replay the attack's sounds and camera work. Both are refused at scan time with the reason, so there is nothing for
the runtime to act on.

The animation a change names must be declared in `assets` like any other. Two changes at the same instant means only
the last one written can happen, and you are told which. Sixty-four changes on one coin is the ceiling.

# FORMS
- a `form` name on an assets entry, and a becomes[] list inside one. A second set of your own drawings that the
  character turns into mid-fight and stays.

```json
"assets": [
  { "name": "Doro_Idle", "motion": "Idle" },
  { "name": "Doro_S1",   "motion": "S1", "index": 0,
    "becomes": [ { "time": 6.0, "form": "transformation1" } ] },

  { "name": "Doro_T1_Idle", "motion": "Idle", "form": "transformation1" },
  { "name": "Doro_T1_S1",   "motion": "S1", "index": 0, "form": "transformation1" }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| form   | text | ON AN ASSETS ENTRY: which set of drawings it belongs to. Left out = the base form, the character as you normally draw it
| becomes| []   | ON AN ASSETS ENTRY: the moments this coin changes the character's form
| becomes[].time | number | required, seconds from the start of the attack, 0..3600
| becomes[].form | text | which form to become. An EMPTY name (or none at all) means "back to normal", which is how you change back
| becomes[].enabled | bool | false switches this one change off

**A form is made by TAGGING, not by declaring.** There is no list of forms anywhere. Draw the second version of
your character, add its animations as ordinary `assets` entries, and give each of them the same `"form"` name -- every
animation carrying that name IS that form. Nothing else creates one, which is why a `becomes` naming a form nothing is
tagged with is reported and ignored: it could never draw anything, and the sentence lists the forms that do exist. A
mod with no tagged animation at all gets a different sentence saying so, rather than one refusal per change. Only an
animation makes a form: a `form` on a prefab, a texture or anything else that is not a `TimelineAsset` creates
nothing. And `form` has to be text -- a number or a list there drops that whole animation declaration, not just the
tag.

**A form is DRAWINGS AND NOTHING ELSE.** Same bundle, same `spritePath`, same `parent`, same `sortingOffset`, same
`hide` settings, same character entry -- one `motions` entry per character, always. The hits, the sounds, the camera
work, the movement and the timing belong to the COIN, and are taken from the untagged animation the form's one stands
in for. A tagged entry that also carries `totalDuration`, `timing`, `phases`, `hitCheckers`, `endCheck`, `cues`,
`speech`, `vfx`, `camera`, `move`, `hideDefaultImpact` or `showDamageCounter` is reported by name and those keys are
ignored. It is not a restriction for tidiness: a form entry is a SECOND declaration for a motion and coin you already
declare, so two of anything on that list would fire twice or overwrite each other, and which won would come down to
the order of the lines in your file.

**What a tagged entry CAN still carry** is everything that is about the picture: `picture`, `loop`, `sync` and its own
`becomes`. `cutIns` is the one exception with no sentence attached -- a cut-in on a form entry is dropped in silence,
so put it on the untagged animation for that coin.

**The form's animation is picked up at the same instant of the attack, not from its own start.** A change at 6
seconds shows the transformed animation 6 seconds in. It is the same coin with the same hit at the same moment, drawn
differently, so the two have to stay in step -- starting it over would rewind the attack under a hit that has already
been scheduled. Author the transformation itself into the END of the untagged animation and let the form's one carry
on from there. If what you want really is a second animation played from ITS beginning at a moment, that is a PICTURE
change, and the two work together on the same coin.

**What a form does not cover falls back to your ordinary animation, and that is the design.** Limbonia looks for the
current form's animation for a motion first and takes the base one when there is none -- so a form that redraws an idle
and one skill is a complete, working form. A motion NEITHER covers still reaches the pack's `fallback` exactly as
before. Each gap is reported once per character, naming the form and the motion, so you can see which ones you might
want to draw -- and once more per body that character wears, so an E.G.O. or a boss phase can repeat the sentence.

**IT LASTS FOR THE REST OF THE BATTLE, and survives things that look like they should end it.** The form is remembered
against the CHARACTER, not against the body it is currently wearing -- so an E.G.O., a wave transition and a boss
changing phase all rebuild the character and it comes back still transformed. It goes back to normal when the battle
ends, or when you reload your mods.

**Changing back is the same thing written the other way.** `{ "time": 20.0, "form": "" }` -- or the same entry with no
`form` at all. "Transform at 6s, change back at 20s" is two entries in one `becomes` list, exactly as it is on a
PICTURE change.

**The same two clips cannot carry one that PICTURE cannot: a pose, and one that owns its own schedule.** A pose is
played through the Animator and has no attack clock to hang a moment on; a clip carrying a `timing` block or
`"sync": "own"` is the clip everything else on the attack is timed against, and swapping what is drawn underneath it
would wind that clock back and replay the attack's sounds and camera work. Both are refused at scan time with the
reason. Watch the second one at the top of the file too: `"sync": "own"` written next to `bundle` applies to every
animation in the entry, so it refuses every `becomes` in the mod at once. A change written on an animation that is
ITSELF part of the form it names is refused too -- the character is already that when it plays, so it could never do
anything; put it on the animation that plays BEFORE the change.

Times here are always seconds, on the same clock as `picture` and `cues`, even on a coin that declares
`"times": "fraction"`. Two changes at the same instant means only the last one written can happen, and the time is
reported. Sixteen form changes on one coin is the ceiling; past it they are counted as changes that could not be
read rather than named individually.

**A script can put a character into a form too**, with the same names your `assets` tags declare -- that is how
"transform when this blow kills something" is written, which no time on a coin can say. A `becomes` beat that is
currently in force wins: it is re-applied for as long as its coin is playing, so it overwrites a form a script set
during that coin. See SCRIPT.

# TARGETGROUP
- targetGroup[] - inside a motions entry. Where the units you are hitting stand.

```json
"targetGroup": [
  { "motion": "S3", "distance": 2.5, "spread": 1.2, "moveMainTarget": true },
  { "spread": 0.8 }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| motion | text | which skill. LEAVE IT OUT to use one set of settings for every skill this character has
| distance | number | how close to the attacker the group is placed
| spread | number | how far apart the units are from one another
| moveMainTarget | bool | whether the main target is drawn to the attacker at all
| enabled | bool | false switches the entry off

Per SKILL, not per coin -- the arrangement happens once before the first coin plays, which is why this sits beside
`assets` rather than inside one. A lone object is read as one entry, so you do not need the list brackets for a single
rule. A named entry beats an unnamed one outright rather than merging with it, and two entries for the same skill
means the first wins and the rest are ignored -- reported, so you can see which.

This needs no bundle either. It moves units the game already has, so a `motions` entry carrying nothing but
`appearance` and `targetGroup` is complete on its own -- the same way one carrying only `gameEffects` is. Name
`assets` or `alwaysOn` without a `bundle`, though, and the entry is refused and told why: both of those name things
your mod SHIPS, and there is no archive to take them out of.

**A negative is a real answer here, not an absence.** The game reads a negative distance or spacing as "use this
character's own", so writing one deliberately is how you take a per-skill script's hard-coded number back to the
default. Anything you do not write at all is forwarded exactly as the game passed it.

**`spread` does nothing on a single-target skill.** The spreading step sits behind the game's own "more than one
target" check, while the other two do not. An entry that sets none of the three is reported: it reads as though the
arrangement had been taken over and it does nothing whatever.

# BUFFS
- buffs[] - top level. One entry describes one status effect completely: what it is AND what it is called.

```json
"buffs": [
  {
    "buff": "emberdebt",
    "new": true,
    "basedOn": "Laceration",
    "type": "negative",
    "maxStack": 10,
    "maxTurn": 3,
    "canBeDespelled": true,
    "icon": "icons/emberdebt.png",
    "sound": "emberdebt",
    "floatingTextSize": 1,
    "floatingTextColor": "#ff7a3c",
    "name": "Ember Debt",
    "desc": "Take {0} damage at the start of this unit's turn.",
    "keywordDesc": "At the start of the turn, deals damage equal to the stack count.",
    "flavor": "It burns for what was borrowed."
  },
  { "buff": "Laceration", "name": "Bleed", "desc": "Loses {0} HP on hit.", "keywordDesc": "..." }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| buff   | text | required. The name the effect is known by, internally. Aliases buffKey, effect, statusEffect
| new    | bool | true = ADD this effect to the game. See below - without this or basedOn, the entry is only a rename
| basedOn| text | which existing effect to shape it from. Also opts the entry in to being added. Left out on an effect that declares a stack or turn count, it is shaped from Bleed
| type   | text | "positive" / "negative" / "neutral". Also good/bad/neutral, buff/debuff. Decides despel and which way a give-multiplier moves it
| maxStack | int | how many stacks one character may hold. 0 or more. Omitted = keep the based-on effect's. Above 0 also decides what the effect is shaped from when there is no basedOn
| maxTurn  | int | how many turns it may last. 0 or more. Omitted = keep the based-on effect's. Only means something on an effect shaped from one that counts - see below
| canBeDespelled | bool | whether a "remove one effect" catches it
| icon   | path or text | a .png/.jpg your mod ships, OR the name of a picture the game already has. Explicit forms: iconFile, iconId
| typoIcon | text | the little symbol beside the line that floats up. Omitted = the based-on effect's
| sound  | path or text | heard when the effect lands. A file path, or a name from your `audio` list. Omitted = SILENT
| floatingTextSize | number | how big the line that floats up is, as a multiplier. The game draws its own effects at 0.75, and Bleed and Burn at 1. No upper limit
| floatingTextColor | text | the colour of that line - "#ff4444". Omitted = the colour the game chooses. Also floatingTextColour
| name   | text | what it is called - on its icon popup, inside "[Bleed]" in skill text, and in the keyword glossary
| desc   | text | what it is doing to THIS character, on the icon popup in battle. Keep "{0}" and "{1}" - the game fills the numbers in
| summary | text | the short form beside the icon. Numbers filled in here too
| flavor | text | the flavour line on the icon popup
| keywordDesc | text | what the effect does in general, in the keyword glossary. The game's own wording here has no numbers in it
| keywordFlavor | text | the flavour line in the glossary

**Adding an effect is opt-in, and it has to be.** This list is shared with the renaming half of the mod system, and
by far the commonest entry in it renames an effect the game already has. So an entry only counts as a NEW effect when
it says `"new": true`, or names a `basedOn`. Without that rule, renaming Bleed would silently append a second, empty
Bleed to the catalogue. Saying both is fine and common -- `new` adds it, `basedOn` decides what it behaves like -- and
you are told so rather than left wondering which won.

**`basedOn` is the game's own key, not the name on screen, and they differ far more often than anyone expects.**
Bleed is `Laceration`. Tremor is `Vibration`. Rupture is `Burst`. Burn is `Combustion`. It is matched exactly,
character for character, so "Bleeding" matches nothing at all. The status-effect list in the companion shows each
effect's real key -- read it there rather than guessing from the tooltip. The same is true of `buff` itself when you
are renaming a vanilla effect.

**A declared effect does nothing on its own.** It arrives with an empty ability list: the game gives you the stack and
turn counters, the expiry, the despel rule, the row on the status bar, the tooltip and the icon, all of it real, and
none of it MEANS anything until a script says what it does. See SCRIPT.

**WHAT AN EFFECT IS SHAPED FROM IS WHAT DECIDES WHETHER IT CAN COUNT, and it matters more than the numbers.** The
game does not have one kind of status effect. Bleed and its relatives are the counting kind: they carry a stack, they
carry a turn count beside it, they tick down and they wear off. Plenty of the game's other effects are not that at
all -- they hold a stack and nothing else, they never count down, and nothing draws a number of turns beside them.
Which one an effect is comes entirely from what it was shaped from, and no number you write can change it: an effect
of the non-counting kind cannot be given a single turn, whatever `maxTurn` says, because the game never asks that
question of it.

**So an entry that declares a count and names no `basedOn` is shaped from Bleed for you, and you are told so.**
Writing `"maxStack": 10` or `"maxTurn": 3` is you saying what kind of effect this is; taking that at its word is the
only reading that can work. If you want a different one, name it: `basedOn` always wins. Two entries are deliberately
left alone -- one that declares nothing at all, and one that says `"maxStack": 0`, which is you saying outright that
the effect does not stack. Those are still shaped from the game's own list, and the first of them is advised to name
a `basedOn`.

**`"maxStack": 0` is read first and settles the question on its own, which is the one trap here.** Write it with a
turn count beside it -- `{ "maxStack": 0, "maxTurn": 5 }` -- and the entry is still left alone, so the turns you
asked for land on an effect that may have no idea what a turn is, with nothing said about it. If you want a count of
any kind, either give `maxStack` a real number or name the `basedOn` yourself.

**Changing `basedOn` on an effect that is already in the game takes a restart.** An effect is added to the game's
list once and never removed while the game is running -- taking it away would leave whatever is carrying it pointing
at nothing. The limits are re-read from your file as you edit it, but the SHAPE is fixed at the moment it was added.
Rescanning after a change of `basedOn` therefore looks like nothing happened; restart and it is right.

**The two ceilings are enforced by different things.** `maxStack` is handed to the game -- written into the effect's
own row and clamped by the identical code that clamps Bleed, on every path, for every shape. `maxTurn` reaches the
game only on the counting kind, so Limbonia truncates against it as well when the effect is applied; that way the
number you wrote means the same thing either way. Both are always written down, so anything inspecting the effect
reads what you declared.

Then there is a third limit you did not write. Some effects carry a TEAM-WIDE total, and an effect shaped from one
inherits it -- and the game checks that FIRST. Declare `"maxStack": 20` on something based on an effect whose whole
side is limited to 5 between them and one character will not get past 5, while every report correctly says 20, because
20 is what you declared. You are told, by name, which effect brought the limit in; base it on a different one if that
is not what you wanted.

**`desc` and `keywordDesc` are the two most confusable keys here, and getting it wrong is silent.** They are different
sentences in different tables: `desc` is the box on the effect's icon during a fight -- the one a player actually
looks at -- and `keywordDesc` is the general rule in the keyword glossary. Set one and not the other and the missing
one reads "Unknown" or the game's placeholder, with everything else looking perfect. You are warned either way round.
`name` is the one key both places want, so it is written to both for you.

**`icon` is one key that takes two different things, told apart by the file extension.** Ending in .png/.jpg/.jpeg
means a picture your mod ships, relative to its folder, 128x128 -- and that one line is the whole declaration, no
`art` entry needed. Anything else is the name of a picture the game already has, so `"icon": "Laceration"` borrows
Bleed's. A path with no extension on the end is the trap: it is read as a picture name, matches nothing, and the
effect quietly draws with the picture of whatever it is based on. That case is reported. Leave `icon` out entirely and
it inherits the based-on effect's picture, which is the reliable answer and needs no file.

**`sound` deliberately does NOT follow that rule.** Leave it out and the effect is silent. An inherited picture is
help; an inherited sound plays in the middle of a fight, appears in no file you wrote, and there is nothing to search
for. Told apart by extension the same way: `"audio/emberdebt.wav"` is a file, `"emberdebt"` is a sound your `audio`
list named -- and every .wav and .ogg under the mod's `audio` folder is registered whether you declared it or not.
Naming it rather than writing the path survives the folder being renamed, and lets you set a `volume` once.

**The line that floats up when the effect lands can be given a size and a colour, and both are optional.** The game
draws that line for its own effects too, and it singles out eleven of them -- each gets a colour of its own at full
size. Everything else in the game, all sixteen hundred or so, takes one flat colour at 0.75 size, and so does an
effect you declare. That is why both keys are off by default: left out, your effect looks exactly like an ordinary
one of the game's, which is usually what you want.

`floatingTextSize` is a **multiplier**, not a point size. 0.75 is what the game uses for almost everything, 1 is what
it gives Bleed and Burn. **There is no upper limit** -- ask for 5 and you get 5. The only sizes refused are ones that
cannot mean anything, a zero or a negative, and you are told when that happens rather than left with a line that never
changed.

`floatingTextColor` is a colour written the way you would write one anywhere else: `"#ff4444"`. The short form
`"#f44"` works, eight digits set how see-through it is -- `"#ff4444cc"` -- and the short form takes that fourth
digit too, `"#f44c"`. The `#` is optional. There is no range on it. **A colour that cannot be read is refused out
loud**, with the reason -- which character was wrong, or how many digits it found -- in the mod's problem list; it
is never quietly ignored.

`type` is **not** secretly the colour, and it never was. The game picks the colour from the effect's own identity, not
from whether it is good or bad, so there is no positive-is-blue convention for a declared effect to inherit -- pick the
colour you want. (`type` still decides whether a "remove one effect" catches it, and which way a give-multiplier moves
it, so it is far from decorative.)

Both apply wherever the game draws the line -- during a fight, at the start of one, when it is applied on spawn. You do
not have to do anything for that to be true.

On the **Mod Author** screen both sit under **The line that floats up when it lands**, on the status-effect page,
beside the icon and the sound -- with a colour picker and a preview showing your line next to one the game's own size,
so you can see what you are choosing before you play a round. (They used to be filed at the bottom of the collapsed
"Wording, and the rest" panel, which is why nobody found them.)

`syncOnBossRaid` is refused if you write it: an effect a mod adds is always kept out of the Boss Raid record.

Switching a mod off does not delete its effects from the game -- a counter already on a unit points straight at the
row, so taking it away mid-battle would crash. The effect stops MEANING anything and the counter ticks down and
expires on its own.

# SCRIPT
- script - top level. A .lua file inside this mod's folder.

```json
{
  "appearance": 10101,
  "script": "scripts/behaviour.lua"
}
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| script | path | a .lua file inside the mod. A folder may be part of it. No "..", no path out of the mod

There is one key and it is a plain string. No default name is picked up, so a .lua file sitting in the folder with
nothing pointing at it never runs. A folder may be part of the name -- **Start a script** puts a new one in the mod's
`scripts` folder and writes that path here -- while anything that would point OUT of the mod is refused outright and
told so: a full path off your own disk, a drive letter, a leading slash, a `..`. A missing file, an unreadable one, or
one too large to be a behaviour file is reported against the mod rather than failing quietly. The script gets the
character from the top-level declaration, so it does not name one itself.

**A mod has ONE script slot, so pointing the key at another file REPLACES the one that runs.** There is no second
slot and nothing is added: the file that was named stays where it is in the folder and simply stops being read.
That is also what "stop running it" does -- it clears the key and leaves your work on disk.

**Move the file and the key does not follow it.** If mod.json still says `behaviour.lua` and the file is now at
`scripts/behaviour.lua`, nothing runs and the mod's problem list reports no such file. The **Behaviour script** field
looks for it before saying that: a file of the same name anywhere else in the mod is offered as "point this mod at
it", which rewrites the key and leaves your file exactly where you put it. If more than one file answers to that
name you are shown all of them and pick; if there is genuinely nothing of that name in the mod, you get the old
answer, which is the one case where starting a new script under that name is right.

**Not every build runs them.** Scripting is decided when Limbonia is compiled, so it is either in your copy or it is
not there at all. The way to tell is the companion: when scripts are in, there is a **Scripts** panel. When they are
not, a mod that ships a `.lua` still loads and everything else in it works -- and the mod's own list of problems
carries one sentence saying the script will never be read, so it does not fail in silence. Check for the panel
before you write one.

What a script does is answer questions a declaration cannot ask. There are four ways to register a handler, and the
word you use is what decides the kind of answer you are allowed to give:

```lua
on(TIMING.BEFORE_GIVE_ATTACK,      function(e) log(e.damage) end)
contribute(DECISION.ATTACK_DAMAGE, function(e) return e.damage + 5 end)
gate(DECISION.RECOVER_SP,          function(e) if e.unit.hp < 20 then return false end end)
veto(DECISION.BREAK,               function(e) return e.unit:hasBuff("Laceration") end)
on(TIMING.ON_KILL_TARGET,          function(e) addBuff(e.unit, "emberdebt", 3, 3) end)
```

`on` runs at a moment and its answer is ignored. `contribute` changes a number the game is in the middle of working
out -- return the new number, or return nothing to leave it alone. `gate` denies something the game would otherwise
allow: return false. `veto` suppresses something it would otherwise do: return true. Those last two only ever move
one way -- a gate cannot turn a no into a yes, and a veto cannot undo a suppression the game has already made. All
four are registrations, so they go at the top level of the file; you cannot add a handler from inside a handler.

`TIMING` and `DECISION` are tables the game itself publishes, and writing the moment or the decision as a plain
string is refused when the script is read, with the name you should have written given back to you.

**Several mods can change the same number and all of them get their way.** Contributions chain: the first handler is
shown the game's own figure, and every one after it is shown what the previous one left. So two mods each writing
`return e.damage + 5` produce ten more damage, not five. What you return REPLACES the number you were shown rather
than being added to it -- which is why "add five" is written as `e.damage + 5` and why a mod that returns a flat
number wipes out what came before it, deliberately. Handing back the number you were shown changes nothing at all.

That last line is the shape of it: "only when this blow finished something off" is not a thing mod.json can say, so
writing it as a declaration would fire on every hit. This is also where a status effect you declared under `buffs`
gets its meaning -- the effect is a counter the game maintains, and the script is what reads it.

**A script decides; the timeline presents.** A whole round is worked out the instant you confirm it and played back
over the following half-minute, so a handler runs seconds before the animation it would be describing. That is why
speech bubbles, shakes, camera moves and sound cues are beats on a coin rather than something a script places: a
script has no way to know which moment on screen its call belongs to. The two things a script can change about what
you see -- floating a status line, and putting a character into one of its FORMS -- are held and landed at a blow
that is actually on screen rather than at the moment the handler ran, so neither of them is an exception: neither
picks a moment.

**At unit spawn a script may watch and may not act.** Asking the game to do something while the stage is still being
put together returns nothing and leaves you a sentence saying where to move the line to. That is containment, not
fussiness: doing per-unit work in that loop softlocks the game.

**The list of what a script can call is not in this document, on purpose.** Limbonia publishes its own surface --
every moment, every decision a `contribute`/`gate`/`veto` can attach to, every field on the event, every method on a
character -- and the companion's script editor completes out of that, live. A written list kept here would describe
one build and then quietly stop being true. The whole of it, written out from the game's own surface, is published
instead: **Open the reference** on the Scripts panel, or **Reference** in the script editor's toolbar, opens it in
your browser.

Write the .lua inside the mod, press Reload, and read the fault list. The sandbox has budgets and a quarantine, but
nothing here assumes a DOWNLOADED script is safe to run.

**Everything above is the `script` KEY.** How to actually write one -- what each of the four verbs lets you answer,
which character a handler is about, coins, criticals, SP, status effects, transformations, and the mistakes that
produce a script which loads cleanly and does nothing -- is in [scripts info.md](scripts%20info.md).

