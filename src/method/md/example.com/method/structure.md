---
template:
  id: http://example.com/method/structure
  name: "Component Decomposition"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Component Decomposition

**Pattern 1 of 7. Answers Q1 (margin against allocation) by establishing the tree that every roll-up walks.**

Declare each component once, say what contains it, and say what it is. Nothing else
belongs here: mass and power figures are entered on the *Part Budgets* page, and budgets
are issued on the *Allocations* page. Three concerns, three files, three owners.

**Why the split.** A decomposition changes when the architecture changes. A mass figure
changes when an estimate matures. If both live in one file, the architect and the mass
engineer collide on every edit, and the file baselines at the pace of the slower of the
two.

**Choosing the kind.** `Assembly`, `Subsystem` and `Part` are disjoint, so the choice is
permanent until someone retypes the instance.

| Kind | Use it when | Consequence |
|---|---|---|
| `Assembly` | The element only groups others; the system root | Mass is a roll-up and must never be asserted |
| `Subsystem` | Budgets are issued and owned at this level | Becomes a focus node for the overrun rule |
| `Part` | Leaf. Nothing below it | The only place `mass` and `powerDraw` may be asserted |

**What is still open.** The structural subsystem is modelled as a frame plus one panel
set, where the SysML breakdown carries eighteen separate 1U panels. One set was chosen
because no per-panel figure exists; splitting it later changes no rule on this page.
`OnboardComputer` is declared with no figures on purpose - it appears in the power budget
and in no mass table, and declaring it makes that gap structural rather than a footnote.

```tree-editor
---
columns: { this: { label: "Component" } }
stylesheet:
  - selector: cell[col === "Kind" && value]
    target: value
    style:
      padding: 2px 10px
      border-radius: 999px
      font-size: 11px
      font-weight: 600
      color: "#ffffff"
      background-color: "#4C51BF"
---
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix oml:  <http://opencaesar.io/oml#> .
@prefix alan: <http://example.com/method/vocabulary#> .
@prefix ui:   <http://example.com/method/ui#> .

alan:ComponentShape
    a sh:NodeShape ;
    sh:targetClass alan:Assembly ;
    sh:targetClass alan:Subsystem ;
    sh:targetClass alan:Part ;
    sh:property [
        sh:path alan:isDirectlyContainedBy ;
        sh:name "Container" ;
        sh:class alan:Component ;
        sh:maxCount 1 ;
        dash:composite true ;
        oml:localReference true ;
        sh:order 0 ;
    ] ;
    sh:property [
        sh:path rdf:type ;
        sh:name "Kind" ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path dc:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path alan:sourceDocument ;
        sh:name "Source" ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:sparql [
        sh:severity sh:Warning ;
        sh:message "This component carries no description. Say what it is, so that a reader who was not in the room can tell whether it is the right component to hang a figure on." ;
        sh:select """
            SELECT $this WHERE {
                FILTER NOT EXISTS { $this <http://purl.org/dc/elements/1.1/description> ?d }
            }
        """ ;
    ] ;
    .

alan:PlacedComponentShape
    a sh:NodeShape ;
    sh:targetClass alan:Subsystem ;
    sh:targetClass alan:Part ;
    sh:property [
        sh:path alan:isDirectlyContainedBy ;
        sh:name "Container" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "Every subsystem and part sits inside exactly one container. An unplaced component is invisible to every roll-up, because the roll-up walks containment." ;
    ] ;
    .

alan:LeafComponentShape
    a sh:NodeShape ;
    sh:targetClass alan:Part ;
    sh:sparql [
        sh:message "This Part contains other components, so it is not a leaf. Either retype it as an Assembly or a Subsystem, or move the children elsewhere - mass may only be asserted on leaves." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                ?child alan:isDirectlyContainedBy $this .
            }
        """ ;
    ] ;
    .
```
