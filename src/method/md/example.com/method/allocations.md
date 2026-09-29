---
template:
  id: http://example.com/method/allocations
  name: "Budget Allocations"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Budget Allocations

**Pattern 2 of 7. Answers Q1: what was promised, before anyone asks what was spent.**

An allocation is a promise issued to one component and carved out of one parent promise.
Issuing a budget is a systems engineering decision; recording what a part actually weighs
is measurement. They are different acts by different people, so they are different files.

**How to add one.** Name the component, name the parent allocation it is carved from, and
enter the two figures. The only allocation without a parent is the system-level one.

**Cite the source.** Every figure here is a commitment someone will be held to. The
`Source` column is where the table or datasheet row goes; a budget with no provenance is
an opinion.

**Known conflict, left visible.** `ADCSAllocation` is 0.531 kg, taken verbatim from the
subsystem total row of TrueSightSAT Table 14. The two line items in that same table sum to
0.585 kg. The allocation is recorded as published rather than silently corrected, and the
*Part Budgets* page reports the overrun. Correcting the source here would erase the
finding.

**Open question.** Communications and propulsion have no allocation, because no sourced
figure exists for either. Until one does, the system margin shown on the next page is an
upper bound.

```table-editor
---
columns: { this: { label: "Allocation" } }
orderBy: Component
---
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix oml:  <http://opencaesar.io/oml#> .
@prefix alan: <http://example.com/method/vocabulary#> .
@prefix ui:   <http://example.com/method/ui#> .

alan:AllocationShape
    a sh:NodeShape ;
    sh:targetClass alan:Allocation ;
    sh:property [
        sh:path alan:allocatedTo ;
        sh:name "Component" ;
        sh:class alan:Component ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "An allocation is issued to exactly one component. If two components share a budget, model the shared parent and carve the budget again." ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path alan:derivesFrom ;
        sh:name "Parent Budget" ;
        sh:class alan:Allocation ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path alan:allocatedMass ;
        sh:name "Mass (kg)" ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path alan:allocatedPower ;
        sh:name "Power (W)" ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path alan:sourceDocument ;
        sh:name "Source" ;
        sh:maxCount 1 ;
        sh:order 5 ;
    ] ;
    sh:property [
        sh:path dc:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 6 ;
    ] ;
    sh:sparql [
        sh:message "This allocation is issued to a component that sits inside another component, but it is not carved out of a parent allocation. Name the budget it came from, or the lineage that traces a subsystem figure back to the system budget is broken." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                $this alan:allocatedTo ?c .
                ?c alan:isDirectlyContainedBy ?parent .
                FILTER NOT EXISTS { $this alan:derivesFrom ?any }
            }
        """ ;
    ] ;
    sh:sparql [
        sh:message "The budgets carved out of this allocation already sum to more than it holds. Re-apportion the children or raise this figure - a parent cannot issue more than it was given." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                {
                    SELECT $this (SUM(?childMass) AS ?committed) WHERE {
                        ?child alan:derivesFrom $this .
                        ?child alan:allocatedMass ?childMass .
                    }
                    GROUP BY $this
                }
                $this alan:allocatedMass ?budget .
                FILTER (?committed > ?budget)
            }
        """ ;
    ] ;
    sh:sparql [
        sh:severity sh:Warning ;
        sh:message "This allocation cites no source document. Record the table or datasheet row it came from before someone is held to it." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                FILTER NOT EXISTS { $this alan:sourceDocument ?s }
            }
        """ ;
    ] ;
    .
```
