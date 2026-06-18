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

# The Aether Psyche

## Specification and Deployment Guide

**SubThought Corporation**

Version 1.0 — June 2026

---

## Table of Contents

1. Overview
2. The Percept Relation
3. Percepts by Channel
4. Common Content Idiom Slots
5. Actuations
6. Cross-Cutting Structure
7. Architecture
8. Deployment File Set
9. Package Layering
10. Adapter Packages
11. Aether Package
12. The Theory
13. Configuration
14. Test Suites
15. Framework Asymmetries
16. Domain Coverage
17. Etymology

---

## 1  Overview

The Aether is a Psyche: the interface between a GIL mind and the internet-as-world. It mediates two flows — percepts traveling inward from the world to the mind's mechanisms, and actuations traveling outward from the mind's mechanisms to the world. The Aether unifies the action surfaces of OmegaClaw (ASI Alliance / OpenCog Hyperon) and OpenClaw (the open-source agent framework) into a single coherent interface.

Where the NAO Psyche interfaces with a humanoid robot and the Ìtẹ́ Psyche interfaces with a game world, the Aether interfaces with the internet itself — web services, file systems, shell processes, messaging channels, browser automation, knowledge stores, external APIs, and (optionally) physical devices via ROS 2.

The internet-world that the Aether interfaces with is not a bounded simulation. It is an open-ended, layered environment spanning local compute, networked services, communication channels, knowledge stores, and physical devices. The set of percepts and actuations is extensible — both source frameworks support plugin/skill architectures that grow the environment over time.

All percepts from the Aether carry `:Modality Aether`. The `:Channel` slot selects the domain. The `:Data` slot names the parsable structure type. The `:Content` slot carries the idiom.

---

## 2  The Percept Relation

The Percept relation from `totality.theory` is canonical. It does not change for the Aether.

```
(relation Percept
  :M                 ; lexified monad
  :Modality          ; which Psyche — Aether for this device
  :Channel           ; sub-channel within the Psyche
  :Address  nil      ; URL or source of the data
  :Data     nil      ; data format or parsable structure type
  :Content  nil      ; idiom {:Slot value ...} — the actual structured data
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

### 2.1  Slot Semantics

| Slot | Meaning | Aether Examples |
|---|---|---|
| `:Modality` | Which Psyche this came through. The Psyche is the sense organ. | `Aether` (always) |
| `:Channel` | Sub-channel within the Psyche. | `Web`, `FileSystem`, `Process`, `Message`, `Browser`, `Memory`, `Service`, `System`, `Embodiment` |
| `:Address` | URL or source of the data. Where it came from. | `"https://google.com/..."`, `"file:///frame_01.png"`, `"ros2://rover-1/imu"`, `nil` |
| `:Data` | Data format or parsable structure type. What the `:Content` contains. | `search-result`, `page-content`, `api-response`, `process-output`, `message-received`, `media`, `knowledge-result`, `sensor-reading` |
| `:Content` | Idiom `{:Slot value ...}` with the actual structured data. | `{:Query "..." :Rank 1 :Url "..." :Title "..." :Snippet "..."}` |
| `:Moment` | When the percept was received. (Template slot.) | `\@m{2026060100000000}` |

For comparison, the NAO Psyche sets `:Modality` to `Visual`, `Auditory`, `Haptic`, `Odometric`, `Sonar`, etc. — the biological sense. The Aether sets `:Modality` to `Aether` — the internet sense. The `:Channel` and `:Data` slots differentiate what kind of internet percept it is.

---

## 3  Percepts by `:Channel`

All percepts from the Aether have `:Modality Aether`. The `:Channel` identifies the domain, `:Data` identifies the format, `:Address` identifies the source, and `:Content` carries the structured data as an idiom.

### 3.1  Channel `Web`

| `:Data` | `:Address` | `:Content` (idiom) | OmegaClaw | OpenClaw |
|---|---|---|---|---|
| `search-result` | `"https://..."` | `{:Query "..." :Rank 1 :Title "..." :Snippet "..." :Source google}` | Tavily agent | `web_search` |
| `page-content` | `"https://..."` | `{:Title "..." :ContentType "text/html" :Encoding "utf-8" :Size 48000}` | Python bridge | `web_fetch` |
| `api-response` | `"https://api..."` | `{:Method GET :Status 200 :Headers {...} :Body {...} :ContentType "application/json" :Latency 120}` | Custom skill | `web_fetch` / MCP |

### 3.2  Channel `FileSystem`

| `:Data` | `:Address` | `:Content` (idiom) | OmegaClaw | OpenClaw |
|---|---|---|---|---|
| `file-content` | `"file:///home/x"` | `{:ContentType "text/plain" :Size 2048 :Modified \@m{...} :Permissions "rwxr-xr-x"}` | Shell I/O | `read` |
| `directory-listing` | `"file:///home"` | `{:Entries {{:Name "src" :Type directory :Modified \@m{...}} {:Name "readme.md" :Type file :Size 2048} ...}}` | Shell | `read` / `exec` |
| `file-event` | `"file:///home/x"` | `{:EventType modified :Size 2100 :OldPath nil}` | Not built-in | `exec` + inotify |

### 3.3  Channel `Process`

| `:Data` | `:Address` | `:Content` (idiom) | OmegaClaw | OpenClaw |
|---|---|---|---|---|
| `process-output` | nil | `{:Command "ls -la" :WorkingDirectory "/home" :Stdout "..." :Stderr "" :ExitCode 0 :Duration 45 :Pid 8821}` | Shell I/O | `exec` |
| `background-output` | nil | `{:SessionId "bg-42" :Stdout "..." :Stderr ""}` | Not built-in | `process` |

### 3.4  Channel `Message`

| `:Data` | `:Address` | `:Content` (idiom) | OmegaClaw | OpenClaw |
|---|---|---|---|---|
| `message-received` | `"telegram://room-7"` | `{:Sender "alice" :SenderId "u123" :SenderType human :Text "hello" :ThreadId "t5" :Attachments {}}` | IRC / Telegram | 25+ channels |
| `agent-reply` | `"agentverse://agent-3"` | `{:Protocol agentverse :SessionId "s9" :InReplyTo "req-7" :Body {...} :Latency 340}` | Agentverse | `sessions` |
| `webhook-event` | `"https://hooks..."` | `{:Source github :EventType push :Body {...}}` | Custom skill | MCP |

### 3.5  Channel `Browser`

| `:Data` | `:Address` | `:Content` (idiom) | OmegaClaw | OpenClaw |
|---|---|---|---|---|
| `browser-content` | `"https://..."` | `{:Title "..." :Selector "article" :ContentType text}` | Not built-in | `browser` |
| `media` | `"file:///frame.png"` | `{:Url "https://..." :Width 1280 :Height 800 :Format png}` | Not built-in | `browser` |
| `accessibility-tree` | `"https://..."` | `{:NodeCount 340 :Tree {...}}` | Not built-in | `browser` |

### 3.6  Channel `Memory`

| `:Data` | `:Address` | `:Content` (idiom) | OmegaClaw | OpenClaw |
|---|---|---|---|---|
| `memory-result` | nil | `{:Query "..." :StoreType embedding :Results {{:Content ... :Score 0.87 :StoredAt \@m{...}} ...}}` | Long-term memory | `memory_search` |
| `knowledge-result` | nil | `{:Query "..." :Results {{:Reifier 12345 :Relation Causes :Arguments {...} :Referents {...} :Confidence 0.9} ...} :InferenceTrail {...}}` | AtomSpace / NAL / PLN | Not built-in |

### 3.7  Channel `Service`

| `:Data` | `:Address` | `:Content` (idiom) | OmegaClaw | OpenClaw |
|---|---|---|---|---|
| `service-resource` | `"https://github.com/..."` | `{:Service github :ResourceType issue :ResourceId "42" :Payload {...}}` | Agentverse agent | MCP / skills |
| `financial-data` | `"https://api.finance..."` | `{:Ticker "TSLA" :Indicators {:SMA 245.3 :RSI 62.1 :MACD {:Signal buy}}}` | Tech Analysis agent | MCP |

### 3.8  Channel `System`

| `:Data` | `:Address` | `:Content` (idiom) | OmegaClaw | OpenClaw |
|---|---|---|---|---|
| `schedule-event` | nil | `{:EventType cron :Schedule "*/5 * * * *" :JobId "poll-github"}` | Continuous loop | `cron` |
| `connection-status` | `"wss://localhost:4242"` | `{:Service openclaw :Status connected :Error nil}` | IRC/Telegram state | `gateway` |

### 3.9  Channel `Embodiment`

| `:Data` | `:Address` | `:Content` (idiom) | OmegaClaw | OpenClaw |
|---|---|---|---|---|
| `media` | `"file:///cam.png"` | `{:Width 640 :Height 480 :Format png :DeviceId Camera1}` | Not built-in | ROS 2 / ClawBody |
| `sensor-reading` | `"ros2://rover-1/imu"` | `{:SensorType imu :Readings {:Pitch 0.02 :Roll -0.01 :Yaw 1.57} :Unit "rad" :CoordinateFrame world}` | Not built-in | ROS 2 / ClawBody |
| `motor-state` | `"ros2://rover-1/elbow-L"` | `{:Position 1.57 :Velocity 0.0 :Torque 2.1 :Unit "Nm"}` | Not built-in | ROS 2 / ClawBody |
| `action-result` | nil | `{:ActionId "move-42" :Status succeeded :Duration 3200}` | Not built-in | ROS 2 / ClawBody |
| `media` | `"file:///shot.png"` | `{:Width 2560 :Height 1440 :Format png :DisplayId main}` | Not built-in | Port42 |
| `transcription` | nil | `{:Text "hello world" :IsFinal yes :Language "en-US" :Confidence 0.95 :Duration 1200}` | Not built-in | Port42 |
| `clipboard` | nil | `{:Text "copied text" :ContentType text :Format plain}` | Not built-in | Port42 |

### 3.10  Percept Examples

```
[Percept :Modality Aether  :Channel Web  :Address "https://google.com/search?q=..."
         :Data search-result
         :Content {:Query "quantum computing" :Rank 1 :Title "..." :Snippet "..."}
         :Moment \@m{...}]

[Percept :Modality Aether  :Channel Browser  :Address "file:///frame_01.png"
         :Data media
         :Content {:Url "https://example.com" :Width 1280 :Height 800 :Format png}
         :Moment \@m{...}]

[Percept :Modality Aether  :Channel Process  :Address nil
         :Data process-output
         :Content {:Command "npm test" :Stdout "..." :ExitCode 0 :Duration 4500}
         :Moment \@m{...}]

[Percept :Modality Aether  :Channel Message  :Address "telegram://room-7"
         :Data message-received
         :Content {:Sender "alice" :SenderType human :Text "hello" :ThreadId "t5"}
         :Moment \@m{...}]

[Percept :Modality Aether  :Channel Memory  :Address nil
         :Data knowledge-result
         :Content {:Query "..." :Results {{:Reifier 12345 :Relation Causes :Confidence 0.9}}
                   :InferenceTrail {...}}
         :Moment \@m{...}]

[Percept :Modality Aether  :Channel Embodiment  :Address "ros2://rover-1/imu"
         :Data sensor-reading
         :Content {:SensorType imu :Readings {:Pitch 0.02 :Roll -0.01 :Yaw 1.57}
                   :Unit "rad" :CoordinateFrame world}
         :Moment \@m{...}]
```

---

## 4  Common `:Content` Idiom Slots

Examining percepts across all `:Channel` × `:Data` combinations reveals shared structure.

### Slots Appearing in 10+ `:Content` Idioms

| Slot | Type | Count | Notes |
|---|---|---|---|
| `:ContentType` / `:Format` / `:SensorType` / `:StoreType` | literal | 22 | Format — describes the kind of data. |
| `:Size` / `:Width` / `:Height` / `:Duration` / `:NodeCount` | integer | 15 | Extent — dimensional measure. |
| `:Source` / `:Service` / `:Sender` / `:DeviceId` | literal | 16 | Origin — who or what produced this data. |

### Slots Appearing in 3–9 `:Content` Idioms

| Slot | Type | Count | Notes |
|---|---|---|---|
| `:Title` | string | 5 | Human-readable label. |
| `:SessionId` / `:ThreadId` / `:JobId` | string | 5 | Conversation or session context. |
| `:Status` / `:ExitCode` | literal/integer | 4 | Outcome indicator. |
| `:Error` | string | 3 | Error details on failure. |
| `:Selector` / `:Query` | string | 4 | What was asked for. |
| `:Confidence` / `:Score` | real | 3 | Certainty of the percept. |
| `:Unit` | literal | 2 | Measurement unit. |
| `:InferenceTrail` | list | 1 | Reasoning provenance (OmegaClaw only). |

---

## 5  Actuations

Actuations are commands the mind's mechanisms dispatch through the Aether to change the state of the world. Following GIL convention, actuations are sent as ATTEMPT structures and results return as RESULT percepts — the mind learns what its body did through perception.

### 5.1  Attempt Structure

```
(structure Attempt
  :Action      nil    ; actuation literal
  :Parameters  nil    ; idiom {:Slot value ...}
  :Token       nil    ; auth token from device registration
  :By          nil    ; moment deadline (nil if no deadline)
)
```

### 5.2  Actuation Table

| # | `:Action` | Description | `:Parameters` idiom |
|---|---|---|---|
| A01 | `exec` | Run a shell command | `{:Command "ls -la" :WorkingDirectory "/home" :Timeout 30000}` |
| A02 | `exec-background` | Start long-running command | `{:Command "npm start" :WorkingDirectory "/app" :YieldMs 5000}` |
| A03 | `process-kill` | Terminate process | `{:SessionId "bg-42"}` |
| A04 | `process-write` | Send input to process | `{:SessionId "bg-42" :Input "yes\r"}` |
| A05 | `file-create` | Create a new file | `{:Path "/home/x.txt" :Content "hello" :Encoding "utf-8"}` |
| A06 | `file-overwrite` | Replace file contents | `{:Path "/home/x.txt" :Content "world" :Encoding "utf-8"}` |
| A07 | `file-edit` | Targeted text replacement | `{:Path "/home/x.txt" :OldText "hello" :NewText "world"}` |
| A08 | `file-patch` | Multi-hunk diff | `{:Path "/home/x.txt" :Patch "...unified diff..."}` |
| A09 | `file-delete` | Remove a file | `{:Path "/home/x.txt"}` |
| A10 | `file-move` | Rename or relocate | `{:SourcePath "/home/x.txt" :DestinationPath "/home/y.txt"}` |
| A11 | `web-search` | Submit a search query | `{:Query "quantum computing" :MaxResults 5}` |
| A12 | `web-fetch` | Retrieve URL content | `{:Url "https://..." :MaxChars 50000 :Format markdown}` |
| A13 | `browser-navigate` | Open a URL | `{:Url "https://..." :Profile chrome}` |
| A14 | `browser-click` | Click an element | `{:Selector "button.submit" :Profile chrome}` |
| A15 | `browser-type` | Enter text in a field | `{:Selector "input.search" :Text "hello" :Profile chrome}` |
| A16 | `browser-execute` | Run JavaScript | `{:Script "document.title" :Profile chrome}` |
| A17 | `browser-capture` | Screenshot | `{:Profile chrome :Scale 1.0}` |
| A18 | `message-send` | Send a chat message | `{:Channel slack :ChannelId "eng" :Text "done" :ThreadId nil}` |
| A19 | `agent-send` | Send to a peer agent | `{:AgentId "agent-3" :SessionId "s9" :Content {...} :Protocol agentverse}` |
| A20 | `agent-spawn` | Create agent session | `{:AgentConfig {...} :Task "summarize" :Context {...}}` |
| A21 | `notify` | System notification | `{:Title "Build Done" :Body "All tests pass" :Sound yes}` |
| A22 | `email-send` | Send email | `{:To {"alice@x.com"} :Subject "Report" :Body "..." :Cc {} :Attachments {}}` |
| A23 | `memory-store` | Write to memory | `{:Key "proj-context" :Content {...} :Scope channel}` |
| A24 | `memory-delete` | Remove from memory | `{:Key "proj-context" :Scope channel}` |
| A25 | `knowledge-assert` | Create a scheme | `{:Reifier 500001 :Relation Causes :Arguments {500002} :Referents {500003} :Confidence 0.85}` |
| A26 | `knowledge-retract` | Remove a scheme | `{:Reifier 500001}` |
| A27 | `nal-infer` | NAL reasoning | `{:Premises {500010 500011} :Query "..." :Depth 3}` |
| A28 | `pln-infer` | PLN reasoning | `{:Premises {500010 500011} :Query "..." :ConfidenceThreshold 0.7}` |
| A29 | `generate-text` | LLM call | `{:Prompt "..." :SystemPrompt "..." :Model "claude-sonnet" :MaxTokens 1000 :Temperature 0.7}` |
| A30 | `generate-image` | Image generation | `{:Prompt "..." :Model "dall-e" :Width 1024 :Height 1024 :Format png}` |
| A31 | `render-ui` | Create visual surface | `{:Html "..." :Title "dashboard" :Target canvas}` |
| A32 | `cron-schedule` | Set scheduled task | `{:Expression "*/5 * * * *" :Job "poll-github" :Description "..."}` |
| A33 | `cron-cancel` | Remove scheduled task | `{:JobId "poll-github"}` |
| A34 | `rest-call` | HTTP request | `{:Url "https://api.stripe.com/..." :Method GET :Headers {} :Secret "stripe" :Timeout 15000}` |
| A35 | `drive-command` | Robot movement | `{:LinearVelocity 0.5 :AngularVelocity 0.0 :Duration 2000}` |
| A36 | `gimbal-command` | Point sensor | `{:Pan 45.0 :Tilt -10.0}` |
| A37 | `joint-command` | Move robotic joint | `{:JointId "elbow-L" :Position 1.57 :Velocity 0.3}` |
| A38 | `light-command` | Control lights | `{:State on :Brightness 0.8 :Color white}` |
| A39 | `speak` | Text-to-speech | `{:Text "hello world" :Voice "Samantha" :Rate 0.5 :Volume 0.8}` |
| A40 | `clipboard-write` | Set clipboard | `{:Content "copied text" :ContentType text}` |
| A41 | `automation-run` | AppleScript/JXA | `{:Script "tell app Finder to ..." :Language applescript :Timeout 30}` |
| A42 | `capture-screen` | Screenshot | `{:DisplayId main :Scale 1.0}` |
| A43 | `capture-camera` | Camera photo | `{:DeviceId Camera1 :Scale 0.5}` |

### 5.3  Actuation Domains

**Shell Execution:** `exec`, `exec-background`, `process-kill`, `process-write`

**File System:** `file-create`, `file-overwrite`, `file-edit`, `file-patch`, `file-delete`, `file-move`

**Web Interaction:** `web-search`, `web-fetch`, `browser-navigate`, `browser-click`, `browser-type`, `browser-execute`, `browser-capture`

**Communication:** `message-send`, `agent-send`, `agent-spawn`, `notify`, `email-send`

**Memory:** `memory-store`, `memory-delete`, `knowledge-assert`, `knowledge-retract`

**Reasoning (OmegaClaw):** `nal-infer`, `pln-infer`

**Content Generation:** `generate-text`, `generate-image`, `render-ui`

**Automation:** `cron-schedule`, `cron-cancel`

**External Services:** `rest-call`

**Device / Embodiment:** `drive-command`, `gimbal-command`, `joint-command`, `light-command`, `speak`, `clipboard-write`, `automation-run`, `capture-screen`, `capture-camera`

### 5.4  ATTEMPT Examples

```
[ATTEMPT :Action exec
         :Parameters {:Command "ls -la" :WorkingDirectory "/home/project" :Timeout 30000}
         :Token ?tok  :By \@m{...}]

[ATTEMPT :Action web-search
         :Parameters {:Query "quantum computing 2026" :MaxResults 5}
         :Token ?tok]

[ATTEMPT :Action message-send
         :Parameters {:Channel slack :ChannelId "eng-team" :Text "build passed"}
         :Token ?tok]

[ATTEMPT :Action knowledge-assert
         :Parameters {:Reifier 500001 :Relation Causes :Arguments {500002}
                      :Referents {500003} :Confidence 0.85}
         :Token ?tok]

[ATTEMPT :Action drive-command
         :Parameters {:LinearVelocity 0.5 :AngularVelocity 0.0 :Duration 2000}
         :Token ?tok]
```

---

## 6  Cross-Cutting Structure

### 6.1  The Four Tuple Types

```
PERCEPT:  [PERCEPT :Modality Aether :Channel C :Address A :Data D :Content {:...} :Moment M :Token T]
ATTEMPT:  [ATTEMPT :Action A :Parameters {:...} :Token T :By M]
RESULT:   [RESULT  :Action A :Status S :Reason R :Moment M :Token T]
URGE:     [URGE    :Need N :Source S :Delta D :Moment M :Token T]
```

The Aether is a bidirectional envelope router: it wraps outbound commands in Attempt structures, dispatches them to the right framework, receives the world's response, and sends it back as a Percept, a Result, or an Urge to the Perceiver. The mind learns what its body did through perception — exactly as implemented in the Ìtẹ́ and NAO Psyches.

### 6.2  Actuation → Percept Pairings

| `:Action` | Produces `:Data` |
|---|---|
| `exec`, `exec-background` | `process-output` |
| `file-create` through `file-patch` | `file-event` |
| `web-search` | `search-result` |
| `web-fetch` | `page-content` |
| `browser-navigate`, `browser-click`, `browser-type` | `browser-content` or `media` |
| `browser-capture` | `media` |
| `message-send`, `agent-send` | `message-received` or `agent-reply` (eventual) |
| `memory-store` | `memory-result` |
| `knowledge-assert` | `knowledge-result` |
| `nal-infer`, `pln-infer` | `knowledge-result` (with `:InferenceTrail`) |
| `rest-call` | `api-response` |
| `capture-screen`, `capture-camera` | `media` |
| `cron-schedule` | `schedule-event` (future) |
| `drive-command`, `joint-command` | `action-result`, then `motor-state` |

Some percepts are also spontaneous — unsolicited messages, webhook events, cron triggers, sensor readings — arriving without a prior Attempt. Homeostatic signals arrive as `[URGE :Need N :Source S :Delta D :Moment \@m{...} :Token T]` following the Ìtẹ́ convention.

---

## 7  Architecture

### 7.1  The Aether as Mediator

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
                          │  AETHER PSYCHE  │
                          │                 │
                          │  :Modality      │  Aether (always)
                          │  :Channel       │  Web | FileSystem |
                          │                 │  Process | Message |
                          │                 │  Browser | Memory |
                          │                 │  Service | System |
                          │                 │  Embodiment
                          │  :Address       │  source URL
                          │  :Data          │  format literal
                          │  :Content       │  idiom {:...}
                          │                 │
                          └────────┬────────┘
                                   │
          ┌────────────────────────┼───────────────────────────┐
          │                        │                           │
    ┌─────┴──────┐          ┌──────┴──────┐           ┌───────┴───────┐
    │  OpenClaw  │          │ OmegaClaw   │           │  Port42 /     │
    │  Gateway   │          │ MeTTa Loop  │           │  Other Hosts  │
    └────────────┘          └─────────────┘           └───────────────┘
```

### 7.2  Perceptual Pipeline

Once a percept arrives at the Perceiver, the Observation component takes over:

**Perception (17 steps):** Perceiver → Detectors (concurrent) → DETECTED traits → Matcher. On miss: Perceiver → Storer → Activator. The experience is now live in the Totality.

**Recollection (13 steps):** Same detection flow, but the Matcher hits. Skip storage, activate the existing content reifier alongside current traits.

**Proprioception (6 steps):** A RESULT arrives. Lexify the action and status, activate the result reifier. The Estimator updates Utility records.

**Interoception (6 steps):** An URGE arrives. Lexify the need, update the Need relation's `:Delta`, activate the urge reifier.

The Aether does not need to know any of this. It delivers percepts and receives attempts. The Psyche is the boundary.

---

## 8  Deployment File Set

The Aether Psyche is deployed as a set of eight files across four categories.

| File | Category | Description |
|---|---|---|
| `openclaw.package` | Adapter | OpenClaw protocol: wraps 26 tools and MCP skill calls over WebSocket/JSON |
| `omegaclaw.package` | Adapter | OmegaClaw protocol: wraps MeTTa bridge, Agentverse, AtomSpace, NAL/PLN |
| `port42.package` | Adapter | Port42 protocol: wraps port42.* bridge API over encrypted WebSocket |
| `aether.package` | Unified Layer | Requires the three adapters. Routing, normalization, convenience functions |
| `aether-psyche.theory` | Theory | Three agents (Sensor, Actuator, Monitor) that connect the adapters to the mind |
| `Aether.daicho` | Configuration | Adapter URLs, channels, routing, polling intervals, urge thresholds |
| `aether.suites` | Tests | 9 test suites: lifecycle, adapter protocols, routing, sensing, acting, percept formation, urge generation, convenience, full lifecycle |

---

## 9  Package Layering

```
aether-psyche.theory           agents: Sensor, Actuator, Monitor
    │
    ▼
aether.package                 routing, normalization, uniform interface
    │
    ├──▶ openclaw.package      OpenClaw protocol (JSON/WebSocket, 26 tools)
    ├──▶ omegaclaw.package     OmegaClaw protocol (MeTTa/Agentverse/AtomSpace)
    └──▶ port42.package        Port42 protocol (WebSocket/E2E encrypted)
```

The theory calls only `aether.package`. The `aether.package` routes to the correct adapter based on `Aether.daicho` configuration. Each adapter package handles the wire protocol for its framework using Premise `tell` and `ask` over TCP, HTTPS, or pipes.

---

## 10  Adapter Packages

### 10.1  `openclaw.package`

Wraps OpenClaw's tool groups: `runtime` (exec, bash, process), `fs` (read, write, edit, apply_patch), `web` (web_search, web_fetch), `ui` (browser, canvas), `sessions` (spawn, send, history), `memory` (search, get), `messaging` (message), `automation` (cron, gateway), and `clawbody` (ROS 2 embodiment). Exposes 30+ functions prefixed `oc-`, plus a generic `oc-dispatch` that translates any Aether `:Action` literal into the corresponding OpenClaw tool call.

### 10.2  `omegaclaw.package`

Wraps OmegaClaw's skill categories: `shell` (OS commands, file I/O), `agentverse` (Tavily Search, Technical Analysis, generic remote agents), `atomspace` (knowledge graph assert/query/retract), `reasoning` (NAL inference, PLN inference), `channels` (IRC, Telegram), and `memory` (embedding search, store, delete). Exposes 20+ functions prefixed `omega-`, plus a generic `omega-dispatch`.

### 10.3  `port42.package`

Wraps Port42's bridge API domains: `browser` (navigate, click, type, execute, screenshot, text, html), `screen` (capture, info, windows), `camera` (capture, stream start/stop), `audio` (speak, capture with transcription), `clipboard` (read, write), `automation` (AppleScript/JXA), `notify` (macOS notifications), and `rest` (HTTP with Keychain credentials). Exposes 20+ functions prefixed `p42-`, plus a generic `p42-dispatch`.

---

## 11  Aether Package

`aether.package` requires all three adapter packages and provides:

**Adapter Registry:** An `Adapter` relation tracking `:Name`, `:Handle`, `:Status`, `:Channels`, and `:Capabilities` for each connected framework.

**Lifecycle:** `aether-connect-adapter`, `aether-disconnect-adapter`, `aether-connect-all`, `aether-disconnect-all`.

**Routing:** `aether-route` (find adapter by channel), `aether-route-action` (find adapter by action capability). Uses the priority lists in `[Routing]` from the daicho.

**Sensing:** `aether-sense` polls an adapter for unsolicited percepts on a channel.

**Acting:** `aether-act` dispatches an `:Action` + `:Parameters` idiom to the correct adapter via its `*-dispatch` function.

**Convenience:** `aether-search`, `aether-fetch`, `aether-exec`, `aether-read`, `aether-write`, `aether-send`, `aether-remember`, `aether-recall`, `aether-assert`, `aether-infer-nal`, `aether-infer-pln`.

---

## 12  The Theory

`aether-psyche.theory` defines three agents:

**Sensor** — polls each channel at the interval configured in `[Polling]`. Wraps each raw percept as a `[PERCEPT :Modality Aether :Channel C :Address A :Data D :Content {...} :Moment M :Token T]` tuple and tells the Perceiver.

**Actuator** — runs as a service listening for ATTEMPT tuples from the Executor. Dispatches each through `aether-act`, wraps the outcome as a `[RESULT :Action A :Status S :Reason R :Moment M :Token T]` tuple, and tells the Perceiver.

**Monitor** — runs on a slow interval (default 30 Moments). Checks homeostatic conditions: adapter connectivity, message backlog, disk usage. When a threshold from `[Urges]` is crossed, emits a `[URGE :Need N :Source S :Delta D :Moment M :Token T]` tuple.

On startup, the theory registers the Aether device with the mind's Registrar, connects all adapters marked `:AutoConnect yes` in the daicho, and starts the three agents.

---

## 13  Configuration

`Aether.daicho` contains seven sections:

| Section | Contents |
|---|---|
| `[Psyche]` | Agent URL, port, delay, modality literal |
| `[OpenClaw]` | URL, adapter name, auto-connect, timeout, channels, capabilities |
| `[OmegaClaw]` | URL, adapter name, auto-connect, timeout, channels, capabilities |
| `[Port42]` | URL, adapter name, auto-connect, timeout, channels, capabilities |
| `[Routing]` | Per-channel adapter priority lists (first connected wins) |
| `[Polling]` | Per-channel polling intervals in Moments (0 = push only) |
| `[Urges]` | Homeostatic need definitions with thresholds |

---

## 14  Test Suites

`aether.suites` contains 9 test suites:

| Suite | Tests |
|---|---|
| `Aether.lifecycle` | Connect, disconnect, adapter status |
| `Aether.multi-adapter` | Multiple simultaneous adapters |
| `OpenClaw.protocol` / `OmegaClaw.protocol` / `Port42.protocol` | Adapter-specific protocol tests |
| `Aether.routing` / `Aether.routing-fallback` | Channel and action routing with fallback |
| `Aether.sensing` | Polling channels for percepts |
| `Aether.acting` / `Aether.acting-memory` / `Aether.acting-cron` | Dispatch exec, file, web, memory, cron |
| `Aether.percept-formation` | Verify PERCEPT, RESULT, URGE tuple structure |
| `Aether.urge-generation` | Monitor detects disconnected adapter |
| `Aether.convenience` | Convenience functions (search, exec, read, write, remember, recall) |
| `Aether.full-lifecycle` | End-to-end: connect, search, file, exec, message, memory, cron, sense, disconnect |

---

## 15  Framework Asymmetries

**OmegaClaw has, OpenClaw lacks:** formal symbolic reasoning (NAL, PLN) with auditable proof trails; self-modifying control loop (MeTTa runtime introspection); three-tier memory with typed hypergraph knowledge store (AtomSpace); continuous autonomous execution loop (always-on, goal-setting).

**OpenClaw has, OmegaClaw lacks:** full browser automation (Playwright-based, multi-profile); 25+ messaging channel integrations; 5,400+ community skills via ClawHub and MCP; structured file editing tools; background process management with PTY support; cron-based scheduling; ROS 2 embodiment bridge (ClawBody).

**Both have:** shell execution, file read/write, web search/fetch, messaging, persistent memory, LLM-based reasoning, extensible skill/plugin systems, multi-agent coordination.

---

## 16  Domain Coverage

| Domain | `:Channel` | Percept `:Data` values | Actuations | Primary Source |
|---|---|---|---|---|
| Local Compute | `FileSystem`, `Process` | `file-content`, `directory-listing`, `file-event`, `process-output`, `background-output` | `exec` through `file-move` | Both (core) |
| Web | `Web` | `search-result`, `page-content`, `api-response` | `web-search`, `web-fetch`, `rest-call` | Both (OpenClaw richer) |
| Communication | `Message` | `message-received`, `agent-reply`, `webhook-event` | `message-send`, `agent-send`, `agent-spawn`, `notify`, `email-send` | Both (OpenClaw richer) |
| Memory | `Memory` | `memory-result`, `knowledge-result` | `memory-store`, `memory-delete`, `knowledge-assert`, `knowledge-retract`, `nal-infer`, `pln-infer` | Both (OmegaClaw richer) |
| Browser | `Browser` | `browser-content`, `media`, `accessibility-tree` | `browser-navigate` through `browser-capture` | OpenClaw / Port42 |
| External Services | `Service` | `service-resource`, `financial-data` | `rest-call` | Both (via skills / MCP / Agentverse) |
| Embodiment | `Embodiment` | `media`, `sensor-reading`, `motor-state`, `action-result`, `transcription`, `clipboard` | `drive-command` through `capture-camera` | OpenClaw (ClawBody / ROS 2) / Port42 |
| Automation | `System` | `schedule-event`, `connection-status` | `cron-schedule`, `cron-cancel` | OpenClaw (OmegaClaw uses continuous loop) |

---

## 17  Etymology

**Aether** (Greek Αἰθήρ, *Aithḗr*) — from the verb αἴθω (*aithō*), "to burn, to ignite." Originally the bright upper atmosphere that the gods breathed, as opposed to the murkier air (*aēr*) of mortals. Aristotle elevated it to the fifth element — the *quinta essentia* — the perfect, unchanging substance of the celestial spheres. In modern physics, the "luminiferous aether" was the medium through which electromagnetic waves propagated.

Every era of meaning reinforces the same core idea: Aether is the medium through which things propagate. The internet is exactly this — a pervasive, invisible medium through which signals propagate, connecting everything. As a Psyche name, Aether says: this is the interface to the medium itself.

**Expanse** (Latin *expansum*, from *expandere*, "to spread out") was the runner-up. First attested in English in 1667, coined by Milton in *Paradise Lost* as his translation of the Hebrew *raqia* from Genesis — the firmament God created to separate the waters. Expanse emphasizes territory and scope; Aether emphasizes medium and propagation.
