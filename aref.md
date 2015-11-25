# Introduction

This document defines an encoding of [*RDF graphs*] called **another RDF
encoding form (aREF)**. The encoding combines and simpilfies parts of existing
RDF serializations [Turtle], [JSON-LD], and [RDF/JSON]. In contrast to these
formats, RDF data in aREF is not serialized as a Unicode string but encoded as
a [*list-map-structure*], as known from the type system of most programming
languages and from data structuring languages such as [JSON] and [YAML].

This specification of aREF is hosted in a public git repository at <{GITHUB}>,
written in in [Pandoc’s
Markdown](http://johnmacfarlane.net/pandoc/demo/example9/pandocs-markdown.html)
and managed with [makespec](https://github.com/jakobib/makespec). Please add
and comment on issues to this specification at
<https://github.com/gbv/aREF/issues>.  The most recent version of this document
is made available at <http://gbv.github.io/aREF/>.

# Background

## Terminology

[*string*]: #terminology
[*strings*]: #terminology

Terms written in "**bold**" refer to terms at the place of their definition in
this document. Terms written in "*italics*" refer to terms defined elsewhere in
this document. Uppercase keywords (MUST, MAY, RECOMMENDED, SHOULD...) are used
as defined in [RFC 2119].  Syntax rules in this document are expressed in ABNF
notation as specified by [RFC 5234].

<div class="note">
Examples and notes in this document are informative only. YAML syntax
is used to express sample aREF documents, unless noted otherwise.
</div>

The following syntax rules are referenced later in this document:

    string = *( %x0-%x10FFFF )

    LOWERCASE = %x61-%x7A ; a-z

The term **string** in this document always refers to Unicode strings as
defined by [Unicode]. A *string* can also be defined with syntax rule `string`.

Strings SHOULD always be normalized to Normal Form C ([NFC]). Applications MAY
restrict *strings* by disallowing selected Unciode codepoints, such as the 66
Unicode noncharacters or the set of Unicode characters not expressible in XML.

## RDF data

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
* An **IRI** (Internationalized Resource Identifier) is a [*string*]
  that conforms to the IRI syntax defined in [RFC 3987].
* A **blank node** is neither an *IRI* nor a *literal node*.
* A **literal node** is a [*string*] tagged by either a
  *language tag* or by a *datatype*. 
* A **simple literal** is a literal
  node with datatype `http://www.w3.org/2001/XMLSchema#string`.
* A **datatype** is an *IRI*.
* A **language tag** is a well-formed laguage tag as defined in [BCP 47].

An *RDF graph* encoded in aREF can also include [*blank node
identifiers*](#blank-nodes) to refer to particular blank nodes within the scope
of the same *RDF graph*.  

<div class="example">
Ask a Semantic Web or Linked Data evangelist for examples of RDF!
</div>

## Lists-map-structures

[*list-map-structure*]: #lists-map-structures
[*list-map-structures*]: #lists-map-structures

A **list-map-structure** is an abstract data structure build of

* [*strings*], which are Unicode strings,
* **lists**, which are a sequences of zero or more *list-map-structures*,
* and **maps**, which are sets of [*strings*] (the maps' **keys**)
  and a mapping from these *keys* to *list-map-structures*.

Every aREF document MUST be given as *map*. Applications MAY restrict aREF
documents to [*non-circular*] *list-map-structures*. All non-circular
*list-map-structures* can be serialized in [JSON] and [YAML].

Applications MAY support special null values, disjoint from [*strings*], as
element in a [*list*] and/or mapped to in a [*map*]. These null values MUST be
ignored on decoding aREF.

<div class="example">
See section [aREF document types](#aref-document-types) and 
[appendix aREF serializations](#aref-serializations) for examples.
</div>

# Encoding

## IRIs

An [*IRI*] in aREF is encoded as [*string*], either as [*plain IRI*], or as
[*explicit IRI*], or as [*qName*]. The special [*string*] “`a`” can further be
used to encode the [*predicate*]
“`http://www.w3.org/1999/02/22-rdf-syntax-ns#type`”.

### Plain IRIs

[*plain IRI*]: #plain-iris

A **plain IRI** is an [*IRI*], as defined in [RFC 3987]. If used as [*object*],
a *plain IRIs* MUST conform to the syntax rule `IRILike` to distinguish from a
[*literal node*].

      IRIlike = LOWERCASE *( LOWERCASE / DIGIT / "+" / "." / "-" ) ":" [ string ]

### Explicit IRIs

An **explicit IRI** is an [*IRI*] enclosed in in angle brackets (“`<`” and “`>`”).

      explicitIRI   = "<" IRI ">"   ; IRI syntax rule from RFC 3987

Applications MAY use the syntax rule `IRILike` instead of `IRI` to facilitate
decoding aREF.

### qNames

A **qName** consists of a **prefix** and a **localName** separated by an
underscore (“`_`”): 

      qName  = prefix "_" localName

The *prefix* is a [*string*] starting with a lowercase letter (`a-z`)
optionally followed by a sequence of lowercase letters and digits (`0-9`).

      prefix = LOWERCASE *( LOWERCASE / DIGIT )    ; a-z *( a-z / 0-9 )

The *localName* is a [*string*] that conforms to the following syntax. 

      localName     = nameStartChar *(nameChar)

      nameStartChar = ALPHA / "_" / %x00C0-%x00D6 / %x00D8-%x00F6 /
                      %x00F8-%x02FF / %x0370-%x037D / %x037F-%x1FFF / 
                      %x200C-%x200D / %x2070-%x218F / %x2C00-%x2FEF / 
                      %x3001-%xD7FF / %xF900-%xFDCF / %xFDF0-%xFFFD /
                      %x10000-%xEFFFF

      nameChar      = nameStartChar / '-' / DIGIT / %xB7 / %x0300-%x036F / %x203F-%x2040

<div class="note">
The syntax rule `localName` is  more restrictive than corresponding definitions
in [Turtle] and [JSON-LD].
</div>

<!--
The restriction allows for both, URI scheme names as URI prefixes
(e.g. "`geo:Point`" and "`tag:Tag`") and absolute URI references such as
"`geo:48.2010,16.3695,183`" ([RFC 5870]) and "`tag:yaml.org,2002:int`" ([RFC 4151]).
-->

A *qName* is mapped to an [*IRI*] by appending its *localName* to the
[*namespace URI*] that corresponds to its *prefix*. Applications SHOULD warn
about unknown prefixes and/or ignore all triples that include a node with an
unknown prefix.

## Literal nodes

A [*literal node*] is encoded as [*string*] in one of three forms:

      literalNode   = languageString / datatypeString / plainLiteral

### Literal nodes with language tag

A [*literal node*] with [*language tag*] is encoded by appending an at sign
("`@`") followed by the [*language tag*] to the node’s [*string*]:

      languageString = string "@" languageTag

      languageTag    = 2*8(ALPHA) *( "-" 1*8( ALPHA / DIGIT ) )

<div class="example">
```json
{
  "_id": "http://example.com/MyResource",
  "skos_prefLabel": [
    "east@en",
    "Osten@de"
    "東@ja",
    "東@ja-Hani",
    "ヒガシ@ja-Kana",
    "higashi@ja-Latn"
  ]
}
```
</div>

<div class="note">
The syntax rule `languageTag` is slightly more
restrictive than the syntax of a language tag in [Turtle] but less restrictive
than the syntax of a language tag in JSON-LD, which refers to well-formed
language tags as defined in [BCP 47].
</div>

### Literal nodes with datatype

A [*literal node*] with *datatype* is encoded by appending a caret ("`^`")
followed by the datatype’s [*IRI*] either [*explicit IRI*] or as [*qName*]:

      datatypeString = string "^" ( qName / explicitIRI )

<div class="example">
```json
{
  "_id": "http://example.org/",
  "dct_modified": [
    "2010-05-29T14:17:39+02:00^xsd_dateDate",
    "2010-05-29^<http://www.w3.org/2001/XMLSchema#date>"
  ]
}
```
</div>

<div class="note">
[Turtle] uses the character sequence “`^^`” instead of a single “`^`”.
</div>

### Simple literals

A [*simple literal*] is encoded either as [literal node with
datatype](#literal-nodes-with-datatype)
“`http://www.w3.org/2001/XMLSchema#string`” or as [*string*] that conforms to
the `plainLiteral` syntax rule. The syntax MUST BE disjoint to the syntax rules
`languageString` and `datatypeString` and to the syntax rules of [IRIs](#iris)
(`explicitIRI`, `IRIlike`, `qName`) and [blank nodes](#blank-nodes)
(`blankNode`).

      plainLiteral = string / string "@" ; MUST NOT match any of rules
                                         ; languageString, datatypeString, 
                                         ; explicitIRI, IRIlike, qName
                                         ; blankNode
 
An at sign ("`@`") can always be appended to the node’s [*string*] to distinguish
from other syntax rules. The at sign MUST be appended if the simple literal
ends with an at sign.

<div class="example">

 aREF string           RDF literal (Turtle syntax)
 --------------------- ----------------------------------
 `@`                   `""`
 `*empty string*`      `""`
 `^xsd_string`         `""`
 `@@`                  `"@"`
 `@^xsd_string`        `"@"`
 `alice@en`            `"alice"@en`
 `alice@example.com`   `"alice@example.com"`
 `123`                 `"123"`
 `忍者@ja`             `"忍者"@ja`
 `Ninja@en`            `"Ninja"@en`
 `Ninja@en@`           `"Ninja@en"`

</div>

## Blank nodes

A [*blank node*] is encoded 

* either as [*predicate map*] without the key "`_id`",
* or as **blank node identifier**, that is a [*string*] which starts with "`_:`", 
  followed by a alphanumerical label (syntax rule `blankNode`),
* or as [*predicate map*] with the key "`_id`" mapped to [*blank node identifier*].

```
blankNode      = "_:" 1*( ALPHA / DIGIT )
```

Within the scope of the same [*RDF graph*], equal *blank node identifiers* MUST
refer to the same *blank node*. *Blank node identifiers* SHOULD NOT be shared
among different *RDF graphs*. 

In the simplest case, a *blank node* in aREF can be encoded as an empty *map*.

<div class="example">
```yaml
_ns:
    foaf: http://xmlns.com/foaf/0.1/
_:alice:
    foaf_knows: _:bob
_:bob:
    foaf_knows:
        _id: _:alice
```
</div>

<div class="example">
```yaml
_ns:
    foaf: http://xmlns.com/foaf/0.1/
_:someone
    foaf_knows:
        foaf_name: "Bob"
```
</div>

<div class="note">
The syntax rule `blankNode` is more restrictive than the rule of
blank node identifiers in [Turtle] and in [JSON-LD].
</div>

## Graphs

An [*RDF graph*] in aREF is encoded as a [*list-map-structure*] that is

* either a [*subject map*],
* or a [*predicate map*] that MUST contain the special *key* "`_id`".

### Subject maps

A **subject map** is a [*map*] with the following constraints:

1.  The [*subject map*] MUST NOT contain the [*key*] "`_id`".

2.  The [*subject map*] MAY contain the key [*key*] "`_ns`", mapped to
    a [*namespace map*].

3.  Additional keys, starting with `_` and not with `_:` SHOULD be ignored.

4.  Every other [*key*] is either a [*plain IRI*] or a [*qName*] or 
    a [*blank node*]. These [*keys*] encode the [*subjects*] of 
    RDF [*triples*].

5.  Every value of a [*key*] that encodes a [*subject*] MUST BE a 
    [*predicate map*] that either does not contain the [*key*] "`_id`"
    or maps the [*key*] "`id`" to an encoding of the same subject.

<div class="example">
```yaml
"http://example.org/alice":
    foaf_knows: http://example.org/bob
    _id: http://example.org/alice  # redundant
```
</div>

### Predicate maps

A [*predicate map*] encodes a set of RDF triples with same [*subject*]. The
subject is given by context, if the [*predicate map*] is part of a [*subject
map*], or explicitly with the [*key*] "`_id`", or the subject is a [*blank
node*].

A **predicate map** is a [*map*] with the following constraints:

1.  The optional [*key*] "`_id`", if given, MUST be mapped to a [*plain IRI*], 
    a [*qName*], or a [*blank node*].

2.  The optional [*key*] "`_ns`", if given, MUST be mapped to a 
    [*namespace map*].

3.  Additional keys, starting with `_` SHOULD be ignored.

4.  Every [*key*], unless it starts with "`_`", MUST be either a 
    [*plain IRI*] or a [*qName*], or the value "`a`" that stands 
    for the [*IRI*] "`http://www.w3.org/1999/02/22-rdf-syntax-ns#type`".
    These [*keys*] encode [*predicates*] of [*triples*].

5.  Every value of a [*key*] that encodes a [*predicate*] MUST BE an
    [*encoded object*].


<div class="example">
```json
{
  "_id": "http://example.org/places#BrewEats",
  "a": [ "http://schema.org/Restaurant", "http://schema.org/Brewery" ]
}
```
</div>

### Encoded objects

[*encoded object*]: #encoded-objects
[*encoded objects*]: #encoded-objects

An **encoded object** encodes zero or more RDF [*objects*] with same
[*subject*] and same [*predicate*]. An [*encoded object*] MUST BE one of, or a
[*list*] of any of the following:

* a [*plain IRI*], [*explicit IRI*], or [*qName*] to encode an [*IRI*]
* a [*string*] that conforms to syntax rule `literalNode` to encode a 
  [*literal node*] (see [literal nodes](#literal-nodes))
* a [*string*] that conforms to syntax rule `blankNode` to encode a
  [*blank node*] (see [blank nodes](#blank-nodes))
* a [*predicate map*]

A [*list*] as [*encoded object*] represents a set of [*objects*], so the order
of elements is irrelevant and duplicates SHOULD NOT be included, independent
from different encoding forms.

<div class="example">
The following encoded objects, expressed in JSON, refer to the same [*IRI*]:

* `http://example.org/`
* `<http://example.org/>`
* `{ "_id": "http://example.org/" }`
* `[ "http://example.org/" ]`
* `[ "<http://example.org/>" ]`
* `[ { "_id": "http://example.org/" } ]`
</div>

## Namespace maps

A *namespace map* can be specified explicitly with the special key "`_ns`" in a
[*subject map*] or in a [*predicate map*].  An aREF document MUST NOT contain
more than one explicit *namespace map*.

A **namespace map** is 

* either a *map* in which every *key* conforms to the `prefix` syntax rule (see
  [*qName*]) and is mapped to an [*IRI*] (syntax rule `IRI` from [RFC 3987]).
  The *IRIs* in a *namespac map* are also called **namespace URIs**. The special
  key underscore (`_`) can further be used to refer to another predefined 
  namespace map given by a *string*. This *string* is also called **namespace
  map identifier**. Mappings explicitly given with *namespace URI*
  precedence over mappings refered to by a *namespace map identifier*.

* or a *namespace map identifier* that refers to a predefined namespace map.

Applications MAY further assume an implicit *namespace map*. Mappings from an
implicit *namespace map* can be overriden by explicit *namespace maps*.  The
following implicit *namespace map* or a superset of it SHOULD be assumed by
default:

```json
{
  "rdf":  "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
  "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
  "owl":  "http://www.w3.org/2002/07/owl#",
  "xsd":  "http://www.w3.org/2001/XMLSchema#"
}
```

**TODO:** *should the default namespace map always precede namespace maps
given by namespace map identifier, so applications can always assume they
are right?*

<div class="example">
The following namespace maps are equivalent:

* `"example"`
* `{ "_": "example" }`

A commonly used *namespace map* is listed at 
<http://www.w3.org/2011/rdfa-context/rdfa-1.1>. If the the *namespace
map identifier* <http://www.w3.org/2013/json-ld-context/rdfa11> refers
to this map, it can be used in aREF as following (examples in YAML):

```
_ns: http://www.w3.org/2013/json-ld-context/rdfa11
```

Custom prefixes can be added and existing prefixes redefined like this:

```
_ns: 
  _: http://www.w3.org/2013/json-ld-context/rdfa11
  dc: http://purl.org/dc/elements/1.1/ # instead of http://purl.org/dc/terms/
  dct: http://purl.org/dc/terms/       # additional prefix
```
</div>

<div class="note">
This specification does *not* include rules how to resolve *namespace maps
identifiers*. The following guidelines are non normative:

* An URL is expected to refer to a JSON-LD document with a 
  [\@context element](http://www.w3.org/TR/json-ld/#the-context). 
  For instance the default aREF *namespace map* could be expressed like this:

    ```json
    {
      "@context": {
        "rdf":  "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
        "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
        "owl":  "http://www.w3.org/2002/07/owl#",
        "xsd":  "http://www.w3.org/2001/XMLSchema#"
      }
    }
    ```

    Note that JSON-LD context documents for particular ontologies usually
    define abbreviations for full URIs and/or default vocabularies (`@vocab`) 
    that cannot be used in aREF documents because a [*qNames*] MUST consists 
    of prefix *and* local name.

* A string of the form `YYYYMMDD` is expected to refer to the *namespace 
  map* defined at this date at <http://prefix.cc> (see [rdfns](https://metacpan.org/pod/rdfns),
  available as package 
  [librdf-ns-perl](https://packages.debian.org/search?searchon=names&keywords=librdf-ns-perl) 
  in Debian for a related command line tool).
  For instance the identifier "20140901" maps prefix "fabio" to
  <http://purl.org/spar/fabio/> and the identifier "20120521" maps it to
  <http://purl.org/spar/fabio#>.

</div>

# aREF document types

*THIS PART OF THE SPEC IS NOT FINISHED YET*

[*circular*]: #aref-document-types
[*non-circular*]: #aref-document-types

Depending on their structure, aREF documents can be classified as
*circular* or *non-circular*, as *flat*, as *consistent*, and as
*normalized*.

An aREF document is **circular** if there is at least one path from a
[*subject map*] to itself by stepping to a next [*subject maps*] that
is part of an [*encoded objects*] of the previous [*subject map*].

<div class="example">
A minimal circular aREF document can be created in JavaScript as following:

```javascript
var aref = { _id: "http://example.org/alice" };
aref.foaf_knows = aref; # alice knows herself
```

Circular aREF documents cannot be serialized in JSON but in YAML, for 
instance this *normalized circular* aREF document:

```yaml
http://example.org/alice: &alice
    foaf_knows: &bob         # alice knows bob
        foaf_knows: *alice   # bob knows alice
http://example.org/bob: *bob
```
</div>

An aREF document is **flat** if all of its [*encoded objects*] are
encoded as [*strings*]. All *flat* aREF documents are *non-circular*.

<div class="note">
The [*list-map-structure*] of a flat aREF document can at most be nested in two
levels, if it is a [*subject map*] and at most one level, if it is a
[*predicate map*]:

```yaml
{
  "http://example.org/": {    # first level: predicate map
    "dct_title": [            # second level: list of encoded objects
      "example@en",
      "Beispiel@de"
    ]
  }
}
```
</div>

An aREF document (or its IRIs) is/are **consistent** iff ... 
same IRI should be encoded the same way (but subtle differences is used 
as subject, predicate, and object)*

<div class="example">
...
</div>

An aREF document is **normalized** according to a given [*namespace map*] if

1. The document must be a [*subject map*]

2. The document contains no null values or ignored keys

3. Its IRIs are encoded consistently

4. All lists have at least two members

4. *what about `_ns`*?

5. The document is 

    - either *flat* and no predicate map contains the key `_id`
      ("*normalized form 1*)

    - or *normalized form 2*:

        - all predicate maps must contain the key `_id` and
          at least one more predicate key

        - all predicate maps must directly be mapped from
          a keys in the subject map.

<div class="note">
...better names for the two forms...
</div>

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

-   Jakob Voß: *RDF-NS*. CPAN Perl Module.
    <https://metacpan.org/release/RDF-NS>


-   Douglas Crockford: *The application/json Media Type for JavaScript Object
    Notation (JSON)*. RFC 4627, July 2006. <https://tools.ietf.org/html/rfc4627>

-   Oren Ben-Kiki, Clark Evens, Ingy döt Net: *YAML Ain’t Markup Language
    (YAML™) Version 1.2*. 1 October 2010.
    <http://yaml.org/spec/1.2/spec.html>

[RFC 2119]: http://tools.ietf.org/html/rfc2119
[RFC 4151]: http://tools.ietf.org/html/rfc4151
[RFC 5870]: http://tools.ietf.org/html/rfc5870

[JSON]: http://json.org/
[YAML]: http://yaml.org/
[Turtle]: http://www.w3.org/TR/turtle/
[JSON-LD]: http://json-ld.org/
[RDF/JSON]: http://www.w3.org/TR/rdf-json/


# Appendix

`aref-query.md`{.include}

`serializations.md`{.include}

<!-- `appendix.md`{.include} -->

[*RDF graph*]: #rdf-data
[*simple literal*]: #rdf-data
[*language tag*]: #rdf-data
[*datatype*]: #rdf-data
[*blank node*]: #rdf-data
[*blank nodes*]: #rdf-data
[*RDF graphs*]: #rdf-data
[*IRI*]: #rdf-data
[*IRIs*]: #rdf-data
[*predicate*]: #rdf-data
[*object*]: #rdf-data
[*objects*]: #rdf-data
[*subject*]: #rdf-data
[*subjects*]: #rdf-data
[*predicates*]: #rdf-data
[*triples*]: #rdf-data

[*list*]: #list-map-structures
[*map*]: #list-map-structures
[*key*]: #list-map-structures
[*keys*]: #list-map-structures

[*namespace map*]: #namespace-maps

[*explicit IRI*]: #explicit-iris
[*qName*]: #qnames
[*qNames*]: #qnames
[*literal node*]: #literal-nodes
[*literal nodes*]: #literal-nodes

[*identified blank node*]: #blank-nodes
[*blank node identifier*]: #blank-nodes

[*subject map*]: #subject-maps
[*subject maps*]: #subject-maps
[*predicate map*]: #predicate-maps

[*namespace URI*]: #namespace-maps

