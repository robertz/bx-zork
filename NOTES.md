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
`leak` (`zorkData.bxs:608`) appears when you push the `blue_button` in the
Maintenance Room (`handlePush`), with the verbatim `BUTTON-F` burst-pipe
message (pushing it again reports it jammed); `trunk` (`zorkData.bxs:421`)
reveals itself via the `reservoir` room's `onEnter` hook once `lowTide` is
true.

The bolt/wrench toggle is no longer an instant flip. `GATE-FLAG`
(`gameState.flags.gateFlag`) — toggled by the Maintenance Room's
`yellow_button`/`brown_button` — now gates whether the bolt turns at all
("The bolt won't turn with your best effort." if not); turning it toggles
`GATES-OPEN` (`gatesOpen`), and `lowTide` only catches up 8 turns later via
a new `tickDam()` per-turn hook (porting `I-RFILL`/`I-REMPTY`'s delayed
transition, not the instant flip this used to be). `dam_room`'s
description now covers the real 4-state `lowTide`×`gatesOpen` matrix from
`DAM-ROOM-FCN`, plus the bubble glowing while `gateFlag` is set.

Pushing the blue button also starts the Maintenance Room's flood timer
(`tickMaintRoom()`, porting `I-MAINT-ROOM`): water rises one `DROWNINGS`
stage per turn (verbatim 9-phrase table), fully floods the room at level
14 — permanently blocking re-entry — and drowns the player via `endGame()`
if they're still inside when it happens. `put putty on leak` stops it
(`fixMaintLeak()`, porting `FIX-MAINT-LEAK`); `put putty on punctured_boat`
repairs the boat (`fixBoat()`, porting `FIX-BOAT` exactly: it moves the
still-*deflated* `inflatable_boat` over and removes `punctured_boat` — you
have to re-inflate it again with the pump, same as ZIL). Not modeled: the
edge case where flooding sweeps an occupied boat over the falls
(`I-MAINT-ROOM`'s `INFLATED-BOAT`/`VEHBIT` clause) — a narrow overlap with
the boat/river system, left out as a documented simplification.

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
`mirror_1`/`mirror_2` (`zorkData.bxs:441,445`): rubbing either mirror swaps
`mirror_room_1`/`mirror_room_2`'s non-scenery contents and teleports the
player to the other room, per `MIRROR-MIRROR` in `1actions.zil`. Not
modeled: the "rub with something other than your hands" tingling message,
since this engine's `rub` grammar has no indirect object yet.

`MIRROR-MUNG` is now implemented: ZIL routes `MUNG`/`THROW`/`ATTACK` against
a mirror to the same breakage branch, but this engine has no `mung` verb
and no `throw` verb yet (a separate, parallel task), so only `attack`
(`handleAttack`, `zorkParser.bxs`) is wired — attacking `mirror_1`/`mirror_2`
special-cases before the normal villain-combat path (mirrors aren't
`actor`s) and sets `gameState.flags.mirrorBroken`, a single flag shared by
both rooms since the two mirrors are a linked pair in ZIL too. Once broken:
`handleRub` refuses the teleport ("The mirror is broken into many
pieces."), `handleExamine` reflects the shattered state, the mirror rooms'
descriptions append ZIL's "Unfortunately, the mirror has been destroyed by
your recklessness." line, and repeat attacks get "Haven't you done enough
damage already?" — all ported verbatim from `MIRROR-MIRROR`/`MIRROR-ROOM`
in `1actions.zil`. The "seven years' supply of good luck" line is ported as
flavor text only; this engine has no mechanical luck system to dock.

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

### Light source fuel timers
`brass_lantern` (185 lit turns) and `candles` (35, already lit at game
start) now run down once per turn while on, via `tickLightFuel()`
(`zorkParser.bxs`) — a flat per-object `fuel` counter standing in for
ZIL's `LAMP-TABLE`/`CANDLE-TABLE` interrupt-queue mechanics, with the same
text and stage thresholds (warnings at 100/170 turns used for the lamp,
20/30 for the candles, permanent burnout one turn after the final
warning). A burned-out light source (`burnedOut`) can never be relit.
`match` (`gameState.matchCount`, 6 per game) is handled separately —
`handleStrikeMatch()` ports `MATCH-FUNCTION` verbatim: each strike burns
for exactly 2 turns (`tickMatch()`) unless put out early, goes out
instantly in `lower_shaft`/`timber_room` (still consuming the match), and
refuses once the matchbook is empty. Warnings/burnout messages only print
when the light source is actually held or in the player's room
(`lightSourceVisible()`), matching ZIL's `HELD?`/`IN HERE` check, but the
countdown itself always runs in the background, matching ZIL's demon.

Not modeled: `torch` has no timer (real ZIL doesn't give it one either —
it's the one light source that never runs out).

Darkness/visibility is now fully implemented — see `roomIsLit()` in
`zorkParser.bxs`. Rooms with `"lit": true` (all outdoor/above-ground rooms
and the house interior) are always lit. Underground rooms require a carried
or in-room light source. In darkness: `describeRoom()` shows the pitch-black
message, `getVisibleObjectIds()` restricts scope to carried items only (so
room objects are invisible and untouchable), and `tickGrue()` fires each turn
after a 2-turn grace period with ZIL-faithful probabilities (65% harmless
warning, ~17% grue description, ~17% death via `endGame()`).

### Throw
New `throw`/`hurl`/`chuck`/`toss` verb (`handleThrow`, `zorkParser.bxs`,
wired into `TWO_OBJECT_VERBS` with an optional `at` preposition) ports
`V-THROW` (`gverbs.zil:1445`) generically: `throw X` alone drops it in the
current room ("Thrown."); `throw X at <villain>` has the villain duck and
the object falls to the ground (no per-villain catch/anger mechanics —
deliberately generic); `throw X at me`/`myself` is the verbatim fatal
"terrific throw" death via `endGame()`. Two object-specific overrides take
priority over that generic behavior, both verbatim-ported: `LANTERN`'s
`THROW` case (`1actions.zil`) smashes `brass_lantern` (removing it
entirely, turning off its light) and places `broken_lamp` (`zorkData.bxs:943`)
in the room instead — the gap this was tracked under is now closed; and
`EGG-OBJECT`'s `OPEN MUNG THROW` case (`1actions.zil`, via the existing
`BAD-EGG`-derived `breakEgg()` helper) breaks the egg even when thrown at a
villain, taking priority over the generic duck response for that one item.

### Diagnose
`handleDiagnose()` (`zorkParser.bxs:775`) used to just return "You are in
perfect health." unconditionally. Now a verbatim port of `V-DIAGNOSE`'s
full text, including the two lines that depend on the mechanics below:
"which will be cured after N moves" (`CURE_WAIT*(WD-1) + cureTimer`) and
"you have been killed once/twice" (`gameState.deaths`). `WD` is the wound
count (`-playerStrengthAdjustment`), `RS` is `fightStrength(false) + WD`
(the *unadjusted* base strength plus wound count — not the wound-adjusted
strength `fightStrength(true)` gives; matches `V-DIAGNOSE`'s own `MS`/`WD`
formula in `1actions.zil:3993`, odd as that addition looks).

### Wound healing over time
`tickCure()` (`zorkParser.bxs`, wired into `processCommand()`'s per-turn
sequence) ports `I-CURE`: while `gameState.cureTimer` is running it counts
down each turn, and on hitting 0 heals `playerStrengthAdjustment` by 1 and
re-arms itself (`CURE_WAIT` = 30 turns) if still wounded, exactly like
`I-CURE`'s self-requeuing `ENABLE <QUEUE I-CURE CURE-WAIT>`.
`applyBlowToPlayer()` arms the timer the moment a blow actually wounds the
player (light/serious wound, not stagger/disarm), matching `WINNER-RESULT`'s
own enable condition.

### Death counter / resurrection
`endGame()` (`zorkParser.bxs`) ports `JIGS-UP`'s core resurrection loop:
every death costs `SCORE_PENALTY_PER_DEATH` (10) points; the first
`DEATH_LIMIT` (2) deaths respawn the player in `forest_1` at
`FIGHT-STRENGTH` 1 ("I can't quite fix you up completely") rather than
ending the session, and any carried `brass_lantern`/`coffin` is left behind
in `living_room`/`egypt_room`. `gameState.deaths` tracks resurrections for
`handleDiagnose()`; the next death after the limit is permanent, verbatim
"suicidal maniac" ending. Deliberately *not* ported: ZIL's `south_temple`/
`,DEAD` "ghost limbo" branch (resurrecting as a translucent, `ALWAYS-LIT`
ghost who has to find their way back to the Hades gate to rejoin the
living) and `RANDOMIZE-OBJECTS`'s full treasure scatter (every other
carried treasure relocated to a random room, `sword`'s trophy value zeroed
forever) — both are substantial separate features layered on top of the
basic respawn, not part of "can you die and keep playing."

## Genuinely unimplemented

### Other single-item gaps
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
