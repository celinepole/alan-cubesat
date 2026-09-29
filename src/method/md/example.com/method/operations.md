---
template:
  id: http://example.com/method/operations
  name: "Operating Modes and Contacts"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Operating Modes and Contacts

**Pattern 5 of 7. Holds the inputs to Q2 (highest sustainable imaging duty cycle) and states plainly what is still missing.**

A mode is a configuration the spacecraft spends part of an orbit in, with a power draw and
a data rate. A contact opportunity is a window in which data can leave. Between them they
carry every input the energy and data balance needs.

**What this page does not do.** It does not compute the balance. Energy in against energy
out across eclipse, buffer fill against downlink opportunity - that is an analysis layer,
and it belongs to Module 5. What the method can enforce here is that the inputs are
complete and internally consistent, which is what the rules below do.

**The duty cycle rule.** Fractions of one orbit cannot sum to more than one orbit. The
reasoner cannot check this: the `Fraction` facet constrains each value to [0,1]
individually, and DL reasoning has no arithmetic to add them up. SPARQL does, so the check
lives here.

**Confidence.** `SafeMode` carries a duty cycle of 0.0 because the source names it as a
state but gives it no time allocation. Zero asserts that safe mode has no nominal
allocation, not that it never occurs. If the sustainable duty cycle question is ever
answered in the affirmative, this is the first figure that has to change.

```table-editor
---
columns: { this: { label: "Mode" } }
orderBy: Duty Cycle
---
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix oml:  <http://opencaesar.io/oml#> .
@prefix alan: <http://example.com/method/vocabulary#> .
@prefix ui:   <http://example.com/method/ui#> .

alan:OperatingModeShape
    a sh:NodeShape ;
    sh:targetClass alan:OperatingMode ;
    sh:property [
        sh:path alan:dutyCycle ;
        sh:name "Duty Cycle" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "A mode with no duty cycle contributes nothing to the orbit-average power figure, which then silently understates the draw." ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path alan:modePowerDraw ;
        sh:name "Power (W)" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path alan:modeDataRate ;
        sh:name "Data Rate (Mb/s)" ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path alan:exercises ;
        sh:name "Exercises" ;
        sh:class alan:Interface ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path dc:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 5 ;
    ] ;
    sh:sparql [
        sh:message "The duty cycles of all operating modes sum to more than one orbit. One of them is overstated, or a mode has been added without taking time from another." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                $this alan:dutyCycle ?own .
                {
                    SELECT (SUM(?d) AS ?total) WHERE {
                        ?mode a alan:OperatingMode ; alan:dutyCycle ?d .
                    }
                }
                FILTER (?total > 1.0)
            }
        """ ;
    ] ;
    sh:sparql [
        sh:message "This mode generates data but exercises no interface, so an interface change can never be traced to it. Name the interface the data leaves by." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                $this alan:modeDataRate ?r .
                FILTER (?r > 0.0)
                FILTER NOT EXISTS { $this alan:exercises ?i }
            }
        """ ;
    ] ;
    .
```

## Contact Opportunities

```table-editor
---
columns: { this: { label: "Contact" } }
---
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix oml:  <http://opencaesar.io/oml#> .
@prefix alan: <http://example.com/method/vocabulary#> .
@prefix ui:   <http://example.com/method/ui#> .

alan:ContactOpportunityShape
    a sh:NodeShape ;
    sh:targetClass alan:ContactOpportunity ;
    sh:rule [
        a sh:SPARQLRule ;
        sh:construct """
            PREFIX alan: <http://example.com/method/vocabulary#>
            PREFIX ui:   <http://example.com/method/ui#>
            CONSTRUCT { $this ui:passCapacity ?formatted }
            WHERE {
                $this alan:contactDuration ?secs ; alan:downlinkRate ?rate .
                BIND(CONCAT(STR(ROUND(?secs * ?rate / 10) / 100), " Gb") AS ?formatted)
            }
        """ ;
    ] ;
    sh:property [
        sh:path alan:contactDuration ;
        sh:name "Duration (s)" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path alan:downlinkRate ;
        sh:name "Rate (Mb/s)" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path ui:passCapacity ;
        sh:name "Capacity per Pass" ;
        dash:readOnly true ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path dc:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    .
```
