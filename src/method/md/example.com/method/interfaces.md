---
template:
  id: http://example.com/method/interfaces
  name: "Interfaces"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Interfaces

**Pattern 4 of 8. The first half of Q4 (interface change impact): the boundaries a change can travel across.**

An interface is a boundary a component declares and owns. Declaring one is a unilateral
act, which is why interfaces have a file, a page and an owner of their own, separate from
the connections that join them.

**How to add one.** Name it after the component and the direction of flow
(`<Component>PowerIn`, `<Component>DataOut`), pick the component that owns it, and say what
crosses it. One interface, one owner: a boundary two components both claim is a boundary
nobody maintains, which is what the rule below enforces.

**Why this page is not the connections page.** Whether a link exists is a fact about the
link, not about either end. Keeping them apart means an owner can declare a boundary before
anyone has agreed what connects to it - which is the normal order of work - and means the
interface file does not change every time a connection is agreed.

**Open question.** `VHFCommandIn` has no connection and never will inside this boundary:
the other end is the ground segment, which is outside the system. The unconnected-interface
warning on the Connections page flags it every run, deliberately. The day the ground segment
enters the model is the day someone should be reminded.

```table-editor
---
columns: { this: { label: "Interface" } }
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

alan:InterfaceShape
    a sh:NodeShape ;
    sh:targetClass alan:Interface ;
    sh:property [
        sh:path alan:interfaceOf ;
        sh:name "Component" ;
        sh:class alan:Component ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "An interface belongs to exactly one component. If two components both claim it, the boundary is in the wrong place." ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path dc:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    .
```
