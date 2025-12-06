# ISO 20022 SPARQL Performance Toolkit

This file provides performance-tuned variants of the core ISO 20022 queries for large triple stores (e.g. Fuseki, GraphDB, Blazegraph, Stardog).

General strategies used:

- Avoid `FILTER(CONTAINS(...))` on large sets where possible (prefer `VALUES` or exact `=`)
- Push as many restrictions as early as possible (e.g. constrain BAs or MessageComponents up front)
- Limit projection columns to what are actually needed
- Use pagination (`LIMIT` / `OFFSET`) for browsing
- Use `VALUES` to constrain targets instead of OR-heavy FILTERs

All examples assume:

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

## 1️⃣ Paginated BusinessComponents

```sparql
SELECT ?bc ?label ?status
WHERE {
  ?bc a owl:Class ;
      iso20022mm:type "BusinessComponent" ;
      rdfs:label ?label .
  OPTIONAL { ?bc iso20022mm:registrationStatus ?status }
}
ORDER BY ?label
LIMIT 200
OFFSET 0   # Increase to 200, 400, ... for paging
```

---

## 2️⃣ Targeted BusinessComponents via VALUES

```sparql
SELECT ?bc ?label ?status ?superType
WHERE {
  VALUES ?bc {
    iso20022:BC_IRI_1
    iso20022:BC_IRI_2
  }

  ?bc a owl:Class ;
      iso20022mm:type "BusinessComponent" ;
      rdfs:label ?label .
  OPTIONAL { ?bc iso20022mm:registrationStatus ?status }
  OPTIONAL { ?bc iso20022mm:superType ?superType }
}
ORDER BY ?label
```

---

## 3️⃣ Efficient MessageComponent Attribute Listing

```sparql
SELECT ?mc ?mcLabel ?prop ?propLabel ?range ?min ?max
WHERE {
  ?mc a owl:Class ;
      iso20022mm:type "MessageComponent" ;
      rdfs:label ?mcLabel .

  ?prop rdfs:domain ?mc .
  VALUES ?propType { owl:ObjectProperty owl:DatatypeProperty }
  ?prop a ?propType .

  OPTIONAL { ?prop rdfs:label ?propLabel }
  OPTIONAL { ?prop rdfs:range ?range }
  OPTIONAL { ?prop iso20022mm:minOccurs ?min }
  OPTIONAL { ?prop iso20022mm:maxOccurs ?max }
}
ORDER BY ?mcLabel ?propLabel
LIMIT 500
OFFSET 0
```

To focus only on one message component, replace the `?mc` pattern with:

```sparql
VALUES ?mc { iso20022:YOUR_MESSAGE_COMPONENT_IRI }
```

---

## 4️⃣ CodeSets and Codes with Pre-filtered CodeSets

```sparql
SELECT ?cs ?csLabel ?code ?codeLabel
WHERE {
  VALUES ?cs {
    iso20022:CS_IRI_1
    iso20022:CS_IRI_2
  }

  ?cs iso20022mm:type "CodeSet" ;
      rdfs:label ?csLabel .

  BIND( IRI(CONCAT("http://purl.org/iso20022/cd/",
                   STRAFTER(STR(?cs), "http://iso20022.org/iso20022/"))) AS ?codeClass )

  ?code a ?codeClass ;
        rdfs:label ?codeLabel .
}
ORDER BY ?csLabel ?codeLabel
LIMIT 500
```

---

## 5️⃣ Choice Components with Pre-resolved IRIs

Discovery step (run once, export IRIs):

```sparql
SELECT ?choice ?label
WHERE {
  ?choice a owl:Class ;
          rdfs:label ?label .
  FILTER(CONTAINS(LCASE(STR(?label)), "choice"))
}
ORDER BY ?label
```

Repeated use with VALUES:

```sparql
SELECT ?choice ?choiceLabel ?prop ?propLabel ?range
WHERE {
  VALUES ?choice {
    iso20022:Choice_IRI_1
    iso20022:Choice_IRI_2
  }

  ?choice rdfs:label ?choiceLabel .

  ?prop rdfs:domain ?choice .
  VALUES ?propType { owl:ObjectProperty owl:DatatypeProperty }
  ?prop a ?propType .

  OPTIONAL { ?prop rdfs:label ?propLabel }
  OPTIONAL { ?prop rdfs:range ?range }
}
ORDER BY ?choiceLabel ?propLabel
```

---

## 6️⃣ Property Chains with Restriction

```sparql
SELECT ?prop ?propLabel ?chain
WHERE {
  ?prop a owl:ObjectProperty ;
        owl:propertyChainAxiom ?chain .
  OPTIONAL { ?prop rdfs:label ?propLabel }
}
ORDER BY ?propLabel
LIMIT 200
OFFSET 0
```

Use `VALUES ?prop { ... }` to focus on a known set of derived properties when possible.

---

## 7️⃣ Store-specific Notes (brief)

- **Fuseki/Jena** – consider using text indices (`text:query`) for label searches instead of `FILTER(CONTAINS(...))` on `rdfs:label`.
- **GraphDB** – if the ISO 20022 ontology sits in a dedicated named graph, use `FROM <graph>` / `FROM NAMED <graph>` to constrain query scope.
- **Blazegraph** – large values of `LIMIT` can be slow; prefer smaller chunks and client-side aggregation.
- **Stardog** – if using reasoning, start with `reasoning = off` for exploratory queries, then re-run with reasoning only for the final checks.
