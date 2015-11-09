## aREF serializations

An aREF document can be expressed both in data structuring languages (JSON,
YAML...) and in type systems of programming languages (Python, Ruby, Perl...).

The following examples express the same aREF document in different languages. The
RDF graph encoded in aREF can be expressed in Turtle syntax as following:

<div class="example">
```turtle
@prefix dct: <http://purl.org/dc/terms/> .
@prefix foaf: <http://xmlns.com/foaf/0.1/> .

<http://example.com/people#alice> a foaf:Person ;
    foaf:name "Alice Smith" ;
    foaf:age 42 ;
    foaf:homepage 
        <http://personal.example.org/~alice/>, 
        <http://work.example.com/asmith/> ;
    foaf:knows [
        foaf:name "John" ;
        dct:description "a nice guy"@en 
    ]
.
```
</div>

<div class="note">
Please add your favorite data or programming language at
<https://github.com/gbv/aREF/issues> to be included here!
</div>

### YAML {.unnumbered}

The most condensed readable serialization of aREF is probably possible in
**YAML**:

<div class="example">
```yaml
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
```
</div>

### JSON {.unnumbered}

The same in **JSON** requires more brackets and delimiters:

<div class="example">
```json
{ 
    "_ns": { 
        "dct": "http://purl.org/dc/terms/",
        "foaf": "http://xmlns.com/foaf/0.1/"
    },
    "_id": "http://example.com/people#alice",
    "a": "foaf_Person",
    "foaf_name": "Alice Smith",
    "foaf_age": "42^xsd_integer",
    "foaf_homepage": [
       "http://personal.example.org/~alice/",
       "http://work.example.com/asmith/" 
    ],
    "foaf_knows": { 
        "_id": "_:1",
        "foaf_name": "John",
        "dct_description": "a nice guy@en" 
    }
}
```
</div>

### JavaScript {.unnumbered}

In **JavaScript** one can omit quotes around map keys by using underscores for
prefixed names:

<div class="example">
```javascript
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
```
</div>

### Perl {.unnumbered}

Similar rules apply to aREF in **Perl**:

<div class="example">
```perl
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
```
</div>

### PHP {.unnumbered}

Although **PHP** does not fully differntiate arrays and maps, one can express
both. A PHP array is a map unless all PHP array keys are numeric:

<div class="example">
```php
[
    "_ns" => [ 
        "dct" => "http://purl.org/dc/terms/",
       "foaf" => "http://xmlns.com/foaf/0.1/"
    ],
    "_id" => "http://example.com/people#alice",
    "a" => "foaf_Person",
    "foaf_name" => "Alice Smith",
    "foaf_age"  => "42^xsd_integer",
    "foaf_homepage" => [
        "http://personal.example.org/~alice/",  /* key "0" */
        "http://work.example.com/asmith/"       /* key "1" */
    ],
    "foaf_knows" => [
        "_id" => "_:1",
        "foaf_name" => "John",
        "dct_description" => "a nice guy@en"
    ]
];
```
</div>

