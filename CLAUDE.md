# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A from-scratch text-adventure parser engine written in [BoxLang](https://boxlang.io), loaded with the world of **Zork I** ported verbatim from the original 1983 Infocom ZIL source (`historicalsource/zork1`, MIT-licensed).

Two files, deliberately separated:

| File | Contents |
|---|---|
| [zorkParser.bxs](zorkParser.bxs) | The engine: tokenizer, grammar matcher, scope resolver, verb dispatcher, combat system, REPL/demo runner. No Zork-specific content. |
| [zorkData.bxs](zorkData.bxs) | The content: every room and object from `1dungeon.zil`/`1actions.zil`, as data. No engine logic. |

`zorkParser.bxs` does `include "zorkData.bxs"` — swap in a different data file for a different game without touching the engine.

## Running it

```sh
boxlang zorkParser.bxs          # interactive REPL, reads stdin until "quit"/"exit"
boxlang zorkParser.bxs demo     # runs the scripted walkthrough in runDemo() instead
```

There is no test suite, linter, or build step — this is two `.bxs` scripts run directly by the `boxlang` CLI. There's also no save/load; each run is a single in-memory session.

## Architecture

### Command pipeline

Every input line flows through one path: `processCommand()` → `dispatchCommand()` (zorkParser.bxs:2113, 2051).

```
tokenize()        lowercase + regex-split into words, strip noise words (a/an/the)
extractVerb()     match first word, or first two words for two-word verbs ("turn on"/"turn off")
grammar dispatch  zero/one/two-object, based on which verb table the verb is registered in
resolveObject()   match noun phrase against current scope (synonyms + adjectives disambiguate)
handle*()         the actual verb logic
```

A bare direction (`"north"`, `"in"`) or `go <direction>` is special-cased before verb lookup.

### Verb tables (zorkParser.bxs:1-110)

Verbs are registered in exactly one of three tables by arity, then routed through `dispatchOneObjectVerb`/`dispatchTwoObjectVerb` (zorkParser.bxs:1972, 1999):

- `ZERO_OBJECT_VERBS` — `look`, `inventory`, `score`, `pray`, `odysseus`, ...
- `ONE_OBJECT_VERBS` — `take`, `open`, `read`, `climb`, `attack`, `raise`/`lower`, `rub`, ...
- `TWO_OBJECT_VERBS` — each entry declares its own preposition(s) used to split the sentence into direct/indirect object (`put X in Y`, `give X to Y`, `unlock X with Y`), mirroring ZIL's per-verb grammar table.

`VERB_SYNONYMS` maps every surface form to a canonical verb; `buildVerbIndex()` flattens it into a synonym → canonical lookup at load time. Adding a verb means: add it to the right arity table, add a `VERB_SYNONYMS` entry, write a `handle*()` function, and wire it into the matching `dispatch*Verb` function.

### Scope and object resolution

`getVisibleObjectIds()` (zorkParser.bxs:266) computes what's currently referenceable: room contents, carried items, contents of any open/transparent container (recursive), plus "global" scenery that lists the current room in its `globalIn` (e.g. the kitchen window is visible from both *Behind House* and the *Kitchen*).

`resolveObject()` (zorkParser.bxs:306) matches against that scope by synonym, using leading adjectives to disambiguate when a word matches multiple objects. Vocabulary known but out-of-scope yields "You don't see that here." rather than "I don't know that word" — same distinction the original ZIL game makes.

### Data model (zorkData.bxs)

**Objects** — built by `makeObject(id, name, synonyms, desc, takeable, options)` (zorkData.bxs:10), a plain struct with ~35 fields covering the ZIL `FLAGS` (container/open/openable, door, light/on/fuel/burnedOut, weapon, readable/text, value/treasureValue for scoring, locked/unlocksWith, etc). No class hierarchy — flags are just false/zero where they don't apply.

The single most important field is **`location`**: a room id, another object's id (nested in a container), or `"player"`. Rooms hold no contents list of their own — `describeRoom()` and the scope resolver always filter the full `objects` struct by `location` live, so moving things around never requires keeping two places in sync.

`hidden` is ZIL's `INVISIBLE`: a correctly-located object not yet in scope because its reveal trigger (a puzzle) hasn't fired or isn't implemented. Always paired with a `note` field describing what should clear it (e.g. trap door, grating, thief).

**Rooms** (zorkData.bxs:961) — `id`, `name`, `description` (or a `dynamicDescription` closure when text depends on game state, e.g. kitchen window open/closed, reservoir water level), and an `exits` map. An exit value is one of:

- a plain string — destination room id, always available
- `{ "blocked": "message" }` — never passable, flavor text only
- `{ "to": id, "requiresOpen"/"requiresFlag"/"requiresNotCarrying"/"requiresEmptyHanded": ... }` — conditionally passable
- `{ "per": "name" }` — routed through `PER_EXITS` (zorkParser.bxs:1833) for exits with custom logic (one-way maze "diodes", the grating, the chimney climb)

Rooms can also declare an `onEnter` hook — most return nothing (pure side effect, e.g. Grating Room revealing the grate), but `enterRoom()` (zorkParser.bxs) also accepts one that returns text to replace the room description entirely (used for the Stone Barrow's win ending) — mirroring ZIL's `M-ENTER`.

### Game state

`gameState` (zorkParser.bxs:112) holds everything not a fixed property of a room/object: current room, turn counter, player wound/strength adjustment, scoring (`baseScore`), and a `flags` struct mirroring ZIL's boolean globals (`trollFlag`, `lowTide`, `lldFlag`, `rainbowFlag`, ...) that puzzles set and that exits/descriptions read. Several puzzles also need their own scalar/counter state alongside the boolean flags — `beachDig`, `boardedVehicle`, `matchCount`, etc. — sitting directly on `gameState` rather than in `flags`.

### Combat

Ported from ZIL's `VILLAIN-BLOW`/`HERO-BLOW` strength tables and message tables (zorkParser.bxs:1172-1531) — not a stub. Troll, thief, and cyclops are all fightable; the cyclops's `STRENGTH 10000` makes direct combat exactly as suicidal as in the original (use `odysseus`/`ulysses`, or give him water).

### Per-turn hooks

`processCommand()` runs a fixed sequence of "demon" checks after every command, each a `tick*()`/`check*()` function returning an extra line to print or `""`: `tickThief()` (random movement/theft), `tickRiver()` (boat drift), `tickLightFuel()`/`tickMatch()` (lamp/candle/match burning down), `checkEndgameTrigger()` (win condition). Adding a new per-turn mechanic means writing one of these and adding it to that sequence — not hooking into a generic event system, there isn't one.

## Working in this codebase

- Engine changes (verbs, parsing, scope, combat math) belong in `zorkParser.bxs`. Content changes (room text, object flags, new puzzle wiring) belong in `zorkData.bxs`. Don't blur this — it's the whole point of the split.
- Unimplemented mechanics are tracked as a `note` field on the relevant object/room in `zorkData.bxs`, or `TODO`, rather than silently faked. Search for `TODO`/`note` there before assuming something is missing entirely. Known gaps, in full detail in [NOTES.md](NOTES.md): no `throw` verb (so `broken_lamp` can never be produced), a few ZIL-internal simplifications called out per-puzzle (the dam's `GATES-OPEN` transition, `MIRROR-MUNG`, boat-patching with putty), and no darkness/visibility system at all (running out of light is flavor-only). Everything else named in the original ZIL source — combat, scoring/endgame, and every puzzle from the Hades exorcism to the coal mine to light-source fuel timers — is implemented.
- When porting more ZIL behavior, match the original source (`1dungeon.zil`/`1actions.zil`) verbatim for text/flags/synonyms rather than improvising.
