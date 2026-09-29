---
template:
  id: http://example.com/method/verification
  name: "Verification Coverage"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Verification Coverage

**Pattern 7 of 7. The half of Q3 that open-world reasoning cannot answer.**

Plan the activity, name the evidence it produces, and attach it to the requirement it
closes. The coverage editor at the bottom writes into this description, not into the
requirements file: the requirement text belongs to the requirements engineer, and planning
its verification does not change it.

**Why the gap needs SHACL.** The vocabulary already derives the positive half. A
requirement verified by an activity that produces evidence classifies as an
`EvidencedRequirement` with nothing asserted by hand. The complement - which requirements
are supported only by assertion - is not derivable: under the open world assumption, the
absence of an activity means unknown, not missing. Closed-world validation is the only way
to ask that question, and the warning below is it.

**Mission needs are exempt.** The coverage warning skips mission-level requirements. A
mission need is validated against stakeholders rather than verified against the article,
and flagging it every run would teach people that the warning is noise.

**Why a warning and not an error.** Three requirements carry no activity today. That is a
real finding, reported every time this page is validated, and it is not a defect in the
model - it is the state of the programme. Errors are for facts that make an analysis
wrong; warnings are for work that has not happened yet. Suppressing it would hide the
answer to Q3; making it an error would train people to ignore the validator.

**Status is not coverage.** Every activity here sits at `Planned`. A requirement with a
planned activity is covered, not verified. The status column is what keeps those two
apart, and it is why `Passed` with no evidence is an error rather than a warning.

```table-editor
---
columns: { this: { label: "Activity" } }
orderBy: Method
---
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix oml:  <http://opencaesar.io/oml#> .
@prefix alan: <http://example.com/method/vocabulary#> .
@prefix ui:   <http://example.com/method/ui#> .

alan:VerificationActivityShape
    a sh:NodeShape ;
    sh:targetClass alan:VerificationActivity ;
    sh:property [
        sh:path alan:verificationMethod ;
        sh:name "Method" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "Name the method - Test, Analysis, Inspection or Demonstration. It decides what the evidence has to look like." ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path alan:verificationStatus ;
        sh:name "Status" ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path alan:produces ;
        sh:name "Evidence" ;
        sh:class alan:Evidence ;
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
        sh:severity sh:Warning ;
        sh:message "This activity produces no identifiable evidence, so it can never close a requirement. Name the report, log or record it will leave behind." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                FILTER NOT EXISTS { $this alan:produces ?e }
            }
        """ ;
    ] ;
    sh:sparql [
        sh:message "An activity cannot be Passed with no evidence attached. Either attach the evidence or return the status to InProgress." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                $this alan:verificationStatus ?s .
                FILTER (STR(?s) = "Passed")
                FILTER NOT EXISTS { $this alan:produces ?e }
            }
        """ ;
    ] ;
    .
```

## Coverage

The requirement column below is context owned by the requirements description. The only
editable field is the activity that verifies it.

```table-editor
---
columns: { this: { label: "Requirement" } }
orderBy: Coverage
---
@prefix sh:   <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix rdf:  <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix xsd:  <http://www.w3.org/2001/XMLSchema#> .
@prefix dc:   <http://purl.org/dc/elements/1.1/> .
@prefix oml:  <http://opencaesar.io/oml#> .
@prefix alan: <http://example.com/method/vocabulary#> .
@prefix ui:   <http://example.com/method/ui#> .

alan:RequirementCoverageShape
    a sh:NodeShape ;
    sh:targetClass alan:Requirement ;
    sh:rule [
        a sh:SPARQLRule ;
        sh:construct """
            PREFIX alan: <http://example.com/method/vocabulary#>
            PREFIX ui:   <http://example.com/method/ui#>
            CONSTRUCT { $this ui:coverage ?state }
            WHERE {
                $this a alan:Requirement .
                OPTIONAL { $this alan:isVerifiedBy ?activity }
                BIND(IF(BOUND(?activity), "Covered", "Assertion only") AS ?state)
            }
        """ ;
    ] ;
    sh:property [
        sh:path ui:coverage ;
        sh:name "Coverage" ;
        dash:readOnly true ;
        sh:maxCount 1 ;
        sh:order 1 ;
    ] ;
    sh:property [
        sh:path alan:requirementId ;
        sh:name "ID" ;
        dash:readOnly true ;
        sh:maxCount 1 ;
        sh:order 2 ;
    ] ;
    sh:property [
        sh:path alan:statement ;
        sh:name "Statement" ;
        dash:readOnly true ;
        sh:maxCount 1 ;
        sh:order 3 ;
    ] ;
    sh:property [
        sh:path alan:isVerifiedBy ;
        sh:name "Verified By" ;
        sh:class alan:VerificationActivity ;
        sh:order 4 ;
    ] ;
    sh:sparql [
        sh:severity sh:Warning ;
        sh:message "This requirement is supported only by assertion: no activity is planned for it. Plan one, or record why it is accepted without verification." ;
        sh:select """
            PREFIX alan: <http://example.com/method/vocabulary#>
            SELECT $this WHERE {
                $this alan:requirementLevel ?level .
                FILTER (STR(?level) != "Mission")
                FILTER NOT EXISTS { $this alan:isVerifiedBy ?v }
            }
        """ ;
    ] ;
    .
```
