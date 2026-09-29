# The ALAN Method

What this method prescribes, and why.

The [vocabulary](src/method/oml/example.com/method/vocabulary.oml) says what *can* be
said about an ALAN-class CubeSat. It is deliberately permissive: it will let you put a
mass on an assembly, leave a requirement unverified, or scatter every fact into one file,
and none of that is a logical contradiction. This document is the layer above it. It says
what *should* be created, where it goes, who owns it, and how you know you did it right.

A new engineer opening this repository asks four questions. The method answers each one in
an artifact rather than in someone's memory.

| The question | Answered by | Where it lives |
|---|---|---|
| What should I build? | Eight description patterns | `src/method/md/example.com/method/*.md` |
| Where does it go? | Eight concern-scoped descriptions | `src/model/oml/example.com/project/**` |
| How do I create it? | A table or tree editor per pattern | the same template files |
| How do I know it is right, and why? | Twelve business rules and the prose around them | the same template files |

---

## 1. Patterns

Patterns were taken from the four questions in Deliverable 1, not invented for coverage.
Each one passes the wiki-page test: there is a page to write called "how to add one of
these," and the pattern is that page.

| # | Pattern | Focal types | Answers | Editor |
|---|---|---|---|---|
| 1 | Component decomposition | `Assembly`, `Subsystem`, `Part` | Q1 | tree |
| 2 | Budget allocations | `Allocation` | Q1 | table |
| 3 | Part budgets | `Part` (work), `Assembly`/`Subsystem` (context) | Q1 | tree |
| 4 | Interfaces | `Interface` | Q4 | table |
| 5 | Connections and flows | `Connection`, `Interface` (context) | Q4 | two tables |
| 6 | Operating modes and contacts | `OperatingMode`, `ContactOpportunity` | Q2 | two tables |
| 7 | Requirements | `Requirement` | Q3 | table |
| 8 | Verification coverage | `VerificationActivity`, `Evidence`, `Requirement` | Q3 | two tables |

Three things follow from that list, and each one is a decision worth defending.

**A class appears in more than one pattern.** `Component` is the focal type of pattern 1,
read-only context in pattern 3, and the value of a picker in patterns 2 and 4. The method
intent changes; the class does not.

**A pattern needs more than one shape.** Pattern 3 has two: `ComponentRollupShape` carries
the hierarchy and the computed totals and is `dash:readOnly`, and `PartFigureShape` is the
editable leaf. Showing the tree makes the task intelligible; making it editable would hand
the mass engineer the architect's file.

**No pattern for `Item`, `Evidence` as a standalone page, or the derived traits.** Items
are created in passing on the connections page, because nobody sits down to author an item.
The derived concepts - `EvidencedRequirement`, `ConstrainedInterface`,
`OperationallyCoupledMode`, `ExceedsMassAllocation` - have no pattern by definition: they
are computed, and a pattern for something nobody authors is overhead.

---

## 2. Organization

One concern per description. Not one class per description, and not one file per
subsystem.

| File | Holds | Owner | Changes when |
|---|---|---|---|
| `structure/components.oml` | The decomposition and nothing else | Architect | The architecture changes |
| `structure/allocations.oml` | Budgets issued and their lineage | Mission systems engineer | A budget is re-apportioned |
| `structure/partbudgets.oml` | Asserted leaf mass and power | Mass and power engineer | An estimate matures |
| `interfaces/interfaces.oml` | Boundaries and their owners | Subsystem interface owner | A component declares a boundary |
| `interfaces/connections.oml` | Items and the links that carry them | Interface owners jointly | Two owners agree a link |
| `operations/modes.oml` | Modes, duty cycles, contacts | Operations engineer | The concept of operations changes |
| `requirements/requirements.oml` | Requirement text and refinement | Requirements engineer | A requirement is agreed |
| `requirements/verification.oml` | Activities, evidence, coverage links | V&V engineer | Verification is planned |

**Why these cuts and not others.** The test is authorship, not size. Structure and mass
were split because the architect and the mass engineer change them for unrelated reasons
and on unrelated schedules; in one file, both would baseline at the pace of the slower.
Interfaces and connections were split because declaring a boundary is unilateral and
agreeing a link is bilateral. Requirements and verification were split for the sharpest
reason of the three: planning a test must not require editing the requirement text, or
every V&V edit shows up in the requirements engineer's diff.

**`ref instance` is what makes the split possible.** `DayCameraIMX264` is *declared* in
`components.oml` and *weighed* in `partbudgets.oml`:

```
// structure/components.oml - the architect's file
instance DayCameraIMX264 : alan:Part [
    alan:isDirectlyContainedBy PayloadSubsystem
]

// structure/partbudgets.oml - the mass engineer's file
ref instance components:DayCameraIMX264 [
    alan:mass 0.020
    alan:powerDraw 4.0
]
```

Same identity, facts merged by the tooling, no integration step. The same device carries
the verification coverage links: `requirements/verification.oml` adds `isVerifiedBy` to
requirements it does not own.

**Split for authoring, bundle for analysis.** `bundle.oml` includes all eight. Reasoning
and analysis run against the bundle; people work in the files.

---

## 3. Authoring

The method owns the view; the project owns the model. Every page under
`src/model/md/ALAN CubeSat/` is four lines: the description it writes to, and a `compose`
block naming the template.

````markdown
---
ontology: http://example.com/project/structure/components
---

```compose
template: http://example.com/method/structure
```
````

Everything the engineer reads and every rule that fires comes from the method template.
The disproportion is the point: change a message in the method and all seven project pages
say the new thing without being touched.

Two of the pages add their own prose after the composed template - the part-budget page
states the two live findings, the verification page states the coverage gap. Project-
specific narrative belongs to the project; the pattern belongs to the method.

One page reads wider than it writes, and it is the only one that needs a parameter. The
part-budget page reads the project bundle, because a margin is a comparison between the
parts and the budgets issued to them and those live in two files owned by two people, but
its edits must land only in the mass engineer's file. Its `target` parameter keeps the two
scopes apart. The alternative - importing the allocations into the part-budget description -
would couple the two files the split exists to separate.

Every other page reads and writes the same description, and takes the default
`${context.ontology}`. An earlier draft had one page hosting two editors writing to two
different files; splitting it into the Interfaces and Connections pages was the better
answer anyway, because a page that writes to two files has two owners and belongs to
neither.

---

## 4. Guidance

### Severity is a policy, not a mood

| Severity | Means | Example |
|---|---|---|
| Violation | A fact that makes an analysis wrong | Mass asserted on a composite; a connection that carries nothing |
| Warning | Work that has not happened yet | A part with no mass; a requirement with no activity |

Five warnings stand open in the model today and every one of them is a real finding.
They are not suppressed, because suppressing them would hide the answer to Q3. They are
not errors, because training people to ignore a red count costs more than the count is
worth.

### The rules

Core SHACL constraints - `sh:minCount`, `sh:maxCount`, `sh:class`, `sh:in` - do the
structural work and are not listed here. These are the twelve rules that needed SPARQL,
with the reason each one cannot be a reasoner rule.

| # | Rule | Page | Why not the reasoner |
|---|---|---|---|
| 1 | A component with no description | Structure | Absence is unknown, not false |
| 2 | A `Part` that contains children | Structure | Needs closed-world negation |
| 3 | A child budget with no parent budget | Allocations | Absence is unknown |
| 4 | Child budgets exceeding the parent | Allocations | Aggregation: SWRL has no `SUM` |
| 5 | A budget with no source document | Allocations | Absence is unknown |
| 6 | Part masses exceeding the allocation | Part budgets | Aggregation, over a transitive path |
| 7 | Mass or power on a composite | Part budgets | Closed-world negation |
| 8 | A part with no mass figure | Part budgets | Absence is unknown |
| 9 | An interface no connection touches | Connections | Closed-world negation |
| 10 | Both ends of a connection on one component | Connections | Expressible in SWRL, but the message is the point |
| 11 | Duty cycles summing above one orbit | Operations | Aggregation; the `Fraction` facet only bounds each value |
| 12 | Non-normative requirement wording | Requirements | Regex over a literal |
| 13 | Two requirements sharing an identifier | Requirements | An OML `key` would do it; the toolchain rejected the declaration |
| 14 | A subsystem requirement with no parent | Requirements | Absence is unknown |
| 15 | `Passed` with no evidence | Verification | Closed-world negation |
| 16 | A requirement supported only by assertion | Verification | **This is Q3.** The positive half is derived by the vocabulary; the complement is not derivable at all |

Rule 16 is the clearest statement of why this module exists. `EvidencedRequirement` is a
defined concept: a requirement verified by an activity that produces evidence classifies
itself, with nothing asserted by hand. Ask the opposite question - which requirements have
*no* activity - and open-world reasoning has no answer, because a missing assertion is not
evidence of absence. Only a closed-world check can answer it.

### Messages are guidance

Every message says what to do, not what failed. "The parts under this component already
weigh more than its allocation. Either the estimate has grown and the budget must be
re-apportioned on the Allocations page, or a part is in the wrong place in the tree." A
message that reads `sh:maxCount violated on alan:mass` teaches nobody anything.

### Prevention before correction

`sh:class alan:Interface` on the `exercises` property does two jobs from one clause: it
rejects a bad value, and it stops the picker offering one. Where a shape can make the right
action the easy action, it should.

---

## 5. Derived columns and the `ui:` namespace

`Total Mass`, `Total Power`, `Mass Margin`, `Capacity per Pass` and `Coverage` are computed
by `sh:rule` CONSTRUCT queries at render time and written to no file. They live in
`http://example.com/method/ui#`, a namespace that exists only to carry computed columns and
is deliberately not part of the vocabulary.

Two reasons. A roll-up is a function of its leaves, so storing it creates a second source of
truth that is wrong the moment a leaf changes. And a term nobody authors is not a term the
vocabulary should carry - putting `totalMass` next to `mass` would invite someone to assert
it.

---

## 6. What stays in prose

The test: could a query produce this sentence? If yes, it is not prose.

| In the model | In the prose |
|---|---|
| `ADCSSubsystem` totals 0.585 kg | Why the 0.531 kg allocation was recorded as published rather than corrected |
| `SafeMode` has a duty cycle of 0.0 | That zero means "no nominal allocation", not "never occurs" |
| `XBandTransceiver` draws 1.81 W | That the figure is a peak averaged over a duty cycle, and is softer than the others |
| Three requirements have no activity | Why they are left uncovered in this increment |

Rationale, rejected alternatives, confidence and open questions are in the templates
because they cannot be derived and should not be invented twice.

---

## 7. The method is now a versioned dependency

Eight project pages compose these templates. A change here is not a private edit.

| Change | Impact |
|---|---|
| Better prose or a clearer message | Safe |
| A new optional property | Usually compatible |
| A new `sh:minCount 1` | Existing models fail validation and need migration |
| A new required template parameter | Every call site breaks |
| A changed `dash:composite` path | The tree reshapes itself |

The rule of thumb: adding a warning is a patch, adding a violation is a major version, and
changing a parameter is a breaking change that has to be announced.

---

## 8. What this method does not prescribe

- **How to compute the energy and data balance.** The inputs are patterned; the analysis
  is Module 5 work and is not pretended to exist.
- **A units vocabulary.** Kilograms and watts are stated in the property descriptions. The
  `isq` and `si` vocabularies enter when the model does arithmetic that unit checking would
  protect.
- **Stakeholders, use cases, scenarios.** No question asks anything about them yet.
- **Whether a figure is right.** The method checks that a figure has a source, not that the
  source is correct. That judgment stays with the engineer, which is the point of the last
  line of Module 4: the goal is not to encode all judgment as rules, it is to stop
  depending on undocumented memory.

---

Author: Celine Polepole - SIE 502, Fall 2026. Project Deliverable 4.
