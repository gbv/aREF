# Example

An aREF document can be expressed both in data structuring languages (JSON,
YAML...) and in type systems of programming languages (Python, Ruby, Perl...).

The following examples express the same aREF document in different languages.
The namespace map could also be abbreviated to just the simple string
`"http://example.org/ontology/"` for the default namespace.

*TODO: add RDF/Turtle of this document*

### YAML

The most condensed readable serialization of aREF is probably possible in
**YAML**:

    ---
    _ns: 
        dct: http://purl.org/dc/terms/
        foaf: http://xmlns.com/foaf/0.1/
    _id: http://example.com/people#alice
    a: foaf_Person
    foaf_name: Alice Smith
    foaf_age: 42^xsd_integer 
    foaf_homepage: 
        - http://personal.example.org/~alice/ 
        - http://work.example.com/asmith/ 
    foaf_knows:
        _id: _:1
        foaf_name: John
        dct_description: a nice guy@en

### JSON

The same in **JSON** requires more brackets and delimiters:

    { 
        "_ns": { 
            "dct": "http://purl.org/dc/terms/",
            "foaf": "http://xmlns.com/foaf/0.1/"
        },
        "_id": "http://example.com/people#alice",
        "a": "foaf:Person",
        "foaf_name": "Alice Smisth",
        "foaf_age": "42^xsd_integer",
        "foaf_homepage": [
           "http://personal.example.org/~alice/",
           "http://work.example.com/asmith/" 
        ],
        "foaf_knows": { 
            _id: "_:1",
            "foaf_name": "John",
            "dct_description": "a nice guy@en" 
        }
    }

### JavaScript

In **JavaScript** one can omit quotes around map keys by using underscores for
prefixed names:

    { 
        _ns: { 
            dct: 'http://purl.org/dc/terms/',
            foaf: 'http://xmlns.com/foaf/0.1/'
        },
        _id: 'http://example.com/people#alice',
        a: 'foaf:Person',
        foaf_name: 'Alice Smisth',
        foaf_age: '42^xsd_integer',
        foaf_homepage: [
           'http://personal.example.org/~alice/',
           'http://work.example.com/asmith/' 
        ],
        foaf_knows: { 
            _id: '_:1',
            foaf_name: 'John',
            dct_description: 'a nice guy@en' 
        }
    }

### Perl

Similar rules apply to aREF in **Perl**:

    {
        _ns => {
           dct => 'http://purl.org/dc/terms/',
           foaf => 'http://xmlns.com/foaf/0.1/',
        },
        _id => 'http://example.com/people#alice',
        a   => 'foaf:Person',
        foaf_name => 'Alice Smith',
        foaf_age  => '42^xsd_integer', 
        foaf_homepage => [
            'http://personal.example.org/~alice/',
            'http://work.example.com/asmith/' 
        ],
        foaf_knows => {
            _id => '_:1'
            foaf_name => 'John',
            dct_description => 'a nice guy@en',
        }
    }

### PHP

Although **PHP** does not fully differntiate arrays and maps, one can express
both. A PHP array is a map unless all PHP array keys are numeric:

    [
        "_ns" => [ 
            "dct" => "http://purl.org/dc/terms/",
           "foaf" => "http://xmlns.com/foaf/0.1/"
        ],
        "_id" => "http://example.com/people#alice",
        "a" => "foaf:Person",
        "foaf:name" => "Alice Smith",
        "foaf:age"  => "42^xsd_integer",
        "foaf:homepage" => [
            "http://personal.example.org/~alice/",  /* key "0" */
            "http://work.example.com/asmith/"       /* key "1" */
        ],
        "foaf:knows" => [
            "_id" => "_:1",
            "foaf:name" => "John",
            "dct:description" => "a nice guy@en"
        ]
    ];

