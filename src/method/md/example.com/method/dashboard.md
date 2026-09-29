---
template:
  id: http://example.com/method/dashboard
  name: "Analysis Dashboard"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Analysis Dashboard

Module 4 prescribed. This page measures. Every section below states an engineering
question, shows live evidence from the model, and says what the evidence means. The
numbers are queried at render time, so the prose is written about meaning rather than
about today's figures.

**Scope.** These queries run against the project bundle, not one description. A query over
`structure/partbudgets` alone would see the masses and none of the budgets, and would
report perfect compliance by seeing nothing to compare.

---

## Q1 · Which subsystems exceed their allocation?

**Question.** Where is mass being spent against what was promised, and how much system
margin is left to spend?

```chart
---
type: bar
data:
  labels: Component
  datasets:
    - label: Budget (kg)
      data: Budget
    - label: Asserted (kg)
      data: Actual
options:
  plugins:
    title:
      display: true
      text: Mass against allocation
    legend:
      position: bottom
---
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?Component ?Budget ?Actual
WHERE {
  ?allocation alan:allocatedTo ?component ;
              alan:allocatedMass ?Budget .
  {
    SELECT ?component (SUM(?leafMass) AS ?Actual)
    WHERE {
      ?leaf alan:isDirectlyContainedBy+ ?component ;
            alan:mass ?leafMass .
    }
    GROUP BY ?component
  }
  BIND(REPLACE(STR(?component), "^.*[#/]", "") AS ?Component)
}
ORDER BY DESC(?Budget)
```

```table
---
orderBy: ["Margin asc"]
stylesheet:
  - selector: cell[col === "Margin" && Number(value) < 0]
    target: value
    style:
      color: "#ffffff"
      background-color: "#DC2626"
      padding: 2px 10px
      border-radius: 999px
      font-weight: 700
---
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?Component ?Budget ?Actual ?Margin ?UsedPercent
WHERE {
  ?allocation alan:allocatedTo ?component ;
              alan:allocatedMass ?Budget .
  {
    SELECT ?component (SUM(?leafMass) AS ?Actual)
    WHERE {
      ?leaf alan:isDirectlyContainedBy+ ?component ;
            alan:mass ?leafMass .
    }
    GROUP BY ?component
  }
  BIND(?Budget - ?Actual AS ?Margin)
  BIND(ROUND(1000 * ?Actual / ?Budget) / 10 AS ?UsedPercent)
  BIND(REPLACE(STR(?component), "^.*[#/]", "") AS ?Component)
}
ORDER BY ?Margin
```

**Interpretation.** One subsystem is over its allocation and two are exactly at it. The
overrun is the ADCS: its allocation is the subsystem total row of TrueSightSAT Table 14,
and the two line items in the same table sum to more. A 54 g discrepancy inside one table,
found by relating two figures that had each been reviewed separately.

The two subsystems sitting at exactly 100 % are not comfortable either. A budget with zero
margin is a budget that will be broken by the next estimate, and neither has a verification
activity planned against it.

Read the system row for what remains: that margin is the only place the ADCS overrun can be
absorbed from, and roughly 1.37 kg of it is already spoken for by the harness, thermal
control and the on-board computer that the decomposition does not yet carry.

---

## Q1b · Where does the power go?

**Question.** Which subsystems draw the orbit-average power, and does the total fit the
generation figure?

```chart
---
type: doughnut
data:
  labels: Subsystem
  datasets:
    - label: Orbit-average draw (W)
      data: Power
options:
  plugins:
    title:
      display: true
      text: Orbit-average power draw by subsystem
    legend:
      position: right
---
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?Subsystem (SUM(?draw) AS ?Power)
WHERE {
  ?subsystem a alan:Subsystem .
  ?leaf alan:isDirectlyContainedBy+ ?subsystem ;
        alan:powerDraw ?draw .
  FILTER (?draw > 0.0)
  BIND(REPLACE(STR(?subsystem), "^.*[#/]", "") AS ?Subsystem)
}
GROUP BY ?Subsystem
ORDER BY DESC(?Power)
```

**Interpretation.** Communications and payload together account for most of the draw, and
the leaf parts total 21.91 W against 18.44 W of orbit-average generation. That is the
static form of the problem. Its dynamic form - what imaging duty cycle the spacecraft can
actually sustain - is on the Energy Balance page, because it needs arithmetic a query
cannot do.

---

## Q3 · Which requirements are supported only by assertion?

**Question.** Does every requirement trace to a verification activity, and by what method?

```matrix
---
rowColumnLabel: "Requirement / Method"
stylesheet:
  - selector: cell[Number(value) > 0]
    style:
      background-color: "#10B981"
      color: "#ffffff"
  - selector: row[cells.every(c => Number(c.value) === 0)]
    style:
      background-color: "#FEE2E2"
---
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?row ?column (COALESCE(?n, 0) AS ?value)
WHERE {
  ?requirement a alan:Requirement ;
               alan:requirementId ?row ;
               alan:requirementLevel ?level .
  FILTER (STR(?level) != "Mission")
  VALUES ?column { "Test" "Analysis" "Inspection" "Demonstration" }
  OPTIONAL {
    SELECT ?row ?column (COUNT(DISTINCT ?activity) AS ?n)
    WHERE {
      ?req alan:requirementId ?row ;
           alan:isVerifiedBy ?activity .
      ?activity alan:verificationMethod ?column .
    }
    GROUP BY ?row ?column
  }
}
ORDER BY ?row ?column
```

**Interpretation.** The zeros are the finding, and they exist only because the query builds
the full requirement-by-method grid before counting. A matrix drawn from the links that
exist would show four covered requirements and look complete.

The all-zero rows are requirements supported by nothing but their own assertion. Two of
them are the subsystem mass constraints the Q1 chart above shows being broken or exactly
met, which is the least comfortable place to have no planned verification.

Mission-level needs are excluded on purpose. A mission need is validated against
stakeholders rather than verified against the article, so including it would put a
permanent red row in a view whose whole value is that red means something.

---

## Q4 · What does a change to an interface touch?

**Question.** If the telemetry interface changes, which requirements, modes, components and
far-end interfaces are structurally reachable from it?

```graph
---
layout: { mode: force, fit: true }
group: { byPredicate: true }
---
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

CONSTRUCT {
  ?requirement alan:constrains ?interface .
  ?mode        alan:exercises  ?interface .
  ?interface   alan:interfaceOf ?component .
  ?interface   alan:connectedTo ?farEnd .
  ?farEnd      alan:interfaceOf ?farComponent .
}
WHERE {
  VALUES ?interface { <http://example.com/project/interfaces/interfaces#TelemetryIF> }
  OPTIONAL { ?requirement alan:constrains ?interface }
  OPTIONAL { ?mode alan:exercises ?interface }
  OPTIONAL { ?interface alan:interfaceOf ?component }
  OPTIONAL {
    { ?connection oml:hasSource ?interface ; oml:hasTarget ?farEnd }
    UNION
    { ?connection oml:hasTarget ?interface ; oml:hasSource ?farEnd }
    OPTIONAL { ?farEnd alan:interfaceOf ?farComponent }
  }
}
```

**Interpretation.** A change to the telemetry interface reaches two requirements, one
operating mode, the component that owns it and two far-end interfaces with their
components. That is the set an impact assessment has to walk; it is not the set that is
necessarily impacted. Reachability is the model's contribution and judgment is the
engineer's.

Note what the seed is: this template hardcodes one interface because a compose template has
no selection to work from. The Component view, offered automatically when you open any
component, is the version of this analysis that takes its subject from what you clicked.

---

## The mass tree, for reading rather than deciding

```tree
---
containment: [Parent]
containmentDirection: parent
orderBy: ["Component asc"]
---
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?Component ?Kind ?Mass ?Parent
WHERE {
  { ?component a alan:Assembly  . BIND("Assembly"  AS ?Kind) }
  UNION
  { ?component a alan:Subsystem . BIND("Subsystem" AS ?Kind) }
  UNION
  { ?component a alan:Part      . BIND("Part"      AS ?Kind) }
  OPTIONAL { ?component alan:mass ?Mass }
  OPTIONAL {
    ?component alan:isDirectlyContainedBy ?container .
    BIND(REPLACE(STR(?container), "^.*[#/]", "") AS ?Parent)
  }
  BIND(REPLACE(STR(?component), "^.*[#/]", "") AS ?Component)
}
```

A part with no mass is visible here as an empty cell rather than as an absent row, which is
the difference between a zero and a gap. `OnboardComputer` is the one to look for.
