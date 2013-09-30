ARSF is defined on top of array-map-structures like known from JSON, YAML, and programming languages such as Perl. In short, an array-map-structure can be expressed in JSON without special treatment of numbers, true, false, and null.

ARSF is similar to [JSON-LD](http://json-ld.org/) and RDF/Turtle but better aligned with YAML.

# Specification

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

## List-Map-Structures

A List-Map-Structure (LMS) is a data structure that consists of **LMS values**.
Each LMS value is either

* a Unicode string,
* a **LMS lists**, which is a sequence of zero or more LMS values, or
* a **LMS map**, which is a a (possibly empty) set of Unicode strings and a
  bijective mapping from these strings to LMS values.

Each **LMS instance** is a LMS value that is not a Unicode string (so it's
either a LMS list or a LMS map). 

*Note that this definition does not exclude recursive structures.*

In the following the terms "list" and "map" refer to LMS lists or LMS maps,
respectively, unless noted otherwise.

## URI References

An URI reference is represented as Unicode string. The string can either be 
an abbreviated URI reference or an absolute URI reference

## Abbreviated URI references

An abbreviated URI reference consists of a **prefix** and a **suffix**
separated by a colon (`:`). The prefix is a Unicode string starting with
a lowercase letter (`a-z`) optionally followed by a sequence of lowercase
letters and digits (`0-9`). A suffix is a Unicode string that is either empty
or conforms to the `name` rule from Turtle syntax specification:

    name          ::= nameStartChar nameChar*

    nameStartChar ::= [A-Z] | "_" | [a-z] | [#x00C0-#x00D6] | [#x00D8-#x00F6] |
                      [#x00F8-#x02FF] | [#x0370-#x037D] | [#x037F-#x1FFF] | 
                      [#x200C-#x200D] | [#x2070-#x218F] | [#x2C00-#x2FEF] | 
                      [#x3001-#xD7FF] | [#xF900-#xFDCF] | [#xFDF0-#xFFFD] |
                      [#x10000-#xEFFFF]

    nameChar      ::= nameStartChar | '-' | [0-9] | #x00B7 | 
                      [#x0300-#x036F] | [#x203F-#x2040]

*Note that this definition of prefixes is more restrictive than the definition
of prefixes in Turtle and JSON-LD. The restriction of suffixes allows for
absolute URI references such as `geo:48.2010,16.3695,183` [RFC5870] and
`tag:yaml.org,2002:int` [RFC4151] while still using these URI scheme names as
URI prefixes (e.g. `geo:Point` and `tag:Tag`).*

## Absolute URI references

...

## Literals

### Plain literals

...

### Language tag literals

...

### Datatype literals

...

## Blank nodes

...

## Statements


## Graphs

Like turtle


    { 
        $subject1 : {
            $predicate1 : $objects,
        },
        $subject2 : {
            $p1 :
            $p2 :
        }
    }


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
```

## explicit _id

    {
      "schema:name": "Manu Sporny",
      "schema:url": { "_id": "http://manu.sporny.org/" },
      "schema:image": { "_id:": "http://manu.sporny.org/images/manu.png" }
    }

## subject URI and shortcut for rdf:type

    {
      "_id": "http://me.markus-lanthaler.com/",
      "schema:name": "Markus Lanthaler",
      "a": "foaf:Person"
    }

## Datatypes like in JSON-LD

    {
      "dct:modified": {
        "_value": "2010-05-29T14:17:39+02:00",
        "_type": "http://www.w3.org/2001/XMLSchema#dateTime"
      }
    }

## Datatypes abbreviated

    {
      "dct:modified": {
        "_value": "2010-05-29T14:17:39+02:00",
        "_type": "xsd:dateTime"
      }
    }
    { "dct:modified": "2010-05-29T14:17:39+02:00^^xsd:dateTime" }
    { "dct:modified": "2010-05-29T14:17:39+02:00^^http://www.w3.org/2001/XMLSchema#dateTime" }

## Language tags:

    "Ninja@en", "忍者@ja"

To avoid interpreting language tags (same with datatypes):

    {
       "foo:bar": {
         "_value": "Ninja@en" # full literal
       }
    }

## Namespace prefixes

    _ns:
      xsd: http://www.w3.org/2001/XMLSchema#

# References

* Graham Klyne; Jeremy J. Carroll:
  *Resource Description Framework (RDF): Concepts and Abstract Syntax.*
  10 February 2004. W3C Recommendation.
  <http://www.w3.org/TR/2004/REC-rdf-concepts-20040210>

* *Turtle. Terse RDF Triple Language*.
  W3C Candidate Recommendation 19 February 2013.
  <http://www.w3.org/TR/turtle/>

* Uniform Resource Identifier (URI) Schemes.
  <http://www.iana.org/assignments/uri-schemes/>

* <http://json-ld.org/spec/latest/>

