# aREF decoding algorithm

*this section ins non-normative. It may reflect a different version of aREF and
may contain errors!*

~~~
regexp PREFIX   = /^[a-z]([a-z]|[0-9])*$/
regexp IRI_LIKE = /^[a-z]([a-z]|[0-9]|+|\.|-)*:.*$/
regexp NAME     = ... /* see specification */
regexp IRI      = ... /* see RFC 3987 */

global namespaces   /* predefined namespace map */
global visted       /* track recursive structures */

function decode( map )
    namespaces = predefined_namespace
    namespace_map( map["_ns"] )
    visted = { map: true }
    if map["_id"]
        subject = iri_or_blank( map["_id"] )
        predicate_map( subject, map )
    else
        foreach key in map except "_ns"
            subject = iri_or_blank(key)
            predicates = map[key]
            if predicates["_id"] and iri_or_blank( predicates["id"] ) != subject
                error "inconsistent _id"
            else
                predicate_map( subject, predicates )

function namespace_map( ns )
    if is_string(ns)
        namespaces[""] = ns[key]
    else if is_map(ns)
        foreach key in ns 
            if key matches PREFIX and ns[key] matches IRI_LIKE
                if key == "_"
                    prefix = ""
                else
                    prefix = key
                namespaces[prefix] = ns[key]

function predicate_map( subject, map )
    if subject and is_map(map)
        foreach key in map except "_id" or "_ns"
            if key == "a"
                predicate = "http://www.w3.org/1999/02/22-rdf-syntax-ns#type"
            else
                predicate = iri_or_blank(key)
                if predicate
                    next key
            object( subject, predicate, map[key] )

function iri_or_blank( string )
    if string matches /^<(.+)>$/
        if $1 matches IRI
            return $1
    else if string matches /^_:([a-z]|[A-Z]|[0-9])+$/
        return blank_node($1)
    else if string matches /^((PREFIX)?[:_])?(NAME)$/
        prefix = $1
        if namespaces[prefix]
            return concat(namespaces[prefix],$3)
    else if string matches IRI_LIKE
        if string matches IRI
            return string
    error "invalid IRI"
    return false

function blank_node( identifier )
    ... /* return a blank node with given identifier or new identifier */

function literal_node( string, language, datatype,  )
    if not language and not datatype
        datatype = "http://www.w3.org/2001/XMLSchema#string"
    ... /* return a literal node, optionally with language or datatype */

function objects( subject, predicate, object )
    if is_array(object)
        foreach obj in object
            single_object( subject, predicate, obj )
    else
        single_object( subject, predicate, obj )

function single_object( subject, predicate, object )
    if is_map( object )
        if object["_id"]
            object_node = iri_or_blank(object["_id"])
            if not object_node
                return
        else 
            object_node = blank_node()

        triple( subject, predicate, object_node )

        if not visited[object]
            visited[object] = true
            predicate_map( object_node, object )
    else
        if object matches /^<(.+)>$/
            if $1 matches IRI
                object_node = $1
            else
                error "invalid IRI"
                return
        else if string matches /^_:([a-z]|[A-Z]|[0-9])+$/
            object_node = blank_node($1)
        else if string matches /^(PREFIX)?:(NAME)$/
            prefix = $1
            if namespaces[prefix]
                object_node = concat(namespaces[prefix],$3)
            else
                error "unknown prefix"
                return
        else if object matches /^(.*)@([a-z]{2,8}(-[a-z0-9]{1,8})*)$/
            object_node = literal_node( $1, $2 )
        else if string matches /^(.*?)^^?(PREFIX)?:(NAME)$/
            prefix = $1
            if namespaces[prefix]
                datatype = concat(namespaces[prefix],$3)
                object_node = literal_node( $1, false, datatype )
            else
                error "unknown prefix in datatype"
                return
        else if string matches /^(.*?)^^?<(IRI_LIKE)>$/
            if $2 matches IRI
                datatype = $2
                object_node = literal_node( $1, false, datatype )
            else
                error "invalid datatype IRI"
                return
        else if string matches IRI_LIKE
            if object matches IRI
                object_node = object
            else
                error "invalid IRI"
                return
        else 
            object_node = literal_node( object )

        triple( subject, predicate, object_node )
~~~
