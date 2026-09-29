---
template:
  id: http://example.com/method/component-view
  name: "Component Analysis"
  rank: 0
  expose:
    - kind: navigation
      match:
        anyTypeOf:
          - http://example.com/method/vocabulary#Assembly
          - http://example.com/method/vocabulary#Subsystem
          - http://example.com/method/vocabulary#Part
  params:
    - id: member
      type: iri
      defaultValue: ${context.member}
      required: true
---
# Component Analysis

This view is offered by the tool whenever a component is opened, and it takes its subject
from the component you clicked rather than from the page. The dashboard's impact section
has to name a seed in its query; this one is handed the seed.

## What it weighs

```table
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?Part ?Mass ?PowerDraw
WHERE {
  VALUES ?subject { <${member}> }
  ?part alan:isDirectlyContainedBy* ?subject .
  OPTIONAL { ?part alan:mass ?Mass }
  OPTIONAL { ?part alan:powerDraw ?PowerDraw }
  FILTER (BOUND(?Mass) || BOUND(?PowerDraw))
  BIND(REPLACE(STR(?part), "^.*[#/]", "") AS ?Part)
}
ORDER BY DESC(?Mass)
```

## What it is promised, and what that leaves

```table
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?BudgetMass ?ActualMass ?Margin ?BudgetPower ?Source
WHERE {
  VALUES ?subject { <${member}> }
  ?allocation alan:allocatedTo ?subject ;
              alan:allocatedMass ?BudgetMass .
  OPTIONAL { ?allocation alan:allocatedPower ?BudgetPower }
  OPTIONAL { ?allocation alan:sourceDocument ?Source }
  {
    SELECT ?subject (SUM(?m) AS ?ActualMass)
    WHERE {
      ?leaf alan:isDirectlyContainedBy+ ?subject ;
            alan:mass ?m .
    }
    GROUP BY ?subject
  }
  BIND(?BudgetMass - ?ActualMass AS ?Margin)
}
```

An empty table here is itself a finding: this component holds no allocation, so nothing
constrains what it weighs.

## What a change to it would reach

```graph
---
layout: { mode: dag, dag: { rankDir: LR }, fit: true }
group: { byPredicate: true }
---
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

CONSTRUCT {
  ?owner     alan:hasInterface ?interface .
  ?interface alan:connectedTo  ?farEnd .
  ?farEnd    alan:interfaceOf  ?farComponent .
  ?requirement alan:constrains ?interface .
  ?mode        alan:exercises  ?interface .
}
WHERE {
  VALUES ?subject { <${member}> }
  ?owner alan:isDirectlyContainedBy* ?subject .
  ?interface alan:interfaceOf ?owner .
  OPTIONAL { ?requirement alan:constrains ?interface }
  OPTIONAL { ?mode alan:exercises ?interface }
  OPTIONAL {
    { ?connection oml:hasSource ?interface ; oml:hasTarget ?farEnd }
    UNION
    { ?connection oml:hasTarget ?interface ; oml:hasSource ?farEnd }
    OPTIONAL { ?farEnd alan:interfaceOf ?farComponent }
  }
}
```

Reachability is not impact. Everything impacted should appear here; not everything here is
impacted. The graph narrows where an engineer has to look, and stops there.
