# Introduction

This document defines an encoding of RDF graphs called **another RDF encoding
form (aRef)**. The encoding combines and simplifies ideas from existing RDF
serializations ([Turtle], [JSON-LD], and [RDF/JSON]). RDF data in aRef is
not serialized as Unicode string but encoded with data structures build of
[lists, maps, and strings]. This way an aRef document can be expressed both
in data structuring languages (JSON, YAML...) and in type systems of
programming languages (Python, Ruby, and Perl...).

[JSON-LD]: http://json-ld.org/
[RDF/JSON]: http://docs.api.talis.com/platform-api/output-types/rdf-json
[Turtle]: http://www.w3.org/TR/turtle/
[lists, maps, and strings]: #lists-maps-and-strings

## Status of this document

This specification is written in [Pandoc’s
Markdown](http://johnmacfarlane.net/pandoc/demo/example9/pandocs-markdown.html)
and managed with [makespec](https://github.com/jakobib/makespec). Updates and
sources can be found at <http://github.com/gbv/aref>. The current version of
this document was last modified at {GIT_REVISION_DATE} with revision
{GIT_REVISION_HASH}.

Please add and comment on issues to this specification at
<https://github.com/gbv/aref/issues>.


## Terminology

* Terms written in **bold** refer to defined concepts in their definition.
* Terms written in *italics* refer to concepts defined elsewhere in this document.
* Uppercase keywords (MUST, MAY, RECOMMENDED...) are used as defined in [RFC 2119].
* Syntax rules in this document are expressed using the [EBNF notation used
  by W3C](http://www.w3.org/TR/REC-xml/#sec-notation).
* The term **string** in this document always refers to Unicode strings as defined by [Unicode].

## RDF data

RDF is a graph-based data structuring languages defined as abstract syntax by
Klyne and Carroll ([2004](http://www.w3.org/TR/rdf-concepts/)). Several RDF
variants exist. RDF data as encoded by aRef can be defined as following:

* An **RDF graph** (also known as to as *RDF data*) is a set of *triples*.
* A **triple** (also known as *statement*) consists of a *subject*,
  a *predicate*, and an *object*.
* A **subject** is either an *IRI* or a *blank node*.
* A **predicate** (also known as *property*) is an *IRI*.
* An **object** is either an *IRI* or a *blank node* or a *literal node*.
* An **IRI** (Internationalized Resource Identifier) is a string
  that conforms to the IRI syntax defined in [RFC 3987].
* A **blank node** is neither an *IRI* nor a *literal node*.
* A **literal node** is a string tagged by either a
  *language tag* or by a *datatype*. A *simple literal* is a literal
  node with datatype `http://www.w3.org/2001/XMLSchema#string`.
* A **datatype** is an *IRI*.
* A **language tag** is a well-formed laguage tag as defined in [BCP 47].
* A **blank node** is neither an *IRI* nor a *literal node*.

This definition neither includes relative IRIs nor blank node identifiers as
known from some RDF serialization forms. To refer to a particular blank node
within the scope of the same RDF graph, aRef supports [identified blank
nodes](#identified-blank-nodes).

## Lists, maps, and strings

A ***list-map-structure*** is an abstract data structure build of

* strings,
* ***lists***, which are a sequences of zero or more *list-map-structures*,
* and ***maps***, which are sets of strings (the maps' *keys*)
  and a mapping from these keys *list-map-structures*.

Every aRef document MUST be given as *map*. Applications MAY restrict aRef to
non-recursive *list-map-structures*.

# Encoding

An *RDF graph* can be encoded in aRef as following.

## IRIs
[encoded IRI]: #iris

An *IRI* can be encoded as string, either in form of *absolute IRIs* or as
*prefixed names*.

An ***absolute IRI*** is either an IRI enclosed in angle brackets (`<` and
`>`), or an IRI that does not match any of the grammar rules
`languageTaggedString` or `datatypedString`.

A ***prefixed name*** consists of a **prefix** and a **name** separated by a
colon (`:`). The prefix is a string starting with a lowercase letter
(`a-z`) optionally followed by a sequence of lowercase letters and digits
(`0-9`).

      prefixedName  ::= prefix ":" name

      prefix        ::= [a-z] ( [a-z] | [0-9] )*

As key in a [predicate map](#predicate-maps) a prefixed name MAY also
use an underscore (“`_`”) instead of a colon.

      uPrefixedName ::= prefix "_" name

A name is a string that is either empty or conforms to the
following syntax:

      name          ::= nameStartChar nameChar*

      nameStartChar ::= [A-Z] | "_" | [a-z] | [#x00C0-#x00D6] | [#x00D8-#x00F6] |
                        [#x00F8-#x02FF] | [#x0370-#x037D] | [#x037F-#x1FFF] | 
                        [#x200C-#x200D] | [#x2070-#x218F] | [#x2C00-#x2FEF] | 
                        [#x3001-#xD7FF] | [#xF900-#xFDCF] | [#xFDF0-#xFFFD] |
                        [#x10000-#xEFFFF]

      nameChar      ::= nameStartChar | '-' | [0-9] | #x00B7 | 
                        [#x0300-#x036F] | [#x203F-#x2040]

This definition of prefixes is more restrictive than the
definition of prefixes in Turtle and JSON-LD. The restriction of
suffixes allows for absolute URI references such as
`geo:48.2010,16.3695,183` [RFC5870] and `tag:yaml.org,2002:int`
[RFC4151] while still using these URI scheme names as URI prefixes (e.g.
`geo:Point` and `tag:Tag`).

## Literal nodes

Literal nodes are encoded as strings conforming to the following rules:

      literalNode          ::= simpleString | languageTaggedString | datatypedString 

      simpleString         ::= string "@"?

      languageTaggedString ::= string "@" languageTag

      languageTag ::= [a-z]{2,8} ( "-" [a-z0-9]{1,8} )*

Note that the syntax rule of a language tag in aRef is slightly more
restrictive than the syntax of a language tag in [Turtle] but less restrictive
than the syntax of a language tag in JSON-LD, which refers to well-formed
language tags as defined in [BCP 47].

### Literal nodes with language tag

Literal nodes with language tag are encoded by appending the character
“`@`” and the language tag to the literal node’s string.

### Simple literals

A simple literal is encoded by optionally appending the character `@` to the
literal node’s string. The character “`@`” MUST be appended if the literal
node’s string matches the IRI syntax as defined in [RFC 3987] or the syntax of
an [identified blank node](#identified-blank-nodes). Simple literals MAY also
but SHOULD NOT be encoded as literal nodes with datatype IRI
`http://www.w3.org/2001/XMLSchema#string`.

### Literal nodes with datatype

A literal node with datatype is encoded by appending one or two
characters “`^`” and the datatype’s IRI either enclosed in “`<`” and
“`>`” or as prefixed name.

      datatypedString      ::= string "^" "^"? ( prefixedName | "<" IRI ">" )

Note that for some reason [Turtle] uses the character sequence “`^^`” instead
of a single “`^`” to serialize literal nodes with datatype.

## Blank nodes

Blank nodes can be encoded as *identified blank nodes* or as *anonymous blank
nodes*.

### Identified blank nodes

An **identified blank node** is a *blank node* that is given a **blank node
identifier** for referencing within the same *RDF graph*. Note that *blank node
identifiers* are always disjoint between different *RDF graphs*, so there is no
simple way to reference a particular *blank node* from an external document. An
*identified blank node* in aRef is encoded as a string that starts with “`_:`”
followed by the *blank node identifier*. The set of *blank node identifiers* is
limited to sequences of letters `a` to `z`, `A` to `Z`, and digits `0` to `9`.

      blankNodeIdentifier ::= "_:" ( [a-z] | [A-Z] | [0-9] )+

Note that this rule is more restrictive than the rule of blank node
identifiers in [Turtle] and in [JSON-LD].

### Anonymous blank nodes

An **anonymous blank node** in aRef is encoded by a [property
map](#property-maps) that does not include the key “`_id`”. In the simplest
case the *blank node* is encoded by an empty *map*.

## Graphs

An *RDF graph* is encoded as *map*. Two kinds of maps can be distinguished: a
*[subject map](#subject-maps)* contains zero or more *subjects* as *keys*,
similar to an *RDF graph* encoded in [RDF/JSON]. A [property map](#property-maps)
has a common *subject* and a set of one ore more *predicates* as *keys*. Both
kinds of *maps* can additionally contain a [namespace map](#namespace-maps). An
*RDF graph* encoded as *property map* MUST contain the special *key* “`_id`”.

### Subject maps

A **subject map** is a *map* where

-   every key, expect the optional key “`_ns`”, is an [encoded
    IRI](#iris),
-   and every value is a *property map* that does not contain the special
    *keys* “`_ns`” and “`_id`”.

### Property maps

A **property map** is a *map* where every *key*, except the optional *keys*
“`_ns`” and “`_id`”, is

-   either an [encoded IRI](#iris) (as absolute IRI or as prefixed name,
    also possible with underscore alternative to colon),
-   or the string “`a`” as alias for the IRI
    “`http://www.w3.org/1999/02/22-rdf-syntax-ns#type`”

and every value of these *keys* is an [encoded object](#encoded-objects).
A *property map* MUST NOT be empty, not counting the optional *keys*.

### Encoded objects

An **encoded object** in aRef represents one or multiple *objects* of RDF
*triples* with same *subject* and same *predicate*. An encoded object can be
any of:

-   an [encoded IRI](#iris),
-   a [literal node](#literal-nodes), encoded as string,
-   a [blank node](#blank-nodes), encoded as string,
-   a non-empty *list* of strings, each encoding an RDF *object*.

A *list* represents a set of RDF objects, so the order of elements is
irrelevant. A list SHOULD NOT contain the same RDF object multiple times
(as same string or in different encoding forms).

The object of a single RDF triple can be encoded both as string and as list of
one string. The former form is RECOMMENDED but the latter form is possible as
well, as known from [RDF/JSON].

### Namespace maps

    this section is not finished yet.

The special key `_ns` MAY be used in a *map* to indicate a namespace
map.

SHOULD/MUST (?) be at the top level of an aRef document (there can only
be one namespaceMap in an aRef document). TODO: What about recursive
documents?

...

The following namespace definitions are assumed by default:

-   xsd
-   rdf
-   rdfs
-   foaf
-   owl
-   dct
-   ...

# References

## Normative references

-   *The Unicode Standard, Version 6.3.0*
    The Unicode Consortium, 2013. ISBN 978-1-936213-08-5.
    <http://www.unicode.org/versions/Unicode6.3.0/>

-   Martin Duest; Michel Suignard: *Internationalized Resource
    Identifiers (IRIs)*. RFC3987, January 2005.
    <http://tools.ietf.org/html/rfc3987>

[Unicode]: http://www.unicode.org/versions/Unicode6.3.0/

-   BCP47: M. Davis (ed.): *Tags for Identifying Languages*.
    <http://tools.ietf.org/html/bcp47>

[BCP 47]: http://tools.ietf.org/html/bcp47

## Other references

-   Graham Klyne; Jeremy J. Carroll (editors): *Resource Description
    Framework (RDF): Concepts and Abstract Syntax*. W3C Recommendation
    10 February 2004
    <http://www.w3.org/TR/rdf-concepts/](http://www.w3.org/TR/rdf-concepts/>

-   Eric Prud’hommeaux; Gavin Carothers (editors): *Turtle. Terse RDF
    Triple Language*. W3C Candidate Recommendation 19 February 2013.
    <http://www.w3.org/TR/turtle/](http://www.w3.org/TR/turtle/>

-   Manu Sporny; Gregg Kellogg; Markus Lanthaler (editors): *JSON-LD
    1.0. A JSON-based Serialization for Linked Data*. W3C Candidate
    Recommendation 10 September 2013.
    <http://www.w3.org/TR/json-ld/](http://www.w3.org/TR/json-ld/>

-   Talis (publisher): *RDF JSON*.
    <http://docs.api.talis.com/platform-api/output-types/rdf-json>

-   Uniform Resource Identifier (URI) Schemes.
    <http://www.iana.org/assignments/uri-schemes/>

[RFC 3987]: http://tools.ietf.org/html/rfc3987
[RFC 2119]: http://tools.ietf.org/html/rfc2119


`examples.md`{.include}

`appendix.md`{.include}

