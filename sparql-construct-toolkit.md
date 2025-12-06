# ISO 20022 SPARQL CONSTRUCT Toolkit

This file provides `CONSTRUCT` queries that generate compact subgraphs (mini-ontologies) for selected ISO 20022 artefacts (BusinessAreas, BusinessComponents, MessageComponents, CodeSets).

Use these to export focused Turtle / RDF datasets.

Assumed prefixes:

```sparql
PREFIX iso20022:   <http://iso20022.org/iso20022/>
PREFIX iso20022mm: <http://purl.org/iso20022/mm/>
PREFIX iso20022cd: <http://purl.org/iso20022/cd/>
PREFIX iso20022dt: <http://purl.org/iso20022/dt/>
PREFIX owl:        <http://www.w3.org/2002/07/owl#>
PREFIX rdf:        <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs:       <http://www.w3.org/2000/01/rdf-schema#>
```

---

## 1️⃣ BusinessArea Subgraph

```sparql
CONSTRUCT {
  ?ba a owl:Class ;
      rdfs:label ?label ;
      iso20022mm:type "BusinessArea" ;
      iso20022mm:registrationStatus ?status ;
      rdfs:comment ?comment .
}
WHERE {
  VALUES ?ba { iso20022:YOUR_BUSINESS_AREA_IRI }

  ?ba a owl:Class ;
      iso20022mm:type "BusinessArea" ;
      rdfs:label ?label .
  OPTIONAL { ?ba iso20022mm:registrationStatus ?status }
  OPTIONAL { ?ba rdfs:comment ?comment }
}
```

BusinessArea with MessageSets:

```sparql
CONSTRUCT {
  ?ba a owl:Class ;
      rdfs:label ?baLabel ;
      iso20022mm:type "BusinessArea" .

  ?ms iso20022mm:type "MessageSet" ;
      rdfs:label ?msLabel ;
      iso20022mm:registrationStatus ?msStatus ;
      iso20022mm:businessArea ?ba .
}
WHERE {
  VALUES ?ba { iso20022:YOUR_BUSINESS_AREA_IRI }

  ?ba a owl:Class ;
      iso20022mm:type "BusinessArea" ;
      rdfs:label ?baLabel .

  ?ms iso20022mm:type "MessageSet" ;
      iso20022mm:businessArea ?ba .
  OPTIONAL { ?ms rdfs:label ?msLabel }
  OPTIONAL { ?ms iso20022mm:registrationStatus ?msStatus }
}
```

---

## 2️⃣ MessageComponent Subgraph

Single MessageComponent with attributes:

```sparql
CONSTRUCT {
  ?mc a owl:Class ;
      rdfs:label ?mcLabel ;
      iso20022mm:type "MessageComponent" ;
      iso20022mm:registrationStatus ?mcStatus ;
      rdfs:comment ?mcComment .

  ?prop a ?propType ;
        rdfs:domain ?mc ;
        rdfs:label ?propLabel ;
        rdfs:range ?range ;
        iso20022mm:minOccurs ?min ;
        iso20022mm:maxOccurs ?max ;
        rdfs:comment ?propComment .
}
WHERE {
  VALUES ?mc { iso20022:YOUR_MESSAGE_COMPONENT_IRI }

  ?mc a owl:Class ;
      iso20022mm:type "MessageComponent" ;
      rdfs:label ?mcLabel .
  OPTIONAL { ?mc iso20022mm:registrationStatus ?mcStatus }
  OPTIONAL { ?mc rdfs:comment ?mcComment }

  ?prop rdfs:domain ?mc ;
        a ?propType .
  FILTER(?propType IN (owl:ObjectProperty, owl:DatatypeProperty))

  OPTIONAL { ?prop rdfs:label ?propLabel }
  OPTIONAL { ?prop rdfs:range ?range }
  OPTIONAL { ?prop iso20022mm:minOccurs ?min }
  OPTIONAL { ?prop iso20022mm:maxOccurs ?max }
  OPTIONAL { ?prop rdfs:comment ?propComment }
}
```

MessageComponent plus CodeSets / datatypes used:

```sparql
CONSTRUCT {
  ?mc a owl:Class ;
      rdfs:label ?mcLabel ;
      iso20022mm:type "MessageComponent" .

  ?prop a ?propType ;
        rdfs:domain ?mc ;
        rdfs:label ?propLabel ;
        rdfs:range ?range .

  ?range rdfs:label ?rangeLabel ;
         iso20022mm:type ?rangeType .

  ?cs a owl:Class ;
      iso20022mm:type "CodeSet" ;
      rdfs:label ?csLabel .
}
WHERE {
  VALUES ?mc { iso20022:YOUR_MESSAGE_COMPONENT_IRI }

  ?mc a owl:Class ;
      iso20022mm:type "MessageComponent" ;
      rdfs:label ?mcLabel .

  ?prop rdfs:domain ?mc ;
        a ?propType ;
        rdfs:range ?range .
  FILTER(?propType IN (owl:ObjectProperty, owl:DatatypeProperty))

  OPTIONAL { ?prop rdfs:label ?propLabel }
  OPTIONAL { ?range rdfs:label ?rangeLabel }
  OPTIONAL { ?range iso20022mm:type ?rangeType }

  OPTIONAL {
    ?cs iso20022mm:type "CodeSet" .
    FILTER(?cs = ?range || STRSTARTS(STR(?range), "http://purl.org/iso20022/cd/"))
    OPTIONAL { ?cs rdfs:label ?csLabel }
  }
}
```

---

## 3️⃣ BusinessComponent Subgraph

```sparql
CONSTRUCT {
  ?bc a owl:Class ;
      rdfs:label ?bcLabel ;
      iso20022mm:type "BusinessComponent" ;
      iso20022mm:registrationStatus ?bcStatus ;
      rdfs:comment ?bcComment .

  ?attr a owl:ObjectProperty ;
        iso20022mm:type "BusinessAttribute" ;
        rdfs:domain ?bc ;
        rdfs:label ?attrLabel ;
        rdfs:range ?attrRange ;
        iso20022mm:minOccurs ?attrMin ;
        iso20022mm:maxOccurs ?attrMax .

  ?end a owl:ObjectProperty ;
       iso20022mm:type "BusinessAssociationEnd" ;
       rdfs:domain ?bc ;
       rdfs:range ?target ;
       rdfs:label ?endLabel ;
       iso20022mm:minOccurs ?endMin ;
       iso20022mm:maxOccurs ?endMax .

  ?target rdfs:label ?targetLabel .
}
WHERE {
  VALUES ?bc { iso20022:YOUR_BUSINESS_COMPONENT_IRI }

  ?bc a owl:Class ;
      iso20022mm:type "BusinessComponent" ;
      rdfs:label ?bcLabel .
  OPTIONAL { ?bc iso20022mm:registrationStatus ?bcStatus }
  OPTIONAL { ?bc rdfs:comment ?bcComment }

  OPTIONAL {
    ?attr a owl:ObjectProperty ;
          iso20022mm:type "BusinessAttribute" ;
          rdfs:domain ?bc .
    OPTIONAL { ?attr rdfs:label ?attrLabel }
    OPTIONAL { ?attr rdfs:range ?attrRange }
    OPTIONAL { ?attr iso20022mm:minOccurs ?attrMin }
    OPTIONAL { ?attr iso20022mm:maxOccurs ?attrMax }
  }

  OPTIONAL {
    ?end a owl:ObjectProperty ;
         iso20022mm:type "BusinessAssociationEnd" ;
         rdfs:domain ?bc ;
         rdfs:range ?target .
    OPTIONAL { ?end rdfs:label ?endLabel }
    OPTIONAL { ?end iso20022mm:minOccurs ?endMin }
    OPTIONAL { ?end iso20022mm:maxOccurs ?endMax }
    OPTIONAL { ?target rdfs:label ?targetLabel }
  }
}
```

---

## 4️⃣ CodeSet Subgraph with Codes

```sparql
CONSTRUCT {
  ?cs a owl:Class ;
      iso20022mm:type "CodeSet" ;
      rdfs:label ?csLabel ;
      iso20022mm:registrationStatus ?csStatus ;
      rdfs:comment ?csComment .

  ?code a ?codeClass ;
        rdfs:label ?codeLabel ;
        rdfs:comment ?codeComment .
}
WHERE {
  VALUES ?cs { iso20022:YOUR_CODESET_IRI }

  ?cs iso20022mm:type "CodeSet" ;
      rdfs:label ?csLabel .
  OPTIONAL { ?cs iso20022mm:registrationStatus ?csStatus }
  OPTIONAL { ?cs rdfs:comment ?csComment }

  BIND( IRI(CONCAT("http://purl.org/iso20022/cd/",
                   STRAFTER(STR(?cs), "http://iso20022.org/iso20022/"))) AS ?codeClass )

  ?code a ?codeClass ;
        rdfs:label ?codeLabel .
  OPTIONAL { ?code rdfs:comment ?codeComment }
}
```

---

## 5️⃣ BusinessArea → MessageSet → MessageComponent Chain

```sparql
CONSTRUCT {
  ?ba  a owl:Class ;
       iso20022mm:type "BusinessArea" ;
       rdfs:label ?baLabel .

  ?ms  iso20022mm:type "MessageSet" ;
       rdfs:label ?msLabel ;
       iso20022mm:businessArea ?ba .

  ?mc  a owl:Class ;
       iso20022mm:type "MessageComponent" ;
       rdfs:label ?mcLabel ;
       iso20022mm:messageSet ?ms .

  ?prop a ?propType ;
        rdfs:domain ?mc ;
        rdfs:label ?propLabel ;
        rdfs:range ?range .

  ?range rdfs:label ?rangeLabel .
}
WHERE {
  VALUES ?ba { iso20022:YOUR_BUSINESS_AREA_IRI }

  ?ba  iso20022mm:type "BusinessArea" ;
       rdfs:label ?baLabel .

  ?ms  iso20022mm:type "MessageSet" ;
       iso20022mm:businessArea ?ba .
  OPTIONAL { ?ms rdfs:label ?msLabel }

  ?mc  a owl:Class ;
       iso20022mm:type "MessageComponent" ;
       iso20022mm:messageSet ?ms .
  OPTIONAL { ?mc rdfs:label ?mcLabel }

  ?prop rdfs:domain ?mc ;
        a ?propType ;
        rdfs:range ?range .
  FILTER(?propType IN (owl:ObjectProperty, owl:DatatypeProperty))

  OPTIONAL { ?prop rdfs:label ?propLabel }
  OPTIONAL { ?range rdfs:label ?rangeLabel }
}
```

Adjust `iso20022mm:businessArea` / `iso20022mm:messageSet` to match your actual predicates if they differ.
