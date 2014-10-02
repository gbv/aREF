## Examples from JSON-LD specification

Example 1/2 from JSON-LD adopted as aRef in JSON or Python:

    {
      "schema_name": "Manu Sporny",
      "schema_url": "http://manu.sporny.org/",
      "schema_image": "http://manu.sporny.org/images/manu.png"
    }

Example 4/5 from JSON-LD adopted as aRef in JSON or Python:

    {
      "_ns": { "schema": "http://schema.org/" },
      "schema_name": "Manu Sporny",
      "schema_url": "http://manu.sporny.org/",
      "schema_image": "http://manu.sporny.org/images/manu.png"
    }

Example 11 from JSON-LD adopted as aRef in YAML:

    ---
    _id: http://me.markus-lanthaler.com/
    schema_name: Markus Lanthaler

Example 14 from JSON-LD adopted as aRef in YAML:

    ---
    _id: http://example.org/places#BrewEats
    a: 
        - schema_Restaurant
        - schema_Brewery

Example 20 from JSON-LD adopted as aRef in YAML:

    ---
    _ns:
        xsd: http://www.w3.org/2001/XMLSchema#
        foaf: http://xmlns.com/foaf/0.1/
    _id: http://me.markus-lanthaler.com/
    a: foaf_Person
    foaf_name: Markus Lanthaler
    foaf_homepage: http://www.markus-lanthaler.com/
    foaf_depiction: http://twitter.com/account/profile_image/markuslanthaler

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

        { "dct_modified": "2010-05-29T14:17:39+02:00^xsd_dateTime" }
        { "dct_modified": "2010-05-29T14:17:39+02:00^<http://www.w3.org/2001/XMLSchema#dateTime>" }


