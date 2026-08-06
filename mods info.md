# UNIT
- int - top level
```json
"appearance": 10916 // Thumb Nursefather Rodion
```

# NAMES

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
| skill    | int    | WHICH skill, counting from 1 — the same "Skill 1 / 2 / 3" the game shows. Needs a character: `appearance` here or once at the top. Aliases skillSlot, slot
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
|buff          | int/string | a STATUS EFFECT, not a character. A number is the effect's id, a word is its name. Aliases buffName, statusEffect, buffId, buffID
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
  {"voiceId": "battle_select_hysteric_10312_1", "file": "voice/hyst.ogg" }
  {
    "situation": "battle_kill",
    "text": "Let's go.", // bubble text
    "file": "sounds/entry.ogg",
    "bubbleSeconds": 2.5, // life of the bubble
  }
]
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| personalityId | int          | aliases appearance, unitId
| voiceId       | string       | the full id, e.g. battle_kill_10916_1 - highest precedence, ignores every other selector
| situation     | string       | alias cue. "*" or omitted = any
| variant       | int          | the trailing number. Battle cues always use 1 
| file / files  | path / array | files picks one, avoiding an immediate repeat 
| mute          | bool         | silence. The only way to get it - a custom character with no recordings should be silent, not speak in its stand-in's voice 
| volume        | number       | clamped to 4.0 max

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

| Field     |  Type  |  Notes  |
|-----------|--------|---------|
| cues[]    |   []     |                                              
| time      | number | seconds, on that coin's clock. Not read when the cue is anchored to the hit
| clip      | string | <modfolder>/<id> - note the namespace prefix 
| volume    | number | optional, overrides the clip's own
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
  "overlay": [],
  "sidecar": [],
  "buffs":   [],
  "script":  "behaviour.lua"
}
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| enabled | bool  | false switches the WHOLE folder off - art, names, voices, sounds, animations, everything. A real yes/no, not "false" in quotes
| appearance | int or text | who this mod is about, said ONCE. Aliases personalityId, unitId, characterId, id - consulted in that order, the first one carrying a number wins
| renderer | text | how your animations are drawn. See SIDECAR
| script | text | the name of a .lua file in this folder. See SCRIPT

**Say who once.** Every section that addresses a character -- `names`, `voices`, `art`, `motions`, `sidecar`, `buffs`,
`script` -- falls back to the top-level declaration when its own entry names nobody. An entry that DOES name somebody
always wins, so a mod about one character can still make one section an exception. This is the same fact written once
instead of five times, and forgetting one of the five is the failure it exists to remove.

Written as a bare number (`10101`), it matches every appearance of that identity -- the base one and each of its
skins -- which is what a whole-character mod wants. Written as a full appearance name
(`"10101_YiSang_BaseAppearance"`), it matches that one and no other. Written as a SINNER number (1-12), it is refused
and reported: a sinner has many identities and a number that low cannot say which you mean, so the rest of the file is
left with no character to fall back on. A string that does not start with digits is ignored rather than misread, so
`"id": "my-mod-guid"` is safe to keep for your own bookkeeping.

**One wrong value never kills the game.** Every section is read defensively and reports rather than throws: a number
where text was expected, a quoted `"false"` where a yes/no was expected, a time that is not a time -- each of them is
skipped, the rest of the file still loads, and the sentence appears on the mod's card in the Mods panel. If mod.json
will not parse at all (a missing comma, bracket or quote) the whole folder is skipped and told so. Read that card
first: almost everything in this document that can go wrong says so there, in words, before you ever start a fight.

**Keys nothing reads.** `bundleId` is an 8-character hex id the Unity build writes at the top of the file so the
bundle it produces keeps the same internal name across rebuilds -- leave it alone, and do not write one by hand. A
top-level `name` is written when the panel creates a mod for you, and is read by nothing: the mod's name is its
FOLDER name, everywhere. There is no `author`, `version` or `description` key.

# MOTIONS
- motions[] - top level. One entry per character: one appearance, one bundle, and everything in it.

```json
"motions": [
  {
    "appearance": "10101_YiSang_BaseAppearance",
    "bundle": "bundles/yisang.bundle",
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
| bundle     | path | required, relative to the mod folder. The .bundle Unity built
| assets     | []   | required. Everything you want out of that bundle - see ASSETS
| coverage   | text | "wholeCharacter" (the default here) or "perMotion". See below
| fallback   | text | which of your motions to play for one you did not supply. Defaults to Idle, then Default
| alwaysOn   | []   | effects that sit on the character for the whole battle - see ALWAYSON
| targetGroup| []   | where the units you are hitting stand - see TARGETGROUP
| renderer   | text | overrides the top-level one for this character only
| parent     | text | scalePivot (default) / appearance / animPivot / renderer - what your drawing hangs off
| spritePath | text | where the sprites load from. Defaults to "auto", the character's own
| sortingOffset | int | added to the character's drawing order, -1000..1000. 0 = exactly where the original drew
| sync       | text | auto (default) / game / own - whose clock your animation runs on. game and master mean the same, as do own and free
| hide       | {}   | main / parts / spine / effects - which of the character's own renderers to switch off. main defaults on, the rest off
| disableSpine | bool | ask the game not to draw its Spine skeleton at all. Default true
| disableTrail | bool | the same for its motion trail. Default true

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
| camera | {} | shake / zoom / rotate - see CAMERA
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
  "zoom":   [ { "time": 0.2, "size": -3.0, "duration": 0.4, "between": 0.5 } ],
  "rotate": [ { "time": 0.2, "z": 8, "duration": 0.5, "speed": 0.05 } ]
}
```
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
| size   | number | HOW MUCH CLOSER. NEGATIVE ZOOMS IN. -12..12; the game's own clips live in 0..-6. Default -2
| duration | number | how long the move to that framing takes, 0.01..6. Default 0.3
| between | number | WHAT to frame: 0 = the acting character, 1 = its target, 0.5 = both. Default 0
| focusSpeed | number | how quickly the camera chases that point, 0.01..2. Never 0. Default 0.2
| axisY  | number | vertical bias, -5..5. The game leaves this at 0 on every clip
| ease   | int | 1..40. Default 6

| rotate |  Type  |  Notes  |
|--------|--------|---------|
| time   | number | required
| x, y, z | number | degrees off the camera's default, -360..360. y and z are mirrored by the acting character's facing, so one number reads the same for a sinner and for an enemy
| duration | number | 0.01..6. Default 0.3
| speed  | number | 0..2. Default 0.05
| ease   | int | 1..40. Default 6

**Nothing here is ever invented for you.** These are cosmetic, so leaving one out means "don't", never "pick
something for me". A time that is not a usable number skips that entry and says so.

**Never write `"ease": 0`.** Zero is DOTween's "unset", which sends the tween onto a curve a mod cannot supply and
which the game does not check for null. It is clamped to 1 rather than passed on, but do not rely on that -- write a
real one.

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
| arriveRadius | number | how far short of the target to stop, in character-widths. Default 0. The game's own clips sit near 1.2
| includeTargetRadius | bool | add the target's body radius to that distance. Default OFF - on, your one number stops being the answer
| includeAttackerRadius | bool | the same for your own body. Default OFF
| chaseY | bool | follow the target's height as well
| refreshDir | bool | let the game turn the character. Default true
| facing | text | "auto" (default), "keep", "left" or "right"
| fixedDirection | bool | the older spelling of `facing: "keep"`. Kept working; prefer `facing`
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
| basedOn| text | which existing effect to shape it from. Alias copyFrom. Also opts the entry in to being added
| type   | text | "positive" / "negative" / "neutral". Also good/bad/neutral, buff/debuff. Decides despel and which way a give-multiplier moves it
| maxStack | int | how many stacks one character may hold. 0 or more. Omitted = keep the based-on effect's
| maxTurn  | int | how many turns it may last. 0 or more. Omitted = keep the based-on effect's
| canBeDespelled | bool | whether a "remove one effect" catches it
| icon   | path or text | a .png/.jpg your mod ships, OR the name of a picture the game already has. Explicit forms: iconFile, iconId
| typoIcon | text | the little symbol beside the line that floats up. Omitted = the based-on effect's
| sound  | path or text | heard when the effect lands. A file path, or a name from your `audio` list. Omitted = SILENT
| floatingTextSize | number | 0.25..3. The game draws its own effects at 0.75 and Bleed at 1
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

**The two ceilings are enforced by different things, and one of them can be quietly beaten.** `maxStack` is handed to
the game -- written into the effect's own row and clamped by the identical code that clamps Bleed, on every path.
`maxTurn` cannot be: the game never reads that field at all, so Limbonia enforces the turn ceiling itself when the
effect is applied. Both are still written down, so anything inspecting the effect reads what you declared.

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

**The colour of the line that floats up cannot be changed.** The game picks it from a hardcoded list of eleven of its
own effects; everything else in the game, its own included, takes one flat colour at 0.75 size. `type` is not secretly
it. `floatingTextSize` is the one part you can move -- raise it to 1 and a declared effect reads like one of the
eleven. Left out, it looks exactly like the game's own, which is the point.

`syncOnBossRaid` is refused if you write it: an effect a mod adds is always kept out of the Boss Raid record.

Switching a mod off does not delete its effects from the game -- a counter already on a unit points straight at the
row, so taking it away mid-battle would crash. The effect stops MEANING anything and the counter ticks down and
expires on its own.

# SCRIPT
- script - top level. The name of a .lua file in this folder.

```json
{
  "appearance": 10101,
  "script": "behaviour.lua"
}
```
| Field  |  Type  |  Notes  |
|--------|--------|---------|
| script | text | the FILE NAME, nothing else. No folders, no "..", no path out of the mod

There is one key and it is a plain string. No default name is picked up, so a .lua file sitting in the folder with
nothing pointing at it never runs. A name with a slash or a `..` in it is refused outright and told so -- the manifest
names a file inside the mod and nothing else. A missing file, an unreadable one, or one too large to be a behaviour
file is reported against the mod rather than failing quietly. The script gets the character from the top-level
declaration, so it does not name one itself.

What a script does is answer questions a declaration cannot ask. It may be told when things happen, adjust numbers the
game is deciding, deny something the game would allow, or suppress something it would do:

```lua
on(TIMING.BEFORE_GIVE_ATTACK, function(e) log(e.damage) end)
contribute("attackDamage", function(e) return e.damage + 5 end)
gate("recoverSp",          function(e) if e.unit.hp < 20 then return false end end)
veto("break",              function(e) return e.unit:hasBuff(12) end)
on(TIMING.ON_KILL_TARGET,  function(e) addBuff(e.unit, 1, 3, 3) end)
```

That last line is the shape of it: "only when this blow finished something off" is not a thing mod.json can say, so
writing it as a declaration would fire on every hit. This is also where a status effect you declared under `buffs`
gets its meaning -- the effect is a counter the game maintains, and the script is what reads it.

**A script presents nothing.** Anything that appears on screen or comes out of the speakers is authored as a beat on
the character's own timeline, on the same clock as the animation. A script cannot place one.

**At unit spawn a script may watch and may not act.** Asking the game to do something while the stage is still being
put together returns nothing and leaves you a sentence saying where to move the line to. That is containment, not
fussiness: doing per-unit work in that loop softlocks the game.

Write the .lua beside mod.json, press Reload, and read the fault list. The sandbox has budgets and a quarantine, but
nothing here assumes a DOWNLOADED script is safe to run.

