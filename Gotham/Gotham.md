<!--
  Copyright © 2013-2026 SubThought Corporation. All Rights Reserved.

  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
  OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
-->

# The Gotham Psyche

## Specification and Deployment Guide

**SubThought Corporation**

Version 1.0 — September 2026

---

## Table of Contents

1. Overview
2. The Percept Relation
3. Percepts by Channel
4. Common Content Idiom Slots
5. Actuations
6. Cross-Cutting Structure
7. World Representation
8. Architecture
9. Deployment
10. Configuration
11. Adapter
12. Theory
13. Test Suites
14. Framework Asymmetries
15. Domain Coverage
16. Etymology

---

## 1 Overview

**The Gotham is a Psyche:** the interface between a GIL mind and the *Grand Theft Auto VI* game world.

It mediates two flows — percepts traveling inward from the game world to the mind's mechanisms, and actuations traveling outward from the mind's mechanisms to the game world.

Gotham interfaces with a persistent, real-time, embodied 3D game world. The mind occupies an avatar in the world and can perceive places, people, vehicles, objects, events, state, and spatial relationships while acting through movement, interaction, navigation, communication, and other game controls.

The Psyche is the boundary. It does not itself model the meaning of the world. It transports observations and actions between the game and the GIL mind. Perception, Totality formation, recollection, planning, coordination, and execution remain responsibilities of the mind.

The Gotham Psyche is therefore analogous to a sensory and motor system for a virtual body.

### 1.1 World

The world is the *Grand Theft Auto VI* game environment: a persistent, simulated, navigable 3D environment containing terrain, roads, buildings, interiors, vehicles, pedestrians, non-player characters, wildlife, weather, time, missions, objects, and events.

The Psyche exposes this environment through structured percepts rather than requiring the mind to consume the game's native representation directly.

### 1.2 Modality

All Gotham percepts carry:

```
:Modality Gotham
```

The `:Channel` slot identifies the perceptual or control domain.

Unlike a biological Psyche, Gotham can expose both rendered visual information and structured world-state information. A visual frame may therefore coexist with symbolic observations of position, entities, velocity, mission state, or world time.

---

## 2 The Percept Relation

The `Percept` relation from `totality.theory` is canonical.

```
(relation Percept
  :M                 ; lexified monad
  :Modality          ; which Psyche — Gotham
  :Channel           ; sub-channel within the Psyche
  :Address  nil      ; source / endpoint / session
  :Data     nil      ; data format or parsable structure type
  :Content  nil      ; idiom {:Slot value ...}
)

(template Percept
  {
    :Modality  nil
    :Channel   nil
    :Address   nil
    :Data      nil
    :Content   nil
    :Moment    0      ; when the percept was received
  })
```

### 2.1 Slot Semantics

| Slot | Meaning | Gotham Examples |
|---|---|---|
| `:Modality` | Which Psyche produced the percept | `Gotham` |
| `:Channel` | Sub-channel within Gotham | `Visual`, `World`, `Avatar`, `Entity`, `Vehicle`, `Mission`, `Navigation`, `Audio`, `Environment`, `System` |
| `:Address` | Source, session, or adapter endpoint | game-session identifier, capture source, adapter endpoint |
| `:Data` | Type of structured percept | `frame`, `world-state`, `entity-state`, `vehicle-state`, `mission-state`, `position`, `event`, `audio`, etc. |
| `:Content` | Structured data idiom | `{:X ... :Y ... :Z ...}` |
| `:Moment` | When the percept was received | `\@m{...}` |

The Gotham Psyche should preserve the distinction between **raw perception** and **structured world state**. A camera frame is not itself an `Object`, `Shape`, `Scene`, or image schema. Those are derived by the perceptual mechanisms of the mind.

---

## 3 Percepts by `:Channel`

### 3.1 Channel `Visual`

Visual percepts represent what the avatar's viewpoint sees.

| `:Data` | Description | `:Content` |
|---|---|---|
| `frame` | Rendered camera image | `{:Width 1920 :Height 1080 :Format png :Camera first-person}` |
| `depth` | Depth image or depth field | `{:Width 1920 :Height 1080 :Unit meter :Data ...}` |
| `segmentation` | Optional semantic segmentation | `{:Classes {...} :Data ...}` |
| `capture` | Camera capture metadata | `{:FrameId ... :Timestamp ... :Camera ...}` |

Example:

```
[PERCEPT :Modality Gotham
         :Channel Visual
         :Address "gotham://session-1/camera"
         :Data frame
         :Content {:Width 1920
                   :Height 1080
                   :Format png
                   :Camera third-person}
         :Moment \@m{...}]
```

Visual frames are inputs to the normal GIL perceptual pipeline. Detectors may derive `Icon`, `Geon`, `Shape`, `Object`, `Image`, and `Scene` representations from them.

### 3.2 Channel `World`

World-state percepts describe the simulated environment.

| `:Data` | Description |
|---|---|
| `world-state` | Current global world state |
| `world-time` | Simulated date/time |
| `weather-state` | Weather and environmental conditions |
| `location-state` | Current region, district, or named location |
| `world-event` | Significant environmental event |

Example:

```
[PERCEPT :Modality Gotham
         :Channel World
         :Data world-state
         :Content {:Time ...
                   :Weather ...
                   :Location ...
                   :WantedLevel ...
                   :Entities {...}}
         :Moment \@m{...}]
```

### 3.3 Channel `Avatar`

Avatar percepts describe the mind's embodied player character.

| `:Data` | Description |
|---|---|
| `avatar-state` | General player-character state |
| `position` | World position and orientation |
| `movement-state` | Standing, walking, running, swimming, falling, etc. |
| `health-state` | Health / damage state |
| `equipment-state` | Current equipment or carried items |
| `wanted-state` | Wanted level or law-enforcement state |

Example:

```
[PERCEPT :Modality Gotham
         :Channel Avatar
         :Data position
         :Content {:X 125.4
                   :Y -48.2
                   :Z 17.8
                   :Yaw 1.57
                   :Pitch 0.0}
         :Moment \@m{...}]
```

### 3.4 Channel `Entity`

Entity percepts describe people, animals, objects, and other identifiable world entities.

```
:Data entity-state
```

Typical content:

```
{:Id "entity-1042"
 :Type pedestrian
 :Position {:X ... :Y ... :Z ...}
 :Velocity {:X ... :Y ... :Z ...}
 :Heading ...
 :Visible yes
 :Distance ...
 :Relation unknown}
```

The Psyche reports entity identity and state when available. The mind is responsible for deciding what the entity means.

### 3.5 Channel `Vehicle`

Vehicle percepts describe cars, motorcycles, boats, aircraft, and other vehicles.

```
:Data vehicle-state
```

Typical content:

```
{:Id "vehicle-22"
 :Type automobile
 :Position {:X ... :Y ... :Z ...}
 :Velocity {:X ... :Y ... :Z ...}
 :Heading ...
 :Occupants {...}
 :State parked}
```

### 3.6 Channel `Mission`

Mission percepts describe mission and objective state.

| `:Data` | Description |
|---|---|
| `mission-state` | Active mission state |
| `objective-state` | Current objective |
| `mission-event` | Mission transition |
| `checkpoint-state` | Progress through an objective |

Example:

```
[PERCEPT :Modality Gotham
         :Channel Mission
         :Data objective-state
         :Content {:MissionId "..."
                   :ObjectiveId "..."
                   :State active
                   :Description "..."
                   :Target ...}
         :Moment \@m{...}]
```

### 3.7 Channel `Navigation`

Navigation percepts expose spatial information useful to route planning.

| `:Data` | Description |
|---|---|
| `map-state` | Map or navigation state |
| `route-state` | Active route |
| `waypoint-state` | Current destination |
| `position` | Position relative to world/navigation frame |

A route percept can contain:

```
{:Origin {:X ... :Y ... :Z ...}
 :Destination {:X ... :Y ... :Z ...}
 :Waypoints {{:X ... :Y ... :Z ...} ...}
 :Distance ...
 :Mode driving}
```

### 3.8 Channel `Audio`

Audio percepts expose sounds or transcribed speech when the adapter provides them.

| `:Data` | Description |
|---|---|
| `audio-frame` | Raw audio |
| `sound-event` | Structured sound event |
| `transcription` | Speech transcription |

### 3.9 Channel `Environment`

Environmental percepts include conditions that affect the simulated world.

```
:Data weather-state
```

Possible fields include:

```
{:Weather rain
 :Intensity ...
 :Visibility ...
 :Wind ...
 :Temperature ...
 :TimeOfDay ...}
```

### 3.10 Channel `System`

System percepts describe the health and state of the Psyche itself.

| `:Data` | Description |
|---|---|
| `connection-status` | Game/adapter connectivity |
| `capture-status` | Capture pipeline state |
| `command-status` | Last command state |
| `session-state` | Game session state |

---

## 4 Common `:Content` Idiom Slots

Gotham percepts share several recurring fields.

| Slot | Type | Meaning |
|---|---|---|
| `:Id` | string | Stable identifier for an entity or event |
| `:Type` | literal | Entity or percept type |
| `:Position` | idiom | World coordinates |
| `:Velocity` | idiom | Motion vector |
| `:Heading` | real | Orientation |
| `:State` | literal | Current state |
| `:Moment` | moment | Observation time |
| `:Distance` | real | Distance from avatar or viewpoint |
| `:Visible` | boolean | Whether entity is currently visible |
| `:Source` | literal/string | Source of observation |
| `:Confidence` | real | Confidence when derived rather than directly supplied |

### 4.1 Spatial Idiom

Gotham uses a common 3D coordinate idiom:

```
{:X n :Y n :Z n}
```

Orientation may be represented as:

```
{:Yaw n :Pitch n :Roll n}
```

The Psyche should preserve the native coordinate convention of the adapter and identify that convention in configuration.

---

## 5 Actuations

Actuations are commands dispatched from the GIL Executor to the game world.

Following GIL convention, outbound commands are represented as `ATTEMPT` structures and their outcomes return as `RESULT` percepts.

### 5.1 Attempt Structure

```
(structure Attempt
  :Act         nil
  :Args        nil
  :Token       nil
  :By          nil
)
```

### 5.2 Actuation Table

| # | `:Action` | Description | `:Parameters` |
|---|---|---|---|
| A01 | `move` | Move avatar | `{:Forward n :Strafe n :Duration n}` |
| A02 | `look` | Change viewpoint | `{:Yaw n :Pitch n :Duration n}` |
| A03 | `jump` | Jump | `{}` |
| A04 | `crouch` | Crouch | `{:State on}` |
| A05 | `sprint` | Sprint | `{:State on}` |
| A06 | `interact` | Interact with nearby entity | `{:EntityId "..."}` |
| A07 | `enter-vehicle` | Enter a vehicle | `{:VehicleId "..."}` |
| A08 | `exit-vehicle` | Exit current vehicle | `{}` |
| A09 | `drive` | Control vehicle movement | `{:Throttle n :Brake n :Steering n}` |
| A10 | `vehicle-look` | Look while driving | `{:Yaw n :Pitch n}` |
| A11 | `attack` | Perform attack/action | `{:TargetId "..."}` |
| A12 | `aim` | Aim or target | `{:X n :Y n}` |
| A13 | `select` | Select equipment/item | `{:ItemId "..."}` |
| A14 | `use` | Use selected item | `{}` |
| A15 | `navigate` | Set navigation destination | `{:X n :Y n :Z n}` |
| A16 | `cancel-route` | Cancel navigation route | `{}` |
| A17 | `mission-action` | Perform mission-specific action | `{:ActionId "..."}` |
| A18 | `camera` | Change camera mode | `{:Mode ...}` |
| A19 | `pause` | Pause game if supported | `{}` |
| A20 | `resume` | Resume game if supported | `{}` |

The exact available actions depend on the adapter and game execution environment.

### 5.3 Actuation Domains

**Avatar:** `move`, `look`, `jump`, `crouch`, `sprint`

**Vehicle:** `enter-vehicle`, `exit-vehicle`, `drive`, `vehicle-look`

**Interaction:** `interact`, `use`, `select`

**Navigation:** `navigate`, `cancel-route`

**Mission:** `mission-action`

**Camera:** `camera`

**System:** `pause`, `resume`

### 5.4 Attempt Example

```
[ATTEMPT :Act  move
         :Args  {:Forward 1.0
                 :Strafe 0.0
                 :Duration 500}
         :Token ?tok
         :By \@m{...}]
```

---

## 6 Cross-Cutting Structure

The Gotham Psyche follows the canonical four-tuple model:

```
PERCEPT: [PERCEPT :Modality Gotham :Channel C :Data D
                  :Content {:...} :Moment M :Token T]

ATTEMPT: [ATTEMPT :Act A :Args {:...}
                  :Token T :By M]

RESULT:  [RESULT :Action A :Status S :Reason R
                  :Moment M :Token T]

URGE:    [URGE :Need N :Source S :Delta D
                  :Moment M :Token T]
```

### 6.1 Actuation → Percept Pairings

| `:Action` | Expected percept |
|---|---|
| `move` | `Avatar.position`, `Avatar.movement-state` |
| `look` | `Visual.frame`, `Avatar.position` |
| `enter-vehicle` | `Avatar.avatar-state`, `Vehicle.vehicle-state` |
| `exit-vehicle` | `Avatar.avatar-state` |
| `drive` | `Vehicle.vehicle-state`, `Visual.frame` |
| `interact` | `Entity.entity-state` or `world-event` |
| `navigate` | `Navigation.route-state` |
| `mission-action` | `Mission.objective-state` or `mission-event` |
| `camera` | `Visual.frame` |
| all actions | `RESULT` |

As with the Aether and Eidos, the mind learns what its body did through subsequent perception.

---

## 7 World Representation

Gotham is a real-time 3D world. The Psyche should expose enough state for GIL's perceptual and spatial systems to construct a `Scene`.

### 7.1 Scene Formation

A typical perceptual cycle is:

```
Game World
    |
    v
Gotham Psyche
    |
    +---- Visual frame
    +---- Depth / geometry
    +---- Entity state
    +---- Avatar state
    +---- Environment
    |
    v
Perceiver
    |
    v
Detectors
    |
    +---- Icon
    +---- Geon
    +---- Shape
    +---- Object
    +---- Image
    +---- Image Schemas
    |
    v
Scene
```

The Psyche does **not** decide that a visual region is an `Object` or `Geon`. It supplies the observations from which the Perceiver constructs those representations.

### 7.2 Spatial State

The adapter should expose a common world coordinate frame whenever possible:

```
World
  ├── Avatar
  ├── Vehicles
  ├── Pedestrians
  ├── Objects
  ├── Terrain
  └── Navigation
```

Spatial relationships such as `Above`, `Behind`, `Beside`, `Contact`, `Container`, `Path`, and `Together` belong to the perceptual/modeling layers rather than to the Psyche itself.

### 7.3 Canvas Compatibility

The Gotham world is a natural source for the GIL imagination system.

A captured or reconstructed game `Scene` may become the perceptual side of a corresponding imaginative `Taxis`. Where a `Scene` and `Taxis` represent two sides of the same monad, the same `:M` may identify their correspondence.

The Psyche itself does not render imaginative geometry. It supplies the percepts from which the mind may construct or reconstruct a `Canvas`.

---

## 8 Architecture

```
                     ┌──────────────────────────────────┐
                     │             THE MIND             │
                     │                                  │
                     │  Perceiver ← PERCEPT / RESULT    │
                     │  Detectors → Matcher → Storer    │
                     │  Coordinator → Planner           │
                     │  Executor ─────────→ ATTEMPT     │
                     └────────────────┬─────────────────┘
                                      │
                              ┌───────┴───────┐
                              │ GOTHAM PSYCHE │
                              │               │
                              │ :Modality     │ Gotham
                              │ :Channel      │ Visual
                              │               │ World
                              │               │ Avatar
                              │               │ Entity
                              │               │ Vehicle
                              │               │ Mission
                              │               │ Navigation
                              │               │ Audio
                              │               │ Environment
                              │               │ System
                              └───────┬───────┘
                                      │
                              ┌───────┴───────┐
                              │ GAME ADAPTER   │
                              │               │
                              │ capture       │
                              │ state         │
                              │ input         │
                              │ events        │
                              └───────┬───────┘
                                      │
                              ┌───────┴───────┐
                              │ GTA VI WORLD   │
                              │               │
                              │ Leonida       │
                              │ Vice City     │
                              │ roads         │
                              │ buildings     │
                              │ people        │
                              │ vehicles      │
                              │ missions      │
                              │ environment   │
                              └───────────────┘
```

### 8.1 Adapter Boundary

The adapter is deliberately outside the Psyche's conceptual model.

The adapter is responsible for whatever mechanism is available to observe and control the game, such as:

- screen/frame capture;
- game telemetry when exposed;
- controller or keyboard/mouse input;
- accessibility interfaces;
- local instrumentation;
- supported modding or scripting interfaces;
- external computer-vision processing.

The Gotham Psyche normalizes those mechanisms into the canonical GIL `PERCEPT`, `ATTEMPT`, `RESULT`, and `URGE` structures.

The Psyche therefore remains independent of any one implementation mechanism.

---

## 9 Deployment

A Gotham deployment may consist of:

| File | Category | Description |
|---|---|---|
| `gotham.adapter` | Adapter | Game capture, state extraction, and input/control interface |
| `gotham.package` | Unified Layer | Percept normalization, action dispatch, routing, and utilities |
| `gotham-psyche.theory` | Theory | Sensor, Actuator, and Monitor agents |
| `Gotham.daicho` | Configuration | Adapter, coordinate system, capture, polling, and urge configuration |
| `gotham.suites` | Tests | Connection, sensing, acting, percept formation, spatial state, and lifecycle tests |

### 9.1 Package Layering

```
gotham-psyche.theory
        |
        v
gotham.package
        |
        v
gotham.adapter
        |
        v
Grand Theft Auto VI
```

The theory calls only `gotham.package`.

The package handles normalization and routing. The adapter handles the actual game interface.

---

## 10 Configuration

`Gotham.daicho` should contain at least:

| Section | Contents |
|---|---|
| `[Psyche]` | Agent URL, port, delay, modality |
| `[Game]` | Game/session identifier and execution mode |
| `[Adapter]` | Adapter URL, protocol, connection, timeout |
| `[Capture]` | Frame rate, resolution, visual/depth capture |
| `[World]` | Coordinate system, units, spatial origin |
| `[Polling]` | State polling intervals |
| `[Routing]` | Adapter capabilities and action routing |
| `[Urges]` | Homeostatic thresholds |
| `[Security]` | Local credentials/tokens where required |

Example conceptual configuration:

```
[Psyche]
:Modality Gotham
:Delay 50

[Game]
:Title "Grand Theft Auto VI"

[Capture]
:Width 1920
:Height 1080
:FrameRate 30

[World]
:CoordinateSystem game-native
:Units meter

[Polling]
:Visual 33
:Avatar 50
:Vehicle 50
:Mission 250
:World 500
```

Actual values are deployment-specific.

---

## 11 Adapter

`gotham.adapter` translates between the game interface and the Psyche.

### 11.1 Adapter Responsibilities

1. Connect to the game session.
2. Capture visual observations.
3. Acquire structured game state when available.
4. Normalize timestamps.
5. Normalize spatial coordinates.
6. Translate game events into percepts.
7. Translate `ATTEMPT` actions into game controls.
8. Return action outcomes as `RESULT` structures.
9. Detect connection failure.
10. Report adapter capabilities.

### 11.2 Capability Registry

An adapter should advertise capabilities such as:

```
{:VisualCapture yes
 :DepthCapture maybe
 :WorldState maybe
 :EntityState maybe
 :VehicleState maybe
 :MissionState maybe
 :NavigationState maybe
 :AudioCapture maybe
 :InputControl yes}
```

`maybe` indicates that a capability depends on the actual integration mechanism.

---

## 12 Theory

`gotham-psyche.theory` defines three principal agents.

### Sensor

The Sensor receives observations from the adapter and wraps them as:

```
[PERCEPT :Modality Gotham
         :Channel C
         :Address A
         :Data D
         :Content {...}
         :Moment M
         :Token T]
```

It tells the mind's Perceiver.

### Actuator

The Actuator listens for:

```
[ATTEMPT :Act A
         :Args {...}
         :Token T
         :By M]
```

It dispatches the action through `gotham.package`, then reports:

```
[RESULT :Action A
        :Status S
        :Reason R
        :Moment M
        :Token T]
```

### Monitor

The Monitor watches the health of the embodiment.

Possible homeostatic conditions include:

| Need | Source | Trigger |
|---|---|---|
| `connectivity` | Adapter | Game connection lost |
| `capture` | Visual | Frame stream unavailable |
| `control` | Input | Command channel unavailable |
| `latency` | Adapter | Round-trip latency exceeds threshold |
| `synchronization` | World | Game state and capture timestamps diverge |

When a threshold is crossed, the Monitor emits an `URGE`.

---

## 13 Test Suites

`gotham.suites` should contain at least:

| Suite | Tests |
|---|---|
| `Gotham.lifecycle` | Connect, initialize, disconnect |
| `Gotham.adapter` | Adapter capability discovery and protocol |
| `Gotham.capture` | Frame acquisition and timestamps |
| `Gotham.sensing` | Visual and structured percept acquisition |
| `Gotham.spatial` | Coordinates, position, orientation, velocity |
| `Gotham.entities` | Entity identification and state |
| `Gotham.vehicles` | Vehicle state and control |
| `Gotham.navigation` | Waypoints and routes |
| `Gotham.missions` | Mission/objective percepts |
| `Gotham.acting` | Action dispatch |
| `Gotham.percept-formation` | PERCEPT/RESULT/URGE structure |
| `Gotham.urge-generation` | Adapter and stream failure detection |
| `Gotham.full-lifecycle` | End-to-end sensing and acting |

---

## 14 Framework Asymmetries

The exact asymmetries depend on the selected GTA VI integration.

### Native / Structured Integration

If the game or an approved interface exposes structured world state, Gotham can receive:

- exact coordinates;
- entity identifiers;
- vehicle state;
- mission state;
- world time;
- environmental state;
- navigation information.

This is preferable for symbolic world modeling because the Psyche does not have to infer every fact from pixels.

### Visual Integration

If only visual capture and input are available, Gotham can still provide a complete embodied interface, but much of the world model must be constructed by the Perceiver:

```
pixels
  → detection
  → matching
  → spatial estimation
  → Scene
  → Totality
```

The distinction is important:

**Gotham senses the game. GIL understands the game.**

---

## 15 Domain Coverage

| Domain | `:Channel` | Percept `:Data` | Actuations |
|---|---|---|---|
| Visual perception | `Visual` | `frame`, `depth`, `segmentation` | `look`, `camera` |
| Avatar | `Avatar` | `avatar-state`, `position`, `movement-state` | `move`, `jump`, `crouch`, `sprint` |
| Entities | `Entity` | `entity-state`, `world-event` | `interact`, `use` |
| Vehicles | `Vehicle` | `vehicle-state` | `enter-vehicle`, `exit-vehicle`, `drive` |
| Missions | `Mission` | `mission-state`, `objective-state`, `mission-event` | `mission-action` |
| Navigation | `Navigation` | `map-state`, `route-state`, `waypoint-state` | `navigate`, `cancel-route` |
| Environment | `Environment` | `weather-state`, `world-event` | — |
| Audio | `Audio` | `audio-frame`, `sound-event`, `transcription` | — |
| System | `System` | `connection-status`, `capture-status`, `command-status` | `pause`, `resume` |

---

## 16 Etymology

**Gotham** is an old name associated with New York City and, in modern popular culture, with the fictional city of Batman.

The name is appropriate here not because GTA VI is Gotham, but because **Gotham evokes a large, dense, urban world in which a character moves among people, vehicles, institutions, streets, buildings, danger, commerce, and events.**

As a Psyche name, Gotham names the **urban-world interface** rather than the game itself.

The correspondence among the current Psyche names is:

| Psyche | World | Naming principle |
|---|---|---|
| **Aether** | Internet | Medium / propagation |
| **Expanse** | Physical reality | Territory / spatial extent |
| **Eidos** | ARC-AGI-3 | Perception becoming understanding |
| **Ìtẹ́** | Chess world | What is at stake — the throne |
| **Gotham** | GTA VI | Urban world / city-as-world |

Gotham therefore functions as the body through which GIL can **inhabit, perceive, navigate, and act within a simulated urban world**.

---

## Summary

The Gotham Psyche provides a canonical GIL boundary around *Grand Theft Auto VI*:

```
                    GIL MIND
                       │
             PERCEPT ← │ → ATTEMPT
                       │
                GOTHAM PSYCHE
                       │
              ┌────────┴────────┐
              │                 │
          PERCEPTION         ACTION
              │                 │
       ┌──────┴──────┐    ┌─────┴─────┐
       │ frame       │    │ movement  │
       │ world       │    │ vehicle   │
       │ entities    │    │ interact  │
       │ vehicles    │    │ navigate  │
       │ missions    │    │ mission   │
       │ navigation  │    │ camera    │
       │ environment │    └───────────┘
       └─────────────┘
              │
              ▼
        GTA VI WORLD
```

**Gotham is the Psyche. Grand Theft Auto VI is the world. GIL is the mind.**
