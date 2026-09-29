---
template:
  id: http://example.com/method/part-budgets
  name: "Part Budgets"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
    - id: target
      type: iri
      required: true
---
# Part Budgets

**Pattern 3 of 7. This page is the answer to Q1.**

Enter mass and orbit-average power on leaf parts. The tree above each part is context, not
your work: the hierarchy comes from the *Component Decomposition* page and is read-only
here. The totals are computed live from the leaves, and the margin column compares them
with the budget issued on the *Allocations* page.

**Why leaves only.** A composite that asserts its own mass gives the roll-up two answers
and no way to choose. The rule below rejects it rather than quietly preferring one.

**Reading scope and writing scope are different here, on purpose.** This page reads the
project bundle, because the margin column has to see both the parts and the budgets issued
to them, and those live in two files owned by two people. It writes only to the part-budget
description, named by the `target` parameter. The alternative - importing the allocations
into the part-budget file - would make the mass engineer's file depend on the systems
engineer's, which is the coupling the split exists to avoid.

**Why the totals are not in the model.** `Total Mass`, `Total Power` and `Mass Margin` are
derived here for the UI and are never written to any file. They are not vocabulary terms -
they live in a `ui:` namespace that exists only to carry computed columns. A roll-up is a
function of the leaves, so storing it would create a second source of truth that is stale
the moment a leaf changes. This is also the boundary the reasoner cannot cross: OWL has no
arithmetic and SWRL has no aggregation, so a sum belongs to SPARQL, here or in Module 5.

**Confidence.** Three figures are apportioned rather than sourced: the 8.0 W payload
camera total is split 4.0 / 4.0 across the two imagers, `XBandTransceiver` carries its
29 W peak averaged over a 6.25 % duty cycle, and `IonThrusterSystem` carries the standby
end of a 0.4 - 19 W range. Treat their margins as softer than the rest.

```tree-editor
---
target: ${target}
columns: { this: { label: "Component" } }
stylesheet:
  - selector: cell[col === "Mass Margin" && value && String(value).startsWith("-")]
    target: value
    style:
      padding: 2px 10px
      border-radius: 999px
      font-weight: 700
      color: "#ffffff"
      background-color: "#DC2626"
---
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix oml:  <http://opencaesar.io/oml#> .
@prefix alan: <http://example.com/method/vocabulary#> .
@prefix ui:   <http://example.com/method/ui#> .

alan:ComponentRollupShape
    a sh:NodeShape ;
    sh:targetClass alan:Assembly ;
    sh:targetClass alan:Subsystem ;
    dash:readOnly true ;
    sh:rule [
        sh:order 0 ;
        a sh:SPARQLRule ;
        sh:construct """
            PREFIX alan: <http://example.com/method/vocabulary#>
            PREFIX ui:   <http://example.com/method/ui#>
            CONSTRUCT { $this ui:totalMass ?formatted }
            WHERE {
                SELECT $this (CONCAT(STR(ROUND(SUM(?m) * 10000) / 10000), " kg") AS ?formatted)
                WHERE {
                    ?leaf alan:isDirectlyContainedBy+ $this .
                    ?leaf alan:mass ?m .
                }
                GROUP BY $this
            }
        """ ;
    ] ;
    sh:rule [
        sh:order 1 ;
        a sh:SPARQLRule ;
        sh:construct """
            PREFIX alan: <http://example.com/method/vocabulary#>
            PREFIX ui:   <http://example.com/method/ui#>
            CONSTRUCT { $this ui:totalPower ?formatted }
            WHERE {
                SELECT $this (CONCAT(STR(ROUND(SUM(?p) * 100) / 100), " W") AS ?formatted)
                WHERE {
                    ?leaf alan:isDirectlyContainedBy+ $this .
                    ?leaf alan:powerDraw ?p .
                }
                GROUP BY $this
            }
        """ ;
    ] ;
    sh:rule [
        sh:order 2 ;
        a sh:SPARQLRule ;
        sh:construct """
            PREFIX alan: <http://example.com/method/vocabulary#>
            PREFIX ui:   <http://example.com/method/ui#>
            CONSTRUCT { $this ui:massMargin ?formatted }
            WHERE {
                SELECT $this (CONCAT(STR(ROUND((?budget - ?total) * 10000) / 10000), " kg") AS ?formatted)
                WHERE {
                    {
                        SELECT $this (SUM(?m) AS ?total) WHERE {
                            ?leaf alan:isDirectlyContainedBy+ $this .
                            ?leaf alan:mass ?m .
                        }
                        GROUP BY $this
                    }
                    ?alloc alan:allocatedTo $this ; alan:allocatedMass ?budget .
                }
            }
        """ ;
    ] ;
    sh:property [
        sh:path alan:isDirectlyContainedBy ;
        sh:name "Container" ;
        dash:composite true ;
        sh:order 0 ;
    ] ;
    sh:property [
        sh:path ui:totalMass ;
        sh:name "Total Mass" ;
        dash:readOnly true ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path ui:totalPower ;
        sh:name "Total Power" ;
        dash:readOnly true ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path ui:massMargin ;
        sh:name "Mass Margin" ;
        dash:readOnly true ;
        sh:maxCount 1 ;
        sh:order 5 ;
    ] ;
    sh:sparql [
        sh:message "The parts under this component already weigh more than its allocation. Either the estimate has grown and the budget must be re-apportioned on the Allocations page, or a part is in the wrong place in the tree." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                {
                    SELECT $this (SUM(?m) AS ?total) WHERE {
                        ?leaf alan:isDirectlyContainedBy+ $this .
                        ?leaf alan:mass ?m .
                    }
                    GROUP BY $this
                }
                ?alloc alan:allocatedTo $this ; alan:allocatedMass ?budget .
                FILTER (?total > ?budget)
            }
        """ ;
    ] ;
    sh:sparql [
        sh:message "A component that contains children must not assert its own mass or power draw - composites roll up their leaves. Move the figure onto the leaf it belongs to." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                ?child alan:isDirectlyContainedBy $this .
                { $this alan:mass ?m } UNION { $this alan:powerDraw ?p }
            }
        """ ;
    ] ;
    .

alan:PartFigureShape
    a sh:NodeShape ;
    sh:targetClass alan:Part ;
    sh:property [
        sh:path alan:isDirectlyContainedBy ;
        sh:name "Container" ;
        dash:composite true ;
        dash:readOnly true ;
        sh:order 0 ;
    ] ;
    sh:property [
        sh:path alan:mass ;
        sh:name "Mass" ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path alan:powerDraw ;
        sh:name "Power Draw" ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:sparql [
        sh:severity sh:Warning ;
        sh:message "This part carries no mass figure, so every total above it is an underestimate. Enter the figure, or record in the decomposition why none exists." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                FILTER NOT EXISTS { $this alan:mass ?m }
            }
        """ ;
    ] ;
    .
```
