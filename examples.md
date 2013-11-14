# Examples

An aREF document can be expressed both in data structuring languages (JSON,
YAML...) and in type systems of programming languages (Python, Ruby, Perl...).

Example 1/2 from JSON-LD adopted as aRef in JSON or Python:

    {
      "schema:name": "Manu Sporny",
      "schema:url": "http://manu.sporny.org/",
      "schema:image": "http://manu.sporny.org/images/manu.png"
    }

Example 4/5 from JSON-LD adopted as aRef in JSON or Python:

    {
      "_ns": { "schema": "http://schema.org/" },
      "schema:name": "Manu Sporny",
      "schema:url": "http://manu.sporny.org/",
      "schema:image": "http://manu.sporny.org/images/manu.png"
    }

Example 11 from JSON-LD adopted as aRef in YAML:

    ---
    _id: http://me.markus-lanthaler.com/
    schema:name: Markus Lanthaler

Example 14 from JSON-LD adopted as aRef in YAML:

    ---
    _id: http://example.org/places#BrewEats
    a: 
        - schema:Restaurant
        - schema:Brewery

Example 20 from JSON-LD adopted as aRef in YAML:

    ---
    _ns:
        xsd: http://www.w3.org/2001/XMLSchema#
        foaf: http://xmlns.com/foaf/0.1/
    _id: http://me.markus-lanthaler.com/
    a: foaf:Person
    foaf:name: Markus Lanthaler
    foaf:homepage: http://www.markus-lanthaler.com/
    foaf:depiction: http://twitter.com/account/profile_image/markuslanthaler

Example 23 from JSON-LD adopted as aRef in Perl:

    {
      _id => "http://example.org/posts#TripToWestVirginia",
      a => "http://schema.org/BlogPosting",
      dct_modified => "2010-05-29T14:17:39+02:00^xsd:dateTime"
    }

Example 24 from JSON-LD adopted as aRef in Perl:

    {
      _id => "http://example.com/people#john",
      foaf_name => "John Smith",
      foaf_age  => "41^xsd:integer",
      foaf_homepage => [qw(
         http://personal.example.org/
         http://work.example.com/jsmith/
      )],
    }

...

More examples:

        { "dct:modified": "2010-05-29T14:17:39+02:00^^xsd:dateTime" }
        { "dct:modified": "2010-05-29T14:17:39+02:00^^<http://www.w3.org/2001/XMLSchema#dateTime>" }


Examples of RDF literals encoded as aRef strings:

  aRef string         RDF literal (in Turtle syntax)
  ------------------- ----------------------------------
  @                   `""`
  *empty string*      `""`
  \^xsd:string        `""`
  \^\^xsd:string      `""`
  @@                  `"@"`
  @\^xsd:string       `"@"`
  alice@en            `"alice"@en`
  alice@example.com   `"alice@example.com"`
  123                 `"123"`
  忍者@ja             `"忍者"@ja`
  Ninja@en@           `"Ninja@en"`
  foo:bar             *not a literal node, but an IRI*


