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

# The Shudan

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
14. Authorization and Access
15. Domain Coverage
16. Etymology

---

## 1  Overview

The Shudan is a Psyche: the interface between a GIL mind and a single organization as world. It mediates two flows — percepts traveling inward from the organization's expressed life to the mind's mechanisms, and actuations traveling outward from the mind's mechanisms back to the organization. Where the Kojin interfaces with one human being, the Shudan interfaces with a collective — a company, team, nonprofit, or institution — treated as a subject with its own preferences, strengths, weaknesses, opportunities, threats, ego, and shadow.

The organization-as-world speaks in many voices at once: meetings, messages, documents, metrics, decisions, departures. The Shudan collects this organizational data — always within authorized scope — and helps the mind build, refine, and act on a model of the organization. That model has two registers: the observable operating record (structure, capabilities, preferences, performance, environment) and the depth structure (a working model of the organization's Jungian ego — its official identity, mission, and self-narrative — and its Jungian shadow — the unacknowledged patterns, taboo topics, values-behavior gaps, and scapegoating that the official narrative disowns).

The Shudan serves four functions: **learn** the organization's preferences, strengths, and weaknesses; **guide** the organization toward opportunities and away from threats; **journal** with the organization through facilitated retrospectives and analysis of shared information; and **model** the organization's ego and shadow so guidance addresses how the organization actually behaves, not just what its mission statement says.

The Kojin and the Shudan share one architecture at different scales. Where the Kojin's unit of dialogue is the utterance, the Shudan's is the meeting and the thread. Where the Kojin's journal is a diary, the Shudan's is the retrospective. Where the Kojin infers a shadow from one person's avoidances, the Shudan infers it from what an entire organization systematically cannot discuss.

All percepts from the Shudan carry `:Modality Shudan`. The `:Channel` slot selects the domain of organizational life. The `:Data` slot names the parsable structure type. The `:Content` slot carries the idiom.

---

## 2  The Percept Relation

The Percept relation from `totality.theory` is canonical. It does not change for the Shudan.

```
(relation Percept
  :M                 ; lexified monad
  :Modality          ; which Psyche — Shudan for this device
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

| Slot | Meaning | Shudan Examples |
|---|---|---|
| `:Modality` | Which Psyche this came through. The Psyche is the sense organ. | `Shudan` (always) |
| `:Channel` | Sub-channel within the Psyche. Domain of organizational life. | `Discourse`, `Record`, `Structure`, `Preference`, `Capability`, `Environment`, `Depth`, `System` |
| `:Address` | Source URI of the data. Where it came from. | `"slack://acme/eng/msg-4471"`, `"shudan://acme/retro-q2"`, `"file:///uploads/board-deck.pdf"`, `nil` |
| `:Data` | Data format or parsable structure type. What the `:Content` contains. | `message-observed`, `meeting-summary`, `retro-entry`, `document-shared`, `metric-reading`, `org-change`, `strength-observed`, `opportunity-detected`, `ego-trait`, `shadow-trait` |
| `:Content` | Idiom `{:Slot value ...}` with the actual structured data. | `{:Summary "..." :Participants 8 :Decisions {...} :Sentiment -0.2 :Topics {roadmap hiring}}` |
| `:Moment` | When the percept was received. (Template slot.) | `\@m{2026080800000000}` |

For comparison, the Kojin's channels partition the person (Dialogue, Journal, Capability, Depth). The Shudan's channels partition the organization: what it says internally (Discourse), what it records (Record), how it is shaped (Structure), what it wants (Preference), what it can do (Capability), what surrounds it (Environment), and who it is beneath the mission statement (Depth).

---

## 3  Percepts by `:Channel`

All percepts from the Shudan have `:Modality Shudan`. The `:Channel` identifies the domain, `:Data` identifies the format, `:Address` identifies the source, and `:Content` carries the structured data as an idiom.

### 3.1  Channel `Discourse`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `message-observed` | `"slack://acme/eng/msg-4471"` | `{:Venue "eng" :Sender "u123" :Summary "..." :Sentiment -0.2 :Topics {deadline} :ThreadId "t9"}` |
| `meeting-summary` | `"shudan://acme/meeting-2210"` | `{:Title "Q3 planning" :Participants 8 :Decisions {...} :ActionItems {...} :Unresolved {...} :Sentiment 0.1}` |
| `retro-entry` | `"shudan://acme/retro-q2"` | `{:Text "..." :Contributor anonymous :Category went-wrong :Topics {launch} :Prompted yes :PromptId "p-3"}` |
| `topic-avoided` | nil | `{:Topic "attrition in support" :Venues {all-hands leadership} :AvoidanceType silence :Occurrences 4}` |

### 3.2  Channel `Record`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `document-shared` | `"file:///uploads/board-deck.pdf"` | `{:ContentType "application/pdf" :DocumentType board-deck :Summary "..." :Entities {...} :Sensitivity high}` |
| `metric-reading` | `"https://api.metrics..."` | `{:Metric "churn" :Value 0.031 :Unit "monthly" :Trend rising :Window \@m{90}}` |
| `decision-recorded` | `"shudan://acme/decision-77"` | `{:Decision "..." :DecidedBy leadership :Rationale "..." :Reversible yes}` |
| `policy-stated` | nil | `{:Policy "..." :Domain hiring :StatedValue "we hire for potential"}` |

### 3.3  Channel `Structure`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `org-change` | nil | `{:ChangeType departure :Unit "support" :Role senior :Voluntary yes :TenureMonths 26}` |
| `role-defined` | nil | `{:Role "..." :Unit "eng" :ReportsTo "..." :Open yes}` |
| `unit-state` | nil | `{:Unit "support" :Headcount 12 :OpenRoles 3 :AttritionRate 0.28 :Window \@m{365}}` |

### 3.4  Channel `Preference`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `preference-signal` | nil | `{:Domain strategy :Object "enterprise segment" :Valence 0.8 :Source stated :Confidence 0.95}` |
| `preference-revealed` | nil | `{:Domain strategy :Object "enterprise segment" :Valence -0.2 :Source behavioral :Evidence {resource-allocation-q2} :Confidence 0.6}` |
| `value-signal` | nil | `{:Value candor :Rank 2 :Source stated :Evidence {handbook-p4}}` |

Stated preferences come from mission statements, strategy documents, and leadership messaging; revealed preferences come from budgets, calendars, promotions, and what actually ships. Their disagreement is itself a Depth percept (§3.7).

### 3.5  Channel `Capability`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `strength-observed` | nil | `{:Competence "rapid prototyping" :Evidence {...} :Confidence 0.8 :Units {eng design}}` |
| `weakness-observed` | nil | `{:Competence "post-launch support" :Evidence {...} :Confidence 0.75 :Impact high}` |
| `growth-signal` | nil | `{:Competence "sales enablement" :Direction improving :Delta 0.1 :Window \@m{180}}` |

### 3.6  Channel `Environment`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `opportunity-detected` | `"https://news..."` | `{:Kind market :Description "..." :FitScore 0.8 :Window \@m{...} :BasisStrengths {...} :BasisPreferences {...}}` |
| `threat-detected` | nil | `{:Kind competitive :Description "..." :Severity 0.7 :Horizon near :BasisWeaknesses {...}}` |
| `environment-event` | `"https://news..."` | `{:EventType regulation :Description "..." :Disruption 0.5 :Domains {data-privacy}}` |

Environment percepts are typically synthesized: the Shudan's Analyst correlates Capability and Preference percepts against external data (often fetched through a sibling Aether Psyche on the same mind).

### 3.7  Channel `Depth`

The depth-psychological channel. Percepts here are *inferences*, always carrying `:Confidence` and `:Evidence`.

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `ego-trait` | nil | `{:Trait "the scrappy disruptor" :Register ego :Confidence 0.85 :Evidence {mission-doc all-hands-msgs} :Stability stable}` |
| `shadow-trait` | nil | `{:Trait "contempt for process" :Register shadow :Confidence 0.6 :Evidence {topic-avoided-4 gap-2 scapegoat-1} :Stability emerging}` |
| `scapegoat-detected` | nil | `{:Target "support team" :Attribution "slow" :ShadowCandidate "underinvestment the org won't own" :Confidence 0.55}` |
| `values-gap` | nil | `{:Stated "we hire for potential" :Revealed "pedigree-weighted hiring" :GapSize 0.6 :Domain hiring}` |
| `integration-signal` | nil | `{:Trait "contempt for process" :Direction acknowledged :Evidence {retro-q2-entries decision-81}}` |

### 3.8  Channel `System`

| `:Data` | `:Address` | `:Content` (idiom) |
|---|---|---|
| `authorization-change` | nil | `{:Scope hr-records :Granted no :GrantedBy "coo" :Moment \@m{...}}` |
| `session-event` | `"shudan://acme/retro-q2"` | `{:EventType opened :SessionId "retro-q2" :SessionType retrospective}` |
| `connection-status` | nil | `{:Source slack :Status connected :Error nil}` |

### 3.9  Percept Examples

```
[Percept :Modality Shudan  :Channel Discourse  :Address "shudan://acme/meeting-2210"
         :Data meeting-summary
         :Content {:Title "Q3 planning" :Participants 8 :Decisions {...}
                   :Unresolved {support-staffing} :Sentiment 0.1}
         :Moment \@m{...}]

[Percept :Modality Shudan  :Channel Discourse  :Address nil
         :Data topic-avoided
         :Content {:Topic "attrition in support" :Venues {all-hands leadership}
                   :AvoidanceType silence :Occurrences 4}
         :Moment \@m{...}]

[Percept :Modality Shudan  :Channel Depth  :Address nil
         :Data values-gap
         :Content {:Stated "we hire for potential" :Revealed "pedigree-weighted hiring"
                   :GapSize 0.6 :Domain hiring}
         :Moment \@m{...}]

[Percept :Modality Shudan  :Channel Environment  :Address "https://news..."
         :Data threat-detected
         :Content {:Kind competitive :Severity 0.7 :Horizon near
                   :BasisWeaknesses {post-launch-support}}
         :Moment \@m{...}]
```

---

## 4  Common `:Content` Idiom Slots

| Slot | Type | Count | Notes |
|---|---|---|---|
| `:Confidence` | real | 9 | Certainty of the inference. Mandatory on all Depth and Capability percepts. |
| `:Evidence` | list | 9 | Provenance — the reifiers of the percepts that support this inference. |
| `:Summary` / `:Text` / `:Description` | string | 9 | The human-readable payload. |
| `:Topics` / `:Domain` / `:Kind` | list/literal | 8 | What area of organizational life this concerns. |
| `:Unit` / `:Units` / `:Venue` | literal/list | 6 | Which part of the organization. |
| `:Sentiment` / `:Valence` | real | 5 | Emotional or evaluative charge. |
| `:Source` | literal | 4 | `stated`, `behavioral`, `inferred`, or a channel set. |
| `:Severity` / `:FitScore` / `:GapSize` | real | 3 | Magnitude of the situation or gap. |
| `:Sensitivity` | literal | 2 | Governs storage and authorization rules. |
| `:Register` | literal | 2 | `ego` or `shadow` — which side of the model. |

As in the Kojin, the signature pair is `:Confidence` + `:Evidence`. The Shudan adds `:Unit`/`:Venue` throughout: an organization's percepts are always localized to some part of the collective body.

---

## 5  Actuations

Actuations are how the mind reaches back to the organization: briefings, facilitation, prompts, guidance. Following GIL convention, actuations are sent as ATTEMPT structures and results return as RESULT percepts.

### 5.1  Actuation Table

| # | `:Action` | Description | `:Parameters` idiom |
|---|---|---|---|
| S01 | `brief` | Deliver a briefing to a venue | `{:Venue leadership :Summary "..." :Detail {...} :Tone neutral}` |
| S02 | `ask` | Pose a question to a venue or role | `{:Text "..." :QuestionId "q-31" :Venue leadership :Purpose elicit-preference}` |
| S03 | `prompt-retro` | Open or prompt a retrospective | `{:SessionId "retro-q3" :Prompts {...} :Anonymous yes :Theme launch}` |
| S04 | `reflect` | Mirror back an organizational pattern | `{:Pattern "..." :Evidence {...} :Venue leadership :Framing tentative}` |
| S05 | `summarize` | Summarize a period, unit, or theme | `{:Scope {\@m{...} \@m{...}} :Unit nil :Theme nil :Length brief}` |
| S06 | `advise-opportunity` | Surface an opportunity | `{:OpportunityId "o-9" :Rationale "..." :Urgency 0.6 :Venue leadership}` |
| S07 | `warn-threat` | Surface a threat | `{:ThreatId "t-5" :Rationale "..." :Severity 0.7 :Mitigations {...} :Venue leadership}` |
| S08 | `suggest-action` | Recommend a concrete step | `{:Action "..." :Basis {strengths preferences} :Owner "coo" :Effort medium}` |
| S09 | `ego-assert` | Write a trait into the ego model | `{:Trait "..." :Confidence 0.85 :Evidence {...}}` |
| S10 | `shadow-assert` | Write a trait into the shadow model | `{:Trait "..." :Confidence 0.6 :Evidence {...}}` |
| S11 | `model-retract` | Remove a trait from either register | `{:Trait "..." :Register shadow :Reason contradicted}` |
| S12 | `preference-record` | Record an organizational preference | `{:Domain strategy :Object "enterprise segment" :Valence 0.8 :Source stated}` |
| S13 | `ingest-document` | Request analysis of a shared file | `{:Path "file:///uploads/x.pdf" :Depth full}` |
| S14 | `authorization-request` | Ask permission for a data scope | `{:Scope hr-records :Rationale "..." :Approver "coo"}` |
| S15 | `schedule-review` | Schedule a future review session | `{:At \@m{...} :SessionType retrospective :Theme "support staffing"}` |

### 5.2  Actuation Domains

**Communication:** `brief`, `ask`, `reflect`, `summarize`

**Facilitation:** `prompt-retro`

**Guidance:** `advise-opportunity`, `warn-threat`, `suggest-action`

**Modeling:** `ego-assert`, `shadow-assert`, `model-retract`, `preference-record`

**Intake:** `ingest-document`, `authorization-request`

**Continuity:** `schedule-review`

### 5.3  ATTEMPT Examples

```
[ATTEMPT :Action prompt-retro
         :Parameters {:SessionId "retro-q3" :Anonymous yes :Theme launch
                      :Prompts {"What did we avoid saying about the launch?"}}
         :Token ?tok]

[ATTEMPT :Action shadow-assert
         :Parameters {:Trait "contempt for process" :Confidence 0.6
                      :Evidence {topic-avoided-4 values-gap-2 scapegoat-1}}
         :Token ?tok]

[ATTEMPT :Action warn-threat
         :Parameters {:ThreatId "t-5" :Severity 0.7 :Venue leadership
                      :Rationale "competitor targeting churned support-heavy accounts"
                      :Mitigations {...}}
         :Token ?tok]
```

---

## 6  Cross-Cutting Structure

### 6.1  The Four Tuple Types

```
PERCEPT:  [PERCEPT :Modality Shudan :Channel C :Address A :Data D :Content {:...} :Moment M :Token T]
ATTEMPT:  [ATTEMPT :Action A :Parameters {:...} :Token T :By M]
RESULT:   [RESULT  :Action A :Status S :Reason R :Moment M :Token T]
URGE:     [URGE    :Need N :Source S :Delta D :Moment M :Token T]
```

### 6.2  Actuation → Percept Pairings

| `:Action` | Produces `:Data` |
|---|---|
| `brief`, `ask` | `message-observed`, `meeting-summary` (eventual) |
| `prompt-retro` | `retro-entry` (eventual, plural) |
| `reflect` | `message-observed` — the org's reaction, sometimes `integration-signal` |
| `advise-opportunity`, `warn-threat`, `suggest-action` | `decision-recorded` and later `metric-reading`, `org-change` |
| `ego-assert`, `shadow-assert` | `ego-trait`, `shadow-trait` (confirmation) |
| `ingest-document` | `document-shared` |
| `authorization-request` | `authorization-change` |
| `schedule-review` | `session-event` (future) |

Many Shudan percepts are spontaneous — observed messages, metric ticks, departures, market events — arriving without a prior Attempt. Homeostatic signals arrive as URGE tuples: `blind-spot` (an authorized source silent beyond threshold), `model-staleness` (Depth model unrefreshed), `unresolved-threat` (warned threat with no mitigation decision recorded), `authorization-gap` (data present without covering authorization).

---

## 7  The Jungian Model

The Shudan maintains a two-register model of the organization in the Totality, transposing Jung from the individual to the collective.

**Ego register:** the organization as it knows and presents itself — mission, stated values, official strategy, the brand narrative, the founding story, owned competences. Built primarily from stated material: `policy-stated`, `document-shared` (decks, handbooks), leadership `message-observed`, Preference (`:Source stated`).

**Shadow register:** the organization as it does not (yet) know itself — the systematically undiscussable topics, the gaps between stated values and resource allocation, the units scapegoated for problems the whole body created, the failures retold as external bad luck. Built from indirect evidence: `topic-avoided`, `scapegoat-detected`, `values-gap`, attrition patterns in `org-change`, recurring `Unresolved` items in `meeting-summary`.

Three invariants govern the model, identical in form to the Kojin's:

1. **Evidence or nothing.** No trait enters either register without an `:Evidence` list pointing at stored percepts. Shadow traits additionally require evidence from at least two independent `:Data` types.
2. **Confidence decays.** Trait `:Confidence` decays unless refreshed; contradicting evidence decrements it; below the floor in `[Model]`, the trait is retracted.
3. **The shadow is offered, never imposed.** Shadow traits shape the mind's *questions and facilitation* (`ask`, `prompt-retro`, `reflect` with `:Framing tentative`), not its assertions to the organization. Only when an `integration-signal` shows the organization acknowledging the pattern itself — in a retro, in a recorded decision — does the trait become citable in guidance.

The purpose is practical: opportunities scored only against the ego model produce strategies the organization cannot execute; threats scored only against the ego model miss the ones the organization is inflicting on itself. Guidance grounded in both registers fits the organization that actually exists.

---

## 8  Architecture

### 8.1  The Shudan as Mediator

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
                          │  SHUDAN PSYCHE  │
                          │                 │
                          │  :Modality      │  Shudan (always)
                          │  :Channel       │  Discourse | Record |
                          │                 │  Structure | Preference |
                          │                 │  Capability | Environment |
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
    │  Comms     │          │  Records /  │           │  Metrics /    │
    │  (chat,    │          │  Documents /│           │  HRIS / CRM / │
    │  meetings, │          │  Decisions  │           │  Finance APIs │
    │  retros)   │          │             │           │               │
    └────────────┘          └─────────────┘           └───────────────┘
```

### 8.2  Two-Stage Perception

Raw material (a message, a meeting, a metric, a departure) produces a first-order percept immediately. The Analyst then runs over accumulated first-order percepts and emits second-order percepts — `topic-avoided`, `preference-revealed`, `strength-observed`, `values-gap`, `scapegoat-detected`, `ego-trait`, `shadow-trait`, `opportunity-detected`, `threat-detected` — each citing its first-order evidence. The mind perceives both the organization's life and the analysis of it through the same Percept relation.

---

## 9  Deployment File Set

| File | Category | Description |
|---|---|---|
| `shudan.package` | Protocol | Session management, source intake, analysis functions, model operations, authorization registry |
| `shudan-psyche.theory` | Theory | Three agents: Observer, Analyst, Counselor |
| `Shudan.daicho` | Configuration | Organization identity, sources, authorization scopes, analysis cadence, model floors, urge thresholds |
| `shudan.suites` | Tests | 9 suites: intake, discourse, retrospectives, preferences, capabilities, depth model, guidance, authorization, full lifecycle |

---

## 10  The Package

`shudan.package` provides:

**Sessions:** `shudan-session-open`, `shudan-session-close`, `shudan-brief`, `shudan-ask`, `shudan-retro-open`, `shudan-retro-collect`.

**Intake:** `shudan-ingest-message`, `shudan-ingest-meeting`, `shudan-ingest-document`, `shudan-ingest-metric`, `shudan-ingest-org-event`.

**Analysis:** `shudan-analyze-sentiment`, `shudan-analyze-topics`, `shudan-find-patterns`, `shudan-detect-avoidance`, `shudan-detect-scapegoat`, `shudan-detect-values-gap`, `shudan-score-fit` (opportunities), `shudan-score-risk` (threats).

**Model:** `shudan-ego-assert`, `shudan-shadow-assert`, `shudan-model-retract`, `shudan-model-decay`, `shudan-preference-record`, `shudan-model-snapshot`.

**Authorization:** `shudan-auth-grant`, `shudan-auth-revoke`, `shudan-auth-check` — every intake and analysis function calls `shudan-auth-check` before touching data in a scoped category.

**Dispatch:** `shudan-dispatch` maps any Shudan `:Action` literal to the corresponding function.

---

## 11  The Theory

`shudan-psyche.theory` defines three agents:

**Observer** — reactive and slow-polling. Receives pushed material (messages, meeting records, uploads, org events) and polls metric sources at the intervals in `[Polling]`. Wraps each as a first-order PERCEPT and tells the Perceiver.

**Analyst** — proactive, slow interval (default 1440 Moments — daily; weekly for Depth analysis). Runs the analysis functions over accumulated first-order percepts, emits second-order PERCEPTs (patterns, preferences, capabilities, Depth traits, opportunities, threats), applies model decay, and monitors homeostatic conditions, emitting URGE tuples for `blind-spot`, `model-staleness`, `unresolved-threat`, and `authorization-gap`.

**Counselor** — reactive. Receives ATTEMPTs from the Executor, dispatches through `shudan-dispatch`, wraps the outcome as a RESULT tuple, and tells the Perceiver.

---

## 12  Configuration

`Shudan.daicho` contains seven sections:

| Section | Contents |
|---|---|
| `[Psyche]` | Agent URL, port, delay, modality literal |
| `[Organization]` | Organization identifier, display name, units, venues, locale |
| `[Sources]` | Comms endpoints, document directories, metric API URIs, org-event feed |
| `[Authorization]` | Scoped categories (general, financial, hr-records, board, customer) with approver and grant status |
| `[Model]` | Confidence floor, decay rate, shadow evidence minimum, analysis cadence |
| `[Polling]` | Per-source polling intervals in Moments (0 = push only) |
| `[Urges]` | Thresholds for blind-spot, model-staleness, unresolved-threat, authorization-gap |

---

## 13  Test Suites

`shudan.suites` contains 9 test suites:

| Suite | Tests |
|---|---|
| `Shudan.intake` | Message, meeting, document, metric, and org-event ingestion; first-order percept formation |
| `Shudan.discourse` | Brief/ask round-trips, venue routing, avoidance detection across venues |
| `Shudan.retrospectives` | Retro open, anonymous collection, entry ingestion, pattern detection |
| `Shudan.preferences` | Stated vs revealed recording, conflict surfacing |
| `Shudan.capabilities` | Competence inference with evidence chains, unit attribution |
| `Shudan.depth-model` | Ego/shadow assert, evidence requirements, decay, retraction, integration |
| `Shudan.guidance` | Opportunity fit scoring, threat risk scoring, advise/warn dispatch |
| `Shudan.authorization` | Grant, revoke, scope enforcement on every intake path |
| `Shudan.full-lifecycle` | End-to-end: connect sources, observe, retro, analyze, model, guide, disconnect |

---

## 14  Authorization and Access

The Shudan's world is an organization made of people; the governance is therefore part of the specification.

**Authorization is scoped, approved, and revocable.** Data categories in `[Authorization]` name an approver; every grant is a `System` percept with `:GrantedBy`. Revocation stops future intake in that scope and marks existing scoped percepts inert.

**Individuals are aggregated, not modeled.** The Shudan models the *organization*. Discourse percepts attribute to roles and units, not to profiles of named individuals; retro entries default to `:Contributor anonymous`. One Shudan, one organization — the individual-as-world is the Kojin's domain, and requires that individual's own consent.

**The organization can see the model.** `shudan-model-snapshot` renders both registers, with evidence, to authorized venues. Shadow material is *presented* per invariant 3 of §7 — through facilitation, not pronouncement.

**The token is the organization's.** The `:Token` on every PERCEPT, RESULT, and URGE tuple is the organization's credential, registered with the GIL Registrar at startup.

---

## 15  Domain Coverage

| Domain | `:Channel` | Percept `:Data` values | Actuations |
|---|---|---|---|
| Communication | `Discourse` | `message-observed`, `meeting-summary`, `retro-entry`, `topic-avoided` | `brief`, `ask`, `reflect`, `summarize`, `prompt-retro` |
| Institutional Memory | `Record` | `document-shared`, `metric-reading`, `decision-recorded`, `policy-stated` | `ingest-document` |
| Shape | `Structure` | `org-change`, `role-defined`, `unit-state` | — (observed) |
| Wants | `Preference` | `preference-signal`, `preference-revealed`, `value-signal` | `preference-record` |
| Competences | `Capability` | `strength-observed`, `weakness-observed`, `growth-signal` | — (inferred) |
| Surroundings | `Environment` | `opportunity-detected`, `threat-detected`, `environment-event` | `advise-opportunity`, `warn-threat`, `suggest-action` |
| Interior | `Depth` | `ego-trait`, `shadow-trait`, `scapegoat-detected`, `values-gap`, `integration-signal` | `ego-assert`, `shadow-assert`, `model-retract` |
| Governance | `System` | `authorization-change`, `session-event`, `connection-status` | `authorization-request`, `schedule-review` |

---

## 16  Etymology

**Shudan** (Japanese 集団, *shūdan*) — a compound of 集 (*shū*), "to gather, to collect" (the character depicts birds 隹 settling on a tree 木), and 団 (*dan*), "group, body, association" — the rounded, bounded mass. The word means the group as such: an organized collective, a body of people acting as one. Japanese social science uses *shūdan* precisely where English uses "the group": 集団心理 (*shūdan shinri*) is "group psychology," and 集団主義 (*shūdan shugi*) is "collectivism."

The compound is exact for this Psyche. 集 names the Shudan's perceptual act — gathering the scattered voices, records, and metrics of an organization into one stream of percepts. 団 names its subject — the bounded collective body that the mind models as a single world.

Jung himself made the transposition the Shudan depends on: he wrote of the collective shadow and observed that groups, like individuals, project what they refuse to own. An organization's mission statement is its ego; its undiscussables are its shadow. The Shudan holds both registers so that guidance addresses the whole institution — the 集団 — and not merely its brand.

**Kojin** (個人, "the individual") is the sibling Psyche: the same architecture turned toward a single human being. The two names are a natural Japanese pair — *kojin to shūdan*, the individual and the group — the two poles of human social existence. Where Aether names the medium, Expanse the territory, Eidos the act of seeing, and Kojin the single subject, Shudan names the collective subject — the many who are, to this Psyche, one world.
