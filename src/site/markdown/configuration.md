# Configuring the API

The `DataDistributor` instances used by the API are configured in the triples of the 
Vitro display model. When the API controller receives a request, it looks in the display model for the description of a `DataDistributor` with the correct action name.

The controller uses that description to create a Java instance which will service the request. The Java class of the instance is taken from the RDF, and any properties that should be provided to the newly-created instance.

When the instance has been created and initialized, it will provide the  contents and type of the HTTP response.

The most common way to create such triples is to add one or more files of RDF to the `rdf/display/everytime` folder in the Vitro home directory.

## Configuring "Hello, World"

Let's review the configuration that was used to test the API: the "Hello, World" distributor.

In that case, we created a file of RDF in Turtle format, and put it in VIVO's `rdf/display/everytime` folder.

```
@prefix : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .
  
:data_distributor_hello
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.examples.HelloDistributor> ;
    :actionName "hello" .
```

### What does this configuration say?

* The `@prefix` line establishes the default namespace. By convention, the properties on a `DataDistributor` instance will use this namespace. For example, we may say that every description of a `DataDistributor` must include a data property for `:actionName`, but this is an abbreviated notation for the actual requirement: a data property for `<http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#actionName>`.
* The description must have a URI, but the URI is arbitrary. It is common to place the URI in the default namespace, simply for brevity, but it is not required.
	* This might seem like the perfect place for a blank URI. However, the API controller does not deal well with blank URIs.
* Two type statements are included. 
	* The first says that this URI represents a `DataDistributor`, but that is an abstract class, and can't be instantiated.
	* The second specifies a concrete class for the instance. 
	* Both are necessary.
* A `DataDistributor` description is useless without an `:actionName` property.
	* When a request comes to the API controller, it will look for a 
 `DataDistributor` whose
 `:actionName` property matches the last part of the request path.
	* A request of `http://localhost:8080/vivo/api/dataRequest/hello` would match the description shown above.

## A more ambitious example

The configuration of a `DataDistributor` may also involve the configuration of related object. In this example, we configure a `DataDistributor` instance that executes a SPARQL SELECT query against an RDF graph. That graph is the result of queries by two `GraphBuilder` instances. Each `GraphBuilder` runs a query against the VIVO content models.

All of this is accomplished through the configuration.

```
@prefix : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .
  
:ambitious
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.SelectFromGraphDistributor> ;
    :actionName "2graphs" ;
    :graphBuilder :ambitious_graph1 ,
                  :ambitious_graph2 ;
    :query """
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      SELECT ?label
      WHERE {
        ?s rdfs:label ?label .
      }
    """ .
    
:ambitious_graph1
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :constructQuery """
        PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
        CONSTRUCT {
          ?concept rdfs:label ?label .
        } WHERE {
          ?concept a skos:Concept .
          ?concept rdfs:label ?label .
        }
    """ .

:ambitious_graph2
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :constructQuery """
        PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
        PREFIX foaf: <http://xmlns.com/foaf/0.1/>
        CONSTRUCT {
          ?person rdfs:label ?label .
        } WHERE {
          ?person a foaf:Person .
          ?person rdfs:label ?label .
        }
    """ .
```

### What does this configuration say?

* The `@prefix` line is the same as in the previous example.
* Again, we describe a `DataDistributor` instance. This one has a concrete class of `SelectFromGraphDistributor`. It will run two graph builders, merge their results into a single graph, and execute a SPARQL SELECT query against that merged graph.
	* As in the previous example, the `DataDistributor` needs a data property for `:actionName`, to match an HTTP request.
	* The `SelectFromGraphDistributor` instance requires one or more object properties for `:graphBuilder`. The values are URIs which describe `GraphBuilder` instances, elsewhere in the configuration.
	* The instance also requires a SPARQL SELECT query, to execute against the results obtained from the `GraphBuilder` instance(s).
* The next section describes a `GraphBuilder` instance.
	* The URI is arbitrary, except that it matches the reference made by the `SelectFromGraphDistributor`.
	* There are two type statements:
		* The first says that this represents a `GraphBuilder`, as the `SelectFromGraphDistributor` requires. This is not sufficient to create the instance, since `GraphBuilder` is an abstract class.
		* The second specifies the concrete class of the instance.
	* The `ConstructQueryGraphBuilder` requires a data property for `:constructQuery`.
		* This query will be run against the VIVO content models, producing an RDF graph as a result.
* The final section describes another `GraphBuilder` instance.

## Creating the configuration files

The configuration is based on triples in the display model of VIVO's configuration triple-store. These triples are usually created by placing one or more files in `{VIVO}/home/src/main/resources/rdf/display/everytime` and running the build script.

RDF files in this folder may use any of these formats:

Format | file extension
--- | ---
RDF/XML | `.rdf`
N-triples | `.nt`
Turtle | `.ttl`
N3 | `.n3`

File names are arbitrary (except for the extensions). Chose names that are meaningful.

The configuration triples may all be in a single file, or may be split among several files. Use whatever scheme you feel is clear and maintainable.

### Other ways to create configuration triples

Some people may prefer to place the configuration file in the `rdf/display/everytime` folder of VIVO's home directory, where it will be picked up at runtime. Be aware, however, that any such files would be overwritten by the next build.

It would also be possible to ingest the triples into VIVO, using the interactive **Site Administration** tools to **Manage Jena Models**.

## RDF syntax errors
A systax error in a configuration file will cause VIVO to ignore that file. This may go unnoticed because it does not preven VIVO from starting, nor does VIVO display any warning message in the browser.

To find RDF syntax errors, you will need to inspect VIVO's log file, `vivo.all.log`. Here is an example of an error in the log file:

```
2017-08-17 13:29:06,649 WARN  [RDFFilesLoader] Could not load file '/vivo/instance/home/rdf/display/everytime/HelloDistributorConfig.ttl' as TURTLE. Check that it contains valid data.
org.apache.jena.riot.RiotException: [line: 1, col: 77] Out of place: [DOT]
```

In this case, the leading `@` was inadvertently removed from the `prefix` statement.

