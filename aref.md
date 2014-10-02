# Introduction

This document defines an encoding of [*RDF graphs*] called **another RDF
encoding form (aREF)**. The encoding combines and simplfies best parts of
existing RDF serializations [Turtle], [JSON-LD], and [RDF/JSON]. In contrast to
these formats, RDF data in aREF is not serialized as a Unicode string but
encoded as a [*list-map-structure*], as supported by the type system of most
programming languages and by data structuring languages such as JSON and YAML.

The specification of aREF is hosted in a public git repository at <{GITHUB}>,
written in in [Pandoc’s
Markdown](http://johnmacfarlane.net/pandoc/demo/example9/pandocs-markdown.html)
and managed with [makespec](https://github.com/jakobib/makespec). Please add
and comment on issues to this specification at
<https://github.com/gbv/aREF/issues>.  The most recent version of this document
is made available at <http://gbv.github.io/aREF/>.

# Background

## Terminology

Terms written in "**bold**" refer to terms at the place of their definition in
this document. Terms written in "*italics*" refer to terms defined elsewhere in
this document. Uppercase keywords (MUST, MAY, RECOMMENDED, SHOULD...) are used
as defined in [RFC 2119].  Syntax rules in this document are expressed in ABNF
notation as specified by [RFC 5234]. 

The term **string** in this document always refers to Unicode strings
as defined by [Unicode]. A *string* can also be defined with the following
syntax rule:

    string = *( %x0-%x10FFFF )

Strings SHOULD be in Normal Form C ([NFC]). Applications MAY restrict strings
by disallowing selected Unciode codepoints, such as the 66 Unicode
noncharacters or the set of Unicode characters not expressible in XML.

## RDF data

[*RDF graphs*]: #rdf-data

RDF is a graph-based data structuring languages defined as abstract syntax by
Klyne and Carroll ([2004](http://www.w3.org/TR/rdf-concepts/)). Several RDF
variants exist (in particular see [Wood, 2013](http://www.w3.org/TR/rdf11-new/)
for a comparision between RDF 1.0 and RDF 1.1). RDF extensions with named
graphs, blank nodes as predicates, and literal nodes as subjects are not
covered by this specification nor expressible in aREF.

RDF data as encoded by aREF is defined as following:

* An **RDF graph** is a set of *triples*.
* A **triple** (also known as "statement") consists of a *subject*,
  a *predicate*, and an *object*.
* A **subject** is either an *IRI* or a *blank node*.
* A **predicate** (also known as "property") is an *IRI*.
* An **object** is either an *IRI* or a *blank node* or a *literal node*.
* An **IRI** (Internationalized Resource Identifier) is a *string*
  that conforms to the IRI syntax defined in [RFC 3987].
* A **blank node** is neither an *IRI* nor a *literal node*.
* A **literal node** is a *string* tagged by either a
  *language tag* or by a *datatype*. 
* A **simple literal** is a literal
  node with datatype `http://www.w3.org/2001/XMLSchema#string`.
* A **datatype** is an *IRI*.
* A **language tag** is a well-formed laguage tag as defined in [BCP 47].

An *RDF graph* encoded in aREF can also include [*blank node
identifiers*](#blank-nodes) to refer to particular blank nodes within the scope
of the same *RDF graph*.  

## Lists-map-structures

[*list-map-structure*]: #lists-map-structures

A **list-map-structure** is an abstract data structure build of

* *strings*, which are Unicode strings,
* **lists**, which are a sequences of zero or more *list-map-structures*,
* and **maps**, which are sets of *strings* (the maps' **keys**)
  and a mapping from these *keys* to *list-map-structures*.

Every aREF document MUST be given as *map*. Applications MAY restrict aREF
documents to non-circular *list-map-structures*. All non-circular
*list-map-structures* can be serialized in JSON and YAML, amomg other formats.

# Encoding

## IRIs

An *IRI* in aREF is encoded as *string*,

* either as *plain IRI*
* or as *explicit IRI* enclosed in angular brackets
* or as [*qname*](#qnames).

### Plain IRIs

[*plain IRI*]: #plain-iris

An **plain IRI** is either an IRI given as string. The syntax rule `IRILike`
can be used to distinguish plain IRIs from literal strings.

An **explicit IRI** is an IRI enclosed in in angle brackets (`<` and `>`).

      IRIlike       = LOWERCASE *( LOWERCASE / DIGIT / "+" / "." / "-" ) ":" [ string ]
                        ; MUST also match IRI syntax rule from RFC 3987
                        ; MUST NOT match syntax rule literalNode

      explicitIRI   = "<" IRI ">"   ; IRI syntax rule from RFC 3987

      LOWERCASE     = %x61-%x7A     ; a-z

### QNames

[*qname*]: #qnames

A **qname** consists of a **prefix** and a **name** separated by an underscore
(`_`): 

      qName  = prefix "_" name

The special string “`a`” if given as *key* in a *predicate map* is always
an alias for the IRI "`http://www.w3.org/1999/02/22-rdf-syntax-ns#type`".

The prefix is a string starting with a
lowercase letter (`a-z`) optionally followed by a sequence of lowercase letters
and digits (`0-9`).

      prefix        = LOWERCASE *( LOWERCASE / DIGIT )    ; a-z *( a-z / 0-9 )

A name is a string that is either empty or conforms to the following syntax:

      name          = nameStartChar *(nameChar)

      nameStartChar = ALPHA / "_" / %x00C0-%x00D6 / %x00D8-%x00F6 /
                      %x00F8-%x02FF / %x0370-%x037D / %x037F-%x1FFF / 
                      %x200C-%x200D / %x2070-%x218F / %x2C00-%x2FEF / 
                      %x3001-%xD7FF / %xF900-%xFDCF / %xFDF0-%xFFFD /
                      %x10000-%xEFFFF

      nameChar      = nameStartChar / '-' / DIGIT / %xB7 / %x0300-%x036F / %x203F-%x2040

This definition is more restrictive than corresponding definitions in Turtle
and JSON-LD. The restriction allows for both, URI scheme names as URI prefixes
(e.g. "`geo:Point`" and "`tag:Tag`") and absolute URI references such as
"`geo:48.2010,16.3695,183`" ([RFC 5870]) and "`tag:yaml.org,2002:int`" ([RFC 4151]).

***TODO:** See [*namespace map*] for how to construct IRIs from qnames*

Applications SHOULD warn about unknown prefixes. Applications MAY ignore all
triples that include a node with an unknown prefix.


## Graphs

An *RDF graph* in aREF is encoded as a [*list-map-structure*] that is either a
[*subject map*], or a [*predicate map*] that MUST contain the special *key*
"`_id`". The special key "`_ns`" MAY further be used to specify a [*namespace
map*].

## Subject maps

[*subject map*]: #subject-maps

A **subject map** is a *map* with the following constraints:

-   Every *key*, except the optional key "`_ns`", is an encoded IRI (as
    [*plain IRI*] or as [*qname*]), or a 
    [*blank node identifier*]. These *keys* encode *subjects* of *triples*
    encoded by the *subject map*.

-   Every value, except if mapped from the optional key "`_ns`", is an 
    [*predicate map*] with the following constraint:

    -   It MUST NOT contain the special key "`_ns`".

    -   If it contains the special *key* "`_id`" then the *key* must
        be mapped to an encoding of the same subject IRI.

## Predicate maps

[*predicate map*]: #predicate-maps

A **predicate map** is a *map* with the following constraints:

-   Every *key*, except the optional keys "`_id`" and "`_ns`", MUST be an
    encoded IRI (as [*plain IRI*], or as [*qname*]). These *keys* encode *predicates* of *triples* encoded 
    by the *predicate map*.

-   Every value, except if mapped from the optional keys "`_id`" or "`_ns`",
    must be an [*encoded object*].

-   The optional key "`_id`", if given, is mapped to an encoded IRI 
    (as [*plain IRI*] or as [*qname*]) or to a 
    [*blank node identifier*]. This value specifies the *subject* of all 
    *triples* encoded by the *predicate map*. 
    If the key does not exist, the *subject* is either given implicitly
    because the *predicate map* is used as part of a [*subject map*] or
    the *subject* is a *blank node*.

## Namespace maps

[*namespace map*]: #namespace-maps

A **namespace map** in aREF is either a *string* that defines a default
namespace, or a *map* where every *key* conforms to the `prefix` syntax rule
(see [*qname*]) and is mapped to an IRI, given as string that conforms
to the `IRI` syntax rule from [RFC 3987]. The IRI is called **namespace URI**.
A *namespace map* can be specified both, explicitly with the special key
"`_ns`" in a [*subject map*] or in a [*predicate map*], and implicitly by
assuming a [predefined namespace map].

## Encoded objects

[*encoded object*]: #encoded-objects

An **encoded object** in aREF represents one or multiple *objects* of RDF
*triples* with same *subject* and same *predicate*. An encoded object can be
any of:

-   a IRI, encoded as [*plain IRI*] or as [*qname*],
-   a [literal node](#literal-nodes), encoded as string,
-   a [blank node](#blank-nodes), encoded as string,
-   a *list* with each element is a string encoding an RDF *object*
    with any of the three methods above.

A *list* represents a set of RDF objects, so the order of elements is
irrelevant. A list SHOULD NOT contain the same RDF object multiple times
(as same string or in different encoding forms).

The object of a single RDF triple can be encoded both as string and as list of
one string. The former form is RECOMMENDED but the latter form is possible as
well, as known from [RDF/JSON].

***TODO**: restrict encoding of IRI in contrast to IRI as subject or predicate.*

## Literal nodes

A literal node in aREF is encoded as string in one of three forms:

      literalNode   = plainLiteral / languageString / datatypeString

### Simple literals

A *simple literal* with datatype `http://www.w3.org/2001/XMLSchema#string` MAY
be encoded as literal node with datatype or by appending an at sign (`@`) to
the simple literal’s string. If the simple literal’s string neither ends with
an at sign (`@`) nor matches to any of the syntax rules of [*plain IRI*]
(`IRIlike` or `explicitIRI`), [*qname*] (`qName`) and [*identified blank
node*] (`blankNode`), the string SHOULD be used as given. 

      plainLiteral   = string "@" / string
                       ; MUST NOT end with "@"
                       ; MUST NOT match syntax rule 
                         explicitIRI, IRIlike, qName, or blankNode

### Literal nodes with language tag

Literal nodes with *language tag* are encoded by appending an at sign (`@`)
followed by the language tag to the literal node’s string:

      languageString = string "@" languageTag

      languageTag    = 2*8(ALPHA) *( "-" 1*8( ALPHA / DIGIT ) )

Note that the syntax rule of a language tag in aREF is slightly more
restrictive than the syntax of a language tag in [Turtle] but less restrictive
than the syntax of a language tag in JSON-LD, which refers to well-formed
language tags as defined in [BCP 47].

### Literal nodes with datatype

A literal node with *datatype* is encoded by appending one carets (`^`)
followed by the datatype’s IRI either enclosed in “`<`” and “`>`” or as
[*qname*]:

      datatypeString = string "^" ( qName / "<" IRI ">" )

Note that [Turtle] uses the character sequence “`^^`” instead of a single “`^`”.

*TODO: use IRILike instead of IRI*

## Blank nodes

[*identified blank node*]: #blank-nodes
[*blank node identifier*]: #blank-nodes

A *blank nodes* in aREF is encoded as string in form of a *blank node
identifier* or by using a [*predicate map*] that either does not contain the
special key "`_id`" or contains the special key "`_id`" mapped to a *blank node
identifier*.

A **blank node identifier** is a string conforming to the following syntax
rule. Note that the syntax rule `blankNode` is more
restrictive than the rule of blank node identifiers in [Turtle] and in
[JSON-LD]:

      blankNode      = "_:" 1*( ALPHA / DIGIT )

Within the scope of the same *RDF graph*, equal *blank node identifiers* MUST
refer to the same *blank node*. *Blank node identifiers* MUST NOT be shared
among different *RDF graphs*. It is RECOMMENDED to avoid *blank node
identifiers* in an aREF document if the *blank node* encoded by a *blank node
identifier* is never referenced in the same RDF graph. In the simplest case, a
*blank node* in aREF can be encoded as an empty *map*.

## Predefined namespace maps

[predefined namespace map]: #predefined-namespace-maps

The following [*namespace map*] MUST be assumed implicitly:

prefix  namespace URI
------- --------------------------------------------
rdf     http://www.w3.org/1999/02/22-rdf-syntax-ns#
rdfs    http://www.w3.org/2000/01/rdf-schema#
owl     http://www.w3.org/2002/07/owl#
xsd     http://www.w3.org/2001/XMLSchema#
------- --------------------------------------------

Applications MAY predefine additional implicit namespace maps. Mappings in an
explicit [*namespace map*] precedence over implicit mappings.

See <http://prefix.cc> and [RDF::NS] for additional common namespace mappings.

Other sources: 
<http://stats.lod2.eu/vocabularies>,
<http://prefix.cc/popular/all.txt>, 
<http://www.w3.org/2011/rdfa-context/rdfa-1.1>, ...

# References

## Normative references

-   *The Unicode Standard, Version 6.3.0*
    The Unicode Consortium, 2013. ISBN 978-1-936213-08-5.
    <http://www.unicode.org/versions/Unicode6.3.0/>

[Unicode]: http://www.unicode.org/versions/Unicode6.3.0/

-   Martin Düst; Michel Suignard: 
    *Internationalized Resource Identifiers (IRIs)*.
    RFC3987, January 2005.
    <http://tools.ietf.org/html/rfc3987>

[RFC 3987]: http://tools.ietf.org/html/rfc3987

-   A. Phillips; M. Davis:
    *Tags for Identifying Languages*. 
    BCP 47, September 2009.
    <http://tools.ietf.org/html/bcp47>

[BCP 47]: http://tools.ietf.org/html/bcp47

-   Mark Davis; Martin Düst:
    *Unicode Normalization Forms*.
    Unicode Standard Annex #15
    <http://www.unicode.org/unicode/reports/tr15/>

[NFC]: http://www.unicode.org/unicode/reports/tr15/

-   D. Crocker; P. Overell:
    *Augmented BNF for Syntax Specifications: ABNF*.
    RFC5234, 2010.
    <http://tools.ietf.org/html/rfc5234>

[RFC 5234]: http://tools.ietf.org/html/rfc5234

## Other references

-   Graham Klyne; Jeremy J. Carroll (editors): 
    *Resource Description Framework (RDF): Concepts and Abstract Syntax*. 
    W3C Recommendation, 10 February 2004
    <http://www.w3.org/TR/rdf-concepts/>

-   Eric Prud’hommeaux; Gavin Carothers (editors):
    *Turtle. Terse RDF Triple Language*. 
    W3C Candidate Recommendation, 19 February 2013.
    <http://www.w3.org/TR/turtle/>

-   Manu Sporny; Gregg Kellogg; Markus Lanthaler (editors): 
    *JSON-LD 1.0. A JSON-based Serialization for Linked Data*.
    W3C Candidate Recommendation, 10 September 2013.
    <http://www.w3.org/TR/json-ld/>

-   Ian Davis; Thomas Steiner; Arnaud J Le Hors (editors):
    *RDF 1.1 JSON Alternate Serialization (RDF/JSON)*.
    W3C Working Group Note, 07 November 2013.
    <http://www.w3.org/TR/rdf-json/>

-   David Boom (editor): *What’s New in RDF 1.1*.
    W3C Working Draft, 17 December 2013.
    <http://www.w3.org/TR/rdf11-new/>

-   Graham Klyne:
    *Uniform Resource Identifier (URI) Schemes*.
    IANA, 21 October 2013.
    <http://www.iana.org/assignments/uri-schemes/>

-   *RDFa Core Initial Context. Vocabulary Prefixes*.
    <http://www.w3.org/2011/rdfa-context/rdfa-1.1>

-   Jakob Voß: *RDF-aREF*. CPAN Perl Module.
    <https://metacpan.org/release/RDF-aREF>

-   JSON ...

-   YAML ...

[RFC 2119]: http://tools.ietf.org/html/rfc2119
[RFC 4151]: http://tools.ietf.org/html/rfc4151
[RFC 5870]: http://tools.ietf.org/html/rfc5870

[Turtle]: http://www.w3.org/TR/turtle/
[JSON-LD]: http://json-ld.org/
[RDF/JSON]: http://www.w3.org/TR/rdf-json/


`examples.md`{.include}

`appendix.md`{.include}

