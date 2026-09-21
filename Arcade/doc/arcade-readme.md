<!--
  ************************************************************************************

   Copyright(c) 2013-2026 SubThought Corporation. All Rights Reserved.

   THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
   OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
   FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.

   IN NO EVENT SHALL THE AUTHOR(S) OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM,
   DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
   ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE, ITS USE, OR OTHER
   DEALINGS IN THE SOFTWARE.

  ************************************************************************************

   arcade-readme.md   --  documentation for the Arcade psyche.

  ************************************************************************************
-->

# The Arcade

**A psyche that lets GIL play video games.**

SubThought Corporation

---

The Arcade is a **Psyche** — the interface between a GIL mind and a world.
Where Eidos gives the mind the ARC-AGI-3 grid, Expanse a robot's body and Aether
the internet, the Arcade gives it **a video game**. Its first title is
**Pacman**: the Premise port in `1.0/pacman/`.

The mind sees what a human player sees — a 1024 × 1024 picture of the screen —
and acts the way a human player acts: by steering. It is told nothing about what
the game is made of. It has to find out.

---

## 1. What the mind receives, and what it does not

**It receives pixels, a sense of what it may do, and three needs.**

Every percept carries the address of a 1024 × 1024 PNG — `frame://arcade/<guid>`,
the same scheme Eidos uses — showing the board drawn by the game's own drawing
code, HUD and all. The score and the lives appear in that
picture as digits and discs, exactly as they do for a person. If the mind comes
to read them, it will have learned to.

**It never receives the game's internals.** Not the maze, not the tile
coordinates, not the ghosts' names, not the score, the lives or the dot count as
numbers, and not the words READY or GAME OVER. A mind handed those would know
what the world was made of before it had looked, and would not be learning
anything.

```
out  →  PERCEPT · URGE · RESULT · REGISTER
in   ←  ATTEMPT · PSYCHE · TERMINATE
```

---

## 2. Two ways for the world to run

Set in `cfg/Arcade.daicho` under `[Play :Mode]`.

### `realtime` — the default

The game runs at its own pace whether or not the mind acts, exactly as it does
for a human. An action buffers a turn and returns at once. A percept goes out
every `:PerceiveEvery` frames — ten a second at the defaults. **The ghosts do not
wait for the mind to think.**

### `turns`

The world advances **only when the mind acts**: each action runs
`:FramesPerAction` frames at a fixed `:Dt`, then sends one percept. Between
actions nothing moves. This is the laboratory setting — deterministic, so the
same actions from the same state give the same frames, which is what lets the
mind confirm that a move does what it thought rather than merely observe it once.

### Why realtime is the default

A world that waits measures **what** the mind understands. A world that does not
also measures **whether it can understand in time**. Pacman was built to be the
second kind, and the mind should meet it as it is. Turns is for debugging, for
confirming a transition, and for a mind still too slow to keep up.

---

## 3. The actions

Offered to the mind when the psyche registers:

| action | what it does |
|---|---|
| `up` `down` `left` `right` | buffer a turn, exactly as a keypress does |
| `wait` | let the world run with no new input |
| `restart` | begin a fresh game — **always** |

**A direction is a buffered turn, not a move.** It takes effect at the next tile
centre where that turn is legal. So `up` in the middle of a corridor is neither
refused nor wasted; it waits for the junction. That is the game's own rule, and
the mind should learn it rather than have it smoothed away.

**`wait` is offered in both modes.** In turns it is how time passes without
steering. In realtime the world runs anyway — but choosing to hold a course is
still a decision, and a mind whose only record of restraint is the *absence* of an
attempt cannot learn from it: there is nothing to correlate a result with. `wait`
makes restraint something the mind did, and so something whose consequences it can
learn. And one repertoire across both modes means what the mind learns about its
actions in one carries to the other.

**`restart` belongs to the Arcade, not to Pacman.** Every title the Arcade hosts
offers it, in every state — playing or finished. A mind that could only begin
again after losing could never learn that abandoning a hopeless position is
itself a choice.

**What may be done changes with the game.** Every percept carries an `:Able` list.
While playing, it is everything. Once the game is over, the moves are withdrawn
and only `restart` remains — steering a finished game changes nothing, and an
action that is not available is refused **with a reason**, not silently ignored.

---

## 4. The three needs

Needs are how the mind is rewarded and punished without being told the rules.
Each arrives as an `URGE` carrying a `:Delta` and a `:Target`.

| need | delta | target | what moves it |
|---|---|---|---|
| **hunger** | dots still on the board | the full board | **falls** as dots are eaten |
| **finish** | dots eaten | the full board | **rises** as hunger falls |
| **survive** | lives lost this game | lives at the start | **rises** on a death |

### Hunger — homeostatic

A setpoint of zero, and a **falling** delta is the reward. Every dot eaten is a
small satisfaction.

### Finish — a goal gradient

The **same quantity seen from the other end.** Its delta **rises** as hunger
falls, so the urge to finish strengthens as the end comes into reach — and it
discharges when the board is cleared, starting again from zero on the next level.

**Why both.** A mind driven only by hunger eats whatever is nearest and leaves
stragglers in far corners, because the last few dots are worth no more than the
first. Finish is what makes those last few worth crossing the maze for — the pull
that grows as a task nears completion. That is Hull's goal gradient, and it is a
real feature of motivated behaviour.

> **Read the direction before reading the number.** Hunger's reward is a
> *falling* delta; finish's intensity is a *rising* one. A mechanism that assumed
> every falling delta was satisfaction would read finish exactly backwards.

### Survive — distress

Lives lost this game. A **rising** delta is distress.

### The score does not cross

Eating a ghost raises the score and changes no need. The score is pixels in the
HUD, and whether the mind comes to value it is something it would have to learn.

---

## 5. Running it

The Arcade runs as **its own process**, with its home at the pacman folder —
because `pacman.theory` loads its parts by relative path:

```
premise --home "…/1.0/pacman/" --grok "../gil/src/psyche/arcade.psyche" \
        --eval "(arcade-start)" --repl no
```

**Grokking the file starts nothing** — as with Eidos, `arcade-start` does. It
opens the window, starts a game, registers with the mind, and then runs the world
until the window closes, holding the process open while it does. Because grokking
alone opens nothing, the test suites can load the psyche without a window or a
live mind.

**The first percept follows the grant** — a percept sent before the mind has
issued a token would be dropped, and the mind's first sight of a world should not
be the one it throws away.

### Before the first run

**Set `[Window :Scale 44]` in `1.0/pacman/cfg/pacman.daicho`.** The board with its
HUD is 19 × 23 tiles, and 44 is the largest scale that fits 1024 pixels. The
psyche checks at startup: a smaller scale works and leaves more of the picture
empty, and says so; a larger one would crop the board, and is refused.

**Create `1.0/gil/etc/frames/in/` and `1.0/gil/etc/archive/arcade/`.** Frames go
into GIL's tree, not pacman's — the first is the working set every psyche shares
and the portal reads, the second the permanent record. Premise cannot create a
folder, so both must exist.

**Start the mind first**, or at least its Registrar. The psyche registers once
at startup.

---

## 6. Files

| file | where |
|---|---|
| `arcade.psyche` | `1.0/gil/src/psyche/` |
| `Arcade.daicho` | `1.0/gil/cfg/` |
| `arcade.suites` | `1.0/gil/qed/` |
| `arcade-readme.md` | `1.0/gil/doc/` |

The game itself — `pacman.theory` and its parts — is unchanged. The Arcade uses
three things from it: `new-game`, `step` and `draw-state`, and reads `Rows` and
`Cols` to size the picture.

---

## 7. Configuration

`cfg/Arcade.daicho`:

| section | keys |
|---|---|
| `Psyche` | `:Url`, `:Delay` |
| `Mind` | `:Registrar`, `:Perceiver`, `:Executor` — fallbacks only; the grant supplies the real addresses |
| `World` | `:Channel` (`pacman`), `:Title` |
| `Raster` | `:Width` `:Height` (1024), `:Hot`, `:Archive` |
| `Play` | `:Mode`, `:PerceiveEvery`, `:FramesPerAction`, `:Dt` |

**Frames are handled exactly as Eidos handles them.** Every frame gets a fresh
guid, so no two frames ever share an address — not even across restarts. The
psyche **deletes nothing**: frames are collected on disuse by consolidation,
never by perception. A ring buffer that reused names by position was considered
for the tree and rejected, because a detector still holding an address could find
a different frame there, silently and only under load.

**One difference from Eidos:** Eidos archives into a folder per run. Premise has
no call to make a folder, so Arcade puts the run in the file name instead —
`arcade_<run>_<sequence>.png` — still ordered, still append-only.

---

## 8. Things worth knowing

**READY! freezes the world for two seconds** at the start of every game. That is
the game's rule and it is left alone. The mind will learn that the start of a
game does not respond.

**Turns mode is deterministic except while the ghosts are frightened** — their
turns are random. Worth remembering when reading evidence gathered then.

**raylib discards the directory** of any path given to its screenshot function
and writes to the working directory instead. The psyche captures under a bare
name and moves the file. Passing a full path looks as if it works and puts every
frame in the wrong place.

**The window stays live in both modes.** raylib reads its event queue when a frame
is drawn, and a window nobody draws to while the mind deliberates is one the
operating system declares hung. So it is redrawn every tick — which also means
you can watch the mind play.

**Every raylib call happens on one thread.** OpenGL binds its context to the
thread that opened the window, so the world runs in a single loop in
`arcade-start`, and the agent that listens to the mind only *queues* what it is
sent. A draw from any other thread is undefined behaviour in the graphics driver —
a black frame, or a crash that never reproduces.

**Reversal waits for a tile centre**, as every turn does. The arcade original lets
Pacman reverse instantly mid-corridor; this port does not. The mind is learning
this game, not the original.

---

## 9. Testing

```
premise --home "…/1.0/pacman/"
(grok "../gil/src/psyche/arcade.psyche")
(grok "../gil/qed/arcade.suites")
(certify Arcade)
```

The suites cover what does not need a window: the repertoire, what may be done in
each state, steering, a new game's setpoints, registration and refusal of it, the
boundary — that a percept carries **none** of the game's numbers — frame
addressing, the raster's fit, and refusing an action that is not on offer.

Drawing, capture, the realtime tick and the turns advance all need raylib and a
window, so they want a scripted run in `turns` mode instead.

---

## 10. Adding a title

The Arcade is named for more than one game. A second title would be:

- a **new channel** on the same psyche — `[World :Channel]` — not a new psyche
- its own **moves**, replacing `arcade-moves`; **`restart` is added by the psyche
  for every title** and cannot be left out
- its own **needs**, if its rewards are different
- the same **raster, registration, beat and actuation** machinery, unchanged

---

Copyright © 2013–2026 SubThought Corporation. All Rights Reserved.
