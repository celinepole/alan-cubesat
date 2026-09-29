---
template:
  id: http://example.com/method/audit
  name: "Conformance and Gap Audit"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Conformance and Gap Audit

Module 4 made five claims: that engineers would create what the patterns expect, that facts
would land in the intended descriptions, that the authoring steps would be used, that the
constraints would be respected, and that projects would reuse the method's templates. This
page tests the first of those, one query per pattern, and it is the reason the method is not
taken on faith.

**What this is not.** These are gaps against the rules the method thought to encode. A
clean run would mean the model satisfies those rules, not that the model is right. Silence
is not coverage.

**Why a query and not the validator.** `oml validate` reports a violation on the instance
being authored, one focus node at a time, while someone is editing. This page counts the
population and shows the shape of what is missing across the whole bundle. The same rule
serves both cadences: prevention while authoring, measurement at review.

## One query per pattern, including the ones that are clean

```chart
---
type: bar
data:
  labels: Pattern
  datasets:
    - label: Open findings
      data: Findings
options:
  plugins:
    title:
      display: true
      text: Findings per pattern
    legend:
      display: false
---
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?Pattern (COUNT(?element) AS ?Findings)
WHERE {
  VALUES ?Pattern {
    "1 Decomposition" "2 Allocations" "3 Part budgets" "4 Interfaces"
    "5 Connections" "6 Operating modes" "7 Requirements" "8 Verification"
  }
  OPTIONAL {

      {
        ?element a alan:Subsystem .
        FILTER NOT EXISTS { ?child alan:isDirectlyContainedBy ?element }
        BIND("1 Decomposition" AS ?p)
      } UNION {
        ?element a alan:Subsystem .
        FILTER NOT EXISTS { ?allocation alan:allocatedTo ?element }
        BIND("2 Allocations" AS ?p)
      } UNION {
        ?element a alan:Part .
        FILTER NOT EXISTS { ?element alan:mass ?mass }
        BIND("3 Part budgets" AS ?p)
      } UNION {
        ?element a alan:Interface .
        FILTER NOT EXISTS { ?c1 oml:hasSource ?element }
        FILTER NOT EXISTS { ?c2 oml:hasTarget ?element }
        BIND("4 Interfaces" AS ?p)
      } UNION {
        ?element a alan:Connection .
        FILTER NOT EXISTS { ?element alan:transfers ?item }
        BIND("5 Connections" AS ?p)
      } UNION {
        ?element a alan:OperatingMode .
        FILTER NOT EXISTS { ?element alan:exercises ?interface }
        BIND("6 Operating modes" AS ?p)
      } UNION {
        ?element a alan:Requirement ; alan:requirementLevel ?lvl .
        FILTER (STR(?lvl) != "Mission")
        FILTER NOT EXISTS { ?element alan:refines ?parent }
        BIND("7 Requirements" AS ?p)
      } UNION {
        ?element a alan:Requirement ; alan:requirementLevel ?lvl2 .
        FILTER (STR(?lvl2) != "Mission")
        FILTER NOT EXISTS { ?element alan:isVerifiedBy ?activity }
        BIND("8 Verification" AS ?p)
      }
    
    FILTER (?p = ?Pattern)
  }
}
GROUP BY ?Pattern
ORDER BY ?Pattern
```

Three patterns report **0**, and that zero is doing real work. It says the query ran, the
population was evaluated, and nothing was found - which is a different statement from a
pattern nobody wrote a query for. The grid is built from a `VALUES` list of all eight
patterns before any counting happens, for the same reason the coverage matrix is built from
a cross product: a view assembled only from what exists can never show what does not.

## The findings themselves

```table
---
orderBy: ["Pattern asc", "Element asc"]
collapseRepeatedCells: true
stylesheet:
  - selector: cell[col === "Pattern" && value]
    target: value
    style:
      font-weight: 600
      color: "#4C51BF"
---
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?Pattern ?Finding ?Element ?Context
WHERE {
  {
    ?element a alan:Subsystem .
    FILTER NOT EXISTS { ?child alan:isDirectlyContainedBy ?element }
    BIND("1 Decomposition" AS ?Pattern)
    BIND("Subsystem carries no parts" AS ?Finding)
  } UNION {
    ?element a alan:Subsystem .
    FILTER NOT EXISTS { ?allocation alan:allocatedTo ?element }
    BIND("2 Allocations" AS ?Pattern)
    BIND("Subsystem holds no budget" AS ?Finding)
  } UNION {
    ?element a alan:Part .
    FILTER NOT EXISTS { ?element alan:mass ?mass }
    BIND("3 Part budgets" AS ?Pattern)
    BIND("No mass figure recorded" AS ?Finding)
  } UNION {
    ?element a alan:Interface .
    FILTER NOT EXISTS { ?c1 oml:hasSource ?element }
    FILTER NOT EXISTS { ?c2 oml:hasTarget ?element }
    BIND("4 Interfaces" AS ?Pattern)
    BIND("No connection touches it" AS ?Finding)
  } UNION {
    ?element a alan:Connection .
    FILTER NOT EXISTS { ?element alan:transfers ?item }
    BIND("5 Connections" AS ?Pattern)
    BIND("Carries no item" AS ?Finding)
  } UNION {
    ?element a alan:OperatingMode .
    FILTER NOT EXISTS { ?element alan:exercises ?interface }
    BIND("6 Operating modes" AS ?Pattern)
    BIND("Exercises no interface" AS ?Finding)
  } UNION {
    ?element a alan:Requirement ;
             alan:requirementLevel ?lvl .
    FILTER (STR(?lvl) != "Mission")
    FILTER NOT EXISTS { ?element alan:refines ?parent }
    BIND("7 Requirements" AS ?Pattern)
    BIND("No parent requirement" AS ?Finding)
  } UNION {
    ?element a alan:Requirement ;
             alan:requirementLevel ?lvl2 .
    FILTER (STR(?lvl2) != "Mission")
    FILTER NOT EXISTS { ?element alan:isVerifiedBy ?activity }
    BIND("8 Verification" AS ?Pattern)
    BIND("Supported only by assertion" AS ?Finding)
  }
  OPTIONAL { ?element dc:description ?Context }
  BIND(REPLACE(STR(?element), "^.*[#/]", "") AS ?Element)
}
```

**How to read a row.** Each one names how many, which one, and what to do next. A finding
with no corrective action is a complaint.

| Finding | What it usually means | Action |
|---|---|---|
| Subsystem carries no parts | The decomposition stopped early | Decompose it, or say in its description why it is a black box |
| Subsystem holds no budget | Nobody apportioned it | Issue an allocation, or record that it is carried in system margin |
| No mass figure recorded | An estimate is missing | Enter the figure, or record why none exists |
| No connection touches it | A boundary to something outside the system, or a missing link | Say so in the description, or add the connection |
| Carries no item | The link is invisible to the power dependency rule | Name the item |
| Exercises no interface | The mode cannot be reached by interface impact analysis | Name the interface, or confirm the mode is passive |
| No parent requirement | The trace to the mission need has a hole | Name the requirement it refines |
| Supported only by assertion | No verification is planned | Plan an activity, or record the acceptance |

**A gap is not always a defect.** Three of these findings are deliberate: the unconnected
VHF interface faces a ground segment outside the boundary, safe mode exercises nothing
because it is a load-shed configuration, and the on-board computer carries no mass because
none is sourced. The right response to each is a sentence in a description, not a number
invented to clear a row.

## Elements whose provenance is unrecorded

```list
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?Element
WHERE {
  {
    ?element a alan:Allocation .
    FILTER NOT EXISTS { ?element alan:sourceDocument ?source }
  } UNION {
    ?element a alan:Requirement .
    FILTER NOT EXISTS { ?element alan:sourceDocument ?source }
  }
  BIND(REPLACE(STR(?element), "^.*[#/]", "") AS ?Element)
}
ORDER BY ?Element
```

Every figure here is one somebody will be held to. An allocation or a requirement with no
`sourceDocument` is an opinion with an identifier, and this list is how the method notices.
