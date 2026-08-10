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

# The Kojin

## Specification and Deployment Guide

**SubThought Corporation**

Version 1.0 — August 2026

---

## Table of Contents

1. Overview
2. The Percept Relation
3. Percepts by Channel
4. Common Content Idiom Slots
5. Actuations
6. Cross-Cutting Structure
7. The Jungian Model
8. Architecture
9. Deployment File Set
10. The Package
11. The Theory
12. Configuration
13. Test Suites
14. Privacy and Consent
15. Domain Coverage
16. Etymology

---

## 1  Overview

The Kojin is a Psyche: the interface between a GIL mind and a single human being as world. It mediates two flows — percepts traveling inward from the person's expressed life to the mind's mechanisms, and actuations traveling outward from the mind's mechanisms back to the person. Where the Aether interfaces with the internet-as-world, the Expanse with physical-reality-as-world, and the Eidos with a grid-as-world, the Kojin interfaces with the human-as-world.

The human-as-world is not a bounded simulation and not a passive data source. It is a living subject who converses, journals, shares documents, expresses preferences, reveals strengths and weaknesses, and moves through a landscape of opportunities and threats. The Kojin collects this personal data — always with consent — and helps the mind build, refine, and act on a model of the person. That model has two registers: the observable life record (biography, preferences, capabilities, circumstances) and the depth-psychological structure (a working model of the person's Jungian ego and Jungian shadow, inferred from what the person says, avoids, repeats, projects, and dreams).

The Kojin serves four functions: **learn** the person's preferences, strengths, and weaknesses; **guide** the person toward opportunities and away from threats; **journal** with the person through conversation and analysis of shared information; and **model** the person's ego and shadow so guidance is grounded in who the person actually is, not just what they say they want.

Unlike the Eidos (turn-based) but like the Aether, the Kojin is event-driven and slow-polling: percepts arrive when the person speaks, writes, or shares, and on periodic review of accumulated material.

All percepts from the Kojin carry `:Modality Kojin`. The `:Channel` slot selects the domain of personhood. The `:Data` slot names the parsable structure type. The `:Content` slot carries the idiom.

---

## 2  The Percept Relation

The Percept relation from `totality.theory` is canonical. It does not change for the Kojin.

```
(relation Percept
  :M                 ; lexified monad
  :Modality          ; which Psyche — Kojin for this device
  :Channel           ; sub-channel within the Psyche
  :Address  nil      ; source URI of the data
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

| Slot | Meaning | Kojin Examples |
|---|---|---|
| `:Modality` | Which Psyche this came through. The Psyche is the sense organ. | `Kojin` (always) |
| `:Channel` | Sub-channel within the Psyche. Domain of personhood. | `Dialogue`, `Journal`, `Document`, `Preference`, `Capability`, `Situation`, `Depth`, `System` |
| `:Address` | Source URI of the data. Where it came from. | `"kojin://alex/session-14"`, `"file:///uploads/resume.pdf"`, `"gmail://alex/msg-9921"`, `nil` |
| `:Data` | Data format or parsable structure type. What the `:Content` contains. | `utterance`, `journal-entry`, `document-shared`, `preference-signal`, `strength-observed`, `weakness-observed`, `opportunity-detected`, `threat-detected`, `ego-trait`, `shadow-trait` |
| `:Content` | Idiom `{:Slot value ...}` with the actual structured data. | `{:Text "..." :Sentiment -0.3 :Topics {work family} :SessionId "s14"}` |
| `:Moment` | When the percept was received. (Template slot.) | `\@m{2026080800000000}` |

For comparison, the Aether's channels partition the internet (Web, FileSystem, Message); the Expanse's channels partition the sensorium (Visual, Auditory, Haptic). The Kojin's channels partition the *person*: what they say (Dialogue), what they write reflectively (Journal), what they share (Document), what they want (Preference), what they can and cannot do (Capability), what surrounds them (Situation), and who they are beneath the surface (Depth).

---

## 3  Percepts by `:Channel`

All percepts from the Kojin have `:Modality Kojin`. The `:Channel` identifies the domain, `:Data` identifies the format, `:Address` identifies the source, and `:Content` carries the structured data as an idiom.

### 3.1  Channel `Dialogue`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `utterance` | `"kojin://alex/session-14"` | `{:Text "I turned down the promotion" :SessionId "s14" :ThreadId "t2" :Sentiment -0.3 :Topics {work} :Attachments {}}` |
| `question-answered` | `"kojin://alex/session-14"` | `{:QuestionId "q-88" :Text "..." :Evasive no :Latency 4200}` |
| `question-avoided` | `"kojin://alex/session-14"` | `{:QuestionId "q-89" :Topic family :AvoidanceType deflection}` |

### 3.2  Channel `Journal`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `journal-entry` | `"kojin://alex/journal/2026-08-08"` | `{:Text "..." :WordCount 340 :Sentiment 0.1 :Topics {sleep career} :Prompted yes :PromptId "p-12"}` |
| `journal-pattern` | nil | `{:PatternType recurring-theme :Theme "fear of being ordinary" :Occurrences 7 :FirstSeen \@m{...} :LastSeen \@m{...}}` |
| `dream-report` | `"kojin://alex/journal/2026-08-08"` | `{:Text "..." :Figures {pursuer water house} :Affect anxious}` |

### 3.3  Channel `Document`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `document-shared` | `"file:///uploads/resume.pdf"` | `{:ContentType "application/pdf" :Size 88213 :DocumentType resume :Summary "..." :Entities {...}}` |
| `message-shared` | `"gmail://alex/msg-9921"` | `{:Counterparty "boss@corp.com" :Direction received :Summary "..." :Tone terse}` |
| `record-shared` | `"file:///uploads/lab-results.pdf"` | `{:RecordType medical :Summary "..." :Sensitivity high}` |

### 3.4  Channel `Preference`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `preference-signal` | nil | `{:Domain food :Object "cilantro" :Valence -0.9 :Source stated :Confidence 0.95}` |
| `preference-revealed` | nil | `{:Domain work :Object "remote work" :Valence 0.7 :Source behavioral :Evidence {...} :Confidence 0.6}` |
| `value-signal` | nil | `{:Value autonomy :Rank 1 :Source inferred :Evidence {...}}` |

Stated preferences come from the person's words; revealed preferences come from their behavior. The two frequently disagree — that disagreement is itself a Depth percept (§3.7).

### 3.5  Channel `Capability`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `strength-observed` | nil | `{:Skill "public speaking" :Evidence {...} :Confidence 0.8 :Source {dialogue document}}` |
| `weakness-observed` | nil | `{:Skill "delegation" :Evidence {...} :Confidence 0.7 :Impact high}` |
| `growth-signal` | nil | `{:Skill "spanish" :Direction improving :Delta 0.1 :Window \@m{90}}` |

### 3.6  Channel `Situation`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `opportunity-detected` | `"https://jobs..."` | `{:Kind career :Description "..." :FitScore 0.85 :Window \@m{...} :BasisStrengths {...} :BasisPreferences {...}}` |
| `threat-detected` | nil | `{:Kind financial :Description "..." :Severity 0.7 :Horizon near :BasisWeaknesses {...}}` |
| `life-event` | nil | `{:EventType relocation :Description "..." :Disruption 0.6}` |

Situation percepts are typically synthesized: the Kojin's Analyst correlates Capability and Preference percepts against external data (often fetched through a sibling Aether Psyche on the same mind).

### 3.7  Channel `Depth`

The depth-psychological channel. Percepts here are *inferences*, always carrying `:Confidence` and `:Evidence`.

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `ego-trait` | nil | `{:Trait "the competent provider" :Register ego :Confidence 0.8 :Evidence {...} :Stability stable}` |
| `shadow-trait` | nil | `{:Trait "resentment of obligation" :Register shadow :Confidence 0.55 :Evidence {avoidance-q89 projection-p3} :Stability emerging}` |
| `projection-detected` | nil | `{:Target "coworker Dan" :Attribution "lazy" :ShadowCandidate "own fear of idleness" :Confidence 0.5}` |
| `persona-gap` | nil | `{:Stated {...} :Revealed {...} :GapSize 0.6 :Domain work}` |
| `integration-signal` | nil | `{:Trait "resentment of obligation" :Direction acknowledged :Evidence {journal-entry-...}}` |

### 3.8  Channel `System`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `consent-change` | nil | `{:Scope medical :Granted no :Moment \@m{...}}` |
| `session-event` | `"kojin://alex/session-14"` | `{:EventType opened :SessionId "s14"}` |
| `connection-status` | nil | `{:Source gmail :Status connected :Error nil}` |

### 3.9  Percept Examples

```
[Percept :Modality Kojin  :Channel Dialogue  :Address "kojin://alex/session-14"
         :Data utterance
         :Content {:Text "I turned down the promotion" :SessionId "s14"
                   :Sentiment -0.3 :Topics {work}}
         :Moment \@m{...}]

[Percept :Modality Kojin  :Channel Journal  :Address nil
         :Data journal-pattern
         :Content {:PatternType recurring-theme :Theme "fear of being ordinary"
                   :Occurrences 7}
         :Moment \@m{...}]

[Percept :Modality Kojin  :Channel Depth  :Address nil
         :Data shadow-trait
         :Content {:Trait "resentment of obligation" :Register shadow
                   :Confidence 0.55 :Evidence {avoidance-q89 projection-p3}}
         :Moment \@m{...}]

[Percept :Modality Kojin  :Channel Situation  :Address "https://jobs..."
         :Data opportunity-detected
         :Content {:Kind career :FitScore 0.85 :BasisStrengths {public-speaking}}
         :Moment \@m{...}]
```

---

## 4  Common `:Content` Idiom Slots

| Slot | Type | Count | Notes |
|---|---|---|---|
| `:Confidence` | real | 9 | Certainty of the inference. Mandatory on all Depth and Capability percepts. |
| `:Evidence` | list | 8 | Provenance — the reifiers of the percepts that support this inference. |
| `:Text` / `:Summary` / `:Description` | string | 9 | The human-readable payload. |
| `:Topics` / `:Domain` / `:Kind` | list/literal | 8 | What area of life this concerns. |
| `:Sentiment` / `:Valence` / `:Affect` | real/literal | 5 | Emotional charge. |
| `:Source` | literal | 4 | `stated`, `behavioral`, `inferred`, or a channel set. |
| `:SessionId` / `:ThreadId` | string | 4 | Conversation context. |
| `:Severity` / `:FitScore` / `:GapSize` | real | 3 | Magnitude of the situation or gap. |
| `:Sensitivity` | literal | 2 | Governs storage and consent rules. |
| `:Register` | literal | 2 | `ego` or `shadow` — which side of the model. |

The Kojin's signature slot pair is `:Confidence` + `:Evidence`: because so many Kojin percepts are inferences about an interior the mind can never observe directly, every inference must carry its certainty and its provenance.

---

## 5  Actuations

Actuations are how the mind reaches back to the person: questions, prompts, reflections, guidance. Following GIL convention, actuations are sent as ATTEMPT structures and results return as RESULT percepts — the mind learns what it said through perception.

### 5.1  Actuation Table

| # | `:Action` | Description | `:Parameters` idiom |
|---|---|---|---|
| K01 | `say` | Send a conversational message | `{:Text "..." :SessionId "s14" :Tone warm}` |
| K02 | `ask` | Ask the person a question | `{:Text "..." :QuestionId "q-90" :Topic work :Purpose elicit-preference}` |
| K03 | `prompt-journal` | Offer a journaling prompt | `{:Text "..." :PromptId "p-13" :Theme obligation :Depth gentle}` |
| K04 | `reflect` | Mirror back a pattern | `{:Pattern "..." :Evidence {...} :Framing tentative}` |
| K05 | `summarize` | Summarize a period or theme | `{:Scope {\@m{...} \@m{...}} :Theme nil :Length brief}` |
| K06 | `advise-opportunity` | Surface an opportunity | `{:OpportunityId "o-4" :Rationale "..." :Urgency 0.6}` |
| K07 | `warn-threat` | Surface a threat | `{:ThreatId "t-2" :Rationale "..." :Severity 0.7 :Mitigations {...}}` |
| K08 | `suggest-action` | Recommend a concrete step | `{:Action "..." :Basis {strengths preferences} :Effort low}` |
| K09 | `ego-assert` | Write a trait into the ego model | `{:Trait "..." :Confidence 0.8 :Evidence {...}}` |
| K10 | `shadow-assert` | Write a trait into the shadow model | `{:Trait "..." :Confidence 0.55 :Evidence {...}}` |
| K11 | `model-retract` | Remove a trait from either register | `{:Trait "..." :Register shadow :Reason contradicted}` |
| K12 | `preference-record` | Record a preference | `{:Domain food :Object "cilantro" :Valence -0.9 :Source stated}` |
| K13 | `ingest-document` | Request analysis of a shared file | `{:Path "file:///uploads/x.pdf" :Depth full}` |
| K14 | `consent-request` | Ask permission for a data scope | `{:Scope medical :Rationale "..."}` |
| K15 | `schedule-checkin` | Schedule a future conversation | `{:At \@m{...} :Theme "the promotion decision"}` |

### 5.2  Actuation Domains

**Conversation:** `say`, `ask`, `reflect`, `summarize`

**Journaling:** `prompt-journal`

**Guidance:** `advise-opportunity`, `warn-threat`, `suggest-action`

**Modeling:** `ego-assert`, `shadow-assert`, `model-retract`, `preference-record`

**Intake:** `ingest-document`, `consent-request`

**Continuity:** `schedule-checkin`

### 5.3  ATTEMPT Examples

```
[ATTEMPT :Action ask
         :Parameters {:Text "When you imagine having taken the promotion, what do you feel first?"
                      :QuestionId "q-90" :Topic work :Purpose depth-probe}
         :Token ?tok]

[ATTEMPT :Action shadow-assert
         :Parameters {:Trait "resentment of obligation" :Confidence 0.55
                      :Evidence {avoidance-q89 projection-p3 journal-pattern-7}}
         :Token ?tok]

[ATTEMPT :Action warn-threat
         :Parameters {:ThreatId "t-2" :Severity 0.7
                      :Rationale "spending trend exceeds income trend by 12%"
                      :Mitigations {...}}
         :Token ?tok]
```

---

## 6  Cross-Cutting Structure

### 6.1  The Four Tuple Types

```
PERCEPT:  [PERCEPT :Modality Kojin :Channel C :Address A :Data D :Content {:...} :Moment M :Token T]
ATTEMPT:  [ATTEMPT :Action A :Parameters {:...} :Token T :By M]
RESULT:   [RESULT  :Action A :Status S :Reason R :Moment M :Token T]
URGE:     [URGE    :Need N :Source S :Delta D :Moment M :Token T]
```

### 6.2  Actuation → Percept Pairings

| `:Action` | Produces `:Data` |
|---|---|
| `say`, `ask` | `utterance`, `question-answered`, or `question-avoided` (eventual) |
| `prompt-journal` | `journal-entry` (eventual) |
| `reflect` | `utterance` — the person's reaction, sometimes `integration-signal` |
| `advise-opportunity`, `warn-threat`, `suggest-action` | `utterance` and later `life-event` |
| `ego-assert`, `shadow-assert` | `ego-trait`, `shadow-trait` (confirmation) |
| `ingest-document` | `document-shared`, `record-shared` |
| `consent-request` | `consent-change` |
| `schedule-checkin` | `session-event` (future) |

Many Kojin percepts are spontaneous — unprompted messages, unsolicited journal entries, shared documents — arriving without a prior Attempt. Homeostatic signals arrive as URGE tuples: `neglect` (no contact beyond threshold), `model-staleness` (Depth model unrefreshed), `unresolved-threat` (warned threat with no mitigation observed), `consent-gap` (data present without covering consent).

---

## 7  The Jungian Model

The Kojin maintains a two-register model of the person in the Totality.

**Ego register:** the person as they know and present themselves — conscious identity, stated values, owned strengths and weaknesses, the narrative "I." Built primarily from stated material: Dialogue, Journal, Preference (`:Source stated`).

**Shadow register:** the person as they do not (yet) know themselves — disowned traits, avoided topics, projections onto others, gaps between stated and revealed preference, recurring dream figures, charged overreactions. Built from indirect evidence: `question-avoided`, `projection-detected`, `persona-gap`, `journal-pattern`, `dream-report`.

Three invariants govern the model:

1. **Evidence or nothing.** No trait enters either register without an `:Evidence` list pointing at stored percepts. Shadow traits additionally require evidence from at least two independent `:Data` types.
2. **Confidence decays.** Trait `:Confidence` decays over time unless refreshed by new evidence; contradicting evidence decrements it. Below the floor in `[Model]`, the trait is retracted.
3. **The shadow is offered, never imposed.** Shadow traits shape the mind's *questions* (`ask`, `prompt-journal`, `reflect` with `:Framing tentative`), not its *assertions to the person*. Only when an `integration-signal` shows the person acknowledging the trait themselves does it become citable in guidance.

The purpose is practical, not clinical: opportunities scored only against the ego model flatter; threats scored only against the ego model miss self-sabotage. Guidance grounded in both registers fits the whole person.

---

## 8  Architecture

### 8.1  The Kojin as Mediator

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
                          │  KOJIN PSYCHE   │
                          │                 │
                          │  :Modality      │  Kojin (always)
                          │  :Channel       │  Dialogue | Journal |
                          │                 │  Document | Preference |
                          │                 │  Capability | Situation |
                          │                 │  Depth | System
                          │  :Address       │  source URI
                          │  :Data          │  format literal
                          │  :Content       │  idiom {:...}
                          │                 │
                          └────────┬────────┘
                                   │
          ┌────────────────────────┼───────────────────────────┐
          │                        │                           │
    ┌─────┴──────┐          ┌──────┴──────┐           ┌───────┴───────┐
    │  Chat /    │          │  Uploads /  │           │  Consented    │
    │  Journal UI│          │  Documents  │           │  Feeds (mail, │
    │            │          │             │           │  calendar...) │
    └────────────┘          └─────────────┘           └───────────────┘
```

### 8.2  Two-Stage Perception

Raw material (an utterance, a journal entry, a document) produces a first-order percept immediately. The Analyst then runs over accumulated first-order percepts and emits second-order percepts — `journal-pattern`, `preference-revealed`, `strength-observed`, `persona-gap`, `ego-trait`, `shadow-trait`, `opportunity-detected`, `threat-detected` — each citing its first-order evidence. The mind perceives both the person's life and the analysis of it through the same Percept relation.

---

## 9  Deployment File Set

| File | Category | Description |
|---|---|---|
| `kojin.package` | Protocol | Session management, intake, analysis functions, model operations, consent registry |
| `kojin-psyche.theory` | Theory | Three agents: Listener, Analyst, Companion |
| `Kojin.daicho` | Configuration | Person identity, sources, consent scopes, analysis cadence, model floors, urge thresholds |
| `kojin.suites` | Tests | 9 suites: intake, dialogue, journaling, preferences, capabilities, depth model, guidance, consent, full lifecycle |

---

## 10  The Package

`kojin.package` provides:

**Sessions:** `kojin-session-open`, `kojin-session-close`, `kojin-say`, `kojin-ask`.

**Intake:** `kojin-ingest-text`, `kojin-ingest-document`, `kojin-ingest-feed`.

**Analysis:** `kojin-analyze-sentiment`, `kojin-analyze-topics`, `kojin-find-patterns`, `kojin-detect-avoidance`, `kojin-detect-projection`, `kojin-score-fit` (opportunities), `kojin-score-risk` (threats).

**Model:** `kojin-ego-assert`, `kojin-shadow-assert`, `kojin-model-retract`, `kojin-model-decay`, `kojin-preference-record`, `kojin-model-snapshot`.

**Consent:** `kojin-consent-grant`, `kojin-consent-revoke`, `kojin-consent-check` — every intake and analysis function calls `kojin-consent-check` before touching data in a scoped category.

**Dispatch:** `kojin-dispatch` maps any Kojin `:Action` literal to the corresponding function.

---

## 11  The Theory

`kojin-psyche.theory` defines three agents:

**Listener** — reactive. Receives raw material from the person (messages, journal entries, uploads), wraps each as a first-order PERCEPT, and tells the Perceiver.

**Analyst** — proactive, slow interval (default 1440 Moments — daily). Runs the analysis functions over accumulated first-order percepts, emits second-order PERCEPTs (patterns, preferences, capabilities, Depth traits, opportunities, threats), applies model decay, and monitors homeostatic conditions, emitting URGE tuples for `neglect`, `model-staleness`, `unresolved-threat`, and `consent-gap`.

**Companion** — reactive. Receives ATTEMPTs from the Executor, dispatches through `kojin-dispatch`, wraps the outcome as a RESULT tuple, and tells the Perceiver.

---

## 12  Configuration

`Kojin.daicho` contains six sections:

| Section | Contents |
|---|---|
| `[Psyche]` | Agent URL, port, delay, modality literal |
| `[Person]` | Person identifier, display name, locale, timezone |
| `[Sources]` | Chat/journal UI endpoint, upload directory, consented feed URIs |
| `[Consent]` | Scoped categories (general, financial, medical, relational, dreams) and grant status |
| `[Model]` | Confidence floor, decay rate, shadow evidence minimum, analysis cadence |
| `[Urges]` | Thresholds for neglect, model-staleness, unresolved-threat, consent-gap |

---

## 13  Test Suites

`kojin.suites` contains 9 test suites:

| Suite | Tests |
|---|---|
| `Kojin.intake` | Text, document, and feed ingestion; first-order percept formation |
| `Kojin.dialogue` | Session lifecycle, say/ask round-trips, avoidance detection |
| `Kojin.journaling` | Prompting, entry ingestion, pattern detection across entries |
| `Kojin.preferences` | Stated vs revealed recording, conflict surfacing |
| `Kojin.capabilities` | Strength/weakness inference with evidence chains |
| `Kojin.depth-model` | Ego/shadow assert, evidence requirements, decay, retraction, integration |
| `Kojin.guidance` | Opportunity fit scoring, threat risk scoring, advise/warn dispatch |
| `Kojin.consent` | Grant, revoke, scope enforcement on every intake path |
| `Kojin.full-lifecycle` | End-to-end: open, converse, journal, ingest, analyze, model, guide, close |

---

## 14  Privacy and Consent

The Kojin's world is a person; the ethics are therefore part of the specification, not an afterthought.

**Consent is scoped and revocable.** Data categories in `[Consent]` gate every intake and analysis path. Revocation stops future intake in that scope and marks existing scoped percepts inert.

**The person can see the model.** `kojin-model-snapshot` renders both registers, with evidence, on request. Nothing about the person is withheld from the person — though shadow material is *presented* per invariant 3 of §7 (offered through questions, not pronounced).

**The token is the person's.** The `:Token` on every PERCEPT, RESULT, and URGE tuple is the person's credential, registered with the GIL Registrar at startup. One Kojin, one human.

---

## 15  Domain Coverage

| Domain | `:Channel` | Percept `:Data` values | Actuations |
|---|---|---|---|
| Conversation | `Dialogue` | `utterance`, `question-answered`, `question-avoided` | `say`, `ask`, `reflect`, `summarize` |
| Reflection | `Journal` | `journal-entry`, `journal-pattern`, `dream-report` | `prompt-journal` |
| Shared Material | `Document` | `document-shared`, `message-shared`, `record-shared` | `ingest-document` |
| Wants | `Preference` | `preference-signal`, `preference-revealed`, `value-signal` | `preference-record` |
| Abilities | `Capability` | `strength-observed`, `weakness-observed`, `growth-signal` | — (inferred) |
| Circumstances | `Situation` | `opportunity-detected`, `threat-detected`, `life-event` | `advise-opportunity`, `warn-threat`, `suggest-action` |
| Interior | `Depth` | `ego-trait`, `shadow-trait`, `projection-detected`, `persona-gap`, `integration-signal` | `ego-assert`, `shadow-assert`, `model-retract` |
| Governance | `System` | `consent-change`, `session-event`, `connection-status` | `consent-request`, `schedule-checkin` |

---

## 16  Etymology

**Kojin** (Japanese 個人, *kojin*) — a compound of 個 (*ko*), "individual, the discrete unit, the counter for single distinct things," and 人 (*jin*), "person, human being." The word means the individual person — the private self as distinct from the group, the public, the role. Japanese law and philosophy use *kojin* precisely where English uses "the individual": 個人情報 (*kojin jōhō*) is "personal data," and 個人主義 (*kojin shugi*) is "individualism."

The compound is exact for this Psyche twice over. First, 個 insists on *one*: the Kojin interfaces with a single human being, one world per mind, one token per person. Second, *kojin jōhō* — personal data — is literally what flows through it.

Jung's own terms echo in the design: the *ego* is the "I" the person knows; the *shadow* (*Schatten*) is, in Jung's phrase, "the thing a person has no wish to be." The Kojin holds both registers so that guidance addresses the whole individual — the 個人 — and not merely the persona.

Where Aether names the medium (propagation), Expanse names the territory (vastness), and Eidos names the act of seeing (knowing), Kojin names the subject itself — the one person who is this Psyche's entire world.

**Shudan** (集団, "the group") is the sibling Psyche: the same architecture turned toward an organization-as-world. Kojin is the individual; Shudan is the collective. Together they cover the two forms of human world a GIL mind can accompany.
