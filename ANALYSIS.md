# ALAN Analysis

Every question this model exists to answer, with its evidence and its finding - or with the
gap that has to close before it can be answered.

[METHOD.md](METHOD.md) is the prescribing half of the methodology: patterns, organization,
authoring, guidance. This is the measuring half. A method that prescribes but never measures
is a method taken on faith.

**Where the evidence lives.** Every figure below is produced by a live query in a
method-owned page, not copied here by hand. The tables and charts are in
[Dashboard](src/model/md/ALAN%20CubeSat/Dashboard.md), the computation in
[Energy Balance](src/model/md/ALAN%20CubeSat/Energy%20Balance.md), the conformance queries
in [Audit](src/model/md/ALAN%20CubeSat/Audit.md). This document carries the meaning, which
is the part that does not change when a leaf mass does.

**Scope.** All analysis pages read `http://example.com/project/bundle`. A query scoped to
one description sees a fraction of the model and reports compliance by having nothing to
compare - the narrow-scope audit page exists to demonstrate exactly that failure.

---

## Q1 · Margin against allocation

**Question.** Which subsystems exceed their mass and power allocations, what system-level
margin remains, and where should it be spent?

**Evidence.** A roll-up over `isDirectlyContainedBy+` summed against each component's
allocation, shown as a chart and a table on the dashboard, and enforced at authoring time by
a SHACL rule on the part-budget page.

| Component | Budget | Asserted | Margin |
|---|---|---|---|
| ADCSSubsystem | 0.531 kg | 0.585 kg | **-0.054 kg** |
| PayloadSubsystem | 0.112 kg | 0.112 kg | 0.000 kg |
| StructuralSubsystem | 0.3043 kg | 0.3043 kg | 0.0000 kg |
| CubesatWhiteBox | 4.0 kg | 2.6263 kg | 1.3737 kg |

**Finding.** **Answered.** One subsystem is over its allocation and two are at exactly
100 %. The ADCS overrun is 54 g, and its cause is in the source: the allocation is the
subsystem total row of TrueSightSAT Table 14 and the two line items in the same table sum to
more. Neither the report review nor the SysML model caught it, because neither related the
two figures in one artifact.

Two subsystems at exactly zero margin are the second finding. A budget with no margin is one
the next estimate breaks, and neither has a verification activity planned against it.

**Action.** Re-apportion the ADCS budget against its line items, or challenge the 0.525 kg
package figure. The 1.37 kg of system margin is the only place the overrun can come from,
and most of it is already owed to harness, thermal control and the on-board computer that
the decomposition does not yet carry.

**Three allocations are missing entirely.** Communications, power and propulsion hold no
budget, so the system margin above is an upper bound rather than a number to spend against.
That is a process gap, not a modelling one: the figures were never apportioned.

---

## Q2 · Highest sustainable imaging duty cycle

**Question.** Can orbital energy generation and downlink capacity support a given imaging
schedule without exceeding depth-of-discharge limits or creating a data backlog?

**Evidence.** A scripted analysis: SPARQL selects the modes, their duty cycles, draws and
data rates, and the contact capacity; Python computes the orbit-average balance and solves
for the duty cycle at which it closes. This is the analysis that could not be a query -
SPARQL can sum, but it cannot solve for the value that makes a balance close.

| | |
|---|---|
| Orbit-average draw | 23.31 W |
| Generation | 18.44 W |
| Margin | **-4.87 W** |
| Sustainable imaging duty cycle, energy-limited | ~63 % |
| Sustainable imaging duty cycle, downlink-limited at 4 passes/day | ~50 % |
| Asserted in the model | 93.75 % |

**Finding.** **Answered conditionally.** The asserted duty cycle is not sustainable on
either constraint, by a wide margin. The computed 23.31 W is the same figure TrueSightSAT
section 5.2 reaches by a different route, where it records a per-orbit deficit and
recommends a reduced imaging duty cycle. The model now derives the report's own conclusion
from the report's own facts, which is the first time the two have been checked against each
other.

The condition is real and worth stating plainly. Which constraint binds - array or ground
segment - depends on how many usable passes a day exist, and that is not a model fact. At
one pass per orbit the downlink never binds; at four passes a day it binds before energy
does. The answer therefore has assumptions printed beside it, and four of them are not in
the model:

| Missing input | Kind of gap | What would close it |
|---|---|---|
| Orbit period and altitude | Vocabulary | An `Orbit` concept; altitude is prose today |
| Passes per day | Vocabulary | A cadence on `ContactOpportunity`, or a `GroundStation` that owns a schedule |
| Battery capacity, depth of discharge | Vocabulary | An `EnergyStore` concept - without it this is an average-power check, not an eclipse-survival check |
| Whether safe mode absorbs unused time | Process | An operational policy, correctly outside the model |

**Action.** Extend the vocabulary with an orbit and an energy store (Module 2 work), or
accept the conditional answer and record the parameters with it. Either way, the imaging
duty cycle in `operations/modes.oml` should not stand at 0.9375 unchallenged.

---

## Q3 · Verification coverage

**Question.** Does every system requirement trace to a verification activity and
identifiable evidence? Which requirements are supported only by assertion?

**Evidence.** A coverage matrix built from the full requirement-by-method cross product,
with `COALESCE(?n, 0)` turning every unmatched pair into an explicit zero. The all-zero rows
are the answer; a matrix drawn from the links that exist could not show them.

**Finding.** **Answered.** Three of six non-mission requirements carry no activity:
`SUB_ADCS_MASS_01`, `SUB_STR_MASS_01` and `OP_MIS_010`. Two of those three are the subsystem
mass constraints that Q1 shows being broken or exactly met, which is the least comfortable
place in the model to have nothing planned.

Coverage is also not verification. All three planned activities sit at `Planned`, so the
covered requirements are covered by intent. `verificationStatus` exists to keep those apart,
and a `Passed` status with no evidence attached is an error rather than a warning for the
same reason.

**Action.** Plan an inspection against the ADCS mass constraint and a test or analysis
against the structural one, or record why each is accepted without verification. Both are
cheap; neither has been done.

**Why this needed a query at all.** The vocabulary already derives the positive half:
`EvidencedRequirement` classifies itself from an `isVerifiedBy` link to an activity that
produces evidence. The complement is not derivable, because under the open world assumption a
missing assertion is not evidence of absence. Only a closed-world question can name the
requirements that have nothing.

---

## Q4 · Interface change impact

**Question.** Which requirements, subsystems and signals are affected by a change to an
interface?

**Evidence.** A `CONSTRUCT` view graph seeded at one interface, following ownership,
connections, the requirements that constrain it and the modes that exercise it. The
dashboard version is seeded at `TelemetryIF`; the Component Analysis view is offered
automatically when any component is opened and takes its seed from what was clicked.

**Finding.** **Answered, with a stated ceiling.** A change to the telemetry interface
reaches two requirements, one operating mode, the owning component and two far-end
interfaces with their components. That is the set an impact assessment must walk.

The ceiling matters: reachability is not impact. Everything impacted is somewhere in the
reachable set, but not everything reachable is impacted. The graph narrows where an engineer
has to look; it does not decide. A query reports what the model says, never what is true of
the spacecraft.

**Action.** None outstanding. This question is answerable today for any interface in the
model.

---

## Gap detection, one query per pattern

The [audit page](src/model/md/ALAN%20CubeSat/Audit.md) runs one absence query for each of
the eight patterns. Nine findings stand open across five of them.

| Pattern | Finding | Element | Deliberate? |
|---|---|---|---|
| 2 Allocations | Subsystem holds no budget | CommunicationSubsystem, PowerSubsystem, PropulsionSubsystem | No - process gap |
| 3 Part budgets | No mass figure recorded | OnboardComputer | Yes - no sourced figure exists |
| 4 Interfaces | No connection touches it | VHFCommandIn | Yes - the far end is the ground segment |
| 6 Operating modes | Exercises no interface | SafeMode | Yes - load-shed, passive by design |
| 8 Verification | Supported only by assertion | SUB_ADCS_MASS_01, SUB_STR_MASS_01, OP_MIS_010 | No - see Q3 |

Three patterns return nothing: decomposition, connections and requirement tracing are clean.
That is a result, not an absence of one, and it is worth saying explicitly because a silent
query looks identical to a query nobody wrote.

A second list reports elements whose provenance is unrecorded - `MNS_1`, `OP_MIS_010` and
`SUB_ADCS_MASS_01` carry no `sourceDocument`. A requirement with no provenance is an opinion
with an identifier.

---

## Every pattern, including the clean ones

The audit builds its grid from a `VALUES` list of all eight patterns before counting, so a
pattern with nothing to report shows a zero rather than disappearing.

| Pattern | Open findings |
|---|---|
| 1 Decomposition | 0 |
| 2 Allocations | 3 |
| 3 Part budgets | 1 |
| 4 Interfaces | 1 |
| 5 Connections | 0 |
| 6 Operating modes | 1 |
| 7 Requirements | 0 |
| 8 Verification | 3 |

A zero here means the query ran and found nothing. A missing row would mean nobody asked.
Those are different claims, and only one of them is evidence.

---

## Diagnosed gaps

Where a question cannot be answered outright, the useful output is a diagnosis: which layer
the gap belongs to, and therefore who fixes it and how.

| Gap | Source | What to do |
|---|---|---|
| Orbit period and altitude are not model facts | **Vocabulary** | Add an `Orbit` concept with altitude and period (Module 2) |
| Ground contact cadence is not recorded | **Vocabulary** | Add a cadence to `ContactOpportunity`, or a `GroundStation` that owns a pass schedule |
| Battery capacity and depth of discharge are absent | **Vocabulary** | Add an `EnergyStore`; without it Q2 is an average-power check, not an eclipse-survival check |
| Requirements carry no provenance requirement | **Pattern** | ~~Revise the requirements pattern~~ **done** - see below |
| Three subsystems hold no allocation | **Process** | Apportion communications, power and propulsion, or record that they are carried in system margin |
| Three requirements have no planned activity | **Process** | Plan the activities, or record the acceptance |
| `OnboardComputer` has no sourced mass | **Process** | Obtain the figure from a vendor or drop the part |
| Whether safe mode absorbs unused imaging time | **Outside the model** | An operational policy; state it as a parameter, do not encode it |
| Where the remaining 1.37 kg of margin *should* be spent | **Outside the model** | Q1 reports what margin exists; ranking claims on it is engineering judgment |

Stated in the form the diagnosis is meant to take:

> We cannot report the sustainable imaging duty cycle unconditionally, because the
> methodology does not require a `ContactOpportunity` to record how often it occurs.

> We could not report missing provenance as a rule violation, because the requirements
> pattern did not require a requirement to name its source document.

### One of these was closed, and it shows the layers connecting

The second sentence is in the past tense on purpose. The audit's provenance list found three
requirements with no `sourceDocument`, and the only way to report it was as a list, because
no rule asked for it. That is a pattern gap, and Module 4 is where pattern gaps are fixed -
so the requirements pattern now carries a provenance rule, and `oml validate` reports those
three as warnings at authoring time instead of leaving them for a reviewer to notice.

The count moved from one error and five warnings to one error and eight warnings. The model
did not get worse; the method started asking a question it had not been asking.

That is the loop the two modules are meant to form. Prescription without measurement is
taken on faith. Measurement without a route back into the prescription is a report nobody
acts on.

---

## What the analysis layer cannot settle

A query reports what the model says. It does not establish reality, and four kinds of
question stay outside it:

| Question | Why no query settles it |
|---|---|
| Is this decomposition right? | Correctness against the real spacecraft is outside the model |
| Is `MNS_1` well written? | Prose quality is not a model property |
| Was 4.0 kg the right dry-mass assumption? | Rationale belongs in narrative |
| Is the model complete? | A query reports gaps against the rules somebody thought to encode |

The last one is the one to keep in view. Nine findings is not a measure of how wrong the
model is; it is a measure of what these eight queries were built to notice. Consistent,
valid and complete are three different claims, and none of them is correct.

---

## Where each check runs, and how hard it bites

The same rule serves different audiences at different cadences, which is itself a
methodology decision.

| Tier | Audience | When | Example here |
|---|---|---|---|
| Authoring | The engineer editing | Continuously | The container picker offers only components; provenance is now asked for here |
| Page | The team | Every render | The part-budget page shows a negative margin in red |
| Review | Reviewers | Every milestone | The coverage matrix and the audit table |
| Gate | The programme | Every commit | Not yet adopted - see below |

Nothing here blocks a commit, deliberately. Three of the nine findings are intended states of
the model, and a gate that fails on intended states teaches people to bypass gates. The
candidate for promotion to a gate is the mass overrun: it is unambiguous, it is already an
error rather than a warning, and nobody intends it.

---

Author: Celine Polepole - SIE 502, Fall 2026. Project Deliverable 5.
