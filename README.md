# ISO-20022
ISO 20022 Representation in OWL

This translation is based upon the work by Henriette Harmse outlined in her UML2OWL paper and https://github.com/henrietteharmse/uml2semantics

The ISO 20022 Repository used for conversion is 20240729_ISO20022_2013_eRepository.iso20022, available from https://www.iso20022.org/iso20022-repository/e-repository

The TSV files can be used with uml2semantics

`java -jar uml2semantics.jar --classes "class.tsv" --attributes "attribute.tsv" --ontology "iso20022.owl" --ontologyIRI "https://iso20022.org/ontology" --ontologyPrefix "iso20022:https://iso20022.org/ontology/" --enumerations "enumeration.tsv" -n "enumerationIndividuals.tsv"`

## Notes

2025-11-27 : New version based on a Python version of UML2Semantics
2024-09-17 : Discussion around whether data types presented via a Metaclass should be first-class Datatypes i.e. ISO 20022 Data Types such as MaxText140 rather than using the built-in. As ISO 20022:2025 is migrating to ISO 11404 General Purpose Datatypes, perhaps there is a bigger picture.

## Combined and fixed Ontology

The files iso20022.rdfxml and iso20022-manchester-owl.omn and RDF/XML and Manchester OWL combined files that have the right Datatypes and DatatypeProperty fixes.

Use this as a good, solid base to load messages instance against.
