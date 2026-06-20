# NOTES.md

A consolidated list of everything not (yet) implemented, gathered from the
`note`/`TODO` annotations scattered through `zorkData.bxs` plus a pass over
`gameState.flags` to see which ones are ever actually set. See the README's
[What's not (yet)](README.md#whats-not-yet) section for the short version —
this is the long one, with pointers.

## Implemented since this was written

### Endgame / scoring
Take points (`value`) are now banked once per treasure in
`gameState.baseScore` (`handleTake`, `zorkParser.bxs:328`), added to the
trophy case's live deposit total (`treasureValue`) in `handleScoreValue()`.
`handleScore()` reports a rank from `scoreRank()`, a verbatim port of
V-SCORE's thresholds. `checkEndgameTrigger()` sets `wonFlag` (and reveals
`map`) once score hits `TREASURE_MAX_SCORE`, the max achievable under
optimal play. Once `wonFlag` is set, `stone_barrow`'s `sw`/`in` exits open,
and walking in there fires the verbatim ZIL ending (`stone_barrow.onEnter`,
`zorkData.bxs:946-968`) via `endGameWin()`.

`computeTreasureMaxScore()` (`zorkParser.bxs:198-230`) can't just sum every
treasure's `value`/`treasureValue` at load — most genuinely-unobtainable
ones start `hidden` or unplaced (`location: ""`), which is exactly how it
tells them apart from ordinary treasures sitting in their starting room.
Two explicit lists patch the cases that heuristic gets wrong:
- `FORCE_REACHABLE_TREASURES` — `trunk`/`scarab`/`pot_of_gold`/`diamond`
  start `hidden`/unplaced too, but their reveal puzzles are now
  implemented and always worth pursuing, so they're force-included.
  Deliberately *not* listed: `broken_egg`/`broken_canary`/`bauble` —
  breaking the egg or winding the canary nets *fewer* points than leaving
  them intact (they're worth little to nothing on their own), so optimal
  play never goes through them; they stay excluded.
- `UNREACHABLE_TREASURES` — for a treasure that doesn't start
  `hidden`/unplaced but is still genuinely gated by a flag the heuristic
  above can't see (`skull`'s only route needed `lldFlag`, before Hades
  exorcism was implemented). Currently empty — kept for the next puzzle
  that needs this treatment.

Current achievable max is 270 (out of the original 350 — the gap is
mostly the unimplemented Hades-demon/fuel-timer point sources, not
missing treasures).

### Hades exorcism
`lldFlag` is now set by the bell/book/candles ceremony, ported from
`LLD-ROOM`'s `M-BEG` clause in `1actions.zil`. Ringing the `bell` at
`entrance_to_hades` (`handleRing`) scares off the `ghosts` and swaps it for
`hot_bell`, dropping any lit `candles` you're holding. Lighting the
`candles` again and then reading the `book` there (`handleLight`/
`handleRead`) completes the ceremony, removes the `ghosts`, and sets
`lldFlag`, unblocking the gate to `land_of_living_dead`. `skull` has been
removed from `UNREACHABLE_TREASURES` accordingly.

### Coffin curse
`coffinCure` is now set by `pray` (`handlePray`) at `south_temple`, as long
as you aren't carrying the `coffin` at the time — unblocking
`south_temple`'s `down` exit to `tiny_cave`.

### Dam / reservoir
The bolt/wrench toggle (`handleTurn`) still flips `lowTide` instantly — the
real `GATES-OPEN` transitional state and gate-flag bubble aren't modeled
(`zorkData.bxs:551`), and neither is the water-level flooding timer
(`I-MAINT-ROOM`/drowning) — both belong with the cross-cutting fuel/timer
work. What's new: `leak` (`zorkData.bxs:608`) now appears when you push the
`blue_button` in the Maintenance Room (`handlePush`), with the verbatim
`BUTTON-F` burst-pipe message (pushing it again reports it jammed); `trunk`
(`zorkData.bxs:421`) now reveals itself via the `reservoir` room's
`onEnter` hook once `lowTide` is true.

### Rainbow / sceptre
`handleWave` now special-cases the `sceptre`, porting `SCEPTRE-FUNCTION`
verbatim: waving it at `aragain_falls` or `end_of_rainbow` toggles
`rainbowFlag` (`zorkParser.bxs:123`), making the rainbow solid/crossable
(the exits gated on `rainbowFlag` already existed, `zorkData.bxs:1913-1944`);
waving it again reverts it. Waving it while standing `on_rainbow` drops the
rainbow out from under you (`endGame`). `pot_of_gold` (`zorkData.bxs:746`)
is revealed the first time the sceptre is waved at `end_of_rainbow`.

### Coal mine area
`handleDig()` now ports `SAND-FUNCTION` verbatim: digging `sand` in
`sandy_cave` with the `shovel` for 4 consecutive turns reveals `scarab`; a
5th dig without taking it first collapses the hole and kills the player
(`gameState.beachDig` tracks the count). `machine_switch` (`zorkData.bxs`)
ports `MSWITCH-FUNCTION` — turning it on with the lid closed turns coal
into `diamond` (anything else into `gunk`); wired into `handleTurnOn`. The
`bat` (`zorkData.bxs`) ports `BATS-ROOM`/`FLY-ME` via `bat_room`'s
`onEnter` hook: entering without `garlic` sweeps the player to a random
room from the `BAT-DROPS` pool. New `raise`/`lower` verbs
(`handleRaise`/`handleLower`) port `BASKET-F`, swapping
`raised_basket`/`lowered_basket` between `shaft_room`/`lower_shaft` along
with their contents, tracked via `gameState.flags.cageTop`.

### Egg / canary
New `breakEgg()` helper (ports `BAD-EGG`) replaces `egg` with `broken_egg`
in place, taking any nested `canary` to `broken_canary` with it; wired into
`handleDrop()` (dropping it, or the nest with the egg still inside, from
`up_a_tree`) and `tickThief()` (the thief stealing it also breaks it — a
deliberate extension past plain ZIL, where theft just relocates treasures
intact). `handleWind()` now triggers the canary's songbird/bauble payoff
(ported from `CANARY-OBJECT`) the first time you wind it intact in a
forest room (`gameState.flags.songSung` makes it one-shot); winding it
afterward, or winding `broken_canary`, does nothing further.

### Other single-item gaps
- `map` (`zorkData.bxs:893`) now reveals itself the moment `wonFlag` is set
  (folded into `checkEndgameTrigger()`, `zorkParser.bxs:1186`, rather than
  a separate one-shot check — real `SCORE-UPD` flips `WON-FLAG` and clears
  the map's `INVISIBLE` in the same breath, so this is the one event, not
  two).
- `gunk` (`zorkData.bxs:904`) is now produced when `putty` (now
  `burnable`) is burned, via a `handleBurn()` special case — note this
  specific mechanic isn't actually in ZIL (real ZIL never lets you burn
  putty), but was implemented as directed.

### Mirror rooms
`rub` is a new one-object verb (`handleRub`, `zorkParser.bxs`) wired up for
`mirror_1`/`mirror_2` (`zorkData.bxs:431,436`): rubbing either mirror swaps
`mirror_room_1`/`mirror_room_2`'s non-scenery contents and teleports the
player to the other room, per `MIRROR-MIRROR` in `1actions.zil`. Not
modeled: `MIRROR-MUNG` (breaking the mirror via MUNG/THROW/ATTACK, which
would permanently disable the swap and dock seven years of luck) and the
"rub with something other than your hands" tingling message, since this
engine's `rub` grammar has no indirect object yet.

### Boat / river
`gameState.boardedVehicle` (`zorkParser.bxs`) tracks whether the player is
in the boat. `board`/`launch`/the new `land` direction
(`handleBoard`/`handleLaunch`/`handleLand`) move the player onto the river
(`RIVER_LAUNCH`) and back to shore (each room's `land` exit key), and
`tickRiver()` advances the boat one room downstream per turn while boarded
(`RIVER_NEXT`); running off the end of `river_5` without landing is a
waterfall death, matching `I-RIVER`'s `JIGS-UP` in `1actions.zil`.
`punctured_boat` (`zorkData.bxs:690`) is produced by `handleBoard()` when
the player carries a weapon aboard, mirroring `RBOAT-FUNCTION`'s `BOARD`
check (the original tests for `SCEPTRE`/`KNIFE`/`SWORD`/`RUSTY-KNIFE`/
`AXE`/`STILETTO` by name; this engine generalizes that to "carrying
anything flagged `weapon`"). Not modeled: hitting the white cliffs' rocks
specifically (the original's puncture trigger is the `BOARD` check itself,
not a per-room probability — there's no separate rocks/probability
mechanic in `1actions.zil` to port), `DBOAT-FUNCTION`'s puncture-repair-with-
putty path (`FIX-BOAT`), and walking onto river rooms without the boat is
disallowed simply because the data has no walkable exit into them (matching
`NONLANDBIT` in practice) rather than via an explicit engine-level check.

## Genuinely unimplemented

### Other single-item gaps
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
