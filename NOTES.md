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
V-SCORE's thresholds. `checkEndgameTrigger()` sets `wonFlag` once score
hits `TREASURE_MAX_SCORE` — the achievable max given what's reachable right
now (it excludes treasures still gated on the unimplemented puzzles below:
trunk, scarab, pot_of_gold, diamond, broken_egg, bauble, skull — see
`zorkParser.bxs:177-198`). Once `wonFlag` is set, `stone_barrow`'s `sw`/`in`
exits open, and walking in there fires the verbatim ZIL ending
(`stone_barrow.onEnter`, `zorkData.bxs:946-968`) via `endGameWin()`.

As each puzzle below gets implemented and starts placing/revealing its
treasure, `TREASURE_MAX_SCORE` recomputes higher automatically — except
`skull`, which needed an explicit removal from `UNREACHABLE_TREASURES`
(`zorkParser.bxs:182`) once Hades exorcism was wired up (done — see below),
since `lldFlag` gating it isn't detectable from `hidden`/`location` alone.

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

## Genuinely unimplemented

### Mirror rooms
- `mirror_1`/`mirror_2` (`zorkData.bxs:434,439`) — rubbing either should
  teleport you to the other one. Not implemented.

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
- `inflated_boat` (`zorkData.bxs:688`) can be created (`handleInflate`,
  `"inflate boat with pump"`) but actually sailing it on river rooms isn't
  implemented.
- `punctured_boat` (`zorkData.bxs:693`) — meant to result from hitting the
  white cliffs' rocks — is never produced.

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
