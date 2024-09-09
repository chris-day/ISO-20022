# ISO-20022
ISO 20022 Representation in OWL

This translation is based upon the work by Henriette Harmse outlined in her UML2OWL paper and https://github.com/henrietteharmse/uml2semantics

The ISO 20022 Repository used for conversion is 20230719_ISO20022_2013_eRepository.iso20022.xml, available from https://www.iso20022.org/iso20022-repository/e-repository

The TSV files can be used with uml2semantics

`java -jar uml2semantics.jar --classes "class.tsv" --attributes "attribute.tsv" --ontology "iso20022.owl" --ontologyIRI "https://iso20022.org/ontology" --ontologyPrefix "iso20022:https://iso20022.org/ontology/" --enumerations "enumeration.tsv" -n "enumerationIndividuals.tsv"`
