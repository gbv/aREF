# Examples

## Examples in different serializations

An aREF document can be expressed both in data structuring languages (JSON,
YAML...) and in type systems of programming languages (Python, Ruby, Perl...).

The following examples express the same aREF document in different languages.
The namespace map could also be abbreviated to just the simple string
`"http://example.org/ontology/"` for the default namespace.

The most condensed readable serialization of aREF is probably possible in
**YAML**:

    ---
    _ns: 
        _:   http://example.org/ontology/
        dct: http://purl.org/dc/terms/
    _id: http://example.com/people#alice
    a: foaf:Person
    foaf:name: Alice Smith
    foaf:age: 42^xsd:integer 
    foaf:homepage: 
        - http://personal.example.org/~alice/ 
        - http://work.example.com/asmith/ 
    foaf:knows:
        _id: _:1
        foaf:name: John
        dct:description: a nice guy@en
    neighbor: _:1

The same in **JSON** requires more brackets and delimiters:

    { 
        '_ns': { 
            '_': 'http://example.org/ontology/',
            'dct': 'http://purl.org/dc/terms/'
        },
        '_id': 'http://example.com/people#alice',
        'a': 'foaf:Person',
        'foaf:name': 'Alice Smisth',
        'foaf:age': '42^xsd:integer',
        'foaf:homepage': [
           'http://personal.example.org/~alice/',
           'http://work.example.com/asmith/' 
        ],
        'foaf:knows': { 
            _id: '_:1',
            'foaf:name': 'John',
            'dct:description': 'a nice guy@en' 
        },
        'neighbor': '_:1' 
    }

In **JavaScript** one can omit quotes around map keys by using underscores for
prefixed names:

    { 
        _ns: { 
            _: 'http://example.org/ontology/',
            dct: 'http://purl.org/dc/terms/'
        },
        _id: 'http://example.com/people#alice',
        a: 'foaf:Person',
        foaf_name: 'Alice Smisth',
        foaf_age: '42^xsd:integer',
        foaf_homepage: [
           'http://personal.example.org/~alice/',
           'http://work.example.com/asmith/' 
        ],
        foaf_knows: { 
            _id: '_:1',
            foaf_name: 'John',
            dct_description: 'a nice guy@en' 
        },
        neighbor: '_:1' 
    }

Similar rules apply to aREF in **Perl**:

    {
        _ns => {
            _ => 'http://example.org/ontology/',
           dct => 'http://purl.org/dc/terms/',
        },
        _id => 'http://example.com/people#alice',
        a   => 'foaf:Person',
        foaf_name => 'Alice Smith',
        foaf_age  => '42^xsd:integer', 
        foaf_homepage => [
            'http://personal.example.org/~alice/',
            'http://work.example.com/asmith/' 
        ],
        foaf_knows => {
            _id => '_:1'
            foaf_name => 'John',
            dct_description => 'a nice guy@en',
        },
        neighbor => '_:1',
    }

Although **PHP** does not fully differntiate arrays and maps, one can express
both. A PHP array is a map unless all PHP array keys are numeric:

    [
        "_ns" => [ 
            "_" => "http://example.org/ontology/",
            "dct" => "http://purl.org/dc/terms/"
        ],
        "_id" => "http://example.com/people#alice",
        "a" => "foaf:Person",
        "foaf:name" => "Alice Smith",
        "foaf:age"  => "42^xsd:integer",
        "foaf:homepage" => [
            "http://personal.example.org/~alice/",  /* key "0" */
            "http://work.example.com/asmith/"       /* key "1" */
        ],
        "foaf:knows" => [
            "_id" => "_:1",
            "foaf:name" => "John",
            "dct:description" => "a nice guy@en"
        ],
        "neighbor" => "_:1"
    ];


## Examples from JSON-LD specification

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


