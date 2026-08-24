# SCRIPTS

A mod's `mod.json` says what the mod *looks and sounds like*. A script says what it *does* — the part of a mod
that has to wait for the fight to ask it a question. "Make this character hit harder" is a declaration. "Make this
character hit harder for every stack of the effect it handed out three turns ago" is a script.

This document is the guide. It explains the ideas, the order things happen in, and the mistakes that produce a mod
that loads cleanly and does nothing. **It is deliberately not the list of what you can call.** Limbonia publishes
that itself, so it can never be out of date; the last section says where to get it. Everything named in an example
here is real and current, but an example is not an index.

Read [mods info.md](mods%20info.md) first if you have not — a script cannot draw anything, and most of what people
want from one turns out to be a `mod.json` key.

---

## Where the file goes

```
Mods/
  YourMod/
    mod.json
    scripts/
      behaviour.lua
```

```json
{
  "appearance": 10101,
  "script": "scripts/behaviour.lua"
}
```

One key, naming one file anywhere inside the mod. A folder may be part of it — **Start a script** puts a new one in
the mod's `scripts` folder and writes that path for you — and a mod that keeps its `.lua` beside mod.json works
exactly as well. What is refused is anything pointing OUT of the mod: a full path off your own disk, a drive letter,
a `..`. There is no default either: **a `.lua` sitting in the folder with nothing pointing at it never runs** — and
"pointing at it" means this key or an `import` from the file this key names, which is how a script is split across
several files. See **Splitting it across several files**. The script inherits the character from the mod's top-level
declaration, so it does not name one itself.

You do not have to type any of that. Everything to do with scripts lives on one screen in the companion: the
**Scripts** panel, last in the Mods group, after Animation Timing. That is where a script is created, edited,
switched on and off, and where everything it says or gets wrong is reported — so it is the screen to have open
while you write one.

On that panel, **Give a mod a script** picks a mod, writes the key into its `mod.json` and creates the file already
containing a working starter — one moment watched, one decision joined in on, and a log line that shows up the first
time a fight reaches it. The starter is generated from the running game's own account of itself, so the moment and
decision names in it are the ones this build really has.

**Edit** opens the pop-out editor. It suggests as you type, and the suggestions come from the running game rather
than from a document, so start the game before you write. **Reload scripts in the game** re-reads every script
without restarting anything.

**Reloading waits for the end of a fight.** Press it mid-battle and you are told so rather than refused: handlers may
be running at that instant, and the scripts they belong to cannot be thrown away underneath them. Your change lands
when the fight ends, which is worth knowing before you conclude an edit did nothing.

A behaviour file is prose, not an asset, and both the size of the file and the time its top level may take are
capped. Neither is a number an ordinary script comes near — but a table of a few thousand entries built at load is
the one way to meet them, and it is reported against your mod like any other problem rather than hanging anything.

**Each script has its own switch.** In **Installed scripts**, `Script on` / `Script off` stops that one script and
nothing else — the mod's art, voices, sounds and animations carry on. That is the switch to reach for when you are
narrowing down which of two mods is responsible for something; throwing the whole mod off the Mods screen changes
far more than you meant to test.

---

## Writing it in VS Code instead

The editor in the companion is not the only one you can use, and if you already live in VS Code you can have the
whole API there.

1. Install **VS Code**, open **Extensions**, search for **Lua**, install the one by **sumneko**.
2. On the Scripts panel, pick your mod and press **Set up *&lt;mod&gt;* for VS Code**.
3. In VS Code, **File → Open Folder** and pick **the mod's own folder** — the one with `mod.json` in it, not a
   file inside it. That last part is the step people miss and the one that matters.
4. Open your script from the sidebar. It is very likely a level down, in `scripts` — open the mod folder anyway.

Because it is set up on the **folder**, every `.lua` in the mod gets the same treatment — so a script split across
several files (see **Splitting it across several files**) is understood the whole way down, with no second step.
A file you add next week is covered the moment you save it; there is nothing per-file to run and nothing to declare.
The editor also **follows your imports**: type a dot after something you imported and you get its real contents,
through as many files as you have chained together, because the setup teaches the editor what `import` means. Leave
the `.lua` off the name and this works; write it and the editor loses the thread, which is the whole reason that
spelling is refused.

**Why the folder and not the script.** What teaches the editor this API is a settings file at the top of the mod,
and an editor only looks for one at the root of whatever it was opened on. Open `scripts`, or the script by itself,
and it finds none of it: no completion, no hovers, no red lines, and no message saying why — which reads exactly
like the setup having failed. It also cannot be moved down to match, because a mod is equally allowed to keep its
script beside `mod.json` and a settings file inside `scripts` would do nothing for that mod at all.

Then everything below happens as you type:

* Type `TIMING.` or `DECISION.` and every one there is appears, with what it is and how often it happens.
* Type `e.` inside a handler and you get **the names that moment actually hands over** — not a list of every name
  there is. At a moment that hands you the character who struck the blow it is offered; at one that does not, it
  is not, and typing it anyway gets a line under it. See [`e` is not one shape](#e-is-not-one-shape--every-moment-has-its-own):
  this is that whole section, enforced while you write.
* Names that arrive only *some* of the time are marked as such, so you are asked to check one before reading
  something off it.
* Type a colon after a character and everything you can ask it to do appears, with what each argument means.
* The parts of Lua that scripts cannot reach are gone from the editor too, so you cannot spend an afternoon on
  something that was never going to run.
* Type a moment's name on its own line and press <kbd>Tab</kbd> to drop in the worked example for it.

**It is written from the build you are running**, which is the point — it cannot describe a version of the game
you do not have. What it writes is stamped with the version of the surface it was taken from, so it is a snapshot
rather than a live connection, and a snapshot goes out of date when Limbonia is updated underneath it.

**Stale reads as a fault in your own script, which is why it is worth recognising.** The usual shape is a name that
is perfectly real — a moment, a field, a method the game gained since you last ran the setup — sitting under a red
line saying it is undefined, or simply never turning up in the suggestions. It runs fine; the editor is describing
an older game. The other direction happens too, and reads the opposite way: a name completes helpfully and then the
game says it has never heard of it.

Either way, press **Set up *&lt;mod&gt;* for VS Code** again. It replaces what it wrote before and tells you which
version it replaced. The settings file it puts at the top of your mod folder is yours, so if you have changed
anything in it your version is kept.

None of these files are part of your mod. The game never reads any of them, and deleting the lot leaves the mod
working exactly as before.

---

## The shape of a script

**It is ordinary Lua 5.4**, so anything the Lua manual says about the language is true here. What is different is the
library: most of the standard one is not reachable, and there is a set of names of our own instead. Both lists are in
the reference.

A script does not run continuously. It is **read once**, and while it is being read it registers handlers. After
that the game calls those handlers at moments during a fight.

```lua
-- read once, at load
on(TIMING.ON_KILL_TARGET, function(e)
	-- run later, every time that moment comes round
	log(e.unit.id, "finished something off")
end)
```

All four registration verbs are **top level only**. Calling one from inside a handler raises, and it has to: the
list of moments must be complete before the first one arrives.

Anything else at the top level runs once too, which makes it the place for your own tables and constants:

```lua
local WATCHING = { "Laceration", "Sinking" }

on(TIMING.ON_START_TURN, function(e)
	for _, name in ipairs(WATCHING) do
		local n = e.unit:buffStacks(name)     -- each one asks the game a question
		if n and n > 0 then log(name, n) end
	end
end)
```

A loop of colon calls is the one shape worth thinking about, because every turn of it asks the game for something
and that is the only thing here that costs the fight any real time — see **Budgets, failures, and one handler going
quiet**.

**If the top level stops with an error, the whole script is abandoned — including the handlers it had already
registered.** Reading the file is one attempt: it either finishes and every registration in it stands, or it fails
and the script is reported as not loaded, with the error and the line on the Scripts panel. There is no partial
load. So a mistake in the last line of a long file costs you the first line too, and a registration the game
*refuses* — a scope it does not serve, a decision handed to the wrong verb, a moment that does not exist — is an
error like any other and takes the rest of the file down with it.

That is only worth working around when you are deliberately writing against more than one build, and then the tool
is `pcall`:

```lua
-- carry on even if this build does not serve that scope
pcall(on, TIMING.ON_START_TURN, { scope = "allies" }, function(e) log("ally") end)
```

For an ordinary mod, do not — a refusal is telling you something, and the sentence it comes with is the answer.

---

## Splitting it across several files

`mod.json` names one script, and that has not changed. What has changed is that the one script can bring in others:

```lua
-- behaviour.lua -- the file mod.json names
local helpers = import("helpers")

on(TIMING.ON_END_COIN, { scope = "any" }, function(e)
	if not helpers.isWounded(e.unit) then return end
	log(e.unit.id, "is down to", helpers.share(e.unit))
end)
```

`import` takes the **name of a file in your script's own folder, without the `.lua` on the end**, and hands back
whatever that file returned. A folder underneath is fine — `import("lib/maths")`.

That is the whole of the mechanism. The rest of this section is the part that is not obvious: where to put what,
and the handful of things that otherwise cost an evening.

### How to arrange it

"You can split the file" and "here is how a split file is arranged" are different things, and only the second is
much use. Three files, one job each, is where a script that has outgrown a single file usually ends up:

```
YourMod/
  mod.json
  scripts/
    behaviour.lua    says WHEN  — the handlers, and nothing else
    helpers.lua      says HOW   — asked questions, answers them
    tuning.lua       the NUMBERS both of them argue about
```

```lua
-- tuning.lua -- every number this mod uses, and nothing else. It imports nothing.
return {
	woundedAt = 0.5,          -- a share of the character's own maximum, not a flat number
	effect    = "Laceration",
}
```

```lua
-- helpers.lua -- the questions this mod asks about a character, in one place.
local tuning = import("tuning")

local H = {}

function H.isWounded(u)
	if not u or not u.hp or not u.maxHp or u.maxHp <= 0 then return false end
	return (u.hp / u.maxHp) < tuning.woundedAt
end

function H.share(u)
	if not u or not u.hp or not u.maxHp or u.maxHp <= 0 then return "?" end
	return math.floor((u.hp / u.maxHp) * 100) .. "%"
end

-- Handed back, so the main file need not import tuning as well to read one number out of it.
H.tuning = tuning

return H
```

**Split along the grain of what changes.** Numbers move most often, the reasoning next, the handlers least. Arrange
it that way and a change to a number is a change to one short file you can take in at a glance — which is what you
will actually be doing at two in the morning. Arrange it the other way and every tweak means reading past the
handlers to find the number.

**The main file names what it needs, not everything underneath it.** `behaviour.lua` imports one file and gets the
chain, because `helpers.lua` imports `tuning.lua` itself and hands it straight back on its own table. A file is free
to re-export what it imported, and that is usually kinder than making the top of the mod list every file in it.

Nothing about three files is a rule. Two is fine, one is fine, and a `lib` folder underneath is fine. What is worth
copying is the idea that each file has one answer to "what is this for".

### Leave the `.lua` off

`import("helpers")`, never `import("helpers.lua")` — and if you write the extension the script stops as it loads and
tells you the spelling to use instead. It is worth knowing why, because otherwise it reads as fussiness.

A name without the extension is one **the editor can resolve to the real file**. Write `import("helpers")` and then
`helpers.` offers everything `helpers.lua` returns, with hovers, go-to-definition and a red line under a field that
is not there — through as many files as you have chained together. Write `import("helpers.lua")` and the editor
reads the dot as a folder separator, goes hunting for a folder called `helpers` with a file called `lua` in it,
finds nothing, and marks every field of whatever came back as a mistake.

Accepting both spellings would have been worse than accepting one. The extension form loads perfectly well in the
game and is silently dead in the editor, and nothing on screen would tell you which of the two you had picked.

### Where it looks

**"Your script's own folder" means beside the file `mod.json` points at, not the top of the mod.** If your script is
`scripts/behaviour.lua` — which is where the companion files a new one — then `import("helpers")` means
`scripts/helpers.lua`. If your script sits beside `mod.json`, so do its imports. You never have to work out which
case you are in: it is always "the folder my script is in".

**Everything you import has to be inside that folder.** A full path, a drive letter or a `..` climbing out is
refused as the file is read, and so is a name that merely looks local and leads somewhere else. That is not a rule
about tidiness — a mod is a folder somebody else downloads, and a name pointing anywhere else would work on the
machine it was written on and nowhere else. For the same reason a script cannot reach another mod's files, however
the name is spelled.

### Bring it in at the top, then use it anywhere

`import` is available only while a file is being read, exactly like the four registration verbs and for the same
reason: a file you bring in may register handlers of its own, and the list of moments has to be complete before the
first one arrives. **A handler cannot import**, and one that tries stops there — mid-fight is far too late to be
reading files.

There is nothing to work around, because there is nothing you lose. Import at the top, keep what came back in a
local, and use it wherever you like — the local is still there when a handler runs, however much later that is:

```lua
local helpers = import("helpers")        -- once, while the file is being read

on(TIMING.ON_START_TURN, function(e)
	log(helpers.share(e.unit))           -- used every turn, imported on none of them
end)
```

### One read, one copy

**A file is read once, however many times it is imported.** Two files that both `import("state")` are handed the
*same* table, not one each — which makes a third file the ordinary way to pass something between them:

```lua
-- state.lua
return { hitsThisFight = 0 }
```

Whichever of them counts, both of them see it. The same rule is why two files importing a third costs one read
rather than two, and why a file's top level never runs twice no matter how many places ask for it.

### An imported file is part of the script, not a library it calls

It is read inside the same sandbox, so everything refused at the top of the main file is refused in there too:
`require`, `package`, `dofile`, `loadfile`, `load`, `io`, `os` and the rest are as absent one file down as they are
at the top. That is the reason for having `import` at all rather than borrowing the name every Lua author already
knows — `require` takes any path on the disk and loads what it finds into the language's own environment, so what it
brought back would arrive holding everything this sandbox turned away. Type `require` anyway and the refusal points
you at `import`.

It also means an imported file can **register handlers on its own account**, using `on`, `contribute`, `gate` and
`veto` just as the main file does. Those handlers are the mod's like any other:

```lua
-- counting.lua
local M = { rounds = 0 }

on(TIMING.ON_START_ROUND, { scope = "any" }, function(e)
	M.rounds = M.rounds + 1
end)

return M
```

```lua
-- behaviour.lua
local counting = import("counting")      -- its handler is registered the moment this line runs

on(TIMING.ON_END_BATTLE, function(e)
	log("rounds this fight:", counting.rounds)
end)
```

A file that returns nothing at all hands you nothing back, which is exactly right when the handlers *are* the reason
you imported it — `import("counting")` on a line of its own is a complete statement.

### What stops the script

All of it happens **while the files are being read, never in the middle of a fight**. The script is dropped whole,
with the reason against your mod on the Scripts panel, rather than left half-loaded — and it is the script that is
dropped, not the mod: its art, voices, sounds and animations carry on regardless.

Every message names the file it is about:

* a name with a dot in it — and the trailing `.lua` gets its own sentence, spelling out the line to write instead;
* a name that leads outside the mod's folder;
* a file that is not there, named as the file on disk rather than as the name you typed, so you know what to look
  for;
* the mod's own script, which would start the whole thing over and register everything in it twice;
* two files that import each other, which names both — neither could ever finish, so move the part they share into
  a third file and import that from both;
* a mistake *inside* an imported file, which names that file and the line in it — the file the mistake is in, not
  the one that asked for it.

**None of this is a limit on how you write.** There is no ceiling on how many files a mod has, how many times one is
imported, or what any of them do. A circle is refused because it cannot finish; a chain of imports far deeper than
any real script would go is refused as the same fault spelled the long way round. Both are caught as the files are
read, and no mod anybody would write meets either.

What splitting does *not* buy you is more room. The size limit applies to each file rather than to the mod, and the
whole load — every file it reaches — shares the one allowance of time, so a slow top level stays slow after you have
spread it over four files.

### When you share the mod

**The imported files go with it, and you do not have to list them anywhere.** Packaging a mod includes every `.lua`
in its folder, whether `mod.json` names it or not, precisely because your script names them instead. Nothing is
parsed and nothing has to be declared; a spare `.lua` you never got round to deleting is carried along too, which is
much the better way round.

---

## Which character a handler is about

Every registration takes an optional table before the handler. It has three keys and nothing else: `scope`, which
picks the character, and `skill` and `skillId`, which pick one of that character's skills — the next section is
about those two. Any other key you put in the table is ignored, so a misspelt one narrows nothing and says nothing.

```lua
on(TIMING.ON_START_TURN, function(e) end)                     -- your character only
on(TIMING.ON_START_TURN, { scope = "any" }, function(e) end)  -- everybody on the field
```

**Leave it out and the handler is about your own character** — the one your `mod.json` declares. That is the right
default and it is usually what you want, but it is also the first thing to check when a handler never fires.

`{ scope = "any" }` runs it for every character in the fight, whoever they belong to. You need it more often than
you would think, because the interesting cases are usually about somebody else: *your* character hands out an
effect, and *the enemies carrying it* are the ones taking extra damage.

**On `contribute`, `gate` and `veto` it does not show you more — it applies your answer to everybody.** That is the
one thing about it worth being careful with, and it is easy to read past because on `on` it really is only a wider
view. Halve incoming damage at `{ scope = "any" }` and you have halved the enemies' too. Widen the scope when you
mean the change to land on whoever the moment is about, and sort out inside the handler who you actually meant —
`e.unit.isAlly` is free to read.

**There is no scope for one side.** Writing `{ scope = "allies" }` is refused, and told so, at load. A handler
cannot be limited by faction — but the moments that stand back and look at the whole fight hand you both sides as
lists, so use `{ scope = "any" }` and read `e.allies` and `e.enemies` there. Every character also knows which side
it is on:

```lua
on(TIMING.ON_START_ROUND, { scope = "any" }, function(e)
	if e.unit.isAlly == false then
		log("an enemy is up:", e.unit.id)
	end
end)
```

**`e.allies` is always the PLAYER'S team** — not "the side this handler is running for". On a handler running for
an enemy it is still your team. Pair it with `e.enemies` rather than reading it as "us".

**If the mod does not declare a character, a default-scope handler is refused at load** with a sentence telling you
to put the identity at the top of `mod.json` or write `{ scope = "any" }`. It is refused rather than accepted and
left silent, because a handler that registers and never runs is the most expensive kind of bug in this layer.

**And there is one moment with no character to match against at all.** `TIMING.ON_FORMATION` arrives before a single
character has been built — see **Before there is a fight** — so the default scope has nothing to compare your mod's
identity to and could never match. **That one is refused at load too**, for the same reason as the paragraph above:
a handler that registers and never runs is worse than one that is turned away with a sentence. Write
`{ scope = "any" }` there and it is the only moment where the scope is not a preference.

---

## One effect on the first skill, a different one on the second

`{ skill = N }` narrows a handler to one skill the same way `scope` narrows it to one character. `N` is the small
number the game prints on the skill card - 1, 2 or 3.

```lua
contribute(DECISION.ATTACK_DAMAGE, { skill = 1 }, function(e) return e.damage + 10 end)
contribute(DECISION.ATTACK_DAMAGE, { skill = 2 }, function(e) return e.damage * 2 end)
```

**Two handlers, not one handler with an `if` in it.** The filter is applied before your function is called, so the
second one simply does not run while the first skill is being used. That matters more than it looks: a handler that
runs and returns early still costs the time it takes to start, and still shows up in the coverage list as having
fired.

**It combines with `scope`**, and the two read the way you would say them out loud:

```lua
contribute(DECISION.CRIT_CHANCE, { scope = "any", skill = 3 }, function(e) return e.critChance + 40 end)
```

**A slot number means the same thing for every character**, which is why it is the one to reach for. If you need one
exact skill and nothing else that shares its slot, use its own number instead:

```lua
on(TIMING.ON_END_ATTACK, { skillId = 10102 }, function(e) log("that specific skill finished") end)
```

**Both are checked when the script loads.** A number too big to be a slot is turned away naming the other key, and a
skill filter on a moment that is never told which skill is being used is turned away too - the stat decisions are the
main example, since they are asked about a character and nothing else. Neither is left to fail silently, because a
handler that never runs looks exactly like an effect that does not work.

**E.G.O. and the defensive actions have no slot**, and that is deliberate. The game does not print a number on them
either, and if they carried one then `{ skill = 1 }` would quietly be true for a block as well as for the first
skill. Read `e.skill.kind` when you need to tell them apart - it is `"skill"`, `"ego"`, `"guard"`, `"evade"`,
`"counter"` or `"other"`, and only `"skill"` has a slot:

```lua
on(TIMING.ON_START_BEHAVIOUR, function(e)
	if e.skill and e.skill.kind == "ego" then log("E.G.O. this turn") end
end)
```

**On the moments about a blow ARRIVING, the skill is the incoming one.** `TIMING.BEFORE_TAKE_ATTACK` and the stagger
moment are told the attacker's action, not yours - so `{ skill = 2 }` there means "while somebody is hitting me with
their second skill", which is usually what you want and is never what you would guess from the name alone.

---

## Who it is aimed at

`e.target` is the character the skill is pointed at. It is not the same question as `e.victim`, and the difference is
the reason both exist:

- **`e.victim`** is who is taking *this* blow. It is settled; the moment reporting it already knows.
- **`e.target`** is who the attack was aimed at in the first place.

On an ordinary attack they are the same character and either will do. **On a skill that strikes several they come
apart** - `e.target` stays the main one while the blow lands on each of them in turn - so comparing the two is how
you tell a main hit from a splash hit:

```lua
contribute(DECISION.ATTACK_DAMAGE, function(e)
	if not e.damage or not e.victim or not e.target then return end
	if e.victim.instanceId ~= e.target.instanceId then
		return e.damage // 2   -- splash: half damage
	end
end)
```

**The decisions made while aiming have `e.target` and nothing else.** When the odds of a critical are worked out,
nobody has been hit yet - there is no victim to name - so `e.target` is the only other character on the payload. That
is the whole reason it is there, and it is not a substitute for anything: it is the character the game itself looked
at when it worked the number out.

**And here is the trap.** Those aiming decisions are settled **once per coin and then applied to everyone that coin
reaches**. So on a skill that hits three enemies there is one critical decision, not three, and `e.target` names the
main one. You cannot give one of the three a different critical chance from the others, because the game does not ask
the question three times. If you need a per-character answer, do it at the damage moments, where `e.victim` is the
character actually being struck.

**`e.target` reads as nothing when there is nothing to name** - a block or a dodge is aimed at nobody, and damage
from a status effect ticking at the end of a round had no attack behind it at all. Check it before you use it.

---

## Rounds and turns are not the same thing

A **round** is the whole exchange: everybody picks, and then the whole thing resolves. A **turn is one action** —
one character carrying out one skill. Those are two different sizes of thing and the names do not make that obvious,
so it is worth reading properly once. Nearly every handler that fires far more often than its author expected is a
handler on a turn moment.

Inside a single round, in this order:

* **`ON_START_ROUND` fires once**, for every character on the field, before anybody has acted. It is one of the
  moments that stands back and hands you both sides as lists, so `e.allies` and `e.enemies` are there.
* Then the actions run one after another in speed order, **interleaving between characters**. Each one raises
  `ON_START_TURN` and then `ON_END_TURN` for the character carrying it out. A character with two skills gets two of
  each. A character that only **defends** gets one for the defence.
* **`ON_END_ROUND` is the end-of-round step** — after every character has taken every turn. Read the next few
  paragraphs before you build on it: it is **not** one event a round.

So `ON_END_TURN` is not "the round ended". It is "this action has finished, and another one is about to start". A
handler on it runs several times in an ordinary round rather than once, and it runs interleaved with everybody
else's: the character it fires for has finished acting, but the round has not.

This is not a pedantic distinction, and it has already cost somebody an evening: the game clears ordinary protection
**at the end of the round**, so a shield handed out at the start of one looks, from the outside, as though it went
away on a turn change — because several turns went past in between. See **Two kinds of protection** below.

**For "once a round", write `ON_START_ROUND`.** It is the round-level moment that always arrives.

`ON_END_ROUND` looks like the natural partner and it is not one, in two separate ways — this is the one place in
this document where the honest answer needs two paragraphs rather than a caveat.

**It does not always arrive.** Nothing announces the round's own end. What carries the moment is the fight *working
out end-of-round damage*, so a round in which nothing ticks may never produce it at all. That is why the reference
says of it that it *works when damage is worked out*.

**And where it does arrive, it arrives once per end-of-round damage calculation rather than once for the round.** A
round in which four characters are bleeding raises it **four times**, and each of those is a separate run of your
handler. That reads as a surprise only because the name describes an event and the moment
counts calculations — see **How often a moment fires is decided by what carries it** below, which is the rule this
is an instance of. A counter incremented in an `ON_END_ROUND` handler does not count rounds; it counts ticks, and
the number it lands on depends on what the enemies happened to be carrying. **There is no way to make it per-round.**
Use it when what you are doing is *about* end-of-round damage anyway, which is what it is genuinely good for;
otherwise do the work at `ON_START_ROUND` at the top of the next round, which costs you the wait and nothing else.

### How often a moment fires is decided by what carries it

The rates are not a table to memorise. Every moment in the vocabulary is carried into a script by one of three
things, and **which of the three decides the rate** — so once you can tell them apart you can predict a moment you
have never used:

* **Moments carried by something that happens to one character** fire **once per character it happens to**, and
  their names are honest. `ON_START_TURN` is that character's action; `ON_START_ROUND` is that character's round.
  Most of the vocabulary is here.
* **Moments carried by the fight working out damage** are counted in **damage calculations**, never in the events
  their names suggest. This is the group that catches people, because the name says one thing and the count is
  another: `ON_END_ROUND` is the worst of them, and `BEFORE_GIVE_ATTACK`, `ON_SUCCESS_ATTACK`,
  `ON_TAKE_ATTACK_DAMAGE`, `ON_END_PARRYING` and `BEFORE_ROLL_COIN_ACTION` are the rest. A blow that computes no
  damage produces none of them; a blow that computes several produces several. The reference marks every one of
  these *works when damage is worked out*, and that line is the whole warning.
* **`ON_HIT` is carried by the picture** rather than by either, so it fires once per blow you actually *see* land —
  which is more often than the damage moments, not less, because one calculation can produce several impacts or
  none.

**So read that line before you read the name.** If a moment only *works when damage is worked out* and you were
about to count something with it, count something else.

The same care applies to the sizes of thing the names refer to, which are not obvious either:

* **`ON_BATTLE_START` and `ON_END_BATTLE` are per character, not per fight.** Six characters means six runs of your
  handler, each carrying the same full rosters. If you want the look taken once, scope the handler to one character
  or keep a flag of your own. And do not read "once per fight" as "once per stage": a stage that comes in waves
  starts a fresh battle phase for each wave, so do not build on a staged fight raising these only once. Count them
  for the stage you care about rather than guessing — the last paragraph of this document's
  **Did my handler ever run?** section says how.
* **A duel is the whole clash; a parrying is one exchange inside it.** They are not two words for the same thing.
  `ON_WIN_DUEL` fires once, at the end, for whoever won the clash; `ON_WIN_PARRYING` fires for each pair of coins
  won along the way, so a clash that runs to several exchanges raises it several times. Reach for a duel moment when
  you mean the outcome and a parrying moment when you mean the back-and-forth.
* **"Parrying" is this game's word for CLASHING, not for defending.** Two attack skills clashing raise every
  parrying moment exactly as a guard does. If you have written a mod around attacking and skipped those moments
  because they sounded defensive, they were yours all along.
* The coin moments are one flip inside one skill.

**Pick the moment at the size of the thing you are describing**, and read how often it says it arrives before you
assume.

---

## The round is decided before you watch any of it

Confirm a round and the game works **the whole of it** out in one burst — every clash, every coin, every point of
damage, every death — and only then starts playing it back. The playback takes the better part of half a minute. The
arithmetic took an instant, and every one of your handlers ran during that instant.

Nobody expects this, and it explains a whole family of "my handler is firing at the wrong time" reports.

**What a handler reads is the value at resolve time, not the value on screen.** Ask a character for its health at
`ON_KILL_TARGET` and you get the health it had when the round was worked out, which may be several blows behind — or
ahead — of whatever the bar is showing at the instant you happen to be watching. There is nothing stale about it: the
fight really is at that point in its own accounting. It is the picture that is behind.

**A handler runs seconds before the thing it describes.** A whole round's worth of `log` lines land within about a
second of each other, before the first animation of that round has even started, in the order the game resolved them
rather than the order you see them. So the log is a record of the round's *reasoning*, not a commentary on the
animation — and reading it as a commentary is how a perfectly correct handler looks as though it fired in the wrong
place.

**`TIMING.ON_HIT` is the one exception, and only half of one.** It is raised at the frame a blow actually lands, so it
*arrives* in step with the picture — which is why it is the moment `showBuff` draws from immediately and the moment
`setForm` takes effect immediately. What it reads is not in step: the character's own numbers were settled back when
the round resolved, like everything else here.

**Which makes it the wrong moment to *change* anything, and this is the paragraph to learn that from rather than an
evening.** `ON_HIT` belongs to the playback. By the time it comes round the whole round has been worked out and the
picture of it has already been built out of that result. So a status effect, a heal, a shield or a drain asked for
there really happens — and **nothing on screen shows it for the rest of that round**, and nothing in the fight reacts
to it, because the round it would have changed is already settled. Every part of the call reports success, so there
is nothing to see except a mod that looks broken. **`TIMING.BEFORE_GIVE_ATTACK` is where a blow's consequences
belong**: it is announced while the round is still being worked out, so what you do there is part of the result the
fight then draws and reacts to.

None of this makes a change unreal at an ordinary moment. Health, protection, SP and status effects move in the
fight's own state the moment your handler asks for them, and the picture of the round is built afterwards, so it
shows. What it means is that **you cannot time anything against the animation from a script**, and that nothing you
read is a reading of what is on screen. See **The three things a script puts on screen deliberately**, which is the
same fact approached from the other end.

---

## The four answers

The verb you register with decides what kind of answer you are allowed to give. That is the whole design: you
cannot accidentally deny something from a handler that was only supposed to watch.

```lua
on(TIMING.BEFORE_GIVE_ATTACK,      function(e) log(e.damage) end)
contribute(DECISION.ATTACK_DAMAGE, function(e) return e.damage + 5 end)
gate(DECISION.RECOVER_SP,          function(e) if e.unit.hp < 20 then return false end end)
veto(DECISION.BREAK,               function(e) return e.unit:hasBuff("Laceration") end)
```

`TIMING` and `DECISION` are tables Limbonia gives every script. **A plain string is refused when the script is read** —
`contribute("attackDamage", ...)` does not load, and the refusal names the constant you should have written. There
is one spelling.

A decision belongs to exactly one verb. Ask for the wrong one and you are told which is right, at load, with the
corrected line written out — `DECISION.BREAK` is a veto, not a gate, and a script that got that backwards would
otherwise register happily and never be asked anything.

### `on` — watch

Its return value is ignored. Use it for logging, for handing out status effects, for triggering a transformation —
anything that is an action rather than an answer.

### `contribute` — change a number

**This is the one to understand properly, because it is what lets two mods coexist.**

Return a number and that is the number the game uses. Return nothing and it is left alone.

```lua
contribute(DECISION.TAKE_DAMAGE, { scope = "any" }, function(e)
	local owed = e.unit:buffStacks("Laceration")
	if not owed or owed <= 0 then return end
	return e.damage + (e.damage * 25 * owed) // 100
end)
```

Two rules, and both of them bite:

**Contributors CHAIN.** The first handler is shown the game's own figure. Every handler after it is shown *what the
previous one left*. So two mods each writing `return e.damage + 5` produce ten more damage, not five, and neither of
them had to know the other existed. It is not "the last one to answer wins", and the returns are not added together
either — each script just starts from where the one before it left off.

Order is therefore not something you can rely on, and there is no way to ask to go last. `+5` then `×2` is not the
same as `×2` then `+5`, so write a handler that composes from **what it was shown** rather than one that assumes a
starting figure, and it will behave the same wherever in the chain it lands.

**What you return REPLACES what you were shown — it is not added to it.** This is why "make it five more" is
written `return e.damage + 5` and never `return 5`. A handler that returns a flat number wipes out everything the
game and every earlier mod worked out, which is occasionally exactly what you want and is otherwise the single
easiest way to break somebody else's mod without noticing. Handing back the number you were shown changes nothing
at all, which makes it the correct way to abstain with a `return` in the line.

The same is true of every contribute decision, not just damage: the SP figure, the crit odds, the crit bonus, the
Power bonus and the stat bonuses all arrive with any earlier script's change already in them. **Several of them are
not the quantity their decision is named after** — see **Power** and **Stats**, where returning a flat number is not
"set it to this" but "add this instead of everything anybody else worked out".

**`e.original` is where the number started**, before the game's own passives touched it — so on the damage decisions
`e.damage - e.original` is how much everything else has already done to this hit, which is a question you cannot ask
from `e.damage` alone. It is an optional field like any other, so guard it:

```lua
contribute(DECISION.ATTACK_DAMAGE, function(e)
	if not e.damage or not e.original then return end
	if e.damage > e.original * 2 then return end   -- already doubled by something; leave it
	return e.damage + 5
end)
```

**Never change anything from inside a `contribute` handler.** It runs every time the game works out a number of
that kind for that character — from an attack, from an effect ticking at the end of a round, from anything — and a
script has no way to know how many of those belong to the same blow. Read, return, and do your acting in an `on`.

### `gate` and `veto` — refuse

`gate` denies something the game would otherwise allow: **return `false`**.
`veto` suppresses something the game would otherwise do: **return `true`**.

Anything else — including returning nothing — leaves the game's answer alone. Both verbs only ever move one way:

* A gate cannot turn a no into a yes. A veto cannot undo a suppression.
* **And a refusal from another script sticks too.** Once any handler has denied something, the handlers after it
  cannot put it back. Every interested handler is still asked, so two mods agreeing does not depend on which of
  them loaded first — but two mods *disagreeing* is not a contest, and the restrictive one wins.

That asymmetry is deliberate. A number two mods argue over can settle at a compromise; a permission cannot, so it
settles at "no". If you want an overridable "never", there is no such thing — write the condition into the handler
instead of hoping to be outvoted.

**Return a real boolean.** `return 1` and `return "no"` are both reported and ignored rather than honoured, because
Lua's own truthiness would take them in opposite directions from what you meant — `0` is true in Lua.

---

## `e` is not one shape — every moment has its own

Every handler is handed one table, written `e` by convention. **There is no shared struct.** Each moment and each
decision hands over its own set of names, and that set is published with the rest of the API:

* **Inside a handler, the editor offers exactly what that moment carries and nothing else.** `e.critical` is not in
  the list at `ON_START_TURN`, because nothing there has ever decided whether a blow was a critical. `e.critChance`
  appears at one decision and nowhere else.
* The reference document lists it per moment too, under **Also carries**.
* **A moment nothing announces says so** instead of offering everything. In place of a list you get one line —
  *nothing announces this moment, so what it would hand over is not known* — and the reference prints *not known* in
  the same column. That is a statement about Limbonia and not about the game: no seam is listening for that moment
  yet, so nobody can say what it would hand over. It is the same set of moments the Scripts panel files under **Not
  seen to arrive**, and a handler registered on one of them never runs.

A name that a moment does not carry is not hidden or greyed — it is not offered, because there is nothing there to
find. If you type one anyway it reads as `nil`, silently, exactly as any absent Lua key does. A name that is not a
field *anywhere* is the one case that says something, and it is the last part of this section.

**The narrowing follows the cursor, so write a handler where it is registered.** The editor works out which moment
you are in by looking back for the nearest registration — which is how a script is laid out anyway, one registration
after another with its handler written inline. Define a handler further up and pass it in by name and there is no
registration to look back to, so `e.` offers the whole vocabulary again: not because that moment carries all of it,
but because nothing can tell which moment you meant.

**Being on the list is not a promise of a value.** Most fields are marked *can be nothing*, and they mean it —
`e.damage` is on the attack moments and is still absent on a swing that computed no number. So `e.damage + 5` is not
a safe line; `(e.damage or 0) + 5` is, and a guard clause is better still:

```lua
on(TIMING.BEFORE_GIVE_ATTACK, function(e)
	if not e.damage then return end
	log("about to deal", e.damage)
end)
```

**A flag has three states, not two.** `e.critical` is `true` when the game has decided a blow is a critical, a real
`false` when it has decided it is not, and **nothing** where the moment came from the side that was never told. So
these ask different questions:

```lua
if e.critical == false then end  -- decided, and it is not a critical
if not e.critical      then end  -- that, OR nobody said
```

The same is true of `u.staggered`, `u.alive` and the rest of a character's optional fields — a `false` that means
"we could not look" would be indistinguishable from a real one, so they read as nothing instead.

**Everything a character tells you is free.** `e.unit.hp`, `u.sp`, `u.shield`, `u.staggered`, `u.isAlly` and the
rest are copied out before your handler runs, so `if u.staggered and u.sp < 0 and u.alive then` is three table
reads, not three requests to the game. Only the calls written with a colon and brackets — `u:hasBuff(...)`,
`u:setForm(...)` — cost anything.

**And that is why they do not change when you do.** They were read once, before your handler started; nothing goes
back and reads them again. So a field you have just acted on is the value from *before* you acted:

```lua
e.unit:giveHealth(20)
log(e.unit.hp)          -- the health it had BEFORE the heal, every time
```

This catches everybody once. It is not a staleness bug to work around — it is the same fact that makes the fields
free — and it is the whole reason the verbs that change a character hand back a number of their own. Read what came
back, never the field.

`u.sp` goes **negative**, and how far is `u.minSp`. Do not assume zero is the floor. `u.maxSp` is the other end.
Both of those are the same figure for every character in this game rather than anything about the one you are
holding — they are published per character so that `u.sp / u.maxSp` is arithmetic on the character you already have,
not so that you can compare two of them.

`u.alive` goes false when a character has died **or** withdrawn, and `u.retreated` is which of the two — worth
checking before you act on a character you were handed at an earlier moment.

### What a character IS, as against what is happening to it

Most of what a character tells you is about the fight in progress — health, protection, SP, whether it is staggered.
A few fields are about the character itself, and they are the ones that let a handler ask *what am I dealing with*
rather than *what is going on right now*.

`u.level` and `u.uptie` are settled before the fight starts and do not move during it. **`u.level` is the number on
the sheet**, which is not always the number the fight is working with — effects that raise or lower a character's
level move the working figure and leave this one alone. It is worth more than it looks, because a character's base
defence is worked out from its level. `u.uptie` is the sync level, 1 to 4 for one of the player's own; enemies carry
one too and it means nothing for them, so pair it with `u.isAlly` before you branch on it.

**Speed is three fields, and that is not redundancy.** The fight does not hand a character a speed — it hands it a
range and rolls inside that range again at the start of every round. `u.minSpeed` and `u.maxSpeed` are the two ends
of it and belong to the character; **`u.speed` is what actually came up this round, and it is the one that decides
turn order.** So "is this a fast character" is `u.maxSpeed`, "is it acting early this round" is `u.speed`, and a
character with the better range is not guaranteed to go first — only more likely to.

```lua
on(TIMING.ON_START_ROUND, function(e)
	if e.unit.speed and e.unit.speed == e.unit.maxSpeed then
		log("rolled the best it could")
	end
end)
```

All of them are free to read, like every other field, and all of them can be nothing — for a different reason from
their neighbours. The rest read as nothing when the character has no battle state to look at; these read as nothing
when its character sheet has not been attached yet, and `u.speed` reads as nothing until the round's speed has
actually been rolled. That last one is the ordinary answer while a fight is still being put together, which is
exactly where a handler is most likely to ask.

### Misspelling a field stops the handler

`e.damage` at a moment with no number in it and `e.critcal` anywhere look identical in Lua — both are a key that is
not there. They are told apart for you, against the same published tables the editor completes from:

* **A real field this moment does not carry** reads as `nil`, in silence. That is the case every defensive line in
  this document is written for, and not one of them changes: `(e.damage or 0) + 5`, the guard clause above, one
  handler registered on three moments guarding on what it finds. All still correct, and all still quiet.
* **A name that is not a field anywhere** stops the handler and says what you wrote, with your file and line in
  front of it the way Lua reports anything:

  ```
  behaviour.lua:12: there is no e.critcal -- did you mean e.critical?
  ```

Where nothing published is near enough to be a slip of the fingers, it says there is no such field and stops there
rather than naming a guess: a name with nothing like it in the list gets no suggestion at all, because sending you to
change a line that was never the problem is worse than saying less.

**It is the same on a character**, and methods count as well as fields: `u.staggerd` answers *a character has no
staggerd — did you mean staggered?*, and `u:hasBuf("Agility")` is caught the same way.

Why this one case is worth an error, when the other is worth silence: there is no honest reason to read a name the
API does not have. Left alone it reads `nil` for the life of the mod, every `if` built on it takes the wrong branch,
and nothing anywhere looks wrong. It is also the one mistake the editor cannot catch for you, because a name it
never offered is exactly the name somebody types from memory.

It is an ordinary failure once it happens — filed against your mod on the Scripts panel with the sentence above, and
counted like any other, so a handler that keeps doing it is switched off the same way anything else that keeps
raising is.

**`e` is not a scratchpad, either.** You can hang a key of your own on it and read it back, but reading one you have
not set yet is indistinguishable from a misspelling and is treated as one. A script's own state belongs in its own
tables at the top level, which is where it survives between moments anyway.

---

## Both sides of a blow

The game works damage out **twice** — once for whoever is swinging and once for whoever is being hit — and the two
halves are told different things:

| | attacking side | receiving side |
|---|---|---|
| `e.attacker` | the one swinging | **nothing** — it is handed the blow, not the character behind it |
| `e.critical` | the game's decision | **nothing** |
| `e.damageSource` | **nothing** | `"COMBAT"`, `"BUFF"`, … |
| `e.effect` | **nothing** | the status effect that did it |

**Exactly one moment can arrive from either of them, and it is `TIMING.BEFORE_GIVE_ATTACK`.** There the four names
above depend on which side announced this particular one, so the editor offers them marked **not every time** and
the reference marks them with a **†**. Everything else that moment carries (`e.damage`, `e.victim`, `e.coin`, …) is
there whichever side it came from.

**Every other damage moment belongs to the receiving side alone**, and that buys you more than it sounds like. At
`ON_SUCCESS_ATTACK`, `ON_TAKE_ATTACK_DAMAGE`, `ON_END_PARRYING`, `ON_END_ROUND` and `BEFORE_ROLL_COIN_ACTION` there is
no attacker and no critical flag to be had at all — those two are not offered there, and typing one is a misspelling
rather than a `nil`. In exchange `e.damageSource` and `e.effect` are there **every time**, so at those five the guard
you would write around them never fails and an absence really does mean "nothing but the blow itself caused this".

**`ON_SUCCESS_ATTACK` is the one whose name misleads**, and it follows from the paragraph above rather than being a
quirk of its own. It reads as "my attack landed"; it is announced from the receiving end, so `e.unit` there is the
character being *hurt*. Scope that handler to your own character and you are watching the wrong end of the blow.

That mark and *can be nothing* are two different claims, and the first is the stronger one. *Can be nothing* says
the moment carries the name and the value may be missing. **not every time** says the moment may not carry the name
at all, and which way it falls is decided by something you cannot see from inside the handler — here, which side
announced this one. A field that earns the second is not given the first as well, because one caution on a
line is one that gets read; treat a **†** exactly as you treat an optional field and check it before you branch on
it. It is not a reason to avoid a name. They are real where they are real, and saying so is the honest middle
between promising them and hiding them.

**The mark turns up in one other place**, for a related reason rather than the same one: `e.target` and `e.skill`
both come off the action behind a blow, and some things that deal damage have no action behind them at all. So the
moments and decisions that carry those two carry them marked as well. See **Who it is aimed at**.

A handler that forgets this puts its effect on its own character every time that character is hit — a real bug,
found the hard way:

```lua
on(TIMING.BEFORE_GIVE_ATTACK, function(e)
	if not e.attacker or not e.victim then return end
	addBuff(e.victim, "Laceration", 2, 3)
end)
```

Whoever is dealing it is `e.attacker`, whoever is on the receiving end is `e.victim`, whoever the blow was pointed at
in the first place is `e.target` — see **Who it is aimed at** — and whoever is offering SP is `e.healer`. Naming them
separately is what makes a handler mean the same thing at every moment that carries them.

### Which kind of damage this is

"Take a quarter more from attacks, but not from the burn at the end of the round" is the second thing everybody
wants, and the receiving side is told exactly that. `e.damageSource` is one word — `"COMBAT"` for a blow, `"BUFF"`
for an effect ticking, and `"PASSIVE"`, `"SKILL"`, `"EGO_GIFT"`, `"STAGE"` and a few more for the rest — and where
an effect was responsible, `e.effect` is which one, spelled the way you would write it yourself:

```lua
contribute(DECISION.TAKE_DAMAGE, { scope = "any" }, function(e)
	if e.damageSource ~= "COMBAT" then return end   -- leave burn, bleed and the rest alone
	if not e.damage then return end
	return e.damage + e.damage // 4
end)
```

**The two damage decisions each belong to one side, and that is why this works.** The uncertainty above belongs to
one *moment* and to no decision at all: `DECISION.TAKE_DAMAGE` is only ever asked by the receiving half of the
calculation and `DECISION.ATTACK_DAMAGE` only by the attacking one. So a take-damage contributor
always has `e.damageSource` and `e.effect` and never has `e.attacker` or `e.critical`, and an attack-damage
contributor is the exact opposite — none of the four is a **†** there, and the editor offers each of them at the one
decision that really carries it.

The same test inside `on(TIMING.ON_TAKE_ATTACK_DAMAGE, ...)` reads exactly the same way, because that moment is the
receiving side's alone too. `TIMING.BEFORE_GIVE_ATTACK` is the one to be careful at: when the attacking side is the
one announcing, there is no `e.damageSource` there at all, so `e.damageSource ~= "COMBAT"` is true for a reason that
has nothing to do with what caused the damage. At that one moment, check the field before you branch on its value.

### A boss made of pieces is two characters, and both of them reach you

Some bosses are built out of destructible pieces — an arm, a head, a tail — and the fight treats every piece as a
character of its own: it has its own health bar, its own status effects, and its own entry in the turn order.

**A blow against a piece asks the two damage decisions twice.** Once about the piece that was struck, and then
again about the whole creature, with the second ask shown whatever the first one left. Both are real, both run your
handler, and `e.unit` — and `e.victim`, or `e.attacker` when a piece is the one swinging — is a *different
character* each time.

That matters most for the numbers on a character:

```lua
-- WRONG on a boss with pieces: on the first ask this is the arm's health bar, not the boss's.
contribute(DECISION.TAKE_DAMAGE, function(e) return e.victim.maxHp end)
```

`e.victim.hp` and `e.victim.maxHp` are the **piece's** bar on the first ask and the creature's on the second. A
handler that reads either is quietly doing two different things on one swing.

#### `isPart` tells the two asks apart

**`u.isPart` is true when the character in hand is one of the pieces**, and false for everybody else — including the
creature the pieces belong to. It is free to read, like every other field, and it is the first thing to reach for at
a damage moment on such a boss:

```lua
-- Right: acts once per blow, about the whole creature, on any boss and on anything else.
contribute(DECISION.TAKE_DAMAGE, function(e)
	if not e.victim or e.victim.isPart then return end
	return e.victim.maxHp
end)
```

The same line is what you want whenever a handler should fire once per blow rather than once per half of the boss —
which is nearly always, because everything else on the field runs your handler exactly once.

#### `partType` says which piece you are holding

`isPart` tells you it is *a* piece. **`u.partType` tells you which one**, in the game's own words:

`"HEAD"`, `"BODY"`, `"LEFT_ARM"`, `"RIGHT_ARM"`, `"LEG"`, `"TAIL"`, `"BACK"`, `"BACK_HEAD"`, `"LEFT_EYE"`,
`"RIGHT_EYE"`, `"HEAD_PLATE"` — and `"NONE"` for a piece the game gives no anatomy to, which is a real answer and
not a failure to look.

It is **nothing on anything that is not a piece**, so it answers `isPart`'s question as well as its own, and like
every other field on a character it is free to read.

```lua
-- Only the head, and only once per blow: the creature's own ask carries no partType at all.
contribute(DECISION.TAKE_DAMAGE, function(e)
	if e.victim and e.victim.partType == "HEAD" then return e.damage + 20 end
end)

-- Or pick one limb out of the list.
on(TIMING.ON_BATTLE_START, function(e)
	for _, p in ipairs(e.unit.parts or {}) do
		if p.partType == "TAIL" then p:takeHealth(20) end
	end
end)
```

**`u.id` is 0 on a piece, and it always will be.** That is the game's answer rather than a gap in ours: a piece is
never given a character number of its own — the number a mod's *appearance* names belongs to whole characters, and
the fight simply never fills one in for a limb. So `partType` is what names a piece, and `whole.id` is what names
the creature it came off. A handler that logs `e.unit.id` on a boss with pieces will print `0` for every one of
them, and nothing has gone wrong.

**It names a piece; it does not promise to be unique.** The game looks a piece up by this type and takes the first
match, so nothing stops a creature having two of something. When you need two pieces told apart for certain,
compare `instanceId` — that is unique per character for the whole fight, on pieces exactly as on everybody else.

#### `parts` and `whole` are the two directions between them

**`u.parts` is the list of pieces a creature is built out of**, and every entry is an ordinary character: it has its
own health, its own effects, and everything you can do to a character you can do to it, `takeHealth` included. For
almost everybody the list is **empty**, and an empty list is a real answer rather than a failure — so `#u.parts == 0`
is "this character has no pieces". Like every optional field it can also be *nothing*, which is the rarer "the
pieces could not be read", so write `ipairs(u.parts or {})` and the two cases look after themselves.

**The list is what the creature has *right now*, in the form it is currently in** — not a fixed roster of everything
it will ever grow. Three things follow from that, and all three surprise people:

* **A piece it has already lost is still in the list.** Destroying a limb does not take it out; it stays, with its
  bar at 0. So the count does not fall as you break the boss apart, and `#u.parts` is not "pieces left standing".
  If you want the ones still up, filter on `p.alive` yourself.
* **A piece it has not made yet is not in the list.** Bosses that grow limbs mid-fight get them one at a time, so
  the count climbs as the fight goes.
* **A boss that changes form drops the whole list and rebuilds it.** For a moment after a transformation a creature
  genuinely has *no* pieces, and then fewer than it will end up with, as the new ones are made one by one.

So read the list at the moment you need it rather than counting it once and remembering the number. If a count ever
looks wrong to you, the log says which of these it was: a line naming the unit, how many pieces the game's own list
held, how many were published, and how many the creature's own data sheet expects for the form it is in.

**`u.whole` is the creature a piece belongs to**, and it is nothing on anything that is not a piece. It is what you
want when a handler was entered about an arm and what you meant was the boss.

```lua
-- Wound every piece of a boss as the fight opens. Reading the list is free; each takeHealth
-- asks the game for something, so a creature made of many pieces does that many times over.
on(TIMING.ON_BATTLE_START, function(e)
	for _, p in ipairs(e.unit.parts or {}) do p:takeHealth(5) end
end)

-- Hurt the boss rather than the limb the blow happened to land on. The guard is what keeps it
-- to once per blow: on a moment that is announced about both halves, the creature's own ask
-- has isPart false and does nothing.
on(TIMING.ON_TAKE_ATTACK_DAMAGE, function(e)
	if e.unit.isPart then e.unit.whole:takeHealth(10) end
end)
```

The two are **one level deep on purpose**: a piece you took out of `u.parts` does not carry a `parts` list of its
own, and a creature you reached through `u.whole` does not either. Both still answer `isPart`. If you need the other
direction from one of those, ask about it at a moment of its own — every piece gets its own moments.

**Neither health bar is a total of the other, and nothing anywhere sums them.** The fight really does keep two
numbers: a blow against a piece spends the *piece's* protection and moves the *piece's* bar, and then hands the
health it took to the creature, which spends its *own* protection and moves its *own* bar. So `maxHp` on a creature
is the creature's bar and never the sum of its limbs, and adding the pieces up yourself would count one blow twice.

**One thing this does *not* affect:** destroying a piece is not a death. `TIMING.ON_DIE` is never announced for one,
and neither is `TIMING.ON_KILL_TARGET` for whoever destroyed it, so a handler counting things it has killed counts
creatures rather than limbs. (A piece that *deals* a killing blow is still the character named as the killer, so
`e.attacker` at `TIMING.ON_DIE` can be a limb.)

**And one that it does: pieces are not in `e.allies` or `e.enemies`.** Those rosters are built from the characters
the fight puts on the field at the start, and pieces arrive by a different route. The creature *is* in the list, so
`e.enemies[i].parts` is how you reach its pieces from there.

**`takeHealth` and `takeSp` aimed at a piece stay on the piece.** A real blow against a limb carries the health it
took through to the creature as well; a script's does not — the piece's bar moves and the creature's does not
follow. So if what you mean is "hurt the boss", aim at the boss: `e.victim.whole:takeHealth(n)`.

---

## Coins

An attack is a run of coin flips, and the coin moments fire once per flip. `e.coin` is **the coin itself** — an
object with four names on it, the same way `e.skill` is an object rather than a skill number:

| | |
|---|---|
| `e.coin.index` | Which coin of the skill this is, counting from zero — the same numbering the skill's own data uses and the Skill Editor shows. |
| `e.coin.result` | How it came down: `"heads"` or `"tails"`, in those spellings. |
| `e.coin.power` | What the skill's Power stands at with this coin counted in. |
| `e.coin.reflipped` | Whether this flip is an extra one rather than the coin's first. |

```lua
on(TIMING.BEFORE_GIVE_ATTACK, function(e)
	if not e.attacker or not e.victim or not e.coin then return end
	addBuff(e.victim, "Laceration", 1 + e.coin.index, 3)   -- later coins hurt more
end)
```

**Check `e.coin` before you read a part of it.** Plenty of moments carry no coin at all — damage from a status
effect ticking at the end of a round is not about a coin — and there `e.coin` is nothing, so `e.coin.index` is an
error rather than a nil. `if not e.coin then return end` is the whole guard; the field is either the object or it
is absent, and there is no falsy-but-present case to worry about the way there is with a number.

**The fields that are plain numbers want the other guard**, and the general rule is worth having in one place:
**write `== nil` when what you mean is "absent" and the value could legitimately be zero.** `0` is true in Lua, so
`if not e.x then` happens to do the right thing there while reading as though it does not — and a guard that is only
accidentally correct is one the next reader has to re-derive. `e.powerBonus` further down is where that bites.

**`power` is a running total, not what the coin was worth.** It is the number a player watches climb as the coins
land. A coin's own contribution is this number minus the previous coin's, and nothing hands you that difference —
if you want it, remember the last one you saw.

**A re-flip keeps the coin's number.** Flip coin 1 twice and `e.coin.index` reads `1` twice, not `1` then `3`. It
is a position in the skill, not a name for one flip, and it starts again at zero for every skill — so it can tell
one attack's coins apart and cannot be remembered across a round. `e.coin.reflipped` is what tells the second flip
from the first.

**A coin can be flipped again from a script.** `DECISION.NEXT_COIN` asks "may the attack move on to the next coin"
— return `false` and this one is flipped again instead:

```lua
gate(DECISION.NEXT_COIN, function(e)
	if e.critical == false then return false end   -- re-flip anything that missed out
end)
```

Three things about it, in order of how much they will cost you:

* **It can only add a re-roll.** One the fight or another script has already decided on happens whatever you
  return — including one a status effect grants, which this decision is never even asked about. So "count every
  re-roll in this fight" is a number that will be quietly short.
* **There is a cap** on how many times one coin may be flipped again, so a handler that always says `false` cannot
  make an attack run forever. It also cannot do what its author meant.
* `e.critical == false` and not `not e.critical`, for the reason in the section above. Get that wrong and you
  re-flip every coin in the fight.

### Before the coin comes down

Everything above happens once a coin has already landed. `TIMING.BEFORE_ROLL_COIN` is the other end of it — the
moment a coin is about to be flipped, while it is still undecided:

```lua
on(TIMING.BEFORE_ROLL_COIN, function(e)
	if e.coin and e.coin.index == 0 and e.skill and e.skill.slot == 3 then
		e.unit:giveShield(3)
	end
end)
```

**`e.coin.result` and `e.coin.power` are nothing here, and only here.** The coin has not come down, so there is no
result to report and the Power is still the previous coin's — both would be answers about a different flip, so
neither is given. `e.coin.index` and `e.coin.reflipped` read as they do everywhere else. If you want to know how a
flip went, `TIMING.ON_END_COIN` is the other end of the same coin.

**It reaches the coins of a clash, which nothing else here is known to.** `DECISION.NEXT_COIN` is asked from
inside the damage calculation, and a clash is settled *before* any of that — the two sides flip, the numbers are
compared, and only then does the winner deal anything. This moment sits on the flip itself, so it sees the lot:
attack coins, clash coins, and the extra flips a re-roll adds.

Two things to hold on to:

* **It does not say whether this flip belongs to a clash.** The fight is not asked that at this point, so the
  moment does not claim to know. `e.rollType` at `DECISION.SKILL_POWER` is where "is this a clash" has an answer.
* **It is the most frequent moment there is** — more than the damage moments, because a clash flips coins that
  never become damage. Whatever you write here runs on every single flip of every fight, so keep it to one test
  and one action.

---

## Power

**Power is the number a coin is flipped with**, and the fight works it out once for each *action* a character takes
rather than once per flip — out of the skill's own Power and everything the character's status effects, passives and
E.G.O. gifts have put on top of it. `DECISION.SKILL_POWER` is where a script joins in on that last part.

```lua
contribute(DECISION.SKILL_POWER, function(e) return e.powerBonus + 20 end)
```

**`e.powerBonus` is the bonus, not the Power.** This is the one decision whose number is not the whole of the thing
it is named after, and it is worth reading twice. What you are shown is what has been *added* — `0` is the ordinary
state, on a character nothing has touched — and the real Power is assembled afterwards out of the skill's own figure
and this. So `return 30` does not say "roll with 30". It says "thirty on top of whatever everyone else decided", and
because a return replaces what you were shown it throws away every buff, passive and earlier mod that had
contributed. Nothing on screen says so; the fight carries on and the rolls are merely wrong. **"Twenty more Power" is
`e.powerBonus + 20`, always.**

The two names say which is which: `DECISION.SKILL_POWER` is the game's own skill-power question, and `e.powerBonus`
is the number actually in your hands.

**And it is signed.** Power Down is an everyday effect, so a negative number here is a normal sight rather than a
sign that something has gone wrong — `-3` means the fight has already taken three off this roll. It is the one
contribute number that goes below zero, and the rule that matters most survives the crossing intact: **handing back
what you were shown changes nothing, under zero as well as over it.** A handler that abstains cannot cancel somebody
else's Power Down. Asking for less is ordinary too: `return e.powerBonus - 2` is a real thing to write, and a result
under zero is not the arithmetic mistake it would be at any other decision.

```lua
contribute(DECISION.SKILL_POWER, function(e)
	if e.powerBonus == nil then return end
	if e.powerBonus < 0 then return 0 end     -- shrug off whatever has taken Power away
end)
```

That guard is `== nil` rather than `not e.powerBonus`, for the same reason the coin guard is: zero is the ordinary
value here and `0` is true in Lua, so `not` happens to work while reading as though it does not. And be clear about
what that handler really does — you are shown one total and not a list, so a character carrying a `-5` and a `+2`
shows `-3`, and lifting that to `0` discards the `+2` along with the `-5`.

**`e.rollType` is how you ask whether this is a clash.** It is one word: `"PARRYING"` for a roll being made in a
clash, `"ACTION"` for an attack nobody is clashing with, `"EVENT"` for a roll the fight asks for outside an attack,
and `"NONE"` for one it does not put in any of those. That is what "only when I am clashing" is written with:

```lua
contribute(DECISION.SKILL_POWER, function(e)
	if e.rollType ~= "PARRYING" then return end   -- ordinary attacks are left alone
	if e.powerBonus == nil then return end
	return e.powerBonus + 10
end)
```

**You cannot see what the other side rolled *here*, and that is the trap in this decision.** "If I am losing this
clash, add 20" is the first thing everybody wants out of it, and it cannot be written at `SKILL_POWER` — not because
a field was left out, but because at this point in the fight there is no instant at which both numbers exist. **A
clash's Power is asked one side at a time.** Whoever rolls first is asked before the other side has rolled at all,
so at that moment the opponent's figure is not a secret, it is not yet a number. Anything built to report it would
answer differently depending on which of your characters happened to roll first, and would sometimes hand back the
*previous* exchange's figure rather than nothing — which tests as working and is wrong in play. So `e.rollType`
says a clash is happening, and nothing at this decision says how it is going.

**That question has a decision of its own**, one step later in the fight, where both rolls are on the table:
`DECISION.CLASH_ROLL`. See the next section. Use `SKILL_POWER` to change what a character rolls *with*, and
`CLASH_ROLL` to react to what the two of them rolled.

**No `e.coin` and no `e.attacker` here.** Power is worked out per action, so this decision is not told which flip it
is for and the coin numbering from the section above does not reach it. If what you want is the Power a particular
coin ended up with, that is `e.coin.power` at a coin moment — the running total after that flip, and a different
question from the bonus being argued about here. And the character rolling is `e.unit`, only
that: a `"PARRYING"` roll belongs to a defensive skill as readily as to an attacking one — **"parrying" is the
game's word for clashing, not for defending** — and an `"EVENT"` roll is not an attack at all, so nothing here
calls the character an attacker.

Being asked once an action rather than once a flip makes this a quieter place to sit than the critical decisions —
still a handful of times a round per character, so it is a `contribute` like any other. Read, return, and act in an
`on`.

---

## Clashing

When two characters aim at each other, the fight does not just compare their attacks — it runs a **clash**: each
side rolls its coins, the two numbers are compared, the loser drops a coin, and it goes round again until one side
runs out. `DECISION.CLASH_ROLL` is asked once per exchange, for each side, **after both sides have rolled and before
the two numbers are compared**.

That timing is the whole point of it, because it is the one place in this API where a handler can see both halves
of a contest:

```lua
-- Losing this exchange? Push back.
contribute(DECISION.CLASH_ROLL, function(e)
	if e.clashRoll < e.opposingRoll then return e.clashBonus + 20 end
end)
```

**Three numbers, and only one of them is yours to change.**

* `e.clashRoll` — what *this* character rolled for this exchange, before any bonus.
* `e.opposingRoll` — what the other side rolled, likewise before its own bonus.
* `e.clashBonus` — the bonus being argued about. This is the one a `return` replaces.

Both rolls are settled facts by the time your handler runs. Neither is a guess, neither is left over from an earlier
exchange, and neither depends on which of the two characters the fight got to first.

**`e.clashBonus` is a bonus, exactly like `e.powerBonus`.** `0` is the ordinary state, negative is ordinary too —
lowering somebody's clash roll is a status effect the game ships — and `return 20` means "twenty on top of nothing",
throwing away every effect that had contributed. **"Twenty more" is `e.clashBonus + 20`.**

**It is asked twice per exchange, not once.** The fight asks each side in turn, so a handler left on the default
scope runs once (for your character), and one written with `{ scope = "any" }` runs twice for the same exchange —
once with your character's numbers and once with the numbers the other way round. Both are real asks; neither is a
duplicate to filter out.

**What you return is a contribution, not the final number.** After every script and every effect has had its say,
the fight still adds a correction of its own when the two sides' skill levels differ, a handful of abilities can
replace an assembled total outright, and a total below zero is treated as zero before the comparison. So a big
enough number wins the exchange in practice, and nothing here *guarantees* it.

**No `e.coin`.** An exchange is a whole roll compared against another roll, not a single flip — the coin numbering
from the section above belongs to attacks, not to this.

**The character is not called the attacker.** Either side of a clash can be holding a defensive skill, so `e.unit`
is simply the one rolling. And on a boss built out of pieces, `e.unit` is the **piece** that is clashing rather than
the creature as a whole — the same distinction the damage decisions have, described above.

If what you want is to react *after* the comparison rather than to change it, `TIMING.ON_WIN_PARRYING` and
`TIMING.ON_LOSE_PARRYING` are announced per exchange, and `ON_WIN_DUEL` / `ON_LOSE_DUEL` once the whole clash is
settled.

---

## Punching through a guard

A character that is aimed at can answer with a **guard, a dodge or a counter**, and a handful of skills in the game
go straight past that answer as though it were not there. `DECISION.DEFENSE_ACTION` is where a script joins in:

```lua
-- This character's third skill ignores whatever the target puts up.
veto(DECISION.DEFENSE_ACTION, function(e)
	return e.skill and e.skill.slot == 3
end)
```

It is a **`veto`**, so `true` suppresses the defence and anything else leaves the fight alone. The fight asks the
question once for **each character a blow reaches**, so on a skill that hits several you are asked several times,
with `e.target` naming the one being asked about that time.

**It is the attacker's decision, and only the attacker's.** The handler runs for the character *swinging* —
`e.unit` and `e.attacker` are both that character — which is what makes "my mod's attacks punch through" a line
you can write. **The opposite is not available**: there is no way to say "my defence cannot be bypassed", because
every refusal in this API only ever moves one way and a suppression the fight has already decided on cannot be
taken back. That is the same rule `gate` and `veto` obey everywhere else, and it lands on this side of the question.

**`e.target` is the character being asked about, not the character the skill was pointed at.** This is the one
decision where those two come apart on purpose: a skill that strikes three characters asks this three times, and
each ask names the one it is about. On a boss built out of pieces that is the **piece** the blow reached.

**There is no `e.victim` here**, and the absence is honest rather than an oversight: at the instant the question is
asked, nothing has landed on anybody. What is being settled is whether the answer to the blow will be allowed at
all.

**Do not confuse it with `DECISION.DEFENSE`**, which is in the next section and is a completely different question.
That one contributes to the *number* a character subtracts from incoming damage. This one decides whether its
defensive *action* happens at all. One English word, two decisions.

---

## Stats, and why they are decisions rather than settings

The companion's **Unit Editor** can force a character's stats outright — its defence, its maximum health — and when
what you mean is "this character is tougher", that is the right tool and a script has nothing to add. What a script
wants is the thing the editor cannot express: **"+20 defence *while it is staggered*"**. That is not a setting. It
is an answer to a question the fight asks over and over, and it is why these arrived as decisions.

The difference is the whole reason this section exists, so it is worth being blunt about. **Forcing a stat**:

* **is permanent** — nothing takes it off, and there is no moment at which a script could put it back;
* **is exclusive** — once a character's defence is forced, every effect on the character stops counting towards it,
  because the game uses the forced figure instead of working one out;
* **is last-writer-wins** — two mods setting it means whichever ran last decides, invisibly, and neither is told.

A contribution has none of that. It is asked **for the one computation**, so a handler that returns more while
`e.unit.staggered` is true has written a conditional buff and nothing has to undo it. It **chains**, so the mod
beside you is shown your number and builds on it rather than replacing it. And it composes with the game's own
effects, because it is a term in the very sum they are terms in.

```lua
contribute(DECISION.DEFENSE, function(e)
	if e.defenseBonus == nil then return end
	if not e.unit.staggered then return end
	return e.defenseBonus + 20
end)
```

Five decisions work this way — defence, speed, level, maximum health and attack weight — and everything the
**Power** section says holds for all of them without amendment:

* **The number is a bonus, not the stat.** `e.defenseBonus` is what has been *added*, and `0` is the ordinary state
  on a character nothing has touched. `return 40` does not say "defence 40"; it says forty on top of everything
  else, with everything else thrown away. **"Twenty more" is `e.defenseBonus + 20`, always.**
* **Every one of them is signed.** Defence Down, Slower, a level reduction and a max-health reduction are all
  everyday effects, so a negative number here is normal rather than a sign something has gone wrong. Handing back
  what you were shown changes nothing, under zero as well as over it — which is what stops an abstaining handler
  from quietly cancelling somebody else's debuff.
* Guard with `== nil` rather than `not`, for the reason the coin guard is written that way: zero is the ordinary
  value and `0` is true in Lua.

### Each one has a catch, and they are five different catches

* **Defence is skipped entirely while a character's own skill level is standing in for it**, which is what happens
  once it has acted. So a defence contribution reaches blows landing on a character that has **not** swung back, and
  is bypassed the rest of the time. That is a real hole rather than an edge case, and whether it matters depends
  entirely on what you were trying to survive.
* **Speed is settled once a round**, asked as the die is about to be thrown. Whatever your handler reads about the
  character there is how it stands as the round *opens* and not at any later point in it. It is also the cheapest
  decision in this document by a wide margin — once per character per round, rather than once per coin.
* **Level moves several things at once.** A character's base defence is worked out from its level and clashes
  compare the two sides' levels, so this is never the small change it reads as. It is also asked often, so keep the
  handler short.
* **Maximum health drags the stagger thresholds with it.** The game recomputes those from the maximum whenever the
  maximum changes; that is its own behaviour for its own max-health effects and not something a script introduces.
* **Attack weight is how many enemies an attack covers**, and the number is how many *more* — `1` means one extra
  enemy on top of however many the skill already hits, not a total of one.

```lua
contribute(DECISION.ATTACK_WEIGHT, function(e)
	if e.attackWeightBonus == nil then return end
	if not e.unit:hasBuff("Agility") then return end
	return e.attackWeightBonus + 1
end)
```

**A widening is a request, and on a few skills it is thrown away.** A handful of the game's abilities replace an
attack's target count outright instead of adding to it, and where one of those is in play the whole assembled figure
goes with it — the skill's own count, the game's own widening effects, and your contribution alongside them. Nothing
inside the handler can tell that it happened, so there is no test to write and no error to read. Ask for the extra
target by all means; do not build a mod whose only idea is that it lands.

**None of these can name a skill**, which is the first thing attack weight invites somebody to try. A handler here
applies to whatever the character is doing, so "only on my third skill" cannot be written. Put the condition on the
**character** instead — whether it is staggered, what it is carrying, how much health it has left — and the handler
means the same thing at every skill it catches.

---

## Criticals

The game keeps three separate questions apart and so does the API.

```lua
contribute(DECISION.CRIT_CHANCE, function(e) return math.min(100, e.critChance + 25) end)
contribute(DECISION.CRIT_DAMAGE, function(e) return e.critDamage + 50 end)
gate(DECISION.CRITICAL,          function(e) return false end)
```

**`critChance` is a percentage from 0 to 100**, not a fraction and not a multiplier. Return 100 and the flip is
certain; return 0 and it cannot happen.

It is the one number with a ceiling of its own, and the ceiling exists *because* contributors chain: two mods each
writing `return e.critChance + 50` against a base of 30 leave 130, and neither of them did anything unreasonable.
Anything past 100 is brought back to 100 and **said so on the Scripts screen**, naming the number you asked for and
the number you got — so the next handler in the chain is never shown a percentage that is not one. That is worth
knowing before you write `e.critChance - 50` somewhere and find it means 80 where you meant 50. The `math.min`
above is not required; it just means the report never appears.

**`critDamage` is how much EXTRA a critical deals, as a percentage on top of the ordinary hit.** A plain critical is
worth 20 before anything modifies it. It is not a doubling, and 400 is a big number but a perfectly sensible one —
there is no ceiling on this one.

**`DECISION.CRITICAL` is the only unoverridable "never".** A contribution to the odds can be raised back by the
next mod or by a vanilla ability that forces a critical; a gate cannot. Use `CRIT_CHANCE` for "make criticals
likelier or rarer" and `CRITICAL` for "this blow is not going to be one".

**All three know who is being aimed at**, which is what makes "crit more against something already bleeding" a line
rather than a wish:

```lua
contribute(DECISION.CRIT_CHANCE, function(e)
	if e.critChance == nil or not e.target then return end
	local bleeding = e.target:buffStacks("Laceration")
	if not bleeding then return end            -- the request was refused; leave the odds alone
	return e.critChance + bleeding * 3
end)
```

Read **Who it is aimed at** before leaning on this. The short version: the odds are worked out once per coin and
applied to everything that coin reaches, so on a skill that hits several, `e.target` is the main one and there is no
per-enemy answer to be had here. The game's own critical maths already reads the target's status effects the same
way - this is the same door, opened for scripts.

---

## SP

Three decisions, one quantity, and the direction is a property of the decision rather than the sign of the number.

```lua
gate(DECISION.RECOVER_SP,      function(e) return false end)                  -- may not regain SP at all
contribute(DECISION.SP_DAMAGE, function(e) return e.sp // 2 end)              -- halve what it is about to lose
veto(DECISION.SP_LOSS,         function(e) return e.unit:hasBuff("Agility") end) -- lose none at all
```

At `RECOVER_SP`, `e.sp` is what the character is being **offered**. At `SP_DAMAGE` and `SP_LOSS` it is what the
character is about to **lose**, as a positive number. All three carry it; every ordinary moment reads nothing
there.

`SP_DAMAGE` and `SP_LOSS` are two decisions on the same seam on purpose. A contribution can be raised back by the
next mod; a veto cannot. "Half as much" is a contribution, "none" is a veto.

**A gate on `RECOVER_SP` that looks ignored usually is not.** A panicking character cannot regain SP at all, and a
low-morale one may not either — so the game had already answered no and no script was asked. `u.panicking` and
`u.lowMorale` are free to read and are the first thing to check.

Those three decisions are about SP the *fight* is moving. Moving it yourself is `giveSp` and `takeSp`, and clearing
a panic is `calm()` — see **Changing a character directly** below, including the part where your own decisions do
not run on SP you moved yourself.

---

## Status effects

A script can give one, take one away, take some off, and ask a character about one:

```lua
on(TIMING.ON_KILL_TARGET, function(e)
	local ok, why = addBuff(e.unit, "Agility", 3, 3)
	if not ok then log("could not:", why) end
end)

on(TIMING.ON_START_TURN, function(e)
	local left = e.unit:buffTurns("Laceration")
	if not left or left == 0 then return end     -- refused, or it has none

	if left == 1 then
		removeBuff(e.unit, "Laceration")          -- last turn of it: take the lot off
	else
		reduceBuff(e.unit, "Laceration", 1)       -- otherwise shave one stack off
	end
end)
```

`buffStacks` is how much of it there is and `buffTurns` is how much longer it lasts; where a character carries the
same effect more than once, `buffTurns` is the longest of them. Both answer `0` for an effect the character does not
have, so neither of them is a way to ask *whether* it has one — that is `hasBuff`.

`removeBuff` takes the effect off entirely and `reduceBuff` takes an amount off. Both report that the game was
*asked*, which is not the same as it having happened: some effects cannot be removed and the game has the final say,
so check with `buffStacks` afterwards if it matters.

**Not from `TIMING.ON_HIT`, however tempting it looks.** All three of these change what a character is carrying, and
that moment belongs to the playback of a round that has already been worked out — so the effect really goes on and
nothing on screen shows it for the rest of the round, and nothing in the fight reacts to it. Nothing refuses and
nothing warns you. **When a blow is supposed to inflict something, write it on `TIMING.BEFORE_GIVE_ATTACK`**, which
is announced while the round is still being decided. See **The round is decided before you watch any of it**.

**Almost everything that reaches into the game answers in two values** — the result, and a sentence saying why not.
Read it with `local ok, why = ...`. Ignoring the second value is fine and normal; ignoring it *while wondering why
nothing happened* is the mistake.

**Names — and the effect you want is probably not called what you think it is.** The word on the status bar and the
word the game files the effect under are two different strings, and for several of the effects you are most likely
to reach for they are not even similar:

| On screen | Write this |
|---|---|
| Bleed | `Laceration` |
| Tremor | `Vibration` |
| Rupture | `Burst` |
| Haste | `Agility` |
| Sinking | `Sinking` |

There is no effect called `Bleeding` at all, and a script that asks for one is told it does not exist rather than
quietly doing nothing. Plenty of others *are* the obvious word — `Sinking`, `Charge`, `Protection`, `Paralysis` —
which is precisely what makes guessing feel like it works right up until it does not.

**So take the name from the editor's completion rather than from the screen.** It offers the game's own name and
shows the on-screen word beside it, so you find `Laceration` by typing what you know it as, and where the game knows
the answer the popup also says how many stacks and turns the effect can hold. The on-screen word does work — the
name is matched against the key first and the translated name second — but it is **translated**, so a script that
says `"Bleed"` stops finding the effect the moment somebody plays the game in another language. Capitals, spaces and
punctuation are ignored either way, and a number works too if you have one — including the large ones your own mod's
effects are given, which are the numbers the companion shows beside them. Only a negative is turned away out of
hand, because an effect number is a position in a list and there is no such thing; above that the game answers for
itself, in its own words, if it does not know the number.

An effect your own mod declares under `buffs` in `mod.json` is addressed by the key you gave it there, and it is
looked for first, ahead of everything the game ships.

**Every effect has a ceiling on both numbers, and the game will not go past either.** What you get is not what you
asked for but what was left underneath the ceiling — ask for eight stacks of something capped at five that already
has four, and one is what lands. Asking for too much is not an error: you get the `true`, *and* a sentence saying how
much less you got and why, which is the second value again. So a mod that quietly stops climbing at five is telling
you so, if you read it.

For an effect **your own mod declares**, the ceiling is the `maxStack` and `maxTurn` you wrote in `mod.json` — so
when a script cannot get past a number, that is the first place to look, and changing it there is the fix. For one
the game ships, the ceiling is the game's and a script cannot raise it.

**Applying an effect floats its own line.** You do not need to announce it; that happens the same way it does for
the game's own effects.

---

## Passives

A passive is one of the standing abilities a character brings into a fight — the ones written on its page rather
than chosen each turn. Most of them do not simply run: they wait for a requirement, usually a number of the chosen
skills sharing a sin, and they act only while that requirement is met. A script can read all of that, and it can
overrule it in either direction.

**There are two families and a script can see both.** A character's *own* passives are its own and its equipped
E.G.O.s'. A *support* passive is one the bench is supplying — it belongs to the whole side and to no character on
it. `passives()` hands back both, from whichever character you are holding, and each entry says which it is:

```lua
on(TIMING.ON_BATTLE_START, function(e)
	for _, p in ipairs(e.unit:passives()) do
		log(p.id, p.name, p.active and "on" or "off", p.support and "(from the bench)" or "")
	end
end)
```

Each entry carries its number (`id`), what the game calls it (`name`), whether it counts right now (`active`),
whether a script has forced it (`forced`), which family it is in (`support`), whether it came from an E.G.O.
(`ego`, and `egoId` for which one), and what it is gated on (`needs`). The full list is in the reference under
**What a passive is**.

`support` is a fact about **the passive**, not about where it happens to be listed — a support passive answers
`true` wherever it turns up. That matters because it is what decides how far a `forcePassive` reaches: read it
before you write one.

**`needs` is the requirement itself.** Each entry names a sin, how much of it is wanted, and which kind of
requirement it is — `"resonance"` for how many of the chosen skills share that sin, `"affinity"` for how much of it
the side has banked. A passive gated on nothing at all has an empty `needs`, which is a real answer, so check its
length rather than treating empty as a failure:

```lua
for _, n in ipairs(p.needs) do
	log(p.name, "wants", n.count, n.sin, n.kind)
end
```

### E.G.O. passives are not there from the start

This is the one thing about the list that catches people out, and it is worth reading before you decide something
is broken.

**A character does not carry its E.G.O.'s passives until it has used that E.G.O.** They arrive at the start of the
round *after* the one it was used in. Until then the E.G.O. contributes nothing to `passives()` — and an E.G.O.
whose passive has not met its own unlock requirement, which is usually a matter of how far that E.G.O. has been
raised, never contributes anything at all.

So an E.G.O. passive that is not there yet **is not a row saying `active = false` — it is no row at all**, and
`hasPassive` answers `false` for it. That is a different thing from a passive sitting dormant waiting on sins, and
the two look identical from inside a script unless you know which you are looking at.

Two ways to work with that:

* **Turn on the "Enable EGO passives" setting.** A team then has its E.G.O. passives from the first round, without
  using the E.G.O. and whatever it has been raised to. This is the reliable way to write and test against them.
* **Or write for both states.** Check `#e.unit:passives()` or look for the `egoId` you care about, and treat "not
  there" as its own case rather than assuming a row exists to read.

`egoId` tells you which E.G.O. brought a passive, and only an E.G.O.'s passive has one — so it doubles as the way
to ask:

```lua
on(TIMING.ON_START_ROUND, function(e)
	for _, p in ipairs(e.unit:passives()) do
		if p.egoId then log(p.name, "came from E.G.O.", p.egoId) end
	end
end)
```

### Asking about one

`hasPassive` and `passiveActive` are the two halves of the question, and they are worth keeping apart:

```lua
if e.unit:hasPassive(40001206) and not e.unit:passiveActive(40001206) then
	log("it has it, and it is sitting dormant")
end
```

A character carries its passives from the moment it arrives, so `hasPassive` is true long before anything happens.
`passiveActive` is the one that says whether the fight is letting it act — and that is the same answer the fight
itself acts on, so a passive reading `false` there is genuinely doing nothing.

### Making one count, or making one stop

`forcePassive` takes the decision away from the fight:

```lua
on(TIMING.ON_START_ROUND, function(e)
	if e.unit:buffStacks("Laceration") >= 5 then
		e.unit:forcePassive(40001206, true)      -- counts, whatever its requirement says
	else
		e.unit:forcePassive(40001206, nil)       -- your opinion off: the fight decides again
	end
end)
```

**Three answers, not two.** `true` makes it count whether or not its requirement is met. `false` makes it *not*
count even where it would have. `nil` takes your opinion back off and hands the decision to the fight. Those are
three different things, so leaving the second part off entirely is refused rather than guessed at.

This changes what the fight *does* and not merely what it shows: a passive made to count acts, and one made not to
count is skipped along with everything it would have done. It takes hold at once — you do not have to wait for
anything.

**It lasts for the round it was asked in.** It is dropped when the round finishes playing out, so say it again from
a moment that comes round every round — `TIMING.ON_START_ROUND` or `TIMING.ON_BATTLE_START` — if you want it to keep
holding. Saying it again is harmless — the same answer twice is the same answer, and it replaces rather than
stacks, which is why the example above is written that way.
Nothing carries over on its own: a character can be rebuilt between rounds, and an opinion left lying about would
end up being answered for by whoever stood there next.

**A support passive is a decision about the whole side.** That is what such a passive is: it is not attached to the
character you named, it is being supplied to everyone. Forcing one on or off changes it for the entire team. One of
the character's own applies to that character alone. `p.support` on the entry tells you which you are about to do,
and it is worth reading before you write the line.

### Passives are numbered, not named

Unlike a status effect, a passive is named by its **number**:

```lua
e.unit:forcePassive(40001206, true)      -- yes
e.unit:forcePassive("Lavacrum", true)    -- refused, with a sentence saying why
```

A status effect has a name the game keeps in one language, so a script can compare against it anywhere. A passive
has only a line translated per language — comparing against that would work for you and for nobody else. `p.name`
is there to be *read*; `p.id` is there to be compared, and `passives()` is where you find the number.

**A number that names no passive on this character, and none its side supplies, is refused** rather than quietly
stored. So is a passive that is in the game's tables in name only, carrying no behaviour of its own. Read the
second value if a line is not doing what you expect:

```lua
local ok, why = e.unit:forcePassive(40001206, true)
if not ok then log("no:", why) end
```

---

## Which skills a character gets offered

Every character carries a **pool** of skills — a bag of numbers. At the start of a round the fight reaches into
that bag at random and pulls out the skills it offers you, plus one it holds back for the round after.

**Copies in the bag are the odds.** That is the whole of the mechanism; there is no percentage anywhere in it. A
character whose bag holds its second skill twice and each of the others once is twice as likely to be offered the
second. So "make this character use its third skill more" and "put another copy of its third skill in the bag" are
the same sentence, and the second one is writable.

```lua
on(TIMING.ON_START_ROUND, function(e)
	for _, id in ipairs(e.unit:skillsByTier(3)) do
		e.unit:setSkillCopies(id, 3)      -- three copies of it in the bag, from now on
	end
end)
```

**`TIMING.ON_START_ROUND` is the moment for all of this.** It comes round before every round's skills are drawn,
so a pool changed there is the pool the fight actually draws from, and it comes round *again* every round, which is
how a change is kept up. Nothing here carries over on its own — a character is rebuilt between waves and between
fights.

### Naming a skill

Skills are numbered, not named — the same rule as passives, for the same reason: a skill's name is one translated
line per language, so a script comparing against it would work for its author and nobody else. There are three
places to read a number off:

```lua
e.unit:skillsByTier(2)    -- this character's second-tier skills, whoever it is
e.unit:skillPool()        -- everything in the bag, as skill number -> how many copies
e.skill.id                -- the skill a handler is running for
```

**`skillsByTier` is the one that makes a script portable.** Every character's skills have different numbers, so
"this character's second skill" is only writable at all because of it. It takes 1, 2 or 3 — the number printed on
the skill card, which is the same number `e.skill.slot` carries.

**It hands back a LIST, and that is not tidiness — a character can have more than one skill at the same tier.**
Two different third skills is an ordinary thing for a character to have. A single answer would have to pick one of
them and say nothing about it, leaving the other unreachable, so you get all of them, lowest number first:

```lua
local threes = e.unit:skillsByTier(3)
if #threes > 0 then log("first of them:", threes[1]) end
if #threes > 1 then log("and there is another:", threes[2]) end
```

It is a list even when there is one in it, so read it the same way every time. **An empty list is a real answer** —
not every character has all three tiers. And it looks in the *pool*, so a skill you removed with
`setSkillCopies(id, 0)` is no longer listed.

**The skill number is the currency; the tier is only how you discover it.** Every verb that changes something takes
an id, never a tier — there is no way to say "the second one of the tier-3 skills" to a write verb, because that
ordinal is not something you could know in advance.

### Looking before you write

Five things you can ask, and the writing verbs are much harder to use without them:

```lua
local pool    = e.unit:skillPool()      -- what it HAS:  { [skillId] = copies }
local left    = e.unit:drawPile()       -- what is LEFT of it this round, duplicates kept
local showing = e.unit:offeredSkills()  -- what is on its panel right now
local waiting = e.unit:readySkills()    -- what it has already drawn for NEXT round
local firsts  = e.unit:skillsByTier(1)  -- every skill it has at tier 1
```

**`skillPool` is a photograph, not the bag.** The fight builds it fresh for the question, so writing into what you
get back changes nothing at all. It is keyed by skill number, so `pool[id]` answers "how many of that one?"
directly and `for id, copies in pairs(pool)` walks the lot — it is not a list, so `#` on it means nothing.

**`drawPile` is the other half of `skillPool`.** One says what the character owns; the other says what has not been
drawn yet this round. Duplicates are kept in it, because the duplicates *are* the copies still to come. It is a
plain list — walk it with `ipairs`.

**`readySkills` is empty until a round has been played,** and this catches everybody once. The skill for the round
after is drawn *as the current one is used*, so on the first round there is nothing waiting. That matters because
the two verbs below need something to be waiting.

An empty answer from any of these is a real answer, not a failure. Check the length rather than treating empty as
broken.

### Deciding what comes next

```lua
on(TIMING.ON_START_ROUND, function(e)
	local firsts = e.unit:skillsByTier(1)
	if #firsts > 0 then
		local ok, why = e.unit:setReadySkill(firsts[1])
		if not ok then log("not this round:", why) end
	end
end)
```

`setReadySkill` puts a skill into the slot for next round whatever the character drew. `swapReadySkill(from, to)`
does the same but only where the character drew one particular skill — use `setReadySkill` when you want the slot
to hold something regardless, and `swapReadySkill` when you want to intercept *one* draw.

**Neither of them changes the pool.** The skill they replace goes back into the pile, so these decide the ORDER and
not the contents.

### Weighting the bag, and taking a skill out of it

`setSkillCopies(id, count)` says how many copies of a skill the pool should **end up** holding. It is not how many
to add, which is what makes it safe to say every round: stating the same number twice is the same as stating it
once.

```lua
e.unit:setSkillCopies(id, 4)    -- four copies: offered far more often
e.unit:setSkillCopies(id, 1)    -- back to one, however many there were
e.unit:setSkillCopies(id, 0)    -- gone. The character is not offered it again.
```

**`0` is a real instruction and it really removes.** The skill leaves the pool and leaves what is left of it this
round, so the character stops being offered it from that moment. That is the way to say "this character can never
reach that skill again".

**One call is refused outright: one that would leave the pool with nothing in it.** A character with an empty pool
has nothing for the fight to draw and no way to say so, so it is not allowed to happen — leave it at least one
skill. If what you want is "none of the old skills, only this one", swap them rather than removing them.

The verb hands back **how many copies the pool really holds now**, read back off it rather than echoed:

```lua
local now, why = e.unit:setSkillCopies(id, 3)
if now ~= 3 then log("did not take:", why) end
```

### Two ways `setSkillCopies` quietly does nothing on the way UP

Both are the game's rules rather than ours, and both are invisible from inside a script:

1. **The number names a skill this character does not carry.** A character can only be given more of what its own
   data already knows about.
2. **A pool is kept one sin at a time,** and the character has no skill of that sin for the new one to go beside.
   A character with no Wrath skills cannot be given a Wrath one, however valid the number is.

The sentence that comes back says which of the two it was. **Asking for more of a skill the character already has
always works** — that is the case worth building on. Coming back *down* has no such catch.

### Replacing a skill outright

```lua
-- everything that was skill 1 is skill 3 from now on, same number of copies
local ones, threes = e.unit:skillsByTier(1), e.unit:skillsByTier(3)
if #ones > 0 and #threes > 0 then e.unit:swapSkill(ones[1], threes[1]) end
```

With no slot named this is the whole character: every copy in the pool becomes the new skill, **the number of
copies is kept**, and what the character is currently holding and has drawn changes with it. Name a slot instead —
`e.unit:swapSkill(from, to, 1)` — and only what that slot is holding changes, leaving the pool alone.

**This is the other way to stop a character having a skill, and the difference from `setSkillCopies(id, 0)` is the
whole reason both exist.** `setSkillCopies(id, 0)` takes the copies *out* and the pool gets smaller.
`swapSkill(from, to)` turns them into something else and the pool stays exactly the size it was. When a character
has only one skill left, the second is the one that still works.

**The skill you swap *to* should be one that character owns.** Nothing stops you passing a number belonging to
somebody else and the fight will try, but it is not something this character is built to use.

### From, then to

Every verb here that names two skills takes them in the same order: **the one that is there now, then the one that
replaces it.** `swapSkill(from, to)`, `swapReadySkill(from, to)`. Getting a pair like this backwards is a silent
kind of wrong — the call is perfectly well formed, it matches nothing, and it does nothing — so it is worth
reading the line back once.

### Every one of them says whether it did anything

The four verbs that change something can all do nothing at all, for reasons a script cannot see from outside. None
of them reports what it *asked* for: each reads the thing it is about to change, asks, reads it again, and answers
out of what it finds.

The three that decide what a slot is holding — `setReadySkill`, `swapReadySkill`, `swapSkill` — answer yes or no:

```lua
local ok, why = e.unit:setReadySkill(id)
```

* `true` means it really changed something.
* **`false` is a real answer and not a refusal** — the call was made and there was nothing to change, which happens
  far more often than it sounds. It always comes with a sentence saying why.
* `nil` plus a sentence is a refusal: the call was turned away and the fight is untouched.

`if not ok then` catches both of the last two, which is usually what you want. `if ok == false then` separates them
when it is not.

`setSkillCopies` answers with a number instead — **what the pool holds now** — so you compare it against what you
asked for. `0` there is a success when `0` is what you asked for, and every number is truthy in Lua, so
`if e.unit:setSkillCopies(id, 0) then` is true either way. Test the value, not its truthiness:

```lua
local now, why = e.unit:setSkillCopies(id, 0)
if now == nil then log("refused:", why)
elseif now ~= 0 then log("did not take:", why) end
```

---

## Changing a character directly

Beyond status effects, a script can move a character's health in either direction, its protection and its SP, and
can bring one out of panicking. They read as one family because they are one idea — hand something over, or take it
away. `takeHealth` is the one that can end a character, and it has a section of its own below:

```lua
on(TIMING.ON_KILL_TARGET, function(e)
	local got = e.unit:giveHealth(15)
	log("healed", got)                  -- what really went on, not the 15
end)

on(TIMING.ON_START_ROUND, function(e)
	e.unit:giveShield(20)               -- gone again when this round ends; see below
end)

on(TIMING.ON_LOSE_DUEL, function(e)
	e.unit:takeSp(4)
end)
```

**What comes back is what happened, not what you asked for**, and that is the point of the family rather than a
detail of it. A heal into a character already at full health is `0`. One into a character with a healing effect on
it can be *more* than you offered, because the fight's own healing modifiers apply to a script's heal exactly as
they do to the game's. `giveSp` into a panicking character is `0`, always. `takeShield(50)` against 12 points of
protection is `12`, and it never spills over into health. `takeHealth(20)` against a character behind a shield is
`0`, with the shield spent. `giveShield` is the one that is always exactly what you asked, because protection has no
ceiling to come up short against.

So read the number:

```lua
on(TIMING.ON_END_TURN, function(e)
	local got, why = e.unit:giveHealth(30)
	if got == 0 then log("nothing to heal:", why or "already full") end
end)
```

**A `0` is a real answer and not a failure** — the call was made and the game had nowhere to put it. A refusal is
nothing plus a sentence, which is a different thing, and `0` is truthy in Lua so `if e.unit:giveHealth(20) then` is
true for both. Test what you actually mean.

**And not one of these from `TIMING.ON_HIT`.** That moment belongs to the playback of a round the game finished
working out before it drew any of it, so a heal, a shield, a drain or a `calm` asked for there is real in the fight
and **invisible for the rest of the round** — the bars were pushed their numbers already — and nothing on the field
reacts to it. It reports success either way, which is what makes it expensive to find. **`TIMING.BEFORE_GIVE_ATTACK`
is the moment to reach for when a blow is meant to move a number.**

`calm()` is the odd one: it takes nothing, and it answers `true` for *asked*, because the game's own method reports
nothing back. Read `u.panicking` first — it is free — and know what it does **not** do: **it clears the panic and
changes nothing else.** A character panics at the bottom of its SP range, so on its own this leaves one standing
there calm and empty. Pair it:

```lua
on(TIMING.ON_START_TURN, function(e)
	if e.unit.panicking then
		e.unit:calm()
		e.unit:giveSp(20)
	end
end)
```

It does not touch low morale either, which is the separate milder state — `u.lowMorale`.

`u.shield` is the readable half: how much protection a character is carrying, every kind of it added up. **`0` is
the ordinary answer** — most characters carry none most of the time — and *nothing* is the different one, meaning
there was no battle state to read.

### Hurting a character

`takeHealth` is `giveHealth` the other way round, and it is the one verb in the family that can end somebody. It is
what makes "this effect takes a bite out of the character carrying it" writable at all — before it, the only way a
script could reduce health was to answer a damage question the fight was already asking, on a blow, which is no help
at the end of a turn.

```lua
on(TIMING.ON_END_TURN, function(e)
	local n = e.unit:buffStacks("MyDebt")
	if n > 0 then e.unit:takeHealth(n * 3) end
end)
```

**It is the same call the game's own bleeding and burning effects use.** Not something near it — the same one. So it
behaves the way those do, all the way down:

* **Protection soaks it first, and is spent.** A character behind a shield loses shield, not health, exactly as it
  would against a real hit. That is why the number can come back `0` with the shield gone.
* **The fight decides what the damage should really be.** Everything on the character that changes incoming damage
  gets its say, so what you asked for is a *proposal*. It can come back smaller, and it can come back bigger.
* **What you get back is the damage that reached HEALTH**, which is the half the verb is named for. When protection
  took some of it you also get a sentence saying how much went where.

**And it can kill.** That is the part worth reading twice before you write one.

**A kill here is an ordinary kill.** Health reaching zero through `takeHealth` is processed exactly as health
reaching zero through a sword: the character dies, everything in the fight that watches for a death is told, and the
fight ends when that was the last one standing. Nothing is left half-done and there is no corpse left standing.

Two things about it are not obvious:

* **Anything that keeps a character alive at 1 health keeps it alive against this too.** The fight asks that question
  here just as it does for a real blow. Your number still reads as the damage the fight settled on — the character
  simply survives on 1.
* **Your own handlers do not run for it.** A script cannot interrupt itself, so your
  `contribute(DECISION.TAKE_DAMAGE)` gets no say in damage you caused yourself, and `TIMING.ON_DIE` and
  `TIMING.ON_KILL_TARGET` do not fire for a death you caused from inside a handler. The game's own effects all run
  normally; only scripts sit that one out. This is the same rule `takeSp` and `giveSp` already live under.

**The number it hands back is the damage, not the health that moved.** A character with 5 health left takes the whole
number and dies — the game does not report having stopped at the bottom. A character that is already down cannot be
hurt again, and the number will not say so either. `e.unit.hp` answers both, for free, before you ask.

**When can you use it?** From any moment, like the rest of the family. One is worth a word: at `TIMING.ON_ADD_UNIT`
the character is still arriving and the fight has not begun, so hurting one there is fine and is how you make one
show up already wounded — but taking one all the way to zero there asks the game to process a death before it has
finished putting the fight together. `TIMING.ON_BATTLE_START` is the first moment where a kill stands on ordinary
ground.

**A number floats for it**, under the same rule as `giveHealth` — see *The floating `+N` follows the moment, not the
verb* below. The figure drawn is the game's own damage figure, which includes the part protection soaked; the figure
handed back to you is the health half. At the few moments that carry no such record the damage lands silently, in
full.

### How big a number you may ask for

**As big as you like. There is no ceiling of ours on any of these.** `giveShield(90000)` grants ninety thousand
protection, exactly: the game stores it as the plain number it is handed and nothing on the way in trims it. The
same is true of stacks and turns on an effect, and of the numbers `showBuff` puts on screen.

The limits you will actually meet belong to the **game**, and telling those apart from ours matters — you cannot
change ours, and you can sometimes change these:

* **An effect's own maximum stacks and turns**, which is the ceiling described under **Status effects** above. For
  an effect your own mod declares it is the `maxStack` and `maxTurn` you wrote in `mod.json`, so that is where you
  change it. For one the game ships it is the game's, and a script cannot raise it.
* **The two healing verbs go strange at absurd numbers.** The game works a heal out through a fraction of its own,
  so past about sixteen million the figure stops being exact, and a heal in the billions is far enough outside what
  that arithmetic can carry that it restores **nothing at all**. That boundary is roughly a hundred times any
  character's health and no number a script means comes near it — it is written down only because "my heal did
  nothing" is otherwise an evening spent looking in the wrong place.

**A number with a fraction on it is rounded down**, so `giveShield(e.unit.maxHp * 1.5)` is an ordinary thing to
write and `20.7` means `20`. That is exactly the arithmetic people reach for once nothing is capping them, so it is
worth knowing it lands rather than being refused.

**Zero means different things to the two halves of this family, deliberately.** `addBuff` and `reduceBuff` take `0`
happily — "add no stacks" is a request the game can carry out — and turn a negative into `0`, because they add and
there is a separate name for taking away. The giving and taking verbs **refuse** both, with a sentence: a `0` there
is nearly always arithmetic in the script that came out wrong, and handing nobody nothing in silence is how the real
cause stays hidden.

### Two kinds of protection

`giveShield` takes an optional second argument, and it decides how long the protection lasts:

```lua
e.unit:giveShield(20)         -- temporary: cleared when the round ends
e.unit:giveShield(20, true)   -- lasting: not cleared there at all
```

**The temporary kind is what you get when you say nothing, and it is cleared AT THE END OF THE ROUND** — after every
character has taken every turn. That is the default deliberately: it is exactly what the game's own shield effects
hand out, so a script that says nothing grants what a vanilla effect grants. The lasting kind is stronger than
anything a buff gives out, and has to be asked for.

**It is not cleared on a turn change**, and that sentence is the one worth remembering, because "the shield
disappeared on turn change" is what it looks like from the outside and is what somebody will type into a search box.
A round contains several turns (see **Rounds and turns are not the same thing**), so protection given at the start
of a round survives every action in that round and is gone by the time the next one starts. If a shield is going
*earlier* than that, something took it — a shield-strip, or a blow spending it — and the round change is not your
culprit.

The clearing is also **gated on the game's own "keeps its shield" effects**. Where one of those is in play the game
does not sweep at all, and a script's temporary shield gets the same reprieve — it is the same kind of shield the
game's own effects grant, so it is covered by the same protections they are.

Everything else about the two is the same:

* **Both soak damage identically**, and **the temporary one is always spent first** — so a character holding both
  loses the perishable half to the next hit and keeps the half that was going to last anyway.
* **Both can be stripped**, by the game's shield-stripping effects and by `takeShield`.
* `u.shield` is the two added together and cannot tell them apart. Nothing else can either, so if the difference
  matters to your mod, keep your own note of what you handed out.

Whichever kind you ask for, the number that comes back is the amount granted and is always exactly what you asked —
the ceiling that makes the rest of this family worth reading does not exist for protection.

**The second argument is an ordinary Lua truth test, so `giveShield(20, 0)` gives the LASTING kind** — `0` is true
in Lua. Nothing here invents a stricter rule than the language's, which would only make this argument behave unlike
every other condition you write. Pass `true`, or leave it out.

### The fight reacts to what a script does

This is the part with consequences beyond the character you touched. A script's action is announced to the fight
**as having happened at the moment your handler is running in**, so the rest of the fight can respond to it: an
effect that watches for something happening at the end of a turn genuinely fires when a script heals at the end of a
turn, and a heal at `ON_WIN_DUEL` is a heal-on-winning-a-clash as far as everything else on the field is concerned.
That is worth knowing in both directions — it is what makes a script's action feel like part of the game, and it is
how a script sets off somebody else's effect without meaning to.

**The moment carried is `e.timing`, and where there is no moment nothing reacts.** That is the whole rule, and you
can check it from inside the handler:

```lua
if e.timing then
	-- whatever this does, the fight will treat it as happening here
end
```

**Which of them carry one follows a shape rather than a list.** A decision that is about a number the fight is
already moving — damage, SP, Power — is told which moment it is moving it at, and carries that moment through. A
decision that is a standing question about a character rather than about an event — may this one regain SP, may this
blow crit, how much defence has this one got — was never asked at a moment and carries none. `TIMING.ON_HIT` carries
none either, and for its own reason: it is ours rather than the game's, raised at the frame a blow lands, and the
game's own moment for that blow went past seconds earlier. That is half of why nothing you change at `ON_HIT` has
any consequence beyond the character you touched — and the other half is that nobody sees it either.

Act where there is no moment and the request goes in marked as belonging to no particular one: it is completely
real, the health or SP moves, and nothing keyed on a moment fires. `if e.timing then` is one line, is right in every
build, and cannot go stale — which is why it is worth writing instead of remembering which side a decision falls on.
And it is one more reason the advice above stands: a `contribute` handler should read and return, and do its acting
in an `on`.

### Your own decisions do not run on what you did yourself

A handler that calls `giveSp` does not re-enter its own `gate(DECISION.RECOVER_SP)`, and the same holds for the
other verbs and their matching channels — `takeSp` does not run your `contribute(DECISION.SP_DAMAGE)` or your
`veto(DECISION.SP_LOSS)`, and `takeHealth` does not run your `contribute(DECISION.TAKE_DAMAGE)`. The game's own
answer stands for anything a script does itself.

**Moments a script causes are the same story.** If `takeHealth` kills somebody, `TIMING.ON_DIE` and
`TIMING.ON_KILL_TARGET` do not fire for that death — the death itself is completely real and every one of the
*game's* own on-death effects runs; it is only your handlers that sit it out.

That is deliberate and it is usually what you want: a mod that blocks SP recovery can still hand SP out on its own
terms, without having to write an exception to its own rule. It does mean the two halves of such a mod are
independent, so a change to the gate does not change what the script itself gives.

### The floating `+N` follows the moment, not the verb

`giveHealth`, `giveSp` and `takeHealth` put the game's own number over the character at most moments. It is written
into the same record the fight keeps of its own effects for the moment your handler is running in, and drawn along
with them — so it appears at `ON_BATTLE_START`, at the per-character moments like `ON_START_TURN` and `ON_END_TURN`,
and at the ones a blow raises. A few moments keep no such record, or keep one the game does not draw numbers out of,
and at those the health, SP or damage goes on **silently**. It lands in full either way: only the number is ever
missing, and it is never drawn on the wrong character.

`takeSp` shows nothing, and that is the game's own behaviour rather than a gap: SP damage has no floating number
anywhere in this game.

`giveShield` shows nothing at all, and that is parity rather than a gap — the game's own protection is silent too,
it simply appears on the bar.

`showBuff` is the tool for putting something on screen at a moment of your own choosing, whatever the game does.

---

## The four things a script puts on screen deliberately

**A script decides; the timeline presents.** Because the round is settled before any of it is drawn — see **The round
is decided before you watch any of it** — a script has no way to know which moment on screen its call belongs to, and
that is why there is no way to move the camera, play a sound or say a line from one. Those are beats on a coin,
authored on the **Animation Timing** screen where you can see them against the artwork.

Changing a character is a different matter and is not what this section is about: a heal moves the health bar and
`giveShield` puts protection on it, because the bars draw whatever the fight currently says. **The change is real
straight away; the picture catches up on the game's own schedule.** Ask at an ordinary moment and the picture for
that round is built afterwards, so it shows. **Do not ask at `TIMING.ON_HIT`** — that moment belongs to the playback,
the picture of the round was finished before it came round, and the change sits there unshown for the rest of the
round while the call reports success. `TIMING.BEFORE_GIVE_ATTACK` is the one to reach for. Neither verb is a way of
*saying* something, which is what the three below are for.

The first three are **held and landed at a moment that is really on screen**, rather than at the moment the handler
ran, so none of them picks a moment either. The fourth does not need that treatment, because what it puts on screen
stays there — see **An effect that stays up** below.

**`showBuff`** floats the game's own status-effect line — a symbol and a number that fades. It applies nothing.
Asked for away from the screen it is held and drawn on the blow it belongs to; asked for from `TIMING.ON_HIT` it
draws straight away, because that moment *is* the blow.

Its last argument, `text`, is what to say **instead of** the effect's name — and when you give it, **the amount is
not drawn either.** That is the whole point of it. The number the game puts on that line means *this much of the
effect was gained*; if what you want to show is what the effect just **did**, that number is answering a different
question, and two numbers on one line with one of them wrong is worse than either alone. So the words replace the
lot, and the symbol stays:

```lua
-- the effect's own line: the Laceration symbol and "+2"
showBuff(e.victim, "Laceration", 2)

-- the same symbol, saying what it cost instead
showBuff(e.victim, "Laceration", 1, 0, 37)

-- and the symbol on its own, with nothing beside it
showBuff(e.victim, "Laceration", 1, 0, "")
```

An **empty** `text` is a real answer and means the symbol alone. **Leaving the argument out is not the same thing** —
left out, the game writes the line as it always did. A number works as well as words: it is written out exactly as
`tostring` would write it. Keep it short. It is a small line that fades while it rises, so a few characters is what
anyone reads; a long one is drawn anyway, with a note back saying the tail was cut.

Because the words replace the whole line, the `amount` and `turns` before them stop being drawn — pass `1, 0` and
give the words. The `1` is there to say *one line*, not to be read.

**`showOnBanner`** writes a status row onto the panel that rises over a character while their skill plays — the one
carrying the skill's name, the coin, and a row for each thing bearing on the attack. It applies nothing either.

```lua
-- the shape to reach for: each coin announces itself as it is rolled
on(TIMING.BEFORE_ROLL_COIN, function(e)
	if not e.coin then return end
	showOnBanner(e.unit, "Ember Debt", nil, 1 + e.coin.index, "Laceration", e.coin.index)
end)

-- or one row for the whole skill, on every coin panel of that character's
on(TIMING.ON_START_TURN, function(e)
	showOnBanner(e.unit, "Ember Debt", "burns deeper on the next hit")
end)
```

**This is the one place the game explains a skill**, and it is built from the skill's *own declared* effects. An
effect your script inflicts is not one of those, so it is invisible there however well it works — the player sees the
counter go up on the status bar and is never told which skill did it. `showOnBanner` is how you say so.

* **That panel is two different things, and there is a name for each.** The wrapping full-width lines at the top —
  *"[On Hit] Trigger Tremor Burst; then, …"* — are the **skill's own text**, written in the voice the game uses to
  explain a skill. Under them sit the **status rows**, one per effect bearing on the attack — a symbol, a short name,
  a number. `showOnBanner` writes one of *those*, so your row reads like *"Ember Debt 8"*, the way an effect in play
  reads there: the row that says *"this is affecting the attack"*. For the wrapping lines above them, use
  `showSkillLine` — it is the next section, and it is the one that can carry the game's own
  colouring.
* **The title is the part that always draws.** The sentence is a bonus. See the layout note below for why.
* **File it against the character whose turn it is** — normally `e.unit`. The panel belongs to whoever's coins are
  being tossed, so the row shows on *that* character's panels and nobody else's: their attack, and their guard or
  evade toss if they make one in the same round. Filed against a character who does nothing at all that round, it is
  dropped rather than held over to the next one — the row is about this round's action, and an announcement that
  arrives a round late is worse than one that does not arrive.
* **The title is which row this is — so keep it the same every time.** Ask again with the same title, on the same
  character, for the same coin, and you *update* that row — its sentence, its number, its symbol — rather than adding
  a second copy. That is how the game treats its own rows: an effect gaining a stack replaces its row instead of
  growing a new one. So a handler that runs once per coin can announce the same thing every time and the player still
  sees one row.

  **The commonest way to get this wrong is to build the title out of a number that moves.** `"Roll " .. n` is a
  different row on every call, and the panel fills up with them. Put the number that changes in `count` — that is
  exactly what it is for — and leave the title alone:

  ```lua
  showOnBanner(e.unit, "Roll", nil, n, "Laceration", e.coin.index)   -- one row, its number changing
  showOnBanner(e.unit, "Roll " .. n, nil, 1, "Laceration", e.coin.index)  -- a NEW row every time
  ```

  Different titles really are different rows, and so is the same title on two different coins. There is no limit on
  how many you may have.
* **A row lasts as long as the skill use it was filed during.** When that character's action is over the row is
  finished with it — it does not carry over into their next one, even if that happens moments later in the same
  round.
* **Each coin can say something different, and `TIMING.BEFORE_ROLL_COIN` is how.** That moment comes round once per
  coin, just before it is rolled, and hands you `e.coin.index` — so a handler there can announce the very coin whose
  roll it is watching, in a clash as well as on an attack. Pass that NUMBER as the sixth argument and the row appears
  on *that* coin's panel only. It wants the number and not the coin: `e.coin` itself is an object, and passing it
  reads as "no coin named" rather than as an error. Leave it out and the row appears on every coin panel of that character's this round,
  which is the right answer for something about the *skill* rather than one coin.

  **How a skill's coins are numbered.** A skill's coins are numbered from zero in the order the skill lists them —
  coin 0, coin 1, coin 2 — and that is the number `e.coin.index` gives you and the one this argument wants. It is the
  coin's place in the *skill*, not a count of flips: a coin that gets re-flipped keeps its own number rather than
  taking a new one. **A coin only gets a panel if the action really rolls it.** A clash ends as soon as one side runs
  out of coins, so a three-coin skill that clashes a two-coin one rolls the rest of its coins afterwards; a skill can
  also stop early. Naming a coin that never comes round is not an error and is not refused — the row simply never
  appears, exactly like filing one against a character who does not act.
* **Whether the rows stack or run across is the game's, not yours — and it decides whether your sentence is read.**
  On some panels the rows sit in a column; on others they run along in a line. That is decided by which panel the
  game has on screen, not by how many rows there are, how long your text is, or anything else you write — the same
  vanilla rows move with it. **Where they run across, the game measures each row by its chip alone, so the next chip
  covers the sentence and only the title and its number are read.** That is why the game's own rows in that layout
  are all bare chips — *"Poise 4"*, *"Shin (心) - Disgrace 1"*. Write for that: **put the meaning in the title** and
  treat the sentence as detail for the panels that have room. Nothing here fights it — forcing a layout would mean
  writing into the game's own UI state and would be undone the next time the panel rebuilt itself.
* **Not from `TIMING.ON_HIT`.** That is the one moment that cannot work, and it is the opposite of `showBuff`'s
  advice. The panel is built while the round is being worked out; `ON_HIT` is the blow *landing*, which is the replay,
  by which point that round's panel is long since finished. Every other moment during the round works —
  `ON_START_TURN` is the straightforward one.
* **The words are yours and stay yours — and they are PLAIN.** Nothing looks the text up, translates it, or
  styles it. Write it in whatever language you wrote the rest of the mod in.
  **A row cannot have the game's look.** Type `[On Hit]` into one and you get those eight characters in the same
  plain colour as everything else, which reads as an imitation of the game's rather than the thing itself. Say it in
  your own words here — **and if the game's look is what you were after, that is what
  `showSkillLine` is for.** It writes the wrapping lines higher up the same panel, it runs your
  text through the game's own pass, and there `[Laceration]` really does come out with the effect's symbol and
  colour.
* **A number is always drawn after the title.** That is how the game composes that chip for its own rows too, which
  is why a vanilla one reads *"Entangled Curse Talisman 6"*. Pass a fourth argument to choose it; leave it out and it
  is 1. There is no way to have no number at all — so use it for a count that means something, like how much of an
  effect you just inflicted.
* **The symbol is optional and yours to choose.** Name an effect as the fifth argument and the chip wears that
  effect's symbol — including one your own mod declared, which wears the icon you shipped. Leave it out and the row
  is drawn with no symbol at all rather than borrowing somebody else's.
* **The panel holds five rows.** The game's own come first, yours after, and anything past five goes to the "+N"
  count the panel already shows. A sentence is drawn on the one line beside the chip and clipped if it runs past the
  row, rather than wrapping onto a second line — so keep it to a phrase. Both are the game's own behaviour for its
  own rows, and neither is something this works around.

**`showSkillLine`** writes a line into **the skill's own text** on that same panel — the wrapping full-width writing
under the skill's name, where the game explains what the skill does. It applies nothing either.

```lua
-- what your effect should look like where the game explains a skill
on(TIMING.BEFORE_ROLL_COIN, function(e)
	if not e.coin then return end
	showSkillLine(e.unit, "[OnSucceedAttack] Inflict 2 [Laceration]", e.coin.index)
end)

-- or one line for the whole skill, on every coin panel of that character's
on(TIMING.ON_START_TURN, function(e)
	showSkillLine(e.unit, "Ember Debt burns deeper on every hit this turn.")
end)
```

**This is the half that carries the game's own look**, and that is the whole reason it is a separate name. A banner
row is a little badge and is printed exactly as you type it, flat. A skill line goes through the game's own text
pass on the way to the panel, so what you write comes out the way the game writes its own.

* **Put an effect in square brackets and it comes out as the game draws it** — the effect's little symbol in front,
  the effect's colour, its name in the *player's* language, and the tooltip link. `[Laceration]` becomes what the
  game calls Bleed, wearing the icon. **An effect your own mod declared works exactly the same way** and wears the
  picture your mod shipped. Effect names are matched forgivingly here, as everywhere else: spacing and capitals do
  not matter.
* **Put a skill tag in square brackets and you get the game's green bracket** — the `[On Hit]`, `[On Crit]`,
  `[On Kill]` that open the game's own lines.

  **The key is not the wording, and this is the one thing to get straight.** The game's table pairs an id with a
  localized wording, like this:

  ```
  { "id": "OnSucceedAttack", "name": "[On Hit]" }
  ```

  The wording **already carries its own square brackets**, because the pass swaps the *whole* `[OnSucceedAttack]` for
  it. So `[On Hit]` is what comes **out**, and typing it **in** gets you a plain `[On Hit]` — which is exactly what
  it looks like when it is not working. Write `[OnSucceedAttack]` and you get the green one.

  The id is also the half that **does not change with the player's language**, which is the other reason it is the one
  to write down: a mod built on the wording would draw correctly in English and silently stop in Korean. **Keys are
  matched exactly** — no forgiveness on spacing or capitals here, unlike effect names.

  **Getting one wrong breaks nothing, and the message names the key you meant.** The line is still filed, still drawn,
  and the effect names in it still get their symbols — the bracket you typed is simply drawn as you typed it. Where
  we can work out what you were reaching for, you are told:

  ```lua
  local ok, why = showSkillLine(e.unit, "[On Hit] Inflict 2 [Laceration]", e.coin.index)
  if why then print(why) end
  -- the line was filed, but "[On Hit]" is what the panel SHOWS, not the key it is
  -- indexed by. Write "[OnSucceedAttack]" to get the game's own colouring
  ```

  **All 72 of them, each paired with its wording, are written into `Limbonia.log`** the first time one is looked up —
  search it for `skill-tag` — and the companion shows the same list in the script status panel
  (`effectText.skillLineTags`). Read it once and write the spellings down; there is no way to guess one from what the
  panel shows. The ones you will reach for most:

  | Key you write | What the panel draws |
  |---|---|
  | `OnSucceedAttack` | `[On Hit]` |
  | `CriticalOnSucceedAttack` | `[On Crit]` |
  | `EnemyKill` | `[On Kill]` |
  | `WhenUse` | `[On Use]` |
  | `WinDuel` / `DefeatDuel` | `[Clash Win]` / `[Clash Lose]` |
  | `BeforeAttack` | `[Before Attack]` |
  | `EndSkill` | `[Attack End]` |
  | `AlwaysUse` | `[Constant]` |
  | `OnSucceedAttackHead` / `OnSucceedAttackTail` | `[Heads Hit]` / `[Tails Hit]` |
* **There is one line per coin panel, not a list.** This is the one place it differs from a banner row, and it
  follows from what the panel is: those wrapping lines are a single block of writing, so a second line for the same
  coin is not a second row, it is a longer paragraph. **Asking again for the same character and the same coin
  replaces what was waiting.** If you want two sentences, write one text with both in it.
* **File it against the character whose turn it is** — normally `e.unit`. The panel is asked whose it is and the
  answer is compared against the character you named, so the line appears on *that* character's panels and nobody
  else's.
* **Each coin can say something different, and `TIMING.BEFORE_ROLL_COIN` is how** — the same as for a banner row.
  Pass `e.coin.index` as the third argument and the line appears on that coin's panel only; leave it out and it
  appears on every coin panel of that character's this round. It wants the number, not the coin object. The
  numbering, and the fact that a coin only gets a panel if the action really rolls it, are exactly as described for
  `showOnBanner` above.
* **A line lasts for the round you filed it in and is thrown away at the start of the next.** It cannot turn up on a
  panel in a round you did not write it for.
* **`TIMING.ON_HIT` is allowed but usually too late** — the opposite of `showOnBanner`, which refuses it outright.
  This line is drawn at the panel rather than filed into the round's workings, so `ON_HIT` is not a dead end; it is
  simply that by the time a blow lands, the panel for that character's coin has normally been built already. The
  line then waits for their next panel this round, and is dropped if there is not one. File from
  `BEFORE_ROLL_COIN` and it is always in time.
* **Nothing is trimmed and nothing is clipped.** The panel measures the text and wraps a long line rather than
  cutting it — which is the other way round from a banner row's sentence.
* **But there has to be room, and where there is not the line is refused rather than drawn.** This is the one real
  limit on this name, so it is worth understanding rather than working around.

  The game draws the **status chips across the bottom of that same block** — the little `Bleed 8` badges. They are
  painted after the text and they are opaque, so an added line ends up *underneath* one: readable at both ends, with
  a chip sitting across the middle of it. That is not something a mod can fix. Where the chips sit is the game's own
  arrangement, and anything that shoved them aside would be undone the next time the panel rebuilt itself — which
  would be worse, because it would work sometimes and not others.

  So the rule is simple: **a coin whose panel already shows chips or passives will not take a line.** One whose
  ability area is empty will. **You are told when it happens** — the next call hands back a sentence saying the last
  panel had no room — and `Script_Status` counts it under `coinLinesNoRoom`:

  ```lua
  local ok, why = showSkillLine(e.unit, "[OnSucceedAttack] Inflict 2 [Laceration]", e.coin.index)
  if why then print(why) end
  ```

  A refused line **stays queued for that character's other coins**, which may have the room this one did not. And if
  what you need is something that always appears, that is what `showOnBanner` is for — a chip is one of the things
  drawn *over* this area, so it never has this problem.
* **Nothing you write here touches the skill.** The line is added to the finished text on its way to the panel being
  drawn. The skill's own words are untouched, so the codex, every other character who uses that skill, and the next
  use of it are all exactly as they were.

**`setForm`** turns a character into another set of drawings its own mod carries, and leaves it that way for the
rest of the fight.

```lua
on(TIMING.ON_KILL_TARGET, function(e)
	if e.unit:whichForm() ~= "transformation1" then
		e.unit:setForm("transformation1")
	end
end)
```

* The name is one your `mod.json` tags animations with (see FORMS in mods info.md). A name the mod does not have
  changes nothing and tells you which ones it does have.
* `setForm("")` goes back to the ordinary drawings. `whichForm()` returns `""` when it is in them.
* **A change you have asked for reads back straight away**, even though the drawings only change at the next blow —
  which is what makes the guard above ask once instead of at every moment.
* If no blow comes, the change is made anyway when the round stops resolving. A form is *state*; arriving late
  costs nothing. A floating line whose moment has passed is meaningless, so that one is dropped instead.
* **Drive a form from a script or from `becomes` beats, not from both.** A `becomes` beat is re-applied on every
  frame of the coin it belongs to and wins for as long as that coin is playing.

`e.unit:forms()` lists what this character's mod declares, and an empty list is a real answer — check the length of
what comes back rather than treating empty as a failure. `e.unit:appearance()` says which character the fight thinks this is, as
the name a mod addresses it by, which is how you recognise **your** character without comparing numbers and how you
tell an E.G.O. or an alternate outfit apart from the ordinary one.

### An effect that stays up — `playVisual` and `stopVisual`

The three above all *say* something at a moment. This one does not: it puts one of the game's own visual effects
on a character — an aura, a glow, a smoulder — and **leaves it there** until you take it off or the fight ends.

```lua
-- this one has drawn blood, and everybody can see it for the rest of the fight
on(TIMING.ON_KILL_TARGET, function(e)
	playVisual(e.unit, "EFFECT_BLOOD")
end)

-- ...until something knocks it out of them
on(TIMING.ON_BREAK_SELF, function(e)
	stopVisual(e.unit, "EFFECT_BLOOD")
end)
```

**You can ask for a colour.** A sixth argument, after the size, the place and the character it is borrowed from:

```lua
on(TIMING.ON_KILL_TARGET, function(e)
	playVisual(e.unit, "EFFECT_BLOOD", 1, "default", "", "#44aaff")
end)
```

These effects are drawn as a shape multiplied by a colour, so one drawn in grey becomes exactly the colour you
ask for. Some have their colour painted into the artwork instead, and those take yours as a shading of what they
already were - red on an orange flame is a deeper orange, not red. The effect list in the **Animation Timing**
editor says which of the two each effect is, and says so plainly when it does not yet know. It works on any effect
you can play: the game gives each character its own copy, so your colour never reaches anybody else's.

**Why this is allowed when a sound or a camera move is not.** The rule at the top of this section is about
*moments*: a script cannot pick a frame, because the round is settled long before the frame is drawn. Something
that **stays up has no frame to be wrong about**. It goes on, it is there, it comes off — at no point does it have
to agree with what is on screen at the instant you asked, so there is nothing for the delay to spoil.

**There is deliberately no "play this burst once".** A burst *is* a frame, and a frame is the one thing a script
cannot pick. Author one as a beat on the character's animation, on the **Animation Timing** screen, where it plays
on the same clock the picture does.

* **A visual is not a status effect, and the names are different.** Everywhere else on this surface `effect` means
  a status effect — `"Laceration"`, `"Sinking"`. A visual is a picture and its names look like `"EFFECT_BLOOD"`.
  Writing a status effect's name here finds nothing and says so.
* **You cannot guess a name.** Open a mod in the **Animation Timing** editor: the effect picker there is fed from
  the game's own catalogue and has a **Try it** button that puts one on a character standing in a battle.
* **The character has to be on screen.** `TIMING.ON_BATTLE_START` is the earliest moment that works; it is refused
  at `TIMING.ON_ADD_UNIT`, because none of the characters is being drawn yet. Every moment after that works,
  including the ones `showBuff` has to be careful about.
* **Asking twice re-plays it** rather than putting a second copy on. The game keeps one copy of each effect per
  character.
* **It ends when the battle does**, whether or not you stop it. Nothing else ever takes it off — so if it is meant
  to come off earlier, some handler of yours has to say so, the way the second one above does.
* `stopVisual` only takes off what `playVisual` put on. An effect a character's own mod declares in its `mod.json`
  belongs to that mod.
* Two more arguments if you want them: a **size** (a multiplier on what the game would give it, `1` by default) and
  **where** it hangs — `"default"`, `"back"`, or `"skin"`, which is the one to try when an effect is on and you
  cannot see it.

**If the same rule is really about a status effect, write it in `mod.json` instead.** *While this character has
Bleed, they smoulder* is one line of `gameEffects` with `"playsWhen": "while"` (see GAMEEFFECTS in mods info.md)
and needs no script at all. Reach for `playVisual` when the rule is something only a script can decide.

---

## Before there is a fight

Three moments arrive before anybody has acted, and they are not interchangeable. Reading them **in order** is most
of what there is to know about setting a mod up:

1. **`TIMING.ON_FORMATION` — deciding.** The game is about to build the fight out of the player's line-up and
   nothing has been made yet. You are handed the whole party, as numbers. Once per fight.
2. **`TIMING.ON_ADD_UNIT` — carrying it out, one character at a time.** Each character is built and put in front of
   you as it arrives. Once per character.
3. **`TIMING.ON_BATTLE_START` — everybody is here and the fight is starting.** Every character exists, nothing has
   acted yet, and `e.allies` and `e.enemies` are the whole field as characters you can read and act on. **Once per
   character**, not once per fight — six of them for a team of six, each carrying the same full rosters — so leave
   the scope off if you want the look taken once.

Hold on to that ordering, because the first two look like answers to the same question and are not. *"What team am I
about to field"* is the first. *"This particular character has arrived — what do I put on it"* is the second.
*"Everything exists now, let me look at it properly"* is the third. Only the first of the three arrives once.

**`e.settingUp` is true at the first two of them.** It is not a way of asking which one you are in — check `e.unit`
for that, because only one of them has a character to hand.

### `ON_FORMATION` — what team am I about to field

This is the earliest anything in this layer can reach, and the only moment that knows the **whole** line-up. It is
announced from the loop that turns the player's formation into characters, entered once, before it has built the
first of them.

**`ON_ADD_UNIT` structurally cannot answer the same question**, and that is worth understanding rather than taking
on trust: it fires from *inside* that same loop, once per character. At the first one there is exactly one character
in existence, and asking it who else is coming is asking something nobody in the process yet knows.

**`e.party` is a list of records, not of characters, and that distinction is the whole moment.** Each entry is a
plain set of numbers read straight off the line-up — `id`, `sinner`, `level`, `uptie`, `order`, `deployed`. Nothing
has been built behind them, so there is no health to read, no effects to ask about and nothing to pass to a verb:
an entry cannot be handed to `addBuff` and cannot be asked `:hasBuff`. **There is no `e.unit` here at all**, which
is why the usual setup line does not work — `addBuff(e.unit, ...)` is arithmetic on nothing, and counts against the
handler like any other Lua mistake.

So this moment is for **deciding**, and `ON_ADD_UNIT` a moment later is where the decision gets carried out.

**Write it `{ scope = "any" }`, and it is not optional here.** There is no character at this moment for the default
scope to match your mod's identity against, so a default-scope handler could never run — and rather than let it
register and sit silent, **the script is refused as it loads**, with a sentence saying so. Refused means refused:
as with every other registration the game turns away, the rest of the file goes down with it. This is the only
moment in the vocabulary that will not take the default scope.

```lua
local doubleUp = false

on(TIMING.ON_FORMATION, { scope = "any" }, function(e)
	if not e.party then return end

	local seen, fielded = {}, 0
	for _, m in ipairs(e.party) do
		if m.deployed then
			fielded = fielded + 1
			if seen[m.sinner] then doubleUp = true end
			seen[m.sinner] = true
		end
	end

	log("fielding", fielded, doubleUp and "with a repeat" or "all different")
end)

-- ...and this is where the decision turns into something
on(TIMING.ON_ADD_UNIT, { scope = "any" }, function(e)
	if doubleUp and e.unit.isAlly then
		addBuff(e.unit, "Agility", 1, 3)
	end
end)

on(TIMING.ON_END_BATTLE, function(e)
	doubleUp = false
end)
```

Four things about that list are worth knowing before you write one:

* **`deployed` is what makes it a party rather than a roster.** A formation can carry slots that are not going into
  this fight, so filter on it before you count anything. `order` is only meaningful on a slot that is deployed.
* **`id` and `sinner` are different questions.** `id` is which identity — the number a mod's `appearance` names,
  and the same number that character reports as `u.id` once it exists, which is exactly what joins a decision made
  here to the character you meet later. `sinner` is which of the twelve it belongs to, and several identities share
  one, so "am I fielding two of the same sinner" is only answerable with `sinner`.
* **`level` and `uptie` are the numbers that character will report.** The fight is built with them, so what you
  read here is what `u.level` and `u.uptie` say afterwards.
* **It is the player's line-up and nothing else.** The enemies are not in it — they are not built from a formation
  — so a handler that wants to know what it is *facing* wants `ON_BATTLE_START` and `e.enemies`.

**Walking `e.party` is free.** Every field on it was copied out before your handler started, exactly as a
character's own fields are, so the loop above asks the game for nothing at all and costs the fight no time
whatever. That is a real difference from the same-shaped loop over `e.allies`, where every `u:buffStacks(...)`
inside it is a request that enters the game.

**And a misspelling inside a party entry is silent, which it is nowhere else.** `e.` and a character both catch a
name that is not a field and stop the handler saying so; a party entry is an ordinary Lua table with nothing
watching it, because there is no character behind it to protect. `m.upti` reads as nothing, takes the wrong branch,
and goes on doing it. Take these six names from the editor's completion rather than from memory.

### `ON_ADD_UNIT` — setting a character up as it arrives

`TIMING.ON_ADD_UNIT` fires once per character as it arrives, before the fight starts. `e.settingUp` is true here,
and unlike the moment before it there **is** a character: `e.unit` is the one that has just been built.

**This is where a character starts the fight carrying something.** Give it a status effect, health, protection or SP
and it is already on when the first round begins — which is the one thing no later moment can say, because by then the
fight has started. `takeHealth` works here too, and is how a character shows up already wounded; **do not take one all
the way to zero here**, though — see **Hurting a character**.

```lua
-- everyone on the player's side arrives with a shield and two stacks of my own effect
on(TIMING.ON_ADD_UNIT, { scope = "any" }, function(e)
	if e.unit.isAlly then
		e.unit:giveShield(15)
		addBuff(e.unit, "emberdebt", 2)
	end
end)
```

Reading works here too — `buffStacks`, `hasBuff`, `e.unit:appearance()` and the rest — so a handler can look at the
character before deciding what to give it.

`setForm` is allowed as well, under the same rule it follows everywhere else: the change is held and made at that
character's first blow of the fight. It is not lost in between, so this is how you say what a character arrives already
meaning to become.

**Three names are refused here** — `showBuff`, `showOnBanner` and `showSkillLine` — and each tells you so when you
try. Nothing is on
screen while the characters are arriving, so a line asked for now would be held, and held lines are cleared when the
fight starts. It could never be drawn, and you would have been told it worked. Say it at the moment you want it seen.
Applying a status effect announces itself without being asked, so a spawn rarely wants `showBuff` anyway.

**A handler here runs once per character**, so whatever it asks the game for is asked again for each of them -- a fight
with fourteen characters runs it fourteen times. That is the number to size a spawn handler against.

Remembering things across the setup and acting on the whole field at once is still the right shape when the decision
depends on who else turned up — **including the enemies**, who are in no formation and so appear in none of what
`ON_FORMATION` hands you:

```lua
local seen = {}

on(TIMING.ON_ADD_UNIT, { scope = "any" }, function(e)
	seen[e.unit.instanceId] = e.unit.id
end)

on(TIMING.ON_BATTLE_START, function(e)
	local n = 0
	for _ in pairs(seen) do n = n + 1 end
	log("the fight started with", n, "characters")
end)

on(TIMING.ON_END_BATTLE, function(e)
	seen = {}
end)
```

`e.unit.id` is which character it is, the same for every copy of it on the field. `e.unit.instanceId` is which
particular copy, unique for the whole fight — that is the one to key a table by.

**Notice that only the first of those three handlers widens its scope.** The collector wants every character, so it
says `{ scope = "any" }`. The other two want to run *once* — and since `ON_BATTLE_START` and `ON_END_BATTLE` arrive
per character, leaving the scope off is exactly what makes that so. Widen the middle one and the same line is logged
for every character on the field, all of them carrying the same number.

**Nothing empties your tables for you.** A script's own variables live for as long as the scripts are loaded, not
for as long as a fight — so anything you fill during one has to be cleared when it ends, or the next fight starts
holding the last one's contents. `TIMING.ON_END_BATTLE` is where that goes. Reloading the scripts clears everything,
which is why a bug like this hides during testing and appears on the second fight of somebody else's session.

---

## Budgets, failures, and one handler going quiet

A script runs on the game thread, on seams that fire thousands of times a fight. Two things are worth knowing, and
only one of them is a ceiling.

* **There is no limit on how much a handler may do.** Ask the game for as many things as you like, in a loop, over
  every character a moment gives you. Nothing is turned away for being too much, and no amount you pass has a
  ceiling of ours on it.
* **A handler is stopped if one run of it takes too long**, by instruction count and by wall clock, and a script has
  a memory ceiling. An overrun surfaces as an ordinary error against your mod, not as a hang.
* `pcall` cannot swallow that stop. A loop that never ends cannot be wrapped out of trouble.

**That limit is there for a loop that never ends, and for nothing else.** It sits far past anything real work needs.
It is not a judgement about how much a mod should be allowed to do — it is the only way back from a script that
never returns, which would otherwise take the whole game with it. When you meet it, the cause is very nearly always
a `while` or a `repeat` whose condition never comes true.

The figures are published with the rest of the surface and are in the reference under **Limits**. Read them from a
live copy rather than from memory or an old saved one.

**What actually costs time is asking the game for something.** Reading a character's fields does not: `e.unit.hp`,
`u.staggered`, `u.sp` and the rest were copied out before your handler started, and cost nothing at all. The colon
calls do, every one of them, including the ones that only ask a question like `buffStacks`. So this:

```lua
on(TIMING.ON_START_ROUND, { scope = "any" }, function(e)
	for _, u in ipairs(e.allies or {}) do
		local n = u:buffStacks("Laceration")
		log(u.id, n or 0)
	end
end)
```

asks the game something once for every character on the field, and that is a perfectly reasonable thing to write.
What it costs is a slightly longer frame, and the moment you write it on decides how much that matters: once a
round is nothing, once for every coin flip is a different proposition entirely.

**If a script makes the game stutter, you will see it** — and the fix is yours to make. Move the work to a moment
that comes round less often, ask about the characters you actually care about, or lift out of the loop whatever
does not need to be inside it.

**A handler that keeps failing is switched off — and it is one handler, not your script.** Repeated failures in one
fight quiet that handler for the rest of it; enough across a session quiet it until the next rescan. Its siblings in
the same file carry on running, which is deliberate: one broken handler silencing a mod's working ones would be far
worse. **Reload scripts in the game** brings it back — you do not need to restart anything.

Every refusal, clamp and failure is filed against the mod that caused it and shows up on the Scripts panel with a
sentence in plain language. That list is the first place to look, not the last.

---

## Did my handler ever run?

A handler that registers cleanly and is never called is the failure mode this whole layer is arranged to make
visible, because nothing about it looks wrong. The Scripts panel answers it directly under **Find out what actually
runs**.

**A zero is three different things, and the panel tells them apart:**

* **Never fired** — it registered and has not been called. Usually `scope`: the handler is watching your character
  and the thing you care about is happening to somebody else. Sometimes the moment simply has not come round yet
  this session, which is not the same as broken. And some moments only *work when damage is worked out* — they are
  carried by the damage calculation, so they arrive only when the game really works damage out at that moment. A
  handler on one of those in a fight where nothing took damage at that point has not failed; there was nothing to
  carry it.
  `ON_END_ROUND` is the one most likely to catch you out here, and in both directions: a round where nothing ticked
  raises it not at all, and a round where four things ticked raises it four times. See **How often a moment fires is
  decided by what carries it**.
* **Not seen to arrive** — no seam in Limbonia announces that moment yet, and nothing has ever seen it arrive. It
  needs work in the DLL before any handler on it can run. This is a claim about evidence, not a proof of
  impossibility: moments have moved out of this column by turning up.
* **Switched off after repeated failures** — it ran, and it kept raising. The sentence beside it says what went
  wrong.

### The seven that will not fire

`TIMING` is very nearly the game's own list of battle moments, and Limbonia does not reach all of it. Registering on
one of these is accepted without complaint and the handler is never called — so it is worth knowing the list before
you spend an evening on one:

| | |
|---|---|
| `ON_START_WAVE`, `ON_END_WAVE` | A wave is a property of the fight rather than of any character, so there is nobody for `e.unit` to be and no shape has been settled for what they would hand over instead. |
| `ON_START_PARRYING` | The game has no per-character function for it to be announced from. Its three siblings — win, lose and end — all work, and all three are about **clashing** rather than defending. |
| `ON_BREAK_TARGET` | "This character staggered somebody." `ON_BREAK_SELF` works, so watch the character that *was* staggered instead. |
| `BEFORE_ROLL_COIN_PARRYING`, `AFTER_ROLL_COIN_PARRYING`, `AFTER_ROLL_COIN_ACTION` | Three of the game's own four coin-flip moments. The fourth, `BEFORE_ROLL_COIN_ACTION`, does arrive — and `BEFORE_ROLL_COIN`, which is Limbonia's own and covers every flip of either kind, arrives every time. |

Thirty-two of the thirty-nine do work, so this is a short list of exceptions rather than a warning about the
vocabulary in general. **And it is a floor, not a verdict** — it says nothing has been seen to announce these, which is
not the same as proving nothing can. The panel and the editor are told by what actually arrives, so a moment that
turns up promotes itself and stops being marked. The reverse never happens: a moment that works does not quietly
stop.

**And when the question is *how often* rather than *whether*, count it yourself.** Every registration on the panel
carries a `fired` count beside it, so three lines at the top of a scratch script will settle any rate question in
this document against the fight you actually care about — including the two that depend entirely on which fight it
is: whether a staged battle raises the battle moments once per wave, and how many end-of-round ticks your particular
enemies produce.

```lua
for _, t in pairs(TIMING) do
	on(t, { scope = "any" }, function(e) end)
end
```

Play one ordinary fight and read the counts. `{ scope = "any" }` is not decoration there: it is what makes the loop
cover every character rather than only yours, and it is what stops `TIMING.ON_FORMATION` refusing the whole script
as it loads. The handlers do nothing whatever, so this cannot change the fight it is measuring — but there are as
many of them as there are moments, several sitting on the busiest seams in the game, so take it out again once you
have your numbers.

`log` is your other instrument, and the only way a handler can say anything. Its lines appear on the Scripts panel
under **What the scripts said** — the last fifty of them — and in `Limbonia.log`, which is emptied each time the
game starts.

---

## A worked mod

An effect the mod hands out, that makes its carrier take more damage, and that transforms your character once it
has spread far enough. Beyond the `appearance` and `script` keys every mod has, it needs only the `buffs` entry
declaring `emberdebt` and the `assets` tags declaring `transformation1`. Every moment it uses is one this build
really delivers.

```lua
local PER_STACK = 25   -- % extra damage per stack
local TO_CHANGE = 10   -- stacks handed out before the transformation

local handedOut = 0

-- Hand it out. Both sides of a blow arrive here, so both parts are checked.
on(TIMING.BEFORE_GIVE_ATTACK, function(e)
	if not e.attacker or not e.victim then return end
	local ok = addBuff(e.victim, "emberdebt", 1 + (e.coin and e.coin.index or 0), 3)
	if ok then handedOut = handedOut + 1 end
end)

-- Anybody carrying it takes more. scope = "any", because the carriers are not ours.
contribute(DECISION.TAKE_DAMAGE, { scope = "any" }, function(e)
	if not e.damage then return end
	local owed = e.unit:buffStacks("emberdebt")
	if not owed or owed <= 0 then return end
	return e.damage + (e.damage * PER_STACK * owed) // 100
end)

-- Transform once, when enough of it is out there. ON_END_TURN comes round after every action, so the
-- guard is what makes this ask once instead of a dozen times a round.
on(TIMING.ON_END_TURN, function(e)
	if handedOut < TO_CHANGE then return end
	if e.unit:whichForm() == "transformation1" then return end
	local ok, why = e.unit:setForm("transformation1")
	if not ok then log("could not transform:", why) end
end)

-- And say so, on a blow that is actually on screen.
on(TIMING.ON_HIT, function(e)
	if not e.victim then return end
	local owed = e.victim:buffStacks("emberdebt")
	if owed and owed > 0 then showBuff(e.victim, "emberdebt", owed) end
end)

-- The counter belongs to this fight, so it is cleared when the fight ends.
on(TIMING.ON_END_BATTLE, function(e)
	handedOut = 0
end)
```

What is worth copying out of it:

* The `contribute` reads and returns and does nothing else. The acting happens in the `on` handlers.
* `return e.damage + ...` and never `return <a number>`, so another mod's change survives.
* The scope changes between handlers in the same file, because the handlers are about different characters.
* Every optional field is checked before it is used.
* `handedOut` is an ordinary local at the top level. It survives between handlers, so it has to be cleared at
  `ON_END_BATTLE` — a reload would also clear it, which is exactly why forgetting this survives your own testing.

---

## Where the real list is

**Not here, and not in any file that is written by hand.** Limbonia publishes its own surface — every moment, every
decision each verb can attach to, which fields each of those hands over, every method on a character, what each is
for, what it hands back and a line of real Lua using it. The editor completes out of that response, live.

* **The editor's suggestions** are the version that is always right. Start the game before you write.
* **In VS Code**, the same surface again — see [Writing it in VS Code instead](#writing-it-in-vs-code-instead).
  Set up once per mod, and pressed again whenever the game is updated.
* **The reference**, which is the same account of the same surface written out as a page you can search and read
  with the game shut. **Open the reference** on the Scripts panel opens it, and so does **Reference** in the
  editor's own toolbar, beside the file you are typing into.

**The reference says at the top which build it was read from**, as a version and a date. That line is the thing to
check before trusting a detail: a written list describes one build and then quietly stops being true, which is why
this document does not keep one and why the page always says which build it is describing.
