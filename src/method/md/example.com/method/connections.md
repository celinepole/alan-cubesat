---
template:
  id: http://example.com/method/connections
  name: "Connections"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Connections and Flows

**Pattern 5 of 8. The second half of Q4: what an interface change can travel along.**

A connection joins two interfaces and carries an item. It is a bilateral agreement between
two owners, so it lives in its own file and changes when the two of them agree something -
not when either one declares a boundary.

**How to add one.** Pick source and target interfaces and say what the connection transfers.
The item matters more than it looks: the vocabulary's `PowerDependency` rule fires only on
connections that carry a `PowerFlow`, so a connection with no item is invisible to the
dependency analysis, and the rule below rejects it.

**A tool limit, stated rather than hidden.** A connection is a relation instance, and the
endpoint columns are read-only here because a table editor creates concept instances more
reliably than relation instances. If your OML Code version will not create one, add the
connection in the connections description and use this editor to maintain what it carries.

**The interface list below it is context, not work.** It is read-only and exists for one
reason: to report an interface that no connection touches. That check cannot live on the
Interfaces page, because the interface description does not import the connections
description - the dependency runs the other way, and inverting it to satisfy a warning would
be the tail wagging the dog.

```table-editor
---
columns: { this: { label: "Connection" } }
---
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix oml:  <http://opencaesar.io/oml#> .
@prefix alan: <http://example.com/method/vocabulary#> .
@prefix ui:   <http://example.com/method/ui#> .

alan:ConnectionShape
    a sh:NodeShape ;
    sh:targetClass alan:Connection ;
    sh:property [
        sh:path oml:hasSource ;
        sh:name "From" ;
        dash:readOnly true ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path oml:hasTarget ;
        sh:name "To" ;
        dash:readOnly true ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path alan:transfers ;
        sh:name "Transfers" ;
        sh:class alan:Item ;
        sh:minCount 1 ;
        sh:message "A connection that carries nothing cannot be reasoned about. Name the item that crosses it - the power dependency rule fires only on connections carrying a PowerFlow." ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path dc:description ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 4 ;
    ] ;
    sh:sparql [
        sh:message "Both ends of this connection belong to the same component. An internal detail of one component is not an interface between two." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            PREFIX oml:  <http://opencaesar.io/oml#>
            SELECT $this WHERE {
                $this oml:hasSource ?src ; oml:hasTarget ?tgt .
                ?src alan:interfaceOf ?c .
                ?tgt alan:interfaceOf ?c .
            }
        """ ;
    ] ;
    .
```

## Interface coverage

```table-editor
---
columns: { this: { label: "Interface" } }
---
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix oml:  <http://opencaesar.io/oml#> .
@prefix alan: <http://example.com/method/vocabulary#> .
@prefix ui:   <http://example.com/method/ui#> .

alan:InterfaceCoverageShape
    a sh:NodeShape ;
    sh:targetClass alan:Interface ;
    dash:readOnly true ;
    sh:rule [
        a sh:SPARQLRule ;
        sh:construct """
            PREFIX alan: <http://example.com/method/vocabulary#>
            PREFIX oml:  <http://opencaesar.io/oml#>
            PREFIX ui:   <http://example.com/method/ui#>
            CONSTRUCT { $this ui:linkCount ?n }
            WHERE {
                SELECT $this (COUNT(?conn) AS ?n) WHERE {
                    $this a alan:Interface .
                    OPTIONAL { ?conn oml:hasSource $this }
                    OPTIONAL { ?conn oml:hasTarget $this }
                }
                GROUP BY $this
            }
        """ ;
    ] ;
    sh:property [
        sh:path alan:interfaceOf ;
        sh:name "Component" ;
        dash:readOnly true ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path ui:linkCount ;
        sh:name "Connections" ;
        dash:readOnly true ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:sparql [
        sh:severity sh:Warning ;
        sh:message "No connection touches this interface. Either it is a boundary to something outside the system, which is worth saying in its description, or a connection is missing." ;
        sh:select """
            PREFIX oml: <http://opencaesar.io/oml#>
            SELECT $this WHERE {
                FILTER NOT EXISTS { ?c oml:hasSource $this }
                FILTER NOT EXISTS { ?c oml:hasTarget $this }
            }
        """ ;
    ] ;
    .
```
