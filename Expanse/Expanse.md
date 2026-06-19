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

# The Expanse 

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
11. Expanse Package
12. The Theory
13. Configuration
14. Test Suites
15. Platform Asymmetries
16. Domain Coverage
17. Etymology

---

## 1  Overview

The Expanse is a Psyche: the interface between a GIL mind and the three-dimensional physical reality that biological organisms currently inhabit. It mediates two flows — percepts traveling inward from the physical world to the mind's mechanisms, and actuations traveling outward from the mind's mechanisms to the physical world. The Expanse unifies the action surfaces of multiple robot platforms — Figure, Optimus, NAO, Unitree G2, and others — into a single coherent interface.

The Expanse encompasses the planet Earth, the oceans, inner space, and outer space. It is the three-dimensional world that robot variants will navigate, perceive, and act upon. Microphones, cameras, depth sensors, LIDAR, haptic sensors, force/torque transducers, IMUs, and GPS receivers provide multimodal percepts from the Expanse to the mind of the Learner.

Where the Aether interfaces with the internet-as-world (a digital medium), the Expanse interfaces with physical reality itself. Where the Aether's channels are Web, FileSystem, Process, and Message, the Expanse's channels are Visual, Auditory, Haptic, Proprioceptive, Spatial, and Communication. Where the Aether dispatches shell commands and API calls, the Expanse dispatches motor commands, gaze shifts, and grasping actions.

All percepts from the Expanse carry `:Modality Expanse`. The `:Channel` slot selects the sensory domain. The `:Data` slot names the parsable structure type. The `:Content` slot carries the idiom.

---

## 2  The Percept Relation

The Percept relation from `totality.theory` is canonical. It does not change for the Expanse.

```
(relation Percept
  :M                 ; lexified monad
  :Modality          ; which Psyche — Expanse for this device
  :Channel           ; sub-channel within the Psyche
  :Address  nil      ; sensor address or device URI
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

| Slot | Meaning | Expanse Examples |
|---|---|---|
| `:Modality` | Which Psyche this came through. The Psyche is the sense organ. | `Expanse` (always) |
| `:Channel` | Sub-channel within the Psyche. Sensory domain. | `Visual`, `Auditory`, `Haptic`, `Proprioceptive`, `Spatial`, `Olfactory`, `Communication`, `System` |
| `:Address` | Sensor address or device URI. Where the data came from. | `"ros2://figure-01/camera/head"`, `"ros2://nao-03/mic/left"`, `"ros2://g2/imu"`, `nil` |
| `:Data` | Data format or parsable structure type. What the `:Content` contains. | `camera-frame`, `depth-map`, `audio-sample`, `transcription`, `contact`, `force-reading`, `joint-state`, `pose`, `point-cloud`, `odometry`, `gps-fix`, `occupancy-grid`, `speech-detected` |
| `:Content` | Idiom `{:Slot value ...}` with the actual structured data. | `{:Width 640 :Height 480 :Format rgb8 :Encoding raw :FrameId "head_camera"}` |
| `:Moment` | When the percept was received. (Template slot.) | `\@m{2026060100000000}` |

For comparison, the Aether sets `:Modality` to `Aether` and uses channels like `Web`, `FileSystem`, `Browser`. The Expanse sets `:Modality` to `Expanse` and uses sensory channels like `Visual`, `Auditory`, `Haptic`. The NAO Psyche sets `:Modality` to `Visual`, `Auditory`, `Haptic` directly — the Expanse consolidates all physical modalities under a single Psyche with channels, enabling a unified interface across robot platforms.

---

## 3  Percepts by `:Channel`

All percepts from the Expanse have `:Modality Expanse`. The `:Channel` identifies the sensory domain, `:Data` identifies the format, `:Address` identifies the sensor, and `:Content` carries the structured data as an idiom.

### 3.1  Channel `Visual`

| `:Data` | `:Address` | `:Content` (idiom) | Platforms |
|---|---|---|---|
| `camera-frame` | `"ros2://figure-01/camera/head"` | `{:Width 640 :Height 480 :Format rgb8 :Encoding raw :FrameId "head_camera" :FOV 90.0}` | All |
| `depth-map` | `"ros2://optimus-01/depth/front"` | `{:Width 640 :Height 480 :Format 32FC1 :MinRange 0.2 :MaxRange 10.0 :FrameId "depth_front"}` | Figure, Optimus, G2 |
| `point-cloud` | `"ros2://g2/lidar/front"` | `{:Width 1024 :Height 1 :PointStep 16 :Fields {:X float :Y float :Z float :Intensity float} :FrameId "lidar_front"}` | Figure, Optimus, G2 |
| `object-detected` | nil | `{:Class "cup" :Confidence 0.92 :BBox {:X 120 :Y 80 :Width 50 :Height 60} :Distance 1.2 :FrameId "head_camera"}` | All (via onboard vision) |
| `face-detected` | nil | `{:FaceId "face-42" :Confidence 0.88 :BBox {:X 200 :Y 100 :Width 80 :Height 90} :Landmarks {...} :Expression neutral}` | All (via onboard vision) |

### 3.2  Channel `Auditory`

| `:Data` | `:Address` | `:Content` (idiom) | Platforms |
|---|---|---|---|
| `audio-sample` | `"ros2://nao-03/mic/array"` | `{:SampleRate 16000 :Channels 4 :Format pcm_s16le :Duration 500 :FrameId "mic_array"}` | All |
| `transcription` | nil | `{:Text "hello there" :IsFinal yes :Language "en-US" :Confidence 0.95 :Speaker "unknown" :Duration 1200}` | All (via onboard ASR) |
| `speech-detected` | nil | `{:Direction 45.0 :Azimuth 45.0 :Elevation 0.0 :Energy 0.72 :FrameId "mic_array"}` | NAO, Figure, G2 |
| `sound-event` | nil | `{:EventType clap :Confidence 0.8 :Direction 90.0 :Duration 200}` | All |

### 3.3  Channel `Haptic`

| `:Data` | `:Address` | `:Content` (idiom) | Platforms |
|---|---|---|---|
| `contact` | `"ros2://figure-01/hand/right/contact"` | `{:Regions {:Palm yes :Fingers {yes yes no no yes}} :Pressure 2.4 :Unit "N" :FrameId "right_hand"}` | Figure, Optimus, G2 |
| `force-reading` | `"ros2://figure-01/wrist/right/ft"` | `{:Force {:X 0.5 :Y -0.2 :Z 9.8} :Torque {:X 0.01 :Y 0.02 :Z -0.005} :Unit "N/Nm" :FrameId "right_wrist"}` | Figure, Optimus |
| `collision` | nil | `{:LinkName "left_forearm" :ContactPoint {:X 0.3 :Y 0.0 :Z 0.8} :Force 15.2 :Unit "N" :FrameId "base_link"}` | All |
| `temperature` | `"ros2://optimus-01/hand/right/temp"` | `{:Value 32.5 :Unit "C" :FrameId "right_hand"}` | Optimus |
| `bumper` | `"ros2://nao-03/bumper/left_foot"` | `{:State pressed :Region left_foot}` | NAO |

### 3.4  Channel `Proprioceptive`

| `:Data` | `:Address` | `:Content` (idiom) | Platforms |
|---|---|---|---|
| `joint-state` | `"ros2://figure-01/joint_states"` | `{:Joints {{:Name "left_shoulder_pitch" :Position 0.5 :Velocity 0.0 :Effort 1.2} {:Name "left_elbow" :Position 1.2 :Velocity 0.1 :Effort 0.8} ...} :Unit "rad"}` | All |
| `imu-reading` | `"ros2://g2/imu"` | `{:Orientation {:Roll 0.02 :Pitch -0.01 :Yaw 1.57} :AngularVelocity {:X 0.0 :Y 0.0 :Z 0.01} :LinearAcceleration {:X 0.1 :Y 0.0 :Z 9.81} :FrameId "imu_link"}` | All |
| `battery-state` | `"ros2://nao-03/battery"` | `{:Voltage 12.4 :Current -1.2 :Percentage 0.73 :Charging no :Unit "V/A"}` | All |
| `motor-temperature` | `"ros2://optimus-01/motor_temps"` | `{:Joints {{:Name "left_hip" :Temperature 42.3} {:Name "right_knee" :Temperature 38.1} ...} :Unit "C"}` | Optimus, Figure |
| `balance-state` | nil | `{:CenterOfMass {:X 0.01 :Y -0.02 :Z 0.85} :CenterOfPressure {:X 0.02 :Y -0.01} :Stable yes :FrameId "base_link"}` | All |

### 3.5  Channel `Spatial`

| `:Data` | `:Address` | `:Content` (idiom) | Platforms |
|---|---|---|---|
| `odometry` | `"ros2://figure-01/odom"` | `{:Position {:X 3.2 :Y 1.5 :Z 0.0} :Orientation {:Roll 0.0 :Pitch 0.0 :Yaw 0.78} :LinearVelocity {:X 0.5 :Y 0.0 :Z 0.0} :FrameId "odom"}` | All |
| `gps-fix` | `"ros2://optimus-01/gps"` | `{:Latitude 37.7749 :Longitude -122.4194 :Altitude 10.2 :HorizontalAccuracy 2.0 :Unit "deg/m"}` | Optimus, outdoor platforms |
| `pose` | `"ros2://g2/pose"` | `{:Position {:X 3.2 :Y 1.5 :Z 0.0} :Orientation {:X 0.0 :Y 0.0 :Z 0.38 :W 0.92} :FrameId "map"}` | All |
| `occupancy-grid` | nil | `{:Width 200 :Height 200 :Resolution 0.05 :Origin {:X -5.0 :Y -5.0} :FrameId "map"}` | All (via SLAM) |
| `landmark` | nil | `{:LandmarkId "door-kitchen" :Position {:X 5.2 :Y 3.1 :Z 0.0} :Type door :Confidence 0.9 :FrameId "map"}` | All (via SLAM) |

### 3.6  Channel `Olfactory`

| `:Data` | `:Address` | `:Content` (idiom) | Platforms |
|---|---|---|---|
| `gas-reading` | `"ros2://optimus-01/gas/mox"` | `{:Sensor "MQ-2" :Value 320 :Unit "ppm" :Gases {:CO 0.12 :Methane 0.05 :Smoke 0.08}}` | Future platforms |
| `air-quality` | `"ros2://optimus-01/air"` | `{:Temperature 22.5 :Humidity 0.45 :PM25 12.3 :CO2 410 :Unit "C/%/ug_m3/ppm"}` | Future platforms |

### 3.7  Channel `Communication`

| `:Data` | `:Address` | `:Content` (idiom) | Platforms |
|---|---|---|---|
| `speech-heard` | nil | `{:Text "pick up the cup" :Speaker "user-01" :Language "en-US" :Confidence 0.93 :Intent {:Action pick-up :Object cup}}` | All |
| `gesture-seen` | nil | `{:GestureType pointing :Direction {:Azimuth 30.0 :Elevation -10.0} :Confidence 0.85 :PersonId "user-01"}` | Figure, Optimus, G2 |
| `touch-signal` | nil | `{:Region head :Duration 1500 :Pressure light :PersonId "user-01"}` | NAO, Figure |
| `display-tap` | nil | `{:WidgetId "confirm-button" :Position {:X 120 :Y 80} :DisplayId "chest"}` | NAO (tablet), future |

### 3.8  Channel `System`

| `:Data` | `:Address` | `:Content` (idiom) | Platforms |
|---|---|---|---|
| `connection-status` | `"ros2://figure-01"` | `{:Platform figure :Status connected :Error nil}` | All |
| `safety-event` | nil | `{:EventType e-stop :Source hardware :Severity critical}` | All |
| `mode-change` | nil | `{:OldMode autonomous :NewMode teleoperated :Reason "user request"}` | All |
| `diagnostic` | nil | `{:Component left_hip_motor :Level warning :Message "temperature high" :Value 58.2}` | All |

### 3.9  Percept Examples

```
[Percept :Modality Expanse  :Channel Visual  :Address "ros2://figure-01/camera/head"
         :Data camera-frame
         :Content {:Width 640 :Height 480 :Format rgb8 :FrameId "head_camera"}
         :Moment \@m{...}]

[Percept :Modality Expanse  :Channel Auditory  :Address nil
         :Data transcription
         :Content {:Text "pick up the cup" :IsFinal yes :Language "en-US" :Confidence 0.95}
         :Moment \@m{...}]

[Percept :Modality Expanse  :Channel Haptic  :Address "ros2://figure-01/wrist/right/ft"
         :Data force-reading
         :Content {:Force {:X 0.5 :Y -0.2 :Z 9.8} :Torque {:X 0.01 :Y 0.02 :Z -0.005}
                   :Unit "N/Nm" :FrameId "right_wrist"}
         :Moment \@m{...}]

[Percept :Modality Expanse  :Channel Proprioceptive  :Address "ros2://g2/joint_states"
         :Data joint-state
         :Content {:Joints {{:Name "left_knee" :Position 0.8 :Velocity 0.0 :Effort 3.1} ...}
                   :Unit "rad"}
         :Moment \@m{...}]

[Percept :Modality Expanse  :Channel Spatial  :Address "ros2://figure-01/odom"
         :Data odometry
         :Content {:Position {:X 3.2 :Y 1.5 :Z 0.0} :Orientation {:Yaw 0.78}
                   :LinearVelocity {:X 0.5} :FrameId "odom"}
         :Moment \@m{...}]

[Percept :Modality Expanse  :Channel Communication  :Address nil
         :Data gesture-seen
         :Content {:GestureType pointing :Direction {:Azimuth 30.0 :Elevation -10.0}
                   :Confidence 0.85 :PersonId "user-01"}
         :Moment \@m{...}]
```

---

## 4  Common `:Content` Idiom Slots

Examining percepts across all `:Channel` × `:Data` combinations reveals shared structure.

### Slots Appearing in 10+ `:Content` Idioms

| Slot | Type | Count | Notes |
|---|---|---|---|
| `:FrameId` | string | 25+ | Coordinate frame of reference. Universal in physical percepts. |
| `:Unit` | literal | 20+ | Measurement unit (rad, N, m, C, ppm, deg, etc.). |
| `:Width` / `:Height` / `:Duration` | integer | 12 | Dimensional measure of the data. |
| `:Confidence` | real | 10 | Certainty of the measurement or detection. |

### Slots Appearing in 3–9 `:Content` Idioms

| Slot | Type | Count | Notes |
|---|---|---|---|
| `:Position` / `:Orientation` | idiom | 8 | Pose in 3D space. |
| `:Format` / `:Encoding` | literal | 7 | Data format (rgb8, pcm_s16le, 32FC1, etc.). |
| `:Direction` / `:Azimuth` / `:Elevation` | real | 5 | Angular direction of a source. |
| `:SampleRate` / `:Channels` | integer | 3 | Audio sampling parameters. |
| `:Velocity` / `:Effort` | real | 5 | Joint or body velocity and effort. |
| `:Force` / `:Torque` / `:Pressure` | idiom/real | 5 | Physical force measurements. |
| `:PersonId` / `:Speaker` / `:FaceId` | string | 5 | Identity of a detected person. |
| `:Language` / `:Text` | string | 4 | Speech and language content. |
| `:Temperature` / `:Value` | real | 3 | Scalar sensor readings. |
| `:EventType` / `:Severity` | literal | 3 | System events and diagnostics. |

---

## 5  Actuations

Actuations are commands the mind's mechanisms dispatch through the Expanse to change the state of the physical world. Following GIL convention, actuations are sent as ATTEMPT structures and results return as RESULT percepts.

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
| A01 | `walk` | Walk in a direction | `{:LinearVelocity 0.5 :AngularVelocity 0.0 :Duration 3000}` |
| A02 | `turn` | Rotate in place | `{:Angle 90.0 :Velocity 0.5 :Unit "deg"}` |
| A03 | `stop` | Halt all locomotion | `{}` |
| A04 | `step` | Take a single step | `{:Foot left :StepLength 0.3 :StepHeight 0.05}` |
| A05 | `stand` | Rise from sitting/crouching | `{}` |
| A06 | `sit` | Lower to seated position | `{}` |
| A07 | `crouch` | Lower center of mass | `{:Height 0.4}` |
| A08 | `navigate` | Navigate to a pose | `{:Goal {:X 5.2 :Y 3.1 :Yaw 1.57} :FrameId "map" :Planner default}` |
| A09 | `look` | Direct gaze at a target | `{:Target {:X 1.0 :Y 0.5 :Z 1.2} :FrameId "base_link"}` |
| A10 | `track` | Continuously track a target | `{:TargetId "face-42" :Mode smooth}` |
| A11 | `reach` | Extend arm to a point | `{:Arm right :Target {:X 0.5 :Y -0.2 :Z 0.8} :FrameId "base_link"}` |
| A12 | `grasp` | Close gripper/hand | `{:Hand right :Force 5.0 :Unit "N"}` |
| A13 | `release` | Open gripper/hand | `{:Hand right}` |
| A14 | `push` | Push an object | `{:Arm right :Force 10.0 :Direction {:X 1.0 :Y 0.0 :Z 0.0} :Unit "N"}` |
| A15 | `pull` | Pull an object | `{:Arm right :Force 8.0 :Direction {:X -1.0 :Y 0.0 :Z 0.0} :Unit "N"}` |
| A16 | `lift` | Lift a grasped object | `{:Arm right :Height 0.3 :Unit "m"}` |
| A17 | `place` | Place an object at a location | `{:Arm right :Target {:X 0.5 :Y 0.0 :Z 0.7} :FrameId "base_link"}` |
| A18 | `point` | Point at a target | `{:Arm right :Target {:X 3.0 :Y 1.0 :Z 1.0} :FrameId "base_link"}` |
| A19 | `wave` | Wave gesture | `{:Arm right :Repetitions 3}` |
| A20 | `nod` | Nod head | `{:Repetitions 2 :Amplitude 15.0 :Unit "deg"}` |
| A21 | `shake-head` | Shake head | `{:Repetitions 2 :Amplitude 20.0 :Unit "deg"}` |
| A22 | `speak` | Produce speech via TTS | `{:Text "hello world" :Language "en-US" :Voice "default" :Rate 1.0 :Volume 0.8}` |
| A23 | `emote` | Display an emotion via LEDs or screen | `{:Emotion happy :Intensity 0.8 :Duration 2000}` |
| A24 | `display` | Show content on a screen | `{:DisplayId "chest" :Content "Ready" :Format text}` |
| A25 | `play-sound` | Play an audio file or tone | `{:Sound "beep" :Volume 0.5 :Duration 500}` |
| A26 | `joint-move` | Move a specific joint | `{:JointName "left_shoulder_pitch" :Position 1.0 :Velocity 0.5 :Unit "rad"}` |
| A27 | `pose-move` | Move to a named or computed pose | `{:PoseName "home" :Duration 2000}` |
| A28 | `trajectory` | Execute a joint trajectory | `{:Joints {"left_shoulder_pitch" "left_elbow"} :Points {{:Positions {0.5 1.0} :TimeFromStart 1000} {:Positions {1.0 0.5} :TimeFromStart 2000}}}` |
| A29 | `set-stiffness` | Set joint stiffness | `{:JointName "all" :Stiffness 1.0}` |
| A30 | `set-led` | Control LEDs | `{:Group "eyes" :Color {:R 0 :G 255 :B 0} :Duration 3000}` |
| A31 | `capture-photo` | Take a photo | `{:Camera "head" :Width 1280 :Height 720 :Format png}` |
| A32 | `start-recording` | Begin audio/video recording | `{:Source "head_camera" :Duration 30000 :Format mp4}` |
| A33 | `stop-recording` | End recording | `{:RecordingId "rec-42"}` |
| A34 | `e-stop` | Emergency stop | `{}` |
| A35 | `set-mode` | Switch operating mode | `{:Mode autonomous}` |

### 5.3  Actuation Domains

**Locomotion:** `walk`, `turn`, `stop`, `step`, `stand`, `sit`, `crouch`, `navigate`

**Gaze:** `look`, `track`

**Manipulation:** `reach`, `grasp`, `release`, `push`, `pull`, `lift`, `place`

**Gesture:** `point`, `wave`, `nod`, `shake-head`

**Communication:** `speak`, `emote`, `display`, `play-sound`

**Motor Control:** `joint-move`, `pose-move`, `trajectory`, `set-stiffness`, `set-led`

**Capture:** `capture-photo`, `start-recording`, `stop-recording`

**System:** `e-stop`, `set-mode`

### 5.4  ATTEMPT Examples

```
[ATTEMPT :Action walk
         :Parameters {:LinearVelocity 0.5 :AngularVelocity 0.0 :Duration 3000}
         :Token ?tok  :By \@m{...}]

[ATTEMPT :Action grasp
         :Parameters {:Hand right :Force 5.0 :Unit "N"}
         :Token ?tok]

[ATTEMPT :Action navigate
         :Parameters {:Goal {:X 5.2 :Y 3.1 :Yaw 1.57} :FrameId "map" :Planner default}
         :Token ?tok]

[ATTEMPT :Action speak
         :Parameters {:Text "I see the cup on the table" :Language "en-US" :Rate 1.0}
         :Token ?tok]

[ATTEMPT :Action look
         :Parameters {:Target {:X 1.0 :Y 0.5 :Z 1.2} :FrameId "base_link"}
         :Token ?tok]
```

---

## 6  Cross-Cutting Structure

### 6.1  The Four Tuple Types

```
PERCEPT:  [PERCEPT :Modality Expanse :Channel C :Address A :Data D :Content {:...} :Moment M :Token T]
ATTEMPT:  [ATTEMPT :Action A :Parameters {:...} :Token T :By M]
RESULT:   [RESULT  :Action A :Status S :Reason R :Moment M :Token T]
URGE:     [URGE    :Need N :Source S :Delta D :Moment M :Token T]
```

The Expanse is a bidirectional envelope router: it wraps outbound commands in Attempt structures, dispatches them to the correct robot platform, receives the physical world's response, and sends it back as a Percept, a Result, or an Urge to the Perceiver. The mind learns what its body did through perception — proprioception.

### 6.2  Actuation → Percept Pairings

| `:Action` | Produces `:Data` |
|---|---|
| `walk`, `turn`, `step`, `navigate` | `odometry`, `joint-state`, `balance-state` |
| `look`, `track` | `camera-frame`, `object-detected`, `face-detected` |
| `reach`, `grasp`, `release` | `joint-state`, `contact`, `force-reading` |
| `push`, `pull`, `lift`, `place` | `force-reading`, `joint-state`, `contact` |
| `speak` | `audio-sample` (echo), `transcription` (if self-monitoring) |
| `joint-move`, `pose-move`, `trajectory` | `joint-state` |
| `capture-photo` | `camera-frame` |
| `navigate` | `odometry`, `pose`, `occupancy-grid` |
| `e-stop` | `safety-event` |

Some percepts are also spontaneous — a person speaks, an object moves, a collision occurs, the battery drops — arriving without a prior Attempt. Homeostatic signals arrive as `[URGE :Need N :Source S :Delta D :Moment \@m{...} :Token T]`.

---

## 7  Architecture

### 7.1  The Expanse as Mediator

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
                          │ EXPANSE PSYCHE  │
                          │                 │
                          │  :Modality      │  Expanse (always)
                          │  :Channel       │  Visual | Auditory |
                          │                 │  Haptic | Proprio- |
                          │                 │  ceptive | Spatial |
                          │                 │  Olfactory | Comm- |
                          │                 │  unication | System
                          │  :Address       │  sensor URI
                          │  :Data          │  format literal
                          │  :Content       │  idiom {:...}
                          │                 │
                          └────────┬────────┘
                                   │
          ┌────────────┬───────────┼───────────┬───────────────┐
          │            │           │           │               │
    ┌─────┴──┐   ┌─────┴──┐  ┌────┴───┐  ┌───┴────┐  ┌───────┴───┐
    │ Figure │   │Optimus │  │  NAO   │  │  G2    │  │  Others   │
    │  01    │   │  01    │  │  03    │  │  01    │  │ (ROS 2)   │
    └────────┘   └────────┘  └────────┘  └────────┘  └───────────┘
```

---

## 8  Deployment File Set

| File | Category | Description |
|---|---|---|
| `figure.package` | Adapter | Figure humanoid protocol: joint groups, grippers, cameras, depth |
| `optimus.package` | Adapter | Tesla Optimus protocol: actuator API, sensor suite, GPS |
| `nao.package` | Adapter | NAO protocol: ALMotion, ALAudioDevice, ALVideoDevice, etc. |
| `g2.package` | Adapter | Unitree G2 protocol: legged locomotion, stereo cameras, LIDAR |
| `generic-ros2.package` | Adapter | Generic ROS 2 bridge for any compliant robot |
| `expanse.package` | Unified Layer | Requires adapter packages. Routing, normalization, convenience |
| `expanse-psyche.theory` | Theory | Single agent: job (sensing + monitoring) + handler (actuation) |
| `Expanse.daicho` | Configuration | Platform addresses, channels, routing, polling, urge thresholds |
| `expanse.suites` | Tests | Lifecycle, adapter protocols, routing, sensing, acting, percept formation, full lifecycle |

---

## 9  Package Layering

```
expanse-psyche.theory          agent: Sensor job + Actuator handler
    │
    ▼
expanse.package                routing, normalization, uniform interface
    │
    ├──▶ figure.package        Figure protocol (ROS 2 / proprietary API)
    ├──▶ optimus.package       Optimus protocol (Tesla API / ROS 2)
    ├──▶ nao.package           NAO protocol (NAOqi / ROS 2)
    ├──▶ g2.package            G2 protocol (Unitree SDK / ROS 2)
    └──▶ generic-ros2.package  Generic ROS 2 topics, services, actions
```

---

## 10  Adapter Packages

### 10.1  `figure.package`

Wraps the Figure humanoid's capabilities: 41+ DOF manipulation, dexterous hands, head/torso cameras, depth sensors, force/torque at wrists, IMU, whole-body control. Communicates via ROS 2 topics, services, and actions. Exposes functions prefixed `fig-`, plus a generic `fig-dispatch`.

### 10.2  `optimus.package`

Wraps the Tesla Optimus platform: 28+ DOF, actuator-level control, stereo cameras, depth, LIDAR, force-sensing hands, GPS (outdoor), battery management. Exposes functions prefixed `opt-`, plus `opt-dispatch`.

### 10.3  `nao.package`

Wraps the NAO humanoid's NAOqi framework: ALMotion (25 DOF), ALAudioDevice (4-mic array), ALVideoDevice (head cameras), ALTextToSpeech, ALFaceDetection, bumpers, head touch. Exposes functions prefixed `nao-`, plus `nao-dispatch`. This package subsumes the existing `nao-psyche.theory` protocol into the Expanse architecture.

### 10.4  `g2.package`

Wraps the Unitree G2 humanoid: legged locomotion, stereo cameras, LIDAR, dexterous grippers, IMU, whole-body motion planning. Communicates via Unitree SDK and ROS 2. Exposes functions prefixed `g2-`, plus `g2-dispatch`.

### 10.5  `generic-ros2.package`

Generic bridge for any ROS 2-compliant robot. Subscribes to standard topics (`/joint_states`, `/odom`, `/camera/image_raw`, `/scan`), calls standard services, and invokes standard actions (`/navigate_to_pose`, `/follow_joint_trajectory`). Platform-agnostic fallback. Exposes functions prefixed `ros2-`, plus `ros2-dispatch`.

---

## 11  Expanse Package

`expanse.package` requires all adapter packages and provides:

**Platform Registry:** A `Platform` relation tracking `:Name`, `:Handle`, `:Status`, `:Channels`, and `:Capabilities` for each connected robot.

**Lifecycle:** `expanse-connect-platform`, `expanse-disconnect-platform`, `expanse-connect-all`, `expanse-disconnect-all`.

**Routing:** `expanse-route` (find platform by channel), `expanse-route-action` (find platform by action capability).

**Sensing:** `expanse-sense` polls a platform for percepts on a sensory channel.

**Acting:** `expanse-act` dispatches an `:Action` + `:Parameters` idiom to the correct platform.

**Convenience:** `expanse-walk`, `expanse-look`, `expanse-grasp`, `expanse-speak`, `expanse-navigate`, etc.

---

## 12  The Theory

`expanse-psyche.theory` defines a single agent with:

**Job (proactive):** runs on a fast delay (default 20ms for real-time sensor data). Polls each sensory channel: camera frames, audio samples, joint states, force readings, odometry, spatial data. Wraps each as a PERCEPT tuple and tells the Perceiver. Monitors homeostatic conditions: battery level, motor temperatures, balance stability, connection health. Emits URGE tuples when thresholds are crossed.

**Handler (reactive):** receives ATTEMPTs from the Executor. Dispatches through `expanse-act`. Returns RESULT tuples to the Perceiver.

```
(agent Expanse expanse-job ?psyche-url expanse-handler ?psyche-delay {sensing acting monitoring})
```

---

## 13  Configuration

`Expanse.daicho` contains sections:

| Section | Contents |
|---|---|
| `[Psyche]` | Agent URL, port, delay (fast: `\@m{0.2}` for 20ms), modality |
| `[Figure]` | ROS 2 namespace, topics, auto-connect, capabilities |
| `[Optimus]` | ROS 2 namespace, topics, auto-connect, capabilities |
| `[NAO]` | NAOqi address or ROS 2 namespace, auto-connect, capabilities |
| `[G2]` | Unitree address or ROS 2 namespace, auto-connect, capabilities |
| `[Routing]` | Per-channel platform priority lists |
| `[Polling]` | Per-channel polling intervals (fast for Visual/Proprioceptive, slower for Spatial) |
| `[Urges]` | Homeostatic needs: low-battery, overheating, imbalance, collision, connection-lost |

---

## 14  Test Suites

`expanse.suites` contains test suites covering: platform lifecycle, adapter protocols (per-platform), routing and fallback, sensing (per-channel), acting (locomotion, manipulation, gaze, speech), percept formation, urge generation, convenience functions, and full end-to-end lifecycle.

---

## 15  Platform Asymmetries

**Figure has, others may lack:** 41+ DOF, dexterous hands with individual finger control, force/torque at both wrists, whole-body dynamic locomotion.

**Optimus has, others may lack:** GPS for outdoor navigation, actuator-level telemetry, battery management system, Tesla-specific vision pipeline.

**NAO has, others may lack:** 4-microphone array with sound localization, capacitive head sensors, chest tablet display, extensive NAOqi behavior library.

**G2 has, others may lack:** Agile legged locomotion, LIDAR-based SLAM, Unitree high-speed motor control.

**All share:** cameras, IMU, joint position/velocity feedback, walking, speaking, grasping (varying DOF), ROS 2 compatibility.

---

## 16  Domain Coverage

| Domain | `:Channel` | Percept `:Data` values | Actuations | Primary Platform |
|---|---|---|---|---|
| Vision | `Visual` | `camera-frame`, `depth-map`, `point-cloud`, `object-detected`, `face-detected` | `look`, `track`, `capture-photo` | All |
| Hearing | `Auditory` | `audio-sample`, `transcription`, `speech-detected`, `sound-event` | `speak`, `play-sound` | All |
| Touch | `Haptic` | `contact`, `force-reading`, `collision`, `temperature`, `bumper` | `grasp`, `release`, `push`, `pull` | All (varying) |
| Body Sense | `Proprioceptive` | `joint-state`, `imu-reading`, `battery-state`, `motor-temperature`, `balance-state` | `joint-move`, `pose-move`, `trajectory`, `set-stiffness` | All |
| Navigation | `Spatial` | `odometry`, `gps-fix`, `pose`, `occupancy-grid`, `landmark` | `walk`, `turn`, `step`, `navigate` | All |
| Smell | `Olfactory` | `gas-reading`, `air-quality` | — | Future |
| Interaction | `Communication` | `speech-heard`, `gesture-seen`, `touch-signal`, `display-tap` | `speak`, `emote`, `display`, `wave`, `nod`, `point` | All |
| Safety | `System` | `connection-status`, `safety-event`, `mode-change`, `diagnostic` | `e-stop`, `set-mode` | All |

---

## 17  Etymology

**Expanse** (Latin *expansum*, from *expandere*, "to spread out, to unfold") — first attested in English in 1667, coined by John Milton in *Paradise Lost* as his translation of the Hebrew *raqia* from Genesis — the firmament God created on the second day to separate the waters above from the waters below. The Hebrew root *raqa* means "to stretch, spread out, then beat or tread out," as one would hammer metal into a thin sheet.

Milton chose "expanse" over "firmament" because he wanted to emphasize openness and vastness rather than solidity. The word entered the English language and stuck. It means a large, open, unbroken area — especially of land, water, or sky. A broad or extensive view. A wide range or scope.

The Expanse captures the physical world's most defining quality: it is vast, open, three-dimensional, and stretches out in every direction without visible boundary. Where the Aether emphasizes the *medium* through which signals propagate (the internet), the Expanse emphasizes the *territory* through which bodies move (the physical world). It is the ground truth — the reality that precedes all digital representations of it.

**Aether** was the runner-up for the internet-world Psyche and was chosen for that role. The two names complement each other: Aether is the invisible medium, Expanse is the visible territory. Together they cover the full range of worlds a GIL mind can inhabit.
