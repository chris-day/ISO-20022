# ISO 20022 SPARQL Starter Toolkit

This toolkit provides a set of initial SPARQL queries useful for exploring a fully expanded ISO 20022 ontology loaded into a SPARQL engine such as Apache Jena Fuseki, GraphDB, Stardog, or Blazegraph.

These queries cover:

- BusinessAreas
- BusinessComponents
- MessageComponents (as structural message “definitions”)
- CodeSets and Code values

Most queries assume the following prefixes:

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

## 1️⃣ Business Areas

### List all BusinessAreas

```sparql
SELECT ?ba ?label ?status
WHERE {
  ?ba a owl:Class ;
      iso20022mm:type "BusinessArea" ;
      rdfs:label ?label .
  OPTIONAL { ?ba iso20022mm:registrationStatus ?status }
}
ORDER BY ?label
```

### Filter BusinessAreas by label text

```sparql
SELECT ?ba ?label
WHERE {
  ?ba a owl:Class ;
      iso20022mm:type "BusinessArea" ;
      rdfs:label ?label .
  FILTER(CONTAINS(LCASE(STR(?label)), "payment"))
}
ORDER BY ?label
```

---

## 2️⃣ Business Components

### Overview of all BusinessComponents

```sparql
SELECT ?bc ?label ?status ?superType ?superClass
WHERE {
  ?bc a owl:Class ;
      iso20022mm:type "BusinessComponent" ;
      rdfs:label ?label .
  OPTIONAL { ?bc iso20022mm:registrationStatus ?status }
  OPTIONAL { ?bc iso20022mm:superType ?superType }
  OPTIONAL { ?bc rdfs:subClassOf ?superClass }
}
LIMIT 200
```

### BusinessAttributes of each BusinessComponent

```sparql
SELECT ?bc ?bcLabel ?attr ?attrLabel ?simpleType ?range ?min ?max
WHERE {
  ?bc a owl:Class ;
      iso20022mm:type "BusinessComponent" ;
      rdfs:label ?bcLabel .

  ?attr a owl:ObjectProperty ;
        iso20022mm:type "BusinessAttribute" ;
        rdfs:domain ?bc .

  OPTIONAL { ?attr rdfs:label ?attrLabel }
  OPTIONAL { ?attr iso20022mm:simpleType ?simpleType }
  OPTIONAL { ?attr rdfs:range ?range }
  OPTIONAL { ?attr iso20022mm:minOccurs ?min }
  OPTIONAL { ?attr iso20022mm:maxOccurs ?max }
}
ORDER BY ?bcLabel ?attrLabel
LIMIT 500
```

### BusinessAssociationEnds (relationships between BCs)

```sparql
SELECT ?bcLabel ?endLabel ?targetLabel ?min ?max
WHERE {
  ?end a owl:ObjectProperty ;
       iso20022mm:type "BusinessAssociationEnd" ;
       rdfs:domain ?bc ;
       rdfs:range ?target .

  ?bc rdfs:label ?bcLabel .
  OPTIONAL { ?target rdfs:label ?targetLabel }
  OPTIONAL { ?end rdfs:label ?endLabel }
  OPTIONAL { ?end iso20022mm:minOccurs ?min }
  OPTIONAL { ?end iso20022mm:maxOccurs ?max }
}
ORDER BY ?bcLabel ?endLabel
LIMIT 500
```

---

## 3️⃣ Message Sets & Message Components

### All MessageSets

```sparql
SELECT ?ms ?label ?status
WHERE {
  ?ms iso20022mm:type "MessageSet" .
  OPTIONAL { ?ms rdfs:label ?label }
  OPTIONAL { ?ms iso20022mm:registrationStatus ?status }
}
ORDER BY ?label
```

### All MessageComponents

```sparql
SELECT ?mc ?label ?status
WHERE {
  ?mc a owl:Class ;
      iso20022mm:type "MessageComponent" ;
      rdfs:label ?label .
  OPTIONAL { ?mc iso20022mm:registrationStatus ?status }
}
LIMIT 200
```

### MessageComponents without superTypes (likely top-level messages)

```sparql
SELECT ?mc ?label
WHERE {
  ?mc a owl:Class ;
      iso20022mm:type "MessageComponent" ;
      rdfs:label ?label .
  FILTER NOT EXISTS { ?mc iso20022mm:superType ?any }
}
ORDER BY ?label
LIMIT 200
```

### Attributes of MessageComponents

```sparql
SELECT ?mcLabel ?propLabel ?range ?min ?max
WHERE {
  ?mc a owl:Class ;
      iso20022mm:type "MessageComponent" ;
      rdfs:label ?mcLabel .

  ?prop a ?propType ;
        rdfs:domain ?mc .

  FILTER(?propType IN (owl:ObjectProperty, owl:DatatypeProperty))

  OPTIONAL { ?prop rdfs:label ?propLabel }
  OPTIONAL { ?prop rdfs:range ?range }
  OPTIONAL { ?prop iso20022mm:minOccurs ?min }
  OPTIONAL { ?prop iso20022mm:maxOccurs ?max }
}
ORDER BY ?mcLabel ?propLabel
LIMIT 500
```

---

## 4️⃣ CodeSets and code values

### All CodeSets

```sparql
SELECT ?cs ?label ?status ?minLen ?maxLen
WHERE {
  ?cs iso20022mm:type "CodeSet" .
  OPTIONAL { ?cs rdfs:label ?label }
  OPTIONAL { ?cs iso20022mm:registrationStatus ?status }
  OPTIONAL { ?cs iso20022mm:minLength ?minLen }
  OPTIONAL { ?cs iso20022mm:maxLength ?maxLen }
}
ORDER BY ?label
```

### Codes belonging to a specific CodeSet (label match)

```sparql
SELECT ?code ?codeLabel ?codeComment
WHERE {
  ?cs iso20022mm:type "CodeSet" ;
      rdfs:label ?csLabel .
  FILTER(?csLabel = "ClearingSystemIdentificationCode")  # modify this

  BIND( IRI(CONCAT("http://purl.org/iso20022/cd/",
                   STRAFTER(STR(?cs), "http://iso20022.org/iso20022/"))) AS ?codeClass )

  ?code a ?codeClass ;
        rdfs:label ?codeLabel .
  OPTIONAL { ?code rdfs:comment ?codeComment }
}
ORDER BY ?codeLabel
```

### Where a CodeSet is used as a BusinessAttribute simpleType

```sparql
SELECT ?ownerLabel ?attrLabel ?csLabel
WHERE {
  ?cs iso20022mm:type "CodeSet" .
  OPTIONAL { ?cs rdfs:label ?csLabel }

  ?attr a owl:ObjectProperty ;
        iso20022mm:type "BusinessAttribute" ;
        iso20022mm:simpleType ?cs ;
        rdfs:domain ?owner .

  OPTIONAL { ?owner rdfs:label ?ownerLabel }
  OPTIONAL { ?attr rdfs:label ?attrLabel }
}
ORDER BY ?ownerLabel ?attrLabel
```

---

## Notes

- `LIMIT` values can be raised once initial performance is validated
- To focus any query, add:

```sparql
FILTER(?entity = iso20022:YOUR-IRI-HERE)
```

- If using GraphDB, use its SPARQL visualisation features to navigate results

---
