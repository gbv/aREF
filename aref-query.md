## aREF query

[*aREF query*]: #aref-query

*This section is non-normative*

**aREF query** is a query language to query [*string*], [*IRIs*], and/or
[*blank nodes*] from a given [*IRI*] or a [*blank node*] in an RDF graph. The
query language can be used as path language for RDF, similar to XPath for XML.

An [*aREF query*] consists a list of [*qNames*], separated by dot ("`.`") and
optionally followed by:

* either a dot to only query [*IRIs*] and [*blank nodes*],
* or an at character ("`@`") to only query [*literal nodes*],
* or an at character followed by a [*language tag*] (syntax rule
  `languageTag`) to only query [*literal nodes*](#rdf-data) with this 
  specific [*language tag*],
* or a caret ("`^`") to only query [*literal nodes*](#rdf-data) with 
  [*datatype*],
* or a caret followed by a [*qName*] to only query 
  [*literal nodes*](#rdf-data)
  with a specific [*datatype*]

```
query  = qName *( "." qName ) [ filter ]
filter = "." | "@" [ languageTag ] | "^" [ qName ]
```

<div class="example">

 aREF query expression     informal description
------------------------ --------------------------------------
`foaf_knows.foaf_knows`   friends of friends
`dct_creator.`            creators unless only given as string
`dct_creator@`            literal node creators
`dct_creator.foaf_name`   author names
`dct_date^xsd_gYear`      date values of datatype `xsd_gYear`
`skos_prefLabel@en`       preferred labels in English

</div>
