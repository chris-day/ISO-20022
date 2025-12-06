# ISO 20022 SHACL Toolkit

This file provides example SHACL shapes that capture key ISO 20022 constraints, ready to be adapted to your concrete model.

It covers:

- Cardinalities
- Choice semantics (via SHACL-SPARQL)
- CodeSet usage
- Datatype facets
- Amount + currency patterns
- Message-level structural checks

Assumed prefixes:

```turtle
@prefix sh:        <http://www.w3.org/ns/shacl#> .
@prefix xsd:       <http://www.w3.org/2001/XMLSchema#> .
@prefix iso20022:  <http://iso20022.org/iso20022/> .
@prefix iso20022dt:<http://purl.org/iso20022/dt/> .
@prefix iso20022cd:<http://purl.org/iso20022/cd/> .
@prefix iso20022mm:<http://purl.org/iso20022/mm/> .
@prefix rdf:       <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs:      <http://www.w3.org/2000/01/rdf-schema#> .
```

---

## 1️⃣ Cardinality Shape for a MessageComponent Property

```turtle
iso20022:PaymentInstructionShape
    a sh:NodeShape ;
    sh:targetClass iso20022:PaymentInstruction ;
    sh:property [
        sh:path iso20022:paymentInformationIdentification ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "paymentInformationIdentification must appear exactly once."@en ;
    ] .
```

---

## 2️⃣ Choice Semantics via SHACL-SPARQL

Exactly one of three properties (Code, Proprietary, AddtlRsnInf) must be present.

```turtle
iso20022:ChoiceExactlyOneShape
    a sh:NodeShape ;
    sh:targetClass iso20022:AcceptedReason10Choice ;
    sh:sparql [
        a sh:SPARQLConstraint ;
        sh:message "Exactly one of Code, Proprietary, or AddtlRsnInf must be present."@en ;
        sh:select """
        SELECT ?this
        WHERE {
          BIND( (EXISTS { ?this iso20022:code ?c }        ) AS ?hasCode )
          BIND( (EXISTS { ?this iso20022:proprietary ?p } ) AS ?hasProp )
          BIND( (EXISTS { ?this iso20022:addtlRsnInf ?a } ) AS ?hasAddtl )

          BIND( (IF(?hasCode,1,0) + IF(?hasProp,1,0) + IF(?hasAddtl,1,0)) AS ?count )

          FILTER(?count != 1)
        }
        """ ;
    ] .
```

Replace property IRIs and `sh:targetClass` as required for other choice patterns.

---

## 3️⃣ CodeSet Usage Shape

```turtle
iso20022:ClearingSystemCodeShape
    a sh:NodeShape ;
    sh:targetClass iso20022:SomeContextClass ;
    sh:property [
        sh:path iso20022:clearingSystemCode ;
        sh:class iso20022cd:ClearingSystemIdentificationCode ;
        sh:message "clearingSystemCode must be an instance of ClearingSystemIdentificationCode."@en ;
    ] .
```

For literal codes instead of individuals, use `sh:in` or `sh:pattern` on a string property.

---

## 4️⃣ Datatype Facet Shape (LEI Example)

```turtle
iso20022:LeiIdentifierShape
    a sh:NodeShape ;
    sh:targetClass iso20022:LegalEntityNode ;
    sh:property [
        sh:path iso20022:lei ;
        sh:datatype xsd:string ;
        sh:minLength 20 ;
        sh:maxLength 20 ;
        sh:pattern "^[A-Z0-9]{20}$" ;
        sh:message "LEI must be a 20-character uppercase alphanumeric string."@en ;
    ] .
```

Targeting all LEI literals regardless of owning class:

```turtle
iso20022:LeiLiteralShape
    a sh:NodeShape ;
    sh:targetObjectsOf iso20022:lei ;
    sh:datatype xsd:string ;
    sh:minLength 20 ;
    sh:maxLength 20 ;
    sh:pattern "^[A-Z0-9]{20}$" ;
    sh:message "LEI literal must be a 20-character uppercase alphanumeric string."@en ;
```

---

## 5️⃣ Amount + Currency Pair Shape

```turtle
iso20022:AmountWithCurrencyShape
    a sh:NodeShape ;
    sh:targetClass iso20022:ActiveCurrencyAndAmount ;
    sh:property [
        sh:path iso20022:amount ;
        sh:datatype xsd:decimal ;
        sh:minCount 1 ;
        sh:message "Amount is mandatory."@en ;
    ] ;
    sh:property [
        sh:path iso20022:currency ;
        sh:class iso20022cd:ActiveCurrencyCode ;
        sh:minCount 1 ;
        sh:message "Currency code is mandatory and must be an ActiveCurrencyCode."@en ;
    ] .
```

Adapt `sh:targetClass` and property IRIs when modelling the pair on other classes.

---

## 6️⃣ Message-Level Structural Shape

```turtle
iso20022:PaymentStatusReportShape
    a sh:NodeShape ;
    sh:targetClass iso20022:PaymentStatusReportV11 ;
    sh:property [
        sh:path iso20022:grpHdr ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
        sh:message "GroupHeader (grpHdr) is mandatory and must occur once."@en ;
    ] ;
    sh:property [
        sh:path iso20022:orgnlGrpInfAndSts ;
        sh:minCount 1 ;
        sh:message "At least one OriginalGroupInformationAndStatus is required."@en ;
    ] .
```

Add further `sh:property` blocks for the remaining mandatory or constrained fields.

---

## 7️⃣ Running SHACL Validation

Example Jena CLI usage:

```bash
shacl validate   --datafile iso20022-data.ttl   --shapesfile iso20022-shapes.ttl
```

You can treat your iso20022 ontology as "data" when validating its structural consistency, or point at instance graphs when validating actual message instances.
