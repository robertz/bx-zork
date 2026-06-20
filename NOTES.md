# NOTES.md

A consolidated list of everything not (yet) implemented, gathered from the
`note`/`TODO` annotations scattered through `zorkData.bxs` plus a pass over
`gameState.flags` to see which ones are ever actually set. See the README's
[What's not (yet)](README.md#whats-not-yet) section for the short version —
this is the long one, with pointers.

## Genuinely unimplemented

### Endgame / scoring
- `wonFlag` (`zorkParser.bxs:115`) never gets set — no win condition.
  `stone_barrow`'s `sw`/`in` exits (`zorkData.bxs:939-940`) are permanently
  blocked as a result.
- `score`/`handleScore()` (`zorkParser.bxs:443,606`) totals treasures
  deposited in the `trophy_case`, but there's no rank table or "you have
  won" sequence.

### Hades exorcism
- `lldFlag` (`zorkParser.bxs:120`) never gets set. The EXORCISE ceremony
  (bell/book/candles) isn't wired up — see `ghosts` (`zorkData.bxs:464`).
  The gate to `land_of_living_dead` stays blocked (`zorkData.bxs:1669-1670`).
- `hot_bell` (`zorkData.bxs:316`) — the alternate state of `bell` after
  being held over the torch — is defined but never produced; no
  bell/torch state-swap logic exists.

### Coffin curse
- `coffinCure` (`zorkParser.bxs:121`) never gets set; praying doesn't lift
  it. `egyptian_room`'s `down` exit to `tiny_cave` stays blocked
  (`zorkData.bxs:1751`).

### Dam / reservoir
- Turning the bolt with the wrench toggles `lowTide` directly
  (`handleTurn`); the real `GATES-OPEN` transitional state and gate-flag
  bubble from the ZIL source aren't modeled (`zorkData.bxs:551`).
- `leak` (`zorkData.bxs:611`) — meant to appear once the dam puzzle is
  underway — is never placed.
- `trunk` (`zorkData.bxs:426`) stays hidden forever; it should reveal once
  the reservoir is drained.

### Mirror rooms

Implemented. `rub` is a new one-object verb (`handleRub`, `zorkParser.bxs`)
wired up for `mirror_1`/`mirror_2` (`zorkData.bxs:431,436`): rubbing either
mirror swaps `mirror_room_1`/`mirror_room_2`'s non-scenery contents and
teleports the player to the other room, per `MIRROR-MIRROR` in
`1actions.zil`. Not modeled: `MIRROR-MUNG` (breaking the mirror via
MUNG/THROW/ATTACK, which would permanently disable the swap and dock seven
years of luck) and the "rub with something other than your hands" tingling
message, since this engine's `rub` grammar has no indirect object yet.

### Rainbow / sceptre
- `rainbowFlag` (`zorkParser.bxs:123`) never gets set; waving the `sceptre`
  at the falls/rainbow should set it (`zorkData.bxs:528`), making the
  rainbow solid and crossable (`zorkData.bxs:1913-1944`).
- `pot_of_gold` (`zorkData.bxs:751`) stays hidden until that happens — never
  placed.

### Coal mine area
- `machine_switch` (`zorkData.bxs:800`) — turning it on with coal in the
  coal room's machine should produce a `diamond` (`zorkData.bxs:806`,
  never placed).
- `bat` (`zorkData.bxs:769`) should swoop you to a random room unless
  you're carrying garlic; currently inert.
- `lowered_basket`/`raised_basket` (`zorkData.bxs:775,782`) — the
  dumbwaiter chain-pull to swap between them isn't implemented.
- `scarab` (`zorkData.bxs:744`) stays hidden; should be diggable with the
  shovel.

### Egg / canary
- `egg` (`zorkData.bxs:829`) should break (and damage any canary inside) on
  rough handling, becoming `broken_egg` (`zorkData.bxs:836`, never placed).
- `canary` (`zorkData.bxs:846`) should be windable for a songbird payoff in
  a forest room, becoming `broken_canary` (`zorkData.bxs:851`) and dropping
  `bauble` (`zorkData.bxs:856`) — none of this is wired up.

### Boat / river

Implemented. `gameState.boardedVehicle` (`zorkParser.bxs`) tracks whether
the player is in the boat. `board`/`launch`/the new `land` direction
(`handleBoard`/`handleLaunch`/`handleLand`) move the player onto the river
(`RIVER_LAUNCH`) and back to shore (each room's `land` exit key), and
`tickRiver()` advances the boat one room downstream per turn while boarded
(`RIVER_NEXT`), running off the end of `river_5` without landing is a
waterfall death, matching `I-RIVER`'s `JIGS-UP` in `1actions.zil`.
`punctured_boat` (`zorkData.bxs:690`) is produced by `handleBoard()` when
the player carries a weapon aboard, mirroring `RBOAT-FUNCTION`'s `BOARD`
check (the original tests for `SCEPTRE`/`KNIFE`/`SWORD`/`RUSTY-KNIFE`/
`AXE`/`STILETTO` by name; this engine generalizes that to "carrying
anything flagged `weapon`"). Not modeled: hitting the white cliffs' rocks
specifically (the original's puncture trigger is the `BOARD` check itself,
not a per-room probability — there is no separate rocks/probability
mechanic in `1actions.zil` to port), `DBOAT-FUNCTION`'s puncture-repair-with-
putty path (`FIX-BOAT`), and walking onto river rooms without the boat is
disallowed simply because the data has no walkable exit into them (matching
`NONLANDBIT` in practice) rather than via an explicit engine-level check.

### Other single-item gaps
- `map` (`zorkData.bxs:893`) — trigger for revealing it isn't implemented.
- `gunk` (`zorkData.bxs:904`) — should be created when `putty` is melted by
  the torch; never placed.
- `broken_lamp` (`zorkData.bxs:909`) — should replace `brass_lantern` if
  it's thrown or burns out; never placed. (Relatedly: there's no lamp/match/
  candle fuel-timer system at all — light sources don't run out.)
- `teeth`, `wall`, `granite_wall` (`zorkData.bxs:72,77,82`) — ZIL
  `GLOBAL-OBJECTS` scenery kept for vocabulary/flavor only; intentionally
  not wired to any room.

## Implemented despite having a `note`

A few objects carry a `note` that's just situational context, not a gap:

- `trap_door` (`zorkData.bxs:300`) — revealed by pushing the rug; this
  works, the note just explains the mechanism.
- `grate` (`zorkData.bxs:386`) — hidden until spotted from the Grating Room
  below, then unlockable with the skeleton key; this works.
- `large_bag`/`stiletto` (`zorkData.bxs:401,406`) — held by the thief until
  he's defeated, then reachable; this works (combat is implemented).
- `axe` (`zorkData.bxs:329`) — held by the troll until defeated, then on
  the ground; this works.
