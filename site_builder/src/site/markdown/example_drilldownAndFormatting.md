# Drilldown with multiple queries, wrapped in a Javascript formatter

This distributor produces a JSON-formatted list of information about the publications written by a given
faculty member. The HTTP request must specify a `person` parameter, containing the URI of the faculty member.

This is two distributors: an `RdfGraphDistributor` wrapped in a `JavaScriptTransformDistributor`.

The `RdfGraphDistributor` uses a structure of `GraphBuilder` instances to produce a model which is emitted in Turtle format. 
The `JavaScriptTransformDistributor` uses a Javascript (with two supporting scripts) to convert that Turtle output into JSON format, structured according to the user requirements.

The `RdfGraphDistributor` uses a `DrillDownGraphBuilder` to create its model. 
The `DrillDownGraphBuilder` runs a `ConstructQueryGraphBuilder` to produce a small model, 
and applies a SELECT query to that model to produce a list of URIs for the publications of the faculty member.

Then, the `DrillDownGraphBuilder` creates and runs a `ConstructQueryGraphBuilder` for each publication in the list,
adding the results of each `ConstructQueryGraphBuilder` to the model. 
If this weren't already complex enough, each of these `ConstructQueryGraphBuilder` instances
runs several CONSTRUCT queries for the URL it is applied to. 
These simple CONSTRUCT queries run more quickly than a large complex query with many OPTIONAL or UNION clauses.

Once all of these `GraphBuilder`s have been run, and their results have been added to the model, 
and the model has been output in Turtle format, then the `JavaScriptTransformDistributor` goes to work.
It defines a `transform()` function which receives the Turtle as a string, converts it to JSON, 
and returns that JSON as output. 
The `transform()` function relies on functions in the supporting scripts to do much of its work.

Final result: a JSON stream with the desired data and structure.

![Instances and data flow](images/example_drilldownAndFormatting.png)

```
#
# ------------------------------------------------------------------------------
#
# Get a JSON formatted list of publications for a specific person.
#
# Run an inner data distributor that creates a TURTLE file for the result, and 
# wrap it in a decorator that convewrts the TURTLE to JSON.
#
# Play games with the inner distributor, including drill-down and multiple 
# sub-queries, to try to speed it up.
# 
# ------------------------------------------------------------------------------
#
 
@prefix : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

#
# The inner distributor uses a GraphBuilder and converts the result to TURTLE.
#

:publist_inner
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.RdfGraphDistributor> ;
    :actionName "_none_" ;
    :graphBuilder :publist_driller .

#
# the GraphBuilder gets the publications for the person, and then drills down 
# to get the information for each publication.
#    
:publist_driller
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.DrillDownGraphBuilder> ;
    :topLevelGraphBuilder :publist_drill_top ;
    :childGraphBuilder :publist_drill_bottom ;
    :drillDownQuery """
        PREFIX bibo: <http://purl.org/ontology/bibo/>
        SELECT ?pub
        WHERE {
          ?pub a bibo:Document .
        }
    """ .

:publist_drill_top
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :uriBinding "person" ;
    :constructQuery """
        PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX bibo:     <http://purl.org/ontology/bibo/>
        PREFIX vivo:     <http://vivoweb.org/ontology/core#>
        CONSTRUCT {
          ?pub a bibo:Document .
          ?pub rdfs:label ?pubLabel .
        }
        WHERE
        {
          ?person vivo:relatedBy ?auth .
          ?auth a vivo:Authorship .
          ?auth vivo:relates ?pub .
          ?pub a bibo:Document .
          ?pub rdfs:label ?pubLabel .
        }
    """ .

#
# For each publication, run a group of queries, rather than one query with
# multiple OPTIONAL or UNION clauses.
#
    
:publist_drill_bottom
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :uriBinding "pub" ;
    :constructQuery """
        PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX foaf:     <http://xmlns.com/foaf/0.1/>
        PREFIX vivo:     <http://vivoweb.org/ontology/core#>
        CONSTRUCT {
          ?pub vivo:relatedBy ?auth .
          ?auth vivo:rank ?rank .
          ?auth vivo:relates ?author .
          ?author a foaf:Person .
          ?author rdfs:label ?authorName .
        }
        WHERE
        {
          ?pub vivo:relatedBy ?auth .
          ?auth a vivo:Authorship .
          ?auth vivo:rank ?rank .
          ?auth vivo:relates ?author .
          ?author a foaf:Person .
          ?author rdfs:label ?authorName .
        }
    """ , """
        PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX vcard:    <http://www.w3.org/2006/vcard/ns#>
        PREFIX vivo:     <http://vivoweb.org/ontology/core#>
        CONSTRUCT {
          ?pub vivo:relatedBy ?auth .
          ?auth vivo:rank ?rank .
          ?auth vivo:relates ?vcard .
          ?vcard vcard:hasName ?vname .
          ?vname vcard:givenName ?givenName .
        }
        WHERE
        {
          ?pub vivo:relatedBy ?auth .
          ?auth a vivo:Authorship .
          ?auth vivo:rank ?rank .
          ?auth vivo:relates ?vcard .
          ?vcard vcard:hasName ?vname .
          ?vname vcard:givenName ?givenName .
        }
    """ , """
        PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX vcard:    <http://www.w3.org/2006/vcard/ns#>
        PREFIX vivo:     <http://vivoweb.org/ontology/core#>
        CONSTRUCT {
          ?pub vivo:relatedBy ?auth .
          ?auth vivo:rank ?rank .
          ?auth vivo:relates ?vcard .
          ?vcard vcard:hasName ?vname .
          ?vname vivo:middleName ?middleName .
        }
        WHERE
        {
          ?pub vivo:relatedBy ?auth .
          ?auth a vivo:Authorship .
          ?auth vivo:rank ?rank .
          ?auth vivo:relates ?vcard .
          ?vcard vcard:hasName ?vname .
          ?vname vivo:middleName ?middleName .
        }
    """ , """
        PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX vcard:    <http://www.w3.org/2006/vcard/ns#>
        PREFIX vivo:     <http://vivoweb.org/ontology/core#>
        CONSTRUCT {
          ?pub vivo:relatedBy ?auth .
          ?auth vivo:rank ?rank .
          ?auth vivo:relates ?vcard .
          ?vcard vcard:hasName ?vname .
          ?vname vcard:familyName ?familyName .
        }
        WHERE
        {
          ?pub vivo:relatedBy ?auth .
          ?auth a vivo:Authorship .
          ?auth vivo:rank ?rank .
          ?auth vivo:relates ?vcard .
          ?vcard vcard:hasName ?vname .
          ?vname vcard:familyName ?familyName .
        }
    """ , """
        PREFIX vivo:     <http://vivoweb.org/ontology/core#>
        CONSTRUCT {
          ?pub vivo:dateTimeValue ?timeStamp .
          ?timeStamp vivo:dateTime ?dateTime .
        }
        WHERE
        {
          ?pub vivo:dateTimeValue ?timeStamp .
          ?timeStamp vivo:dateTime ?dateTime .
        }
    """ , """
        PREFIX rdfs:     <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX vivo:     <http://vivoweb.org/ontology/core#>
        CONSTRUCT {
          ?pub vivo:hasPublicationVenue ?journal .
          ?journal rdfs:label ?journalName .
        }
        WHERE
        {
          ?pub vivo:hasPublicationVenue ?journal .
          ?journal rdfs:label ?journalName .
        }
    """ , """
        PREFIX bibo:     <http://purl.org/ontology/bibo/>
        CONSTRUCT {
          ?pub bibo:doi ?doi .
        }
        WHERE
        {
          ?pub bibo:doi ?doi .
        }
    """ , """
        PREFIX bibo:     <http://purl.org/ontology/bibo/>
        CONSTRUCT {
          ?pub bibo:volume ?volume .
        }
        WHERE
        {
          ?pub bibo:volume ?volume .
        }
    """ , """
        PREFIX bibo:     <http://purl.org/ontology/bibo/>
        CONSTRUCT {
          ?pub bibo:issue ?issue .
        }
        WHERE
        {
          ?pub bibo:issue ?issue .
        }
    """ , """
        PREFIX bibo:     <http://purl.org/ontology/bibo/>
        CONSTRUCT {
          ?pub bibo:pageStart ?pageStart .
          ?pub bibo:pageEnd ?pageEnd .
        }
        WHERE
        {
          ?pub bibo:pageStart ?pageStart .
          ?pub bibo:pageEnd ?pageEnd .
        }
    """ .    

#
# The decorator converts the TURTLE RDF model into a JSON structure.
#
:publist_distributor
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.decorator.JavaScriptTransformDistributor> ;
    :actionName "listPublications" ;
    :contentType "application/json" ;
    :child :publist_inner ;
    :supportingScript "/js/scholars-vis/rdflib.js";
    :supportingScript "/js/scholars-vis/rdf-helper-methods.js";
    :script """
        function transform(turtleRdf) {
            var graph = populateGraph(turtleRdf);
            standardPrefixes();
            
            var results = [];
            getPubStatements().forEach(processPublication);
            
            return JSON.stringify(results, null, 2);
                
            function processPublication(stmt) {
                var pubNode = stmt.subject;
 
                var result = { uri: pubNode.uri };
                addRequiredField(result, "label", findPublicationName);
                addRequiredField(result, "authors", findAuthors);
                addOptionalField(result, "pubDate", findPubDate);
                addOptionalField(result, "journalName", findJournalName);
                addOptionalField(result, "doi", findDoi);
                addOptionalField(result, "volume", findVolume);
                addOptionalField(result, "issue", findIssue);
                addOptionalField(result, "pages", findPages);
                results.push(result);
                    
                function findPublicationName() {
                    return getAnyValue(graph, pubNode, RDFS('label'));
                }
                
                function findAuthors() {
                    return findAuthorshipNodes().map(getAuthorInfo).sort(rankSorter);
                    
                    function findAuthorshipNodes() {
                        return graph.each(pubNode, VIVO('relatedBy'));
                    }
                    
                    function getAuthorInfo(authorshipNode) {
                        var rank = parseInt(getAnyValue(graph, authorshipNode, VIVO('rank')));
                        var authorNode = graph.any(authorshipNode, VIVO('relates'));
                        
                        if (authorIsPerson()) {
                            return personAuthorInfo();
                        } else {
                            return vcardAuthorInfo();
                        }
                        
                        function authorIsPerson() {
                            return graph.statementsMatching(authorNode, RDF('type'), FOAF('Person')).length > 0;
                        }
                        
                        function personAuthorInfo() {
                            return {
                                rank: rank,
                                label: getAnyValue(graph, authorshipNode, VIVO('relates'), RDFS('label'))
                            };
                        }
                        
                        function vcardAuthorInfo() {
                            var firstName = findVCardFirstName();
                            var middleName = findVCardMiddleName();
                            var lastName = findVCardLastName();
                            return {
                                rank: rank,
                                label: combineNameParts()
                            }
                            
                            function findVCardFirstName() {
                                return getAnyValue(graph, authorNode, VCARD('hasName'), VCARD('givenName'));
                            }

                            function findVCardMiddleName() {
                                return getAnyValue(graph, authorNode, VCARD('hasName'), VIVO('middleName'));
                            }

                            function findVCardLastName() {
                                return getAnyValue(graph, authorNode, VCARD('hasName'), VCARD('familyName'));
                            }
                            
                            function combineNameParts() {
                                if (!lastName) {
                                    return "NO NAME";
                                }
                                var name = lastName;
                                
                                if (firstName || middleName) {
                                    name += ",";
                                }
                                if (firstName) {
                                    name += " " + firstName;
                                    if (firstName.length == 1) {
                                        name += ".";
                                    }
                                }
                                if (middleName) {
                                    name += " " + middleName;
                                    if (middleName.length == 1) {
                                        name += ".";
                                    }
                                }
                                return name;
                            }
                        }
                        
                    }

                    function rankSorter(a, b) {
                        return a.rank - b.rank;
                    }
                    
                }
                
                function findPubDate() {
                    return getAnyValue(graph, pubNode, VIVO('dateTimeValue'), VIVO('dateTime')).substring(0, 4);
                }
                
                function findJournalName() {
                    return getAnyValue(graph, pubNode, VIVO('hasPublicationVenue'), RDFS('label'));
                }
                
                function findDoi() {
                    return getAnyValue(graph, pubNode, BIBO('doi'));
                }

                function findVolume() {
                    return getAnyValue(graph, pubNode, BIBO('volume'))
                }

                function findIssue() {
                    return getAnyValue(graph, pubNode, BIBO('issue'))
                }
                
                function findPages() {
                    var pageStart = getAnyValue(graph, pubNode, BIBO('pageStart'));
                    var pageEnd = getAnyValue(graph, pubNode, BIBO('pageEnd'));
                    if (pageStart == null && pageEnd == null) {
                        return null;
                    } else {
                        return {
                            pageStart: pageStart,
                            pageEnd: pageEnd
                        }
                    }
                }
            }
                
            function getPubStatements() {
                return graph.statementsMatching(undefined, RDF('type'), BIBO('Document'));
            }
        }
    """ .

```