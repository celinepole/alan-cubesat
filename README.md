# ALAN Monitoring and Assessment System

An OML model of a 3U CubeSat mission that measures artificial light at night (ALAN)
over defined regions and delivers radiance assessments to municipalities,
observatories, astronomers and researchers.

SIE 502 project repository. Built incrementally across the quarter, starting from
the scope statement in Project Deliverable 1.

## Repository layout

```
src/method/oml/example.com/method/vocabulary.oml   the method: terms, restrictions, rules
src/method/oml/example.com/method/bundle.oml       vocabulary bundle: closes the taxonomy
src/method/md/example.com/method/*.md              the method: patterns, editors, rules, guidance

src/model/oml/example.com/project/
    structure/components.oml                       the decomposition
    structure/allocations.oml                      budgets issued, and their lineage
    structure/partbudgets.oml                      asserted leaf mass and power
    interfaces/interfaces.oml                      boundaries and their owners
    interfaces/connections.oml                     items, and the links that carry them
    operations/modes.oml                           modes, duty cycles, contacts
    requirements/requirements.oml                  requirement text and refinement
    requirements/verification.oml                  activities, evidence, coverage
    bundle.oml                                     description bundle: the reasoning scope

src/model/md/index.md                              the authoring pages, in method order
src/model/md/ALAN CubeSat/*.md                     one thin page per pattern (8)
```

The two bundles carry no terms of their own. They define what "the model" means for a
given reasoning run — see *Reasoning scope and reproducibility* below.

The descriptions are split by concern rather than by class, because a file boundary is an
ownership boundary: the architect changes the decomposition, the mass engineer changes the
figures, and neither should wait for the other. `ref instance` merges the facts back
together, and `bundle.oml` puts all eight into one reasoning scope.

One page reads wider than it writes: the part-budget page reads the bundle, because a
margin is a comparison between two files, and writes only to `structure/partbudgets.oml`.

Above the authoring layer sits an analysis layer: a dashboard, a conformance and gap audit,
and a scripted energy and data balance, all method-owned and invoked by thin project pages.
Every question this model exists to answer now ends in an answer or a diagnosed gap, written
up in [ANALYSIS.md](ANALYSIS.md).

The method is no longer only a vocabulary. `src/method/md/` carries eight description
patterns as SHACL shapes, each one wrapped in an editor and a compose template, with the
rationale beside it. The project pages under `src/model/md/` are four lines each: they name
a description and compose a method template. What the method prescribes, and why each rule
exists, is in [METHOD.md](METHOD.md).

## Building

Requires OML CLI and the OML Code extension at **v0.26.0 or later**. Open the repository
root in VS Code and run from the integrated terminal:

```bash
oml lint       # syntax and well-formedness
oml reason     # DL consistency, writes build/owl
oml reason -e  # explain any inconsistency
oml validate   # SHACL: the method's rules, closed-world
```

`oml reason`, `oml validate` and the analysis pages ask three different questions and none
replaces another. Reasoning asks what follows logically; validation asks whether an instance
matches the shape the method expects; a query asks what the population looks like and where
the holes between valid instances are.

`oml reason` and `oml validate` ask different questions and neither replaces the other.
Reasoning asks what follows logically and what contradicts; validation asks whether the
data matches the shape the method expects. The mass overrun below is invisible to the
first and reported by the second.

To work in the model rather than on it, open `src/model/md/index.md` and start at the
first page.

Asserted facts land in `build/owl/example.com/project/description.ttl`; derived facts
land alongside in `description__entailments.ttl`. The separation matters: everything in
the entailments file was computed, not written.

## The system

The system of interest is a 3U CubeSat carrying a dual-mode optical payload with
onboard neuromorphic processing, in a 505 km sun-synchronous orbit. Within the
boundary sit the payload, ADCS, communications, power, propulsion, and structural
subsystems. Outside it sit the ground segment, the launch vehicle and CubeSat
manufacturers, the observed region, the Sun, and the public website through which
assessments are distributed.

The design baseline is the TrueSightSAT mission design report (University of
Luxembourg, SnT, June 2025), a team report co-authored by the model owner, together
with an existing SysML model of the same system (SIE 458/558, Spring 2026).

## The questions this model exists to answer

These are acceptance criteria, not aspirations. Each names something the SysML model
cannot currently do.

**Q1 — Margin against allocation.** Which subsystems exceed their mass and power
allocations, what system-level margin remains, and where should it be spent?
*The SysML model rolls up totals but cannot identify allocation violations or
responsible subsystems.*

**Q2 — Highest sustainable imaging duty cycle.** Can orbital energy generation and
downlink capacity support a given imaging schedule without exceeding depth-of-discharge
limits or creating a data backlog? *Energy and data span all subsystems and are not
currently modelled.*

**Q3 — Verification coverage.** Does every system requirement trace to a verification
activity and identifiable evidence? Which requirements are supported only by assertion?
*The SysML model has derive and satisfy relationships but no verification relationships
or activities.*

**Q4 — Interface change impact.** Which requirements, subsystems and signals are
affected by a change to an interface? *Interfaces and item types exist, but the model
cannot trace their downstream dependencies.*

## How each question maps to the vocabulary

| Question | Concepts | Properties | Relations |
|---|---|---|---|
| Q1 | `Part`, `Subsystem`, `Assembly`, `Allocation` | `mass`, `powerDraw`, `allocatedMass`, `allocatedPower` | `isDirectlyContainedBy` → `isContainedBy`, `allocatedTo`, `derivesFrom` → `budgetDerivesFrom` |
| Q2 | `OperatingMode`, `ContactOpportunity` | `dutyCycle`, `modePowerDraw`, `modeDataRate`, `contactDuration`, `downlinkRate` | `exercises` |
| Q3 | `Requirement`, `VerificationActivity`, `Evidence`, `EvidencedRequirement`, `EvidencedActivity` | `requirementLevel`, `verificationMethod`, `verificationStatus`, `statement` | `refines` → `tracesTo`, `isVerifiedBy`, `produces`, `hasEvidence` |
| Q4 | `Interface`, `ConstrainedInterface`, `Item`, `DataFlow`, `PowerFlow`, `Connection`, `OperationallyCoupledMode` | — | `hasInterface` / `interfaceOf`, `transfers`, `constrains`, `dependsOnPower`, `affectsMode` |

Arrows read *asserted* → *derived*: you write the direct relation, the reasoner
populates the transitive one. Concepts in the Q3 and Q4 rows written in the same style
(`EvidencedRequirement`, `ConstrainedInterface`, `OperationallyCoupledMode`) are defined
concepts — nothing is ever typed as one by hand.

Q2's row is new this increment. The terms are now present; the computation that consumes
them is not. See *What is deliberately absent*.

## What the reasoner does with this

Nothing below is asserted anywhere in `description.oml`.

**Derived facts.** `isContainedBy` is transitive, so one asserted link from
`DayCameraIMX264` to `PayloadSubsystem` yields containment up to `CubesatWhiteBox`.
`tracesTo` does the same for the three-level requirement chain, so
`SUB_ADCS_MASS_01 tracesTo MNS_1` falls out of two asserted `refines` links.

**Derived classifications.** `VA_MassProperties` produces evidence, so it classifies as
`EvidencedActivity`; `SYS_MASS_01` is verified by it, so it classifies as
`EvidencedRequirement`. `OP_MIS_010` constrains `TelemetryIF`, so that interface
classifies as `ConstrainedInterface`; `DownlinkMode` exercises it, so the mode
classifies as `OperationallyCoupledMode`. Four classifications from two asserted links.

**Rule-derived relations.** `PowerDependency` binds a `Connection` instance and inspects
what it transfers, deriving `dependsOnPower(DayCameraIMX264, BatteryEPS)` from the power
connection. It correctly declines to fire on the data connection, because
`PayloadImageData` is a `DataFlow` rather than a `PowerFlow`. `SafeMode` likewise
declines to classify as `OperationallyCoupledMode`, because it exercises no interface.
The refusals are the evidence that the derivations discriminate.

**What it cannot do, and why that is not a modelling failure.**

Both live findings below are invisible to the reasoner, for two unrelated reasons.

The ADCS mass overrun needs the leaf masses under a subsystem *summed*. SWRL has no
aggregation, and DL reasoning has no arithmetic. `MassOverAllocation` compares a single
asserted mass against a single allocation and is included to make the boundary concrete,
not because it answers Q1. Roll-up totals are a SPARQL concern.

The unverified requirements cannot be found by reasoning at all. Under the open world
assumption, the absence of an `isVerifiedBy` assertion is not evidence that none exists.
`EvidencedRequirement` gives the positive half of Q3; the complement is a closed-world
question and belongs to `oml validate` and SHACL.

## Modelling decisions worth defending

**Concepts are disjoint; aspects overlap.** Vocabulary bundle closure makes sibling
concepts disjoint automatically, so the choice of concept versus aspect *is* the
disjointness decision. Things that must not overlap are concepts —
`Subsystem`/`Assembly`/`Part`, `DataFlow`/`PowerFlow`, and the top-level kinds under
`Element`. Things that must overlap are aspects — `Container` and `Contained`, so that a
`Component` can be both; and the two derived traits `ExceedsMassAllocation` and
`NonNormativeRequirement`, so that an over-budget `Part` is a finding rather than a
contradiction. `BudgetedComponent` was considered and rejected for this reason: as a
defined concept under `Component` it would be made disjoint from `Part`.

**Containment is split in two.** OWL 2 DL forbids a transitive property from being
functional, irreflexive or asymmetric. `isContainedBy` is transitive and carries none of
those; `isDirectlyContainedBy` specializes it and carries all three. The same split
applies to `derivesFrom`/`budgetDerivesFrom` and `refines`/`tracesTo`. Direct structure
is asserted, transitive structure is derived.

**`Connection` is reified; containment is not.** What flows across a link is a fact about
the link, not about either endpoint — and more practically, the `PowerDependency` rule
opens with `Connection(i1,c,i2)` so that `transfers(c,f)` has a connection instance to
bind. Containment carries no facts of its own and is a plain relation.

**Requirement text is a semantic property, not an annotation.** Everything else that was
`alan:description` is now `@dc:description`, outside the logical semantics. Requirement
text stays semantic as `statement`, because the `NonNormativeWording` rule reads it and
an annotation property cannot be a rule predicate.

**One defined concept per parent.** Two defined children of the same concept would be
made disjoint by closure, and a requirement can legitimately satisfy two definitions at
once. Where a second classification was wanted, it is an aspect.

## Reasoning scope and reproducibility

"The reasoner passed" is not a reproducible claim. A citable result names three things:

| | |
|---|---|
| Model version | commit `<2d8cbad047d8301b24b07dd97455a498e317f8a1>` |
| Reasoning scope | `http://example.com/project/bundle#` |
| Reasoning assumptions | unique names assumption **on** (`oml reason`, the default) |

The third matters. Because UNA governs the whole run rather than a single file, the same
files can be consistent under one setting and inconsistent under another. Change any of
the three and the outcome can change.

## What is deliberately absent

A term nobody asks questions about is overhead. Each of the following was considered
and excluded, with a reason.

**`Stakeholder`.** Deliverable 1 names an audience for each question, but no question
asks anything *about* stakeholders. It enters when a question needs it.

**`UseCase`.** Q4 mentions use cases among the things an interface change touches. This
increment addresses the operational leg differently: `OperatingMode` and the `exercises`
relation let the `InterfaceOperationalImpact` rule derive which modes a requirement
change reaches. A full `UseCase` concept still requires scenarios and operational
activities and is not present.

**A units vocabulary.** Masses are in kilograms and power in watts, stated in each
property's description rather than carried as typed quantities. The `isq` and `si`
vocabularies enter when the model performs arithmetic that unit checking would protect.
Converting `mass` to a quantity property would also trade away the `NonNegativeDecimal`
facet, since a quantity property takes a quantity rather than a range — a real trade,
deferred deliberately.

**Q2's energy balance.** This is the most consequential exclusion, so it needs the
fullest reason.

The vocabulary now holds the operational inputs: per-mode power draw and data rate, duty
cycle, contact duration and downlink rate. Declaring them was straightforward. But
answering *"what is the highest sustainable imaging duty cycle"* is a computation over an
orbit: energy in against energy out, battery state across eclipse, buffer fill against
downlink opportunity. That is an analysis layer, not a graph query.

The `Fraction` scalar enforces that any single `dutyCycle` lies in [0,1] — a facet the
reasoner treats as a logical constraint, not a warning. It cannot enforce that the modes
*sum* to 1, for the same reason it cannot sum masses. The terms are in place so that the
parametric layer has something to attach to; the answer is not.

Declaring the properties without the layer that consumes them would produce exactly the
condition this project set out to expose: a structurally complete model that still cannot
answer the question asked of it. This increment splits the difference honestly — the
vocabulary exists, the claim of coverage does not.

## Findings already visible

The model was built from two sources that were each internally reviewed. Relating their
shared quantities in one artifact surfaces conflicts that neither review caught.

As of this increment these are no longer prose. `oml validate` reports the ADCS overrun as
a violation on the part-budget page, and reports the unverified requirements, the
unconnected interface and the part with no mass as warnings. The findings below are what
the tool says, not what the author remembered to write down.

### Two allocation violations

**ADCS mass.** `ADCSPackIADCS200` (0.525 kg) and `SunSensorSet` (0.060 kg) sum to
**0.585 kg** against an allocation of **0.531 kg**. The allocation is taken verbatim from
the TrueSightSAT Table 14 subsystem total row, which does not equal the sum of its own
two line items. A 54 g discrepancy inside a single table.

**System power.** The eleven leaf parts draw **21.91 W** orbit-average against an
allocation of **18.44 W**, the orbit-average solar generation from TrueSightSAT §5.2.
A 3.47 W overrun. The report reaches the same conclusion by a different route in the
same section, recording a −8.0 Wh per-orbit deficit and recommending a reduced imaging
duty cycle. The static form of that problem is visible here; the dynamic form is Q2.

Neither violation makes the model inconsistent, and that is correct. An overrun is an
engineering finding about the design, not a contradiction in the knowledge. The reasoner
is silent on both because both require summation.

**Two routes to orbit-average power.** Summing the eleven leaf parts gives 21.91 W.
Weighting the two active operating modes by duty cycle gives 23.31 W, which matches
Table 20's 27.98 W once its 20% design margin is removed. The 1.40 W gap resolves
exactly: +1.80 W because Table 20 charges ADCS at 4.5 W where Table 14 charges 2.7 W,
and −0.40 W because Table 20 omits the propulsion subsystem. Neither figure falls
within the 18.44 W orbit-average generation.


### Five contradictions in the source material

| Quantity | Value A | Value B |
|---|---|---|
| ADCS mass | 0.585 kg (sum of parts, Table 14) | 0.531 kg (total row, Table 14) |
| ADCS power | 2.7 W (Table 14) | 4.5 W (Table 20) |
| Daily data volume | 33.3 GB (§4.1) | 100 GB (§4.1, two lines later) |
| Mission duration | 3 years (§1.1.3 constraints) | 5 years (§5.3 battery cycle count) |
| Mission duration | 3 years (§1.1.3 constraints) | 5 years (§5.3 battery cycle count) |
| Propulsion power | 0.4 W standby (Table 17) | absent from the power budget (Table 20) |

### One absence

There is no on-board computer anywhere — not in the TrueSightSAT mass tables, not in the
SysML block breakdown — yet the power budget charges 1.0 W to "AI Payload / OBC". The
containment tree makes the gap structural rather than a footnote.

These are the conditions Deliverable 1 §4 predicted: individually defensible chapters
whose data-volume, compression, downlink and link-budget assumptions conflict, invisible
until a model artifact captures and relates the shared quantities.

## Known gaps in this increment

**`StructuralSubsystem` is decomposed, coarsely.** It now carries a frame (0.2428 kg) and
one 1U panel set (0.0615 kg) against an allocation of 0.3043 kg. The SysML breakdown
carries eighteen separate panels; one set is modelled because no per-panel figure exists.
Splitting it later changes no rule.

**Roughly 1.37 kg of the 4.0 kg dry mass assumption is unaccounted.** Thirteen parts now
total 2.6263 kg, the figure the part-budget page computes. The remainder is harness,
thermal control, the OBC and margin. `OnboardComputer` is declared with no figures, so the
validator reports it as an incomplete entry every run rather than leaving the gap to a
footnote.

**Two requirements are unverified by design.** `SUB_ADCS_MASS_01` and `OP_MIS_010` have
no `isVerifiedBy`. Q3 asks which requirements are supported only by assertion; if every
requirement carried an activity, that query would return nothing and demonstrate nothing.
Note also that `VA_MassProperties` sits at `"Planned"` — so even the verified requirement
is not yet evidenced, which is the distinction `verificationStatus` exists to make.

**Keys are still not used.** `Requirement` has a `requirementId` with a pattern facet, but
no `key` declaration; the declaration was written and rejected by the toolchain. A SHACL
rule on the requirements page now catches two requirements sharing an identifier, which is
the closed-world half of what a key would have given. The key remains a candidate for the
next increment, because it would make the duplicate a logical inconsistency rather than a
validation finding.

**Term counts exceed the Deliverable 2 targets.** Deliverable 2 asked for 5–10 relations
and 15–30 instances; this model has 17 and 39. Six relations exist only to satisfy the
DL constraint that a transitive property cannot be functional, three are derived by rule
and never asserted, and the additional instances exist so that the rules and defined
concepts have something to bind to. Every addition traces to a Module 3 construct rather
than to Sierra.

## Sourcing

Component masses and power figures are from the TrueSightSAT report (Tables 8, 14, 17,
20, 21) and from vendor datasheets via SatCatalog for the ISISPACE VHF/UHF transceiver
(0.075 kg, 4.0 W), the IQ Spacecom XLink transceiver (0.180 kg, 3.5 W), and the ISISPACE
3U structure (0.304 kg).

Provenance is carried in the model itself: the `sourceDocument` annotation property
records where a figure came from, and `reviewStatus` records elements known to be
incomplete. Both are annotations rather than semantic properties, because changing a
citation should not change an engineering conclusion.

Three figures are the model owner's apportionment rather than sourced values:

- The 8.0 W payload camera figure is split 4.0 / 4.0 across the two imagers; the report
  gives a payload total, not a per-camera breakdown.
- `XBandTransceiver` is recorded at 1.81 W, the 29 W peak averaged over its 6.25% duty
  cycle, because `powerDraw` is declared orbit-average.
- `IonThrusterSystem` is recorded at 0.4 W, the standby end of its 0.4–19 W range.
- `SafeMode` carries 5.5 W (OBC 1.0 + ADCS 4.5) and a duty cycle of 0.0; the report
  names safe mode as a top-level state but gives it no power line or time allocation.
  Zero duty cycle asserts that safe mode has no nominal allocation, not that it never
  occurs.
---

Author: Celine Polepole · SIE 502, Fall 2026