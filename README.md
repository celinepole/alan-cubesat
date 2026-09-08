# ALAN Monitoring and Assessment System

An OML model of a 3U CubeSat mission that measures artificial light at night (ALAN)
over defined regions and delivers radiance assessments to municipalities,
observatories, astronomers and researchers.

SIE 502 project repository. Built incrementally across the quarter, starting from
the scope statement in Project Deliverable 1.

## Repository layout

```
src/method/oml/example.com/method/vocabulary.oml    the method: terms and rules
src/model/oml/example.com/project/description.oml   the model: this system
```

## Building

Open the repository root in VS Code and run in the integrated terminal:

```bash
oml lint       # syntax and well-formedness
oml validate   # closed-world checks
oml reason     # DL consistency
```

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
| Q1 | `Part`, `Subsystem`, `Assembly`, `Allocation` | `mass`, `powerDraw`, `allocatedMass`, `allocatedPower` | `contains` / `isContainedBy`, `allocatedTo`, `derivesFrom` |
| Q2 | — | — | — |
| Q3 | `Requirement`, `VerificationActivity`, `Evidence` | `requirementLevel`, `verificationMethod`, `verificationStatus` | `refines`, `isVerifiedBy`, `produces` |
| Q4 | `Interface`, `Item`, `DataFlow`, `PowerFlow` | — | `hasInterface` / `interfaceOf`, `carries`, `constrains` |

Q1 is answerable by summing leaf masses under a subsystem and comparing to its
allocation, then walking `derivesFrom` upward to find remaining system margin.
Q3 is answerable by finding requirements with no `isVerifiedBy`. Q4 is answerable by
traversing outward from an interface to everything that points at it.

Q2 is deferred. See below.

## What is deliberately absent

A term nobody asks questions about is overhead. Each of the following was considered
and excluded, with a reason.

**`Stakeholder`.** Deliverable 1 names an audience for each question, but no question
asks anything *about* stakeholders. It enters when a question needs it.

**`UseCase`.** Q4 mentions use cases among the things an interface change touches.
Modelling them meaningfully requires scenarios and operational activities; a `UseCase`
concept with nothing behind it would answer nothing.

**A units vocabulary.** Masses are in kilograms and power in watts, stated in each
property's description rather than carried as typed quantities. The `isq` and `si`
vocabularies enter when the model performs arithmetic that unit checking would protect.

**Q2's energy balance.** This is the most consequential exclusion, so it needs the
fullest reason.

The vocabulary could hold the inputs — solar generation, battery capacity,
depth-of-discharge limit, eclipse fraction, per-mode power draw and data rate. Declaring
them would be straightforward. But answering *"what is the highest sustainable imaging
duty cycle"* is a computation over an orbit: energy in against energy out, battery state
across eclipse, buffer fill against downlink opportunity. That is an analysis layer, not
a graph query.

Declaring the properties without the layer that consumes them would produce exactly the
condition this project set out to expose: a structurally complete model that still cannot
answer the question asked of it. The properties would look like coverage and deliver none.

Q2 is therefore scoped to a later increment, arriving with the parametric material.
The current model still surfaces the underlying problem — see the power violation below.

## Findings already visible

The model was built from two sources that were each internally reviewed. Relating their
shared quantities in one artifact surfaces conflicts that neither review caught.

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

### Four contradictions in the source material

| Quantity | Value A | Value B |
|---|---|---|
| ADCS mass | 0.585 kg (sum of parts, Table 14) | 0.531 kg (total row, Table 14) |
| ADCS power | 2.7 W (Table 14) | 4.5 W (Table 20) |
| Daily data volume | 33.3 GB (§4.1) | 100 GB (§4.1, two lines later) |
| Mission duration | 3 years (§1.1.3 constraints) | 5 years (§5.3 battery cycle count) |

### One absence

There is no on-board computer anywhere — not in the TrueSightSAT mass tables, not in the
SysML block breakdown — yet the power budget charges 1.0 W to "AI Payload / OBC". The
containment tree makes the gap structural rather than a footnote.

These are the conditions Deliverable 1 §4 predicted: individually defensible chapters
whose data-volume, compression, downlink and link-budget assumptions conflict, invisible
until a model artifact captures and relates the shared quantities.

## Known gaps in this increment

**`StructuralSubsystem` is not decomposed.** The ISISPACE 3U structure is 0.304 kg
(242.8 g primary, 304.3 g primary plus secondary), sourced and ready, but the subsystem
has no parts in this increment.

**Roughly 1.7 kg of the 4.0 kg dry mass assumption is unaccounted.** Eleven parts total
2.322 kg. The remainder is structure, harness, thermal control, the missing OBC, and
margin.

**Two requirements are unverified by design.** `SUB_ADCS_MASS_01` and `OP_MIS_010` have
no `isVerifiedBy`. Q3 asks which requirements are supported only by assertion; if every
requirement carried an activity, that query would return nothing and demonstrate nothing.
Note also that `VA_MassProperties` sits at `"Planned"` — so even the verified requirement
is not yet evidenced, which is the distinction `verificationStatus` exists to make.

## Sourcing

Component masses and power figures are from the TrueSightSAT report (Tables 8, 14, 17,
20, 21) and from vendor datasheets via SatCatalog for the ISISPACE VHF/UHF transceiver
(0.075 kg, 4.0 W), the IQ Spacecom XLink transceiver (0.180 kg, 3.5 W), and the ISISPACE
3U structure (0.304 kg).

Three figures are the model owner's apportionment rather than sourced values:

- The 8.0 W payload camera figure is split 4.0 / 4.0 across the two imagers; the report
  gives a payload total, not a per-camera breakdown.
- `XBandTransceiver` is recorded at 1.81 W, the 29 W peak averaged over its 6.25% duty
  cycle, because `powerDraw` is declared orbit-average.
- `IonThrusterSystem` is recorded at 0.4 W, the standby end of its 0.4–19 W range.

## Next increments

1. On-board computer and structural decomposition — both sourced, both currently absent.
2. Q2's analysis layer: energy balance across an orbit, and the duty-cycle solve.
3. `UseCase` and operational scenarios, completing Q4's operational dimension.
4. A bundle, so `oml validate` has SHACL targets and Q3 becomes a closed-world check
   rather than a manual inspection.

---

Author: Celine Polepole · SIE 502, Fall 2026