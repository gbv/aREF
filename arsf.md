# Another RDF Serialization Form (ARSF)

ARSF is defined on top of array-map-structures like known from JSON, YAML, and programming languages such as Perl. In short, an array-map-structure can be expressed in JSON without special treatment of numbers, true, false, and null.

ARSF is similar to [JSON-LD](http://json-ld.org/) and RDF/Turtle but better aligned with YAML.


## Specification

```
Graph       = { ( Subject : PropertyMap )* Namespaces? } 
            | { ( "_id" : Subject ) ( Predicate : Objects )* Namespaces? }
PropertyMap = { ( Predicate : Objects )* }

Subject     = URIRef
Predicate   = URIRef | "a"
Objects     = Object | [ Object* ]

Object      = URIRef | SimpleString | StringWithDatatype | StringWithLanguage
            | { ( "_id" : URIRef ) ( Predicate : Objects )* }
            | { ( Predicate : Objects )* }
            | { ( "_value" : String ) }
            | { ( "_value" : String ) ( "_type" : URIRef ) }
            | { ( "_value" : String ) ( "_lang" : Language ) }

URIref      = FullURI | PrefixedURI | BlankNodeIdentifier
Namespaces  = ( "_ns" : { ( Prefix : FullURI )* } )

```

# Summary

## keywords:

* `_id`
* `_value`
* `_type`
* `_lang`
* `a`
* `_ns`

## Predicate-Map

```
{
   $p1 : $objects1,
   $p2 : $objects2,
   ...
}
```

## Same-Subject-Graphs


```
{
   _id : $subject,
   $p1 : $objects1,
   $p2 : $objects2,
   ...
}
```

## Graphs

```
{
   $s1 : $PredicateMap1,
   $s1 : $PredicateMap2,
   ...
}
```

Plus optional keyword `_ns` (prefixes).

## URI references

a) full URI (string)
b) with namespace prefix (string)
c) blank node identifier (string) (`_:xxx`)
c) map 
    a) with `_id`
    b) with `_value` (and possible `_lang` or `_type`)
    c) anonymous blank node (=PredicateMap)

## Objects

a) a single objects (string or map)
b) a list of objects (array)

# Examples

Taken and adopted from the JSON-LD specification:

```
{
  "schema:name": "Manu Sporny",
  "schema:url": "http://manu.sporny.org/",
  "schema:image": "http://manu.sporny.org/images/manu.png"
}

# explicit _id
{
  "schema:name": "Manu Sporny",
  "schema:url": { "_id": "http://manu.sporny.org/" },
  "schema:image": { "_id:": "http://manu.sporny.org/images/manu.png" }
}

# subject URI and shortcut for rdf:type
{
  "_id": "http://me.markus-lanthaler.com/",
  "schema:name": "Markus Lanthaler",
  "a": "foaf:Person"
}

# Datatypes like in JSON-LD
{
  "dct:modified": {
    "_value": "2010-05-29T14:17:39+02:00",
    "_type": "http://www.w3.org/2001/XMLSchema#dateTime"
  }
}
# Datatypes abbreviated
{
  "dct:modified": {
    "_value": "2010-05-29T14:17:39+02:00",
    "_type": "xsd:dateTime"
  }
}
{ "dct:modified": "2010-05-29T14:17:39+02:00^^xsd:dateTime" }
{ "dct:modified": "2010-05-29T14:17:39+02:00^^http://www.w3.org/2001/XMLSchema#dateTime" }

# Language tags:
"Ninja@en", "忍者@ja"

# To avoid interpreting language tags (same with datatypes):
{
   "foo:bar": {
     "_value": "Ninja@en" # full literal
   }
}

prefixes:

_ns:
  xsd: http://www.w3.org/2001/XMLSchema#
```