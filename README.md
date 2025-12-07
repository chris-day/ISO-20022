# ISO-20022

## ISO 20022 Representation in OWL

This translation is based upon the work by Henriette Harmse, outlined in her UML2OWL paper and https://github.com/henrietteharmse/uml2semantics

The ISO 20022 Repository used for conversion is 20250424_ISO20022_2013_eRepository.iso20022, available from https://www.iso20022.org/iso20022-repository/e-repository

The TSV files can be used with uml2semantics-python found here https://github.com/chris-day/uml2semantics-python

```
uml2semantics -c class.tsv -a attribute.tsv -e enumeration.tsv -n enumerationIndividuals.tsv --datatypes datatypes.tsv  --annotation-properties AnnotationProperties.tsv --annotations Annotations.tsv --output iso20022.ttl --prefix "iso20022:http://iso20022.org/iso20022/,iso20022cd:http://purl.org/iso20022/cd/,iso20022dt:http://purl.org/iso20022/dt/,iso20022mm:http://purl.org/iso20022/mm/,xsd:http://www.w3.org/2001/XMLSchema#" --ontology-iri "http://iso20022.org/iso20022" --format turtle
```

## Notes

2025-11-27 : New version based on a Python version of UML2Semantics here https://github.com/chris-day/uml2semantics-python
