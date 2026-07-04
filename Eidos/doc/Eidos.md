<!--
  Copyright © 2013-2026 SubThought Corporation. All Rights Reserved.

  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
  OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.

  IN NO EVENT SHALL THE AUTHOR(S) OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM,
  DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE,
  ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE, ITS USE, OR OTHER
  DEALINGS IN THE SOFTWARE.
-->

# The Eidos 

## Specification and Deployment Guide

**SubThought Corporation**

Version 1.0 — June 2026

---

## Table of Contents

1. Overview
2. The Percept Relation
3. Percepts
4. Common Content Idiom Slots
5. Actuations
6. Cross-Cutting Structure
7. Grid Representations
8. Architecture
9. Deployment File Set
10. The Package
11. The Theory
12. Configuration
13. Test Suites
14. API Key and Authentication
15. Etymology

---

## 1  Overview

The Eidos is a Psyche: the interface between a GIL mind and the ARC-AGI-3 interactive reasoning benchmark. It mediates two flows — percepts traveling inward from the grid world to the mind's mechanisms, and actuations traveling outward from the mind's mechanisms to the grid world. Unlike the Aether (internet) and the Expanse (physical reality), the Eidos interfaces with a turn-based puzzle environment where the world changes only when the agent acts.

ARC-AGI-3 tests four capabilities that map directly to GIL's architecture: exploration (the Perceiver senses the grid), modeling (the Totality stores patterns as knowledge), goal-setting (the coordination component drives action through needs and urges), and planning and execution (the Executor dispatches attempts). The Eidos Psyche connects these mechanisms to the ARC grid world.

The world is a 64×64 grid with 16 colors and 7 standardized actions. The agent receives no instructions — it must discover game mechanics through exploration, figure out the goal through observation, and solve the puzzle efficiently. Performance is scored by RHAE (Relative Human Action Efficiency): `(human_actions / ai_actions)²`. The closer to human efficiency, the higher the score.

The Eidos is pure Premise — no DLL, no bridge adapter. Premise's built-in `encode`/`decode json` and native HTTPS `tell`/`ask` talk directly to the ARC-AGI-3 REST API.

All percepts from the Eidos carry `:Modality Eidos`. The single channel is `Grid`. The `:Data` slot identifies the percept type.

---

## 2  The Percept Relation

The Percept relation from `totality.theory` is canonical. It does not change for the Eidos.

```
(relation Percept
  :M                 ; lexified monad
  :Modality          ; which Psyche — Eidos for this device
  :Channel           ; sub-channel — Grid (always)
  :Address  nil      ; API URL
  :Data     nil      ; data format or parsable structure type
  :Content  nil      ; idiom {:Slot value ...} — the actual structured data
)
```

### 2.1  Slot Semantics

| Slot | Meaning | Eidos Examples |
|---|---|---|
| `:Modality` | Which Psyche this came through. | `Eidos` (always) |
| `:Channel` | Sub-channel within the Psyche. | `Grid` (always) |
| `:Address` | API URL. | `"https://three.arcprize.org"` |
| `:Data` | Data format or percept type. | `grid-state`, `grid-delta`, `game-event` |
| `:Content` | Idiom with the actual structured data. | `{:GameId "ls20" :Level 1 :Width 64 :Height 64 :Grid {...} :State playing ...}` |
| `:Moment` | When the percept was received. | `\@m{2026060100000000}` |

For comparison, the Aether has 9 channels (Web, FileSystem, Process, etc.) because the internet is a rich, multi-domain environment. The Expanse has 8 sensory channels (Visual, Auditory, Haptic, etc.) because the physical world has multiple modalities. The Eidos has one channel — Grid — because the ARC world is a single visual grid. The complexity lives not in the number of channels but in the patterns within the grid.

---

## 3  Percepts

All percepts from the Eidos have `:Modality Eidos` and `:Channel Grid`.

### 3.1  `grid-state`

The primary percept. Delivered after every action.

```
[Percept :Modality Eidos  :Channel Grid
         :Address "https://three.arcprize.org"
         :Data grid-state
         :Content {:GameId           "ls20"
                   :Level            1
                   :Width            64
                   :Height           64
                   :Format           raw
                   :Grid             {0 0 0 0 0 3 3 3 0 ...}
                   :State            playing
                   :LevelsCompleted  0
                   :TotalLevels      5
                   :ActionsUsed      12
                   :HumanActions     8
                   :AvailableActions {RESET ACTION1 ACTION2 ACTION3 ACTION4 ACTION5 ACTION7}}
         :Moment \@m{...}]
```

| Slot | Type | Description |
|---|---|---|
| `:GameId` | string | ARC-AGI-3 game identifier (e.g., "ls20", "ft09") |
| `:Level` | integer | Current level within the game |
| `:Width` | integer | Grid width — always 64 |
| `:Height` | integer | Grid height — always 64 |
| `:Format` | literal | Grid encoding — `raw`, `sparse`, or `rle` |
| `:Grid` | list | The grid data in the configured format |
| `:State` | literal | Game state — `playing`, `win`, or `game-over` |
| `:LevelsCompleted` | integer | How many levels solved so far |
| `:TotalLevels` | integer | Total levels in this game |
| `:ActionsUsed` | integer | Agent's action count so far |
| `:HumanActions` | integer | Human baseline action count |
| `:AvailableActions` | list | Which of the 7 actions are valid right now |

### 3.2  `grid-delta`

Emitted alongside `grid-state` when the grid format is `raw`. Shows only what changed since the previous frame.

```
[Percept :Modality Eidos  :Channel Grid
         :Data grid-delta
         :Content {:Changes {{:I 66 :From 0 :To 3}
                              {:I 67 :From 0 :To 3}
                              {:I 68 :From 0 :To 3}}
                   :Count 3}]
```

### 3.3  `game-event`

Emitted when the game state transitions out of `playing`.

```
[Percept :Modality Eidos  :Channel Grid
         :Data game-event
         :Content {:State win
                   :LevelsCompleted 3
                   :TotalLevels 5}]
```

---

## 4  Common `:Content` Idiom Slots

| Slot | Type | Count | Notes |
|---|---|---|---|
| `:Grid` | list | 1 | The grid data in the configured format |
| `:State` | literal | 3 | Game state — playing, win, game-over |
| `:GameId` | string | 2 | Game identifier |
| `:Level` | integer | 2 | Current level |
| `:LevelsCompleted` / `:TotalLevels` | integer | 3 | Progress tracking |
| `:ActionsUsed` / `:HumanActions` | integer | 2 | Efficiency tracking |
| `:AvailableActions` | list | 1 | Valid actions for the current state |
| `:Format` | literal | 1 | Grid encoding format |
| `:Width` / `:Height` | integer | 1 | Grid dimensions (always 64) |
| `:Changes` / `:Count` | list/integer | 1 | Delta changes between frames |

---

## 5  Actuations

The ARC-AGI-3 world has 7 standardized actions. The grid changes only when the agent acts — this is a turn-based world.

### 5.1  Actuation Table

| `:Action` | ARC Name | Description | `:Parameters` idiom |
|---|---|---|---|
| `reset` | `RESET` | Start or restart game/level | `{:GameId "ls20"}` or `{}` for level reset |
| `action1` / `up` | `ACTION1` | Simple — semantically up | `{}` |
| `action2` / `down` | `ACTION2` | Simple — semantically down | `{}` |
| `action3` / `left` | `ACTION3` | Simple — semantically left | `{}` |
| `action4` / `right` | `ACTION4` | Simple — semantically right | `{}` |
| `action5` / `interact` | `ACTION5` | Simple — interact, select, rotate | `{}` |
| `action6` / `click` | `ACTION6` | Complex — requires x,y coordinates | `{:X 32 :Y 16}` |
| `action7` / `undo` | `ACTION7` | Undo the last action | `{}` |

The dispatch accepts both ARC names (`action1`) and semantic names (`up`, `click`, `undo`).

### 5.2  ATTEMPT Examples

```
[ATTEMPT :Action reset
         :Parameters {:GameId "ls20"}
         :Token ?tok]

[ATTEMPT :Action up
         :Parameters {}
         :Token ?tok]

[ATTEMPT :Action click
         :Parameters {:X 32 :Y 16}
         :Token ?tok]

[ATTEMPT :Action undo
         :Parameters {}
         :Token ?tok]
```

---

## 6  Cross-Cutting Structure

### 6.1  The Four Tuple Types

```
PERCEPT:  [PERCEPT :Modality Eidos :Channel Grid :Address A :Data D :Content {:...} :Moment M :Token T]
ATTEMPT:  [ATTEMPT :Action A :Parameters {:...} :Token T :By M]
RESULT:   [RESULT  :Action A :Status S :Reason R :Moment M :Token T]
URGE:     [URGE    :Need N :Source S :Delta D :Moment M :Token T]
```

### 6.2  Actuation → Percept Flow

Every action produces exactly one `grid-state` percept. Optionally also a `grid-delta` (if format is raw and the grid changed) and a `game-event` (if the game state changed to win or game-over).

### 6.3  Urges

| Need | Source | Trigger |
|---|---|---|
| `connectivity` | `Grid` | API connection lost |
| `efficiency` | `Grid` | Agent action count exceeds 3× human baseline |

The efficiency urge is unique to the Eidos. When the agent has used more than three times the human action count, the RHAE score drops below 0.11 — essentially failing. The urge signals the mind to change strategy or abandon the current approach.

---

## 7  Grid Representations

The Eidos supports three grid representations, configurable via the daicho or at runtime.

### 7.1  Raw

Flat list of 4096 integers (64 × 64), left to right, top to bottom. Color values 0–15. Background is 0.

```
:Format raw
:Grid   {0 0 0 0 0 0 0 0 ... 0 0 3 3 3 0 0 0 ... 0 0 3 0 3 0 0 0 ...}
```

Row from index: `y = index / 64`. Column: `x = index mod 64`. Index from coordinates: `(y * 64) + x`.

### 7.2  Sparse

Only non-zero (non-background) cells. List of `{:I index :C color}` pairs.

```
:Format sparse
:Grid   {{:I 66 :C 3} {:I 67 :C 3} {:I 68 :C 3} {:I 130 :C 3} {:I 132 :C 3}}
```

Compact for mostly-empty grids. A grid with 30 colored cells goes from 4096 values to 30 pairs.

### 7.3  Run-Length Encoding

Pairs of count and color, inline.

```
:Format rle
:Grid   {64 0  2 0  3 3  59 0  2 0  1 3  1 0  1 3  59 0  2 0  3 3  ...}
```

Read as: 64 zeros, 2 zeros, 3 threes, 59 zeros... Compact for grids with large uniform regions.

### 7.4  Utility Functions

The `eidos.package` provides conversion functions: `grid-to-sparse`, `grid-to-rle`, `sparse-to-grid`, `rle-to-grid`. Also: `grid-cell` (x,y lookup), `grid-row`, `grid-diff` (delta between frames), `grid-colors` (unique non-zero colors), `grid-count` (cells of a given color).

---

## 8  Architecture

### 8.1  The Eidos as Mediator

```
                    ┌─────────────────────────────────────────┐
                    │              THE MIND                    │
                    │                                         │
                    │  Perceiver ← ─ ─ PERCEPT / URGE / RESULT│
                    │  Detectors → Matcher → Storer → Activator│
                    │  Executor  ─ ─ ─ → ATTEMPT              │
                    │                                         │
                    └──────────────┬──────────────────────────┘
                                   │
                          ┌────────┴────────┐
                          │  EIDOS PSYCHE   │
                          │                 │
                          │  :Modality      │  Eidos (always)
                          │  :Channel       │  Grid (always)
                          │  :Data          │  grid-state |
                          │                 │  grid-delta |
                          │                 │  game-event
                          │                 │
                          │  encode json    │
                          │  tell / ask     │
                          │  decode json    │
                          │                 │
                          └────────┬────────┘
                                   │ HTTPS
                          ┌────────┴────────┐
                          │  ARC-AGI-3 API  │
                          │  REST / JSON    │
                          │  64×64 grid     │
                          │  16 colors      │
                          │  7 actions      │
                          └─────────────────┘
```

### 8.2  Turn-Based Flow

```
Mind: "I want to move up."
  → [ATTEMPT :Action up :Parameters {} :Token ?tok]
    → Eidos handler receives ATTEMPT
      → (encode {:action "ACTION1"} json) → JSON string
      → (tell ?handle ...) → POST /api/v1/command
      → (decode ?response json) → Premise idiom
      → eidos-wrap-frame → grid-state percept
    → [PERCEPT :Modality Eidos :Channel Grid :Data grid-state :Content {...}]
  → Perceiver receives the new grid
  → Detectors analyze patterns, shapes, colors, changes
  → Mind reasons about what happened and decides next action
```

No polling. No timer. One action in, one grid out. The mind's speed of thought is the clock.

---

## 9  Deployment File Set

| File | Category | Description |
|---|---|---|
| `eidos.package` | Protocol | Pure Premise. API calls, actions, grid compression, grid utilities, scorecards |
| `eidos-psyche.theory` | Theory | Single agent: minimal job (health check) + handler (action dispatch) |
| `Eidos.daicho` | Configuration | API URL, API key, grid format, urge thresholds |
| `eidos.suites` | Tests | 7 suites: grid compression, utilities, connection, actions, percepts, scorecards, lifecycle |

No DLL. No provider. No bridge. Four files, pure Premise.

---

## 10  The Package

`eidos.package` contains 10 sections:

**API calls:** `eidos-api-post` and `eidos-api-get` wrap the `encode json` → `tell`/`ask` → `decode json` pattern.

**Game management:** `eidos-reset` (start/restart game by ID), `eidos-reset-level` (restart current level), `eidos-list-games`.

**Actions:** `eidos-step` (generic), `eidos-step-xy` (with coordinates), plus named convenience functions: `eidos-up`, `eidos-down`, `eidos-left`, `eidos-right`, `eidos-interact`, `eidos-click`, `eidos-undo`.

**Scorecards:** `eidos-scorecard-open`, `eidos-scorecard-close`, `eidos-scorecard-get` — for tracking performance across multiple games.

**Grid compression:** `grid-to-sparse`, `grid-to-rle`, `sparse-to-grid`, `rle-to-grid`.

**Grid utilities:** `grid-cell`, `grid-row`, `grid-diff`, `grid-colors`, `grid-count`.

**Dispatch:** `eidos-dispatch` maps both ARC names and semantic names to the correct API call.

---

## 11  The Theory

`eidos-psyche.theory` defines a single agent:

**Job (proactive):** nearly empty — checks API connection health every 60 Moments. The grid world is turn-based, so there is nothing to poll.

**Handler (reactive):** receives ATTEMPTs from the Executor. Dispatches the action through `eidos-dispatch`. Calls `frame-to-percept` which emits up to three percepts per action (grid-state, grid-delta, game-event) and monitors action efficiency. Returns a RESULT to the Perceiver.

```
(agent Eidos eidos-job ?psyche-url eidos-handler ?psyche-delay {sensing acting})
```

---

## 12  Configuration

`Eidos.daicho` contains four sections:

| Section | Contents |
|---|---|
| `[Psyche]` | Agent URL, port, delay (`\@m{60}` — slow, turn-based world) |
| `[API]` | ARC-AGI-3 REST URL and API key |
| `[Grid]` | Grid format: `raw`, `sparse`, or `rle` |
| `[Urges]` | Connectivity loss and efficiency degradation thresholds |

---

## 13  Test Suites

`eidos.suites` contains 7 test suites:

| Suite | Tests |
|---|---|
| `Eidos.grid-sparse` | Sparse compression round-trip |
| `Eidos.grid-rle` | Run-length compression round-trip |
| `Eidos.grid-utilities` | Cell access, row extraction, diff, colors, count |
| `Eidos.connection` | API connection, status, game listing |
| `Eidos.actions` | Reset, simple actions, complex action, undo |
| `Eidos.percept-formation` | PERCEPT, RESULT, URGE tuple structure |
| `Eidos.scorecard` | Open, get, close scorecard lifecycle |
| `Eidos.full-lifecycle` | End-to-end: connect, list, scorecard, play, format switching, teardown |

---

## 14  API Key and Authentication

The ARC-AGI-3 API key serves dual purposes:

**Authentication:** passed in the HTTP Authorization header of every request to the ARC-AGI-3 REST API. Without a key, an anonymous key is used with reduced rate limits and no access to public games at release.

**Device credential:** registered with the GIL Registrar during startup. The key becomes the `:Token` slot on every PERCEPT, RESULT, and URGE tuple, identifying which device produced the data.

To obtain a key: visit `https://three.arcprize.org/user`, create an account, and copy your API key. Paste it into the `[API :Key]` slot in `Eidos.daicho`.

---

## 15  Etymology

**Eidos** (Greek εἶδος, *eidos*) — from the verb εἴδω (*eidō*), "to see." In Greek, seeing and knowing are the same word. The perfect tense of *eidō* is οἶδα (*oida*) — literally "I have seen," but meaning "I know." To have seen is to know.

Plato elevated eidos to mean the ideal Form — the unchanging reality behind appearances. Every chair is an imperfect copy of the eidos of Chair. Every triangle is an imperfect copy of the eidos of Triangle.

Every ARC-AGI-3 puzzle asks: what is the eidos behind these colored tiles? The surface is a 64×64 grid of colors. The eidos is the transformation rule — the hidden form that governs how inputs become outputs, how actions change the grid, what the goal is. Finding the eidos is solving the puzzle.

The Eidos is where seeing becomes knowing. The mind looks at the grid, sees the pattern, and in that moment knows the rule. That is the essence of abstraction and reasoning — and the essence of the word.

Where Ìtẹ́ names what's at stake (the throne), Aether names the medium (propagation), and Expanse names the territory (vastness), Eidos names the act of perception itself — the moment where raw sensation becomes understanding.
