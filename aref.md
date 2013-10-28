# Introduction

This document defines an encoding of RDF graphs called **Another RDF
Encoding Form (AREF)**. The encoding can be used as readable
representation of RDF data in serialization formats and data binding languages.

## Related RDF serializations

AREF is similar to [JSON-LD](http://json-ld.org/) and RDF/Turtle but better
aligned with YAML.

This document uses Turtle syntax to express RDF data.

# Core concepts

## RDF Data

RDF is a graph-based data structuring languages defined as abstract syntax by
Klyne and Carroll (2004). Several RDF variants exist. RDF data as encoded by
AREF documents can be defined as following:

* An ***RDF graph*** (also known as to as *RDF data*) is a set of *triples*.
* A ***triple*** (also known as *statement*) consists of a *subject*, 
  a *predicate*, and an *object*.
* A ***subject*** is either an *IRI* or a *blank node*.
* A ***predicate*** (also known as *property*) is an *IRI*.
* An ***object*** is either an *IRI* or a *blank node* or a *literal node*.
* An ***IRI*** (Internationalized Resource Identifier) is a string that
  conforms to the syntax defined in [RFC3987].
* A ***blank node*** is neither an *IRI* nor a *literal node*. 
* A ***literal node*** is a Unicode string optionally tagged by either
  a *datatype* or by a *language tag*.
* A ***datatype*** is an *IRI*.
* A ***language tag*** is a well-formed laguage tag as defined by (BCP47).

*NOTE: language tags conform to the regular expression
`[a-zA-Z]{1,8}(’-’[a-zA-Z0-9]{1,8})`. Case can be ignored.*

*NOTE: a **blank node identifier** MAY be used to refer to a particular
blank note within the scope of an AREF document. Blank node identifiers
are not part of RDF data.*

## List-map-structures

List-map-structure (LMS) consist of:

* Unicode strings
* ***LMS lists***, which are a sequences of zero or more *LMS values*
* ***LMS maps***, which are sets of Unicode strings (also known as *keys*)
  together with bijective (?) mappings from these strings to Unicode strings,
  *LMS lists* and *LMS maps*.

In the following the terms *list* and *map* refer to LMS lists or LMS maps,
respectively, unless noted otherwise.

*NOTE: This definition of List-map-structures does not exclude recursive
structures but applications MAY restrict LMS instances to non-recursive
structures. All non-recursive list-map-structures can be expressed in JSON
without special treatment of numbers, true, false, and null.*

# Specification

An RDF graph can be encoded as LMS instance as following.

## IRIs

IRIs are encoded as Unicode strings, either as absolute IRI or as prefixed name.

*TODO: include full grammar of IRI?*

### Absolute IRI

An absolute IRI is a Unicode string that is either an IRI with the following
constraints or an IRI enclosed in '<' and '>':

* An absolute IRI must not end with `"@" ( languageTag )?`
* An absolute IRI must not end with `"^^" ( prefixedName | "<" IRI ">" )`

*NOTE: no escape sequences as in Turtle because string escaping is one level
above!*

### Prefixed names

A prefixed name consists of a **prefix** and a **name** separated by a colon
(`:`). The prefix is a Unicode string starting with a lowercase letter (`a-z`)
optionally followed by a sequence of lowercase letters and digits (`0-9`). 

    prefixedName  ::= prefix ":" name

    prefix        ::= [a-z] ( [a-z] | [0-9] )*

A name is a Unicode string that is either empty or conforms to the `name` rule
from Turtle syntax specification (*FIXME: Turtle is more liberal*):

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

*NOTE: Compare <http://www.w3.org/TR/json-ld/#compact-iris> and <http://www.w3.org/TR/json-ld/#compact-iris>*

## Literals

Literals are encoded as Unicode strings.

### Plain literals

A trailing `@` is removed. If a literal may be confused with an IRI, its
representation in AREF contains an additional `@` at the end.

...

### Language tag literals

...see above...

### Datatype literals

...see above...

## Blank nodes

Blank nodes can be encoded with a blank node identifier or as anonymous blank
node.

### Identified blank node

...same as Turtle (?): <http://www.w3.org/TR/turtle/#BNodes>.

    blankNodeIdentifier ::= "_:" ( [a-z] | [A-Z] | [0-9] )+

### Anonymous blank node

same rule as a property mapgraph but without

## Graphs

A graph can be encoded as LMS map with the following constraints:

* The special key `_ns` MAY be used to indicate a namespace map
* If the key `_id` is given, then
    * its value MUST encode an IRI (= a subject)
    * every other key MUST encode an IRI or be the value "a" (= a predicate)
    * every other value MUST be an object list
* Otherwise
    * every key MUST encode an IRI (= a subject)
    * every value MUST be a property map

~~~
Graph       = { ( Subject : PropertyMap )* Namespaces? } 
            | { ( "_id" : Subject ) ( Predicate : Objects )* Namespaces? }
~~~

Like turtle


    { 
        $subject1 : {
            $predicate1 : $objects,
        },
        $subject2 : {
            $p1 : [ $obj1, $obj2 ],
            $p2 : $obj
        }
    }

Same-Subject-Graph:


```
{
   _id : $subject,
   $p1 : $objects1,
   $p2 : $objects2,
   ...
}
```


*NOTE: There can only be one namespaceMap in an AREF document.*

### Property maps

... aka predicate map (?)


```
{
   $p1 : $objectslist,
   $p2 : $objectslist,
   ...
}
```

### Object list

Is on either

* An IRI
* or a blank node
* or a literal

or a non empty LMS list of any of the above (= multiple objects).

...

### Namespace maps

...

# Summary/Notes

## keywords:

* `_id`
* `_value`, `_type`, `_lang`?
* `a`
* `_ns`

## nodes

a) full URI (string)
b) with namespace prefix (string)
c) blank node identifier (string) (`_:xxx`)
c) map 
    a) with `_id` ?
    b) with `_value` (and possible `_lang` or `_type`) ?
    c) anonymous blank node (=PredicateMap)

# Examples

Taken and adopted from the JSON-LD specification:

```
{
  "schema:name": "Manu Sporny",
  "schema:url": "http://manu.sporny.org/",
  "schema:image": "http://manu.sporny.org/images/manu.png"
}
```

## explicit _id (allowed?)

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
    { "dct:modified": "2010-05-29T14:17:39+02:00^^<http://www.w3.org/2001/XMLSchema#dateTime>" }

## Language tags

    "Ninja@en", "忍者@ja"

To avoid interpreting language tags (same with datatypes):

    {
       "foo:bar": {
         "_value": "Ninja@en" # full literal
       }
    }

or

    "foo:bar": "Ninja@en@"

## Namespace prefixes

    _ns:
      xsd: http://www.w3.org/2001/XMLSchema#

## Informal Grammar

```
Graph       = { ( Subject : PropertyMap )* Namespaces? } 
            | { ( "_id" : Subject ) ( Predicate : Objects )* Namespaces? }
PropertyMap = { ( Predicate : Objects )* }

Subject     = IRI
Predicate   = IRI | "a"
Objects     = Object | [ Object+ ]

Object      = IRI | SimpleString | StringWithDatatype | StringWithLanguage
            | { ( "_id" : IRI ) ( Predicate : Objects )* }
            | { ( Predicate : Objects )* }
            | { ( "_value" : String ) }
            | { ( "_value" : String ) ( "_type" : IRI ) }
            | { ( "_value" : String ) ( "_lang" : Language ) }

URIref      = FullURI | PrefixedURI | BlankNodeIdentifier
Namespaces  = ( "_ns" : { ( Prefix : FullURI )* } )
```

# References

* Graham Klyne; Jeremy J. Carroll (editors):
  *Resource Description Framework (RDF): Concepts and Abstract Syntax*.
  W3C Recommendation 10 February 2004
  <http://www.w3.org/TR/rdf-concepts/>

* Eric Prud'hommeaux; Gavin Carothers (editors):
  *Turtle. Terse RDF Triple Language*.
  W3C Candidate Recommendation 19 February 2013.
  <http://www.w3.org/TR/turtle/>

* Manu Sporny; Gregg Kellogg; Markus Lanthaler (editors):
  *JSON-LD 1.0. A JSON-based Serialization for Linked Data*.
  W3C Candidate Recommendation 10 September 2013.
  <http://www.w3.org/TR/json-ld/>

* Uniform Resource Identifier (URI) Schemes.
  <http://www.iana.org/assignments/uri-schemes/>

* BCP47: M. Davis (ed.): *Tags for Identifying Languages*.
  <http://tools.ietf.org/html/bcp47>


