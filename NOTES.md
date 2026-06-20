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

### Dam / reservoir
- Turning the bolt with the wrench toggles `lowTide` directly
  (`handleTurn`); the real `GATES-OPEN` transitional state and gate-flag
  bubble from the ZIL source aren't modeled (`zorkData.bxs:551`).
- `leak` (`zorkData.bxs:611`) — meant to appear once the dam puzzle is
  underway — is never placed.
- `trunk` (`zorkData.bxs:426`) stays hidden forever; it should reveal once
  the reservoir is drained.

### Mirror rooms
- `mirror_1`/`mirror_2` (`zorkData.bxs:434,439`) — rubbing either should
  teleport you to the other one. Not implemented.

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

## Implemented since this was written

- **Hades exorcism**: `lldFlag` is now set by the bell/book/candles
  ceremony, ported from `LLD-ROOM`'s `M-BEG` clause in `1actions.zil`.
  Ringing the `bell` at `entrance_to_hades` (`handleRing`) scares off the
  `ghosts` and swaps it for `hot_bell`, dropping any lit `candles` you're
  holding. Lighting the `candles` again and then reading the `book`
  there (`handleLight`/`handleRead`) completes the ceremony, removes the
  `ghosts`, and sets `lldFlag`, unblocking the gate to
  `land_of_living_dead`.
- **Coffin curse**: `coffinCure` is now set by `pray` (`handlePray`) at
  `south_temple`, as long as you aren't carrying the `coffin` at the
  time — unblocking `south_temple`'s `down` exit to `tiny_cave`.
