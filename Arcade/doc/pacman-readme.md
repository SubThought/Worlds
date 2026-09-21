# Premise Pacman

A Premise 3.4 port of the raylib maze-chase game (from the Clojure
original). One immutable idea kept intact: a single Game state threaded
through `step` and `draw-state`, decisions taken only at tile centres,
everything derived from the maze strings.

All commands below run from the `1.0/pacman/` folder — the entry file
groks its parts by relative path.

## Layout

```
1.0/pacman/
├── pacman.theory              entry: boot order, theory assembly, banner
├── cfg/pacman.daicho          configuration ledger (all tuning lives here)
├── src/axioms/game.theory     the game half — no raylib anywhere in it
├── src/portal/draw.theory     the drawing half — reads state, changes nothing
├── src/portal/main.theory     the loop: input, step, draw, unattended args
├── pkg/raylib/raylib.package      Option A: direct binds to system raylib
├── pkg/raylib/raylib-kit.package  Option B: binds to the RaylibKit shim
├── pkg/raylib/RaylibKit.c         Option B shim source (flat int32 ABI)
├── qed/pacman.suites          test suites over the pure axioms
└── doc/pacman-readme.md       this file
```

## Requirements

Premise 3.4 and raylib 5.x as a shared library. The library name is set
in `cfg/pacman.daicho` under `[Raylib :Library]`:

| Platform | value                                  |
|----------|----------------------------------------|
| Linux    | `"libraylib.so"` (default)             |
| macOS    | `"libraylib.dylib"`                    |
| Windows  | `"raylib.dll"` — on PATH, beside the Premise executable, or an absolute path |

## Running

```
premise pacman.theory
```

then, at the REPL:

```
(Pacman.run)                  ; play until the window closes
(Pacman.run 10)               ; quit after ten seconds of game time
(Pacman.run 3 "out.png" 76)   ; quit after three, capture frame 76
```

`run` returns the final score. Arguments omitted fall back to the
`[Run]` section of the daicho, so an unattended run can be configured
entirely from `cfg/pacman.daicho` (`:Deadline`, `:ShotPath`,
`:ShotFrame`) and started with a plain `(Pacman.run)` — useful for
scripted smoke tests and screenshots.

## Controls

Arrows or WASD steer. A press buffers the turn; it takes effect at the
next tile centre where that turn is legal, so pressing a little early
before a junction is exactly right. ENTER restarts after a game over.
ESC or the close button quits.

## First run: the smoke test (Phase 0)

Before playing, verify the four FFI assumptions listed at the top of
`pacman.theory`, in two calls:

```
(grok "cfg/pacman.daicho")
(require raylib from "pkg/raylib/raylib.package")
(raylib.InitWindow 300 200 "probe")     ; float + string + int32 params
(raylib.GetFrameTime)                   ; float return
(raylib.ClearBackground -16777216)      ; packed color (opaque black)
(raylib.CloseWindow)
```

A window that opens, a small real number back, no crash on the packed
color: Option A stands and nothing more is needed.

## Option B: the RaylibKit shim

If Phase 0 fails on float marshalling, the packed-color struct, or
`rlColor4ub`, switch to the shim. Everything crossing the FFI boundary
becomes int32 or string; the game files never change.

1. Build the shim (commands also in the header of `RaylibKit.c`):

   ```
   # Linux
   gcc -shared -fPIC -O2 -o RaylibKit.so RaylibKit.c -lraylib
   # Windows (mingw)
   gcc -shared -O2 -o RaylibKit.dll RaylibKit.c -lraylib -lopengl32 -lgdi32 -lwinmm
   # macOS
   clang -shared -fPIC -O2 -o RaylibKit.dylib RaylibKit.c -lraylib
   ```

2. Point `[Raylib :Library]` in the daicho at the built binary, e.g.
   `"pkg/raylib/RaylibKit.dll"`.

3. In `pacman.theory`, change the require line to
   `(require raylib from "pkg/raylib/raylib-kit.package")`.

## Configuration

Everything tunable sits in `cfg/pacman.daicho`. Speeds are tiles per
second, timers are seconds.

| Section  | Keys |
|----------|------|
| `Window` | `:Title`, `:Scale` (pixels per tile), `:Fps` |
| `Game`   | `:PacSpeed`, `:GhostSpeed`, `:FrightSpeed`, `:ChaseSecs`, `:ScatterSecs`, `:FrightSecs`, `:Lives`, `:DtMax` |
| `Run`    | `:Deadline`, `:ShotPath`, `:ShotFrame` |
| `Raylib` | `:Library` |

`:DtMax` is the frame-time clamp: below twenty frames a second the game
slows down instead of letting anything tunnel through a wall.

## Testing

```
premise pacman.theory
(grok "qed/pacman.suites")
(certify Pacman)              ; .: QED, or a list of issues
```

The suites cover the pure axioms only — tunnel wrapping, cell codes,
grid boundaries, the door (a wall to Pacman, not to a ghost), the
pinned dot count (198), and the ghost-combo score ladder (200, 400,
800, 1600). No window opens.

## How it works, briefly

The maze is nineteen columns by twenty-one rows of strings in
`game.theory` — walls `#`, dots `.`, power pellets `o`, the ghost-house
door `-`, house interior `G`, the start `P`. Everything else (grid,
dot count, door tile, start tiles) is derived by reading the strings;
there is no second table to fall out of sync. The layout is original
and machine-verified: every dot reachable, ghosts can leave the house,
row 10 is an open tunnel that wraps.

State is a handful of ephemerons (`Game`, `Pac`, four `Ghost`s) —
frame-loop state, deliberately never persisted. The loop in
`main.theory` polls keys onto the Pac, calls `step` with a clamped
`dt`, then `draw-state`; the axioms decide, the portal renders, and
neither knows the other exists.

Movement has one rule: directions change only at tile centres. Each
step advances by `speed * dt`; when a step reaches a centre, the
entity stops there, chooses (buffered turn first, then current
heading, else halt), and spends the remaining distance on the new
heading. A small epsilon absorbs floating-point near-misses.

The four ghosts differ only in their target arithmetic: Blinky aims at
Pacman's tile, Pinky four tiles ahead of it, Inky reflects Blinky
through the point two ahead, and Clyde chases until inside eight tiles,
then breaks for his corner. Chase and scatter alternate on the daicho
timers; a power pellet frightens everyone (random legal turns, slower,
edible), and eating ghosts on one pellet climbs the 200-400-800-1600
ladder. A ghost in the house targets the tile above the door, which
only ghosts may cross.

## Known divergence

Reversal, like any turn, waits for the next tile centre — the arcade
lets you reverse instantly mid-corridor. This follows the source's
decide-at-centres rule; noted in case it reads differently in play.

## Attribution

Copyright © 2013-2026 SubThought Corporation. All rights reserved.
Provided as-is, without warranty of any kind; see the header of any
source file for the full notice.

This is an original Premise implementation — code, maze layout, and the
RaylibKit shim were written fresh for this port; no source text was
copied from the originals. The game design follows:

- **raylib-pacman** by b12n-oss (Clojure) — the direct source of the
  port: state model, centre-decision movement, ghost personalities,
  unattended-run arguments. License: [verify in that repo's LICENSE
  file before distributing].
- **pacman.clj** by Michiel Borkent, from the babashka/ffi examples —
  the origin of the design. Copyright © Michiel Borkent, MIT License.
- **raylib** by Ramon Santamaria — the graphics library this program
  binds at runtime. Zlib license.

Per the MIT terms of the origin, the notice above is preserved with
this documentation. Pac-Man is a trademark of Bandai Namco; this is a
non-commercial study port and uses no original assets.
