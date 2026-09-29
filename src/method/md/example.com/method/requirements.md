---
template:
  id: http://example.com/method/requirements
  name: "Requirements"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Requirements

**Pattern 6 of 7. The requirement half of Q3; the verification half is the next page.**

A requirement is an identifier, a normative sentence, a level, and a parent. Write the
sentence as it will be read in a review: one obligation, testable, no commentary. The
reasoning behind it goes in the description column, not into the statement.

**Why the text is a model fact and not an annotation.** `statement` is a semantic property,
so a rule can read it. The vocabulary's `NonNormativeWording` rule inspects it and
classifies anything hedged as `NonNormativeRequirement`; the wording rule below catches the
same thing at authoring time, before it is ever committed. An annotation property could do
neither.

**Identifiers.** The `RequirementId` facet already rejects a malformed identifier as a
logical inconsistency. What it cannot do is notice that two requirements share one: an OML
`key` declaration would, and the toolchain rejected it. So the duplicate check lives here
as a SHACL rule, which is the honest place for it until the key works.

**Provenance is required, and this rule arrived late.** The analysis layer noticed that three
requirements cited no source, and the audit could only report it as a list because no rule
asked for it. That was a pattern gap rather than a data gap: the method never required a
requirement to name where it came from. The rule below closes it, and the three requirements
that prompted it now carry a warning until someone records their source.

**Levels and tracing.** Mission-level requirements stand alone. Everything below one must
name the requirement it refines, or the trace from a subsystem constraint back to the
mission need has a hole in it. Under the open world assumption the missing link is merely
unknown, which is exactly why this is a SHACL check and not a reasoner check.

```table-editor
---
columns: { this: { label: "Requirement" } }
orderBy: Level
stylesheet:
  - selector: cell[col === "Level" && value]
    target: value
    style:
      padding: 2px 10px
      border-radius: 999px
      font-size: 11px
      font-weight: 600
      color: "#7a6cff"
      background-color: "#ede9ff"
---
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix oml:  <http://opencaesar.io/oml#> .
@prefix alan: <http://example.com/method/vocabulary#> .
@prefix ui:   <http://example.com/method/ui#> .

alan:RequirementShape
    a sh:NodeShape ;
    sh:targetClass alan:Requirement ;
    sh:property [
        sh:path alan:requirementId ;
        sh:name "ID" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "Every requirement is known by an identifier in the requirements document. Without one it cannot be cited in a review." ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path alan:requirementLevel ;
        sh:name "Level" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path alan:statement ;
        sh:name "Statement" ;
        dash:editor dash:TextAreaEditor ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "A requirement with no statement is a placeholder. Write the obligation, or delete the instance." ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path alan:refines ;
        sh:name "Refines" ;
        sh:class alan:Requirement ;
        sh:order 4 ;
    ] ;
    sh:property [
        sh:path alan:constrains ;
        sh:name "Constrains" ;
        sh:class alan:Interface ;
        sh:order 5 ;
    ] ;
    sh:property [
        sh:path dc:description ;
        sh:name "Rationale" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
        sh:order 6 ;
    ] ;
    sh:sparql [
        sh:message "A system or subsystem requirement must refine a parent requirement. Name the requirement it comes from, or nothing traces it to the mission need." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                $this alan:requirementLevel ?level .
                FILTER (STR(?level) != "Mission")
                FILTER NOT EXISTS { $this alan:refines ?parent }
            }
        """ ;
    ] ;
    sh:sparql [
        sh:message "This statement is not normative. A requirement says shall; should, will, may and can read as intent and cannot be failed in a review." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                $this alan:statement ?s .
                FILTER (!REGEX(STR(?s), "shall"))
            }
        """ ;
    ] ;
    sh:sparql [
        sh:severity sh:Warning ;
        sh:message "This requirement cites no source document. Record where it came from - a requirement with no provenance is an opinion with an identifier." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                FILTER NOT EXISTS { $this alan:sourceDocument ?source }
            }
        """ ;
    ] ;
    sh:sparql [
        sh:message "Another requirement already uses this identifier. Two requirements with one identifier make every compliance report ambiguous." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                $this alan:requirementId ?id .
                ?other alan:requirementId ?id .
                FILTER (?other != $this)
            }
        """ ;
    ] ;
    .
```
