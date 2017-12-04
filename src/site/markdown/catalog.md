# Implementations provided

The Data Distribution API is intended to be extensible, and developers are encouraged to create their own `DataDistributor` classes. However, many sites will find that the existing implementations are sufficient for their needs.

* [`/example` package](#aexample_package)
	* [`HelloDistributor`](#HelloDistributor)
* [`/file` package](#afile_package)
	* [`FileDistributor`](#FileDistributor)
	* [`SelectingFileDistributor`](#SelectingFileDistributor)
* [`/rdf` package](#ardf_package)
	* [`SelectFromContentDistributor`](#SelectFromContentDistributor)
	* [`RdfGraphDistributor`](#RdfGraphDistributor)
	* [`SelectFromGraphDistributor`](#SelectFromGraphDistributor)
* [`/rdf/graphbuilder` package](#ardfgraphbuilder_package)
	* [`EmptyGraphBuilder`](#EmptyGraphBuilder)
	* [`ConstructQueryGraphBuilder`](#ConstructQueryGraphBuilder)
	* [`IteratingGraphBuilder`](#IteratingGraphBuilder)
	* [`DrillDownGraphBuilder`](#DrillDownGraphBuilder)
* [`/decorator` package](#adecorator_package)
	* [`JavaScriptTransformDistributor`](#JavaScriptTransformDistributor)

## /example package

### `HelloDistributor`
![UML for HelloDistributor](images/uml_HelloDistributor.png)

**Description**

A simple implementation, used for demonstrating the installation procedures.

**Configuration properties**

| Property | Meaning | How many? |
| --- | --- | --- |
| `actionName` | Associates the instance with an HTTP request. | exactly one |

**Request parameters**

| Parameter | Meaning |
| --- | --- |
| name | The name that will be echoed in the response |

**Example configuration:** 

```  
@prefix : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:data_distributor_hello
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.examples.HelloDistributor> ;
    :actionName "hello" .
```

**Example request:** 

```
http://localhost:8080/vivo/api/dataRequest/hello?name=World
```

----------------------------------------------

## /file package

### `FileDistributor`

![UML for FileDistributor](images/uml_FileDistributor.png)

**Description**

Sends the contents of a file as an HTTP response. 

**Configuration properties**

| Parameter | Meaning | How many? |
| --- | --- | --- |
| `actionName` | Associates the instance with an HTTP request. | exactly one |
| `contentType` | The MIME type to be sent in the HTTP response header. | exactly one |
| `path` | The location of the file. If a relative path, it is relative to the Vitro home directory. | exactly one |

**Request parameters**

| Parameter | Meaning |
| --- | --- |
| _none_ | |

**Example configuration:** 

```
@prefix : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:data_distributor_organization_research_areas
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.file.FileDistributor> ;
    :actionName "research_areas" ;
    :contentType "text/turtle" .
    :path "visualizationData/bme_research_areas.ttl" ;
```

**Example request:** 

```
http://localhost:8080/vivo/api/dataRequest/research_areas
```

----------------------------------------------

### `SelectingFileDistributor `

![UML for SelectingFileDistributor](images/uml_SelectingFileDistributor.png)

**Description**

Serves the contents of a file. Selects the file by extracting a value from the HTTP request, and using it to construct the file path.

**Configuration properties**

| Parameter | Meaning | How many? |
| --- | --- | --- |
| `actionName` | Associates the instance with an HTTP request. | exactly one |
| `contentType` | The MIME type to be sent in the HTTP response header. | exactly one |
| `parameterName` | The request parameter that will contain the file selector. | exactly one |
| `parameterPattern` | A regular expression to extract the file selector from the parameter value. | exactly one |
| `filepathTemplate` | A template for constructing the file path from the selection value. If the constructed path is relative, it is relative to the Vitro home directory. | exactly one |
| `emptyResponse` |  A string to be served as an "empty data set", if the file is not found. | exactly one |


**Request parameters**

| Parameter | Meaning |
| --- | --- |
| _named in the configuration_| Provides a file selector value. |

**Example configuration:** 

```
@prefix : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:sf_distributor
  a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
      <java:edu.cornell.library.scholars.webapp.controller.api.distribute.file.SelectingFileDistributor> ;
  :actionName "collaboration_sunburst" ;
  :contentType "application/json" ;
  :parameterName "department" ;
  :parameterPattern "[^/#]+$";
  :filepathTemplate "crossunit-\\0.json" ;
  :emptyResponse "[]" .
```

**Example request:** 

```
http://localhost:8080/vivo/api/dataRequest/collaboration_sunburst?department=http%3A%2F%2Fmy.domain.edu%2Findividual%2Fn1234
```

----------------------------------------------

## /rdf package

### `SelectFromContentDistributor`

![UML for SelectFromContentDistributor](images/uml_SelectFromContentDistributor.png)

**Description**

Executes a SPARQL SELECT query against Vitro's content triple-store, and 
returns the results in JSON format: `application/sparql-results+json`.

You may specify the names of variables in the SPARQL query that should be bound
to request parameters before the query is executed. Each value may be bound as
a URI or as a plain literal value. If the request does not contain exactly one 
parameter for each specified binding, the response will be `400 Bad Request` 
with an informative message.

**Configuration properties**

| Parameter | Meaning | How many? |
| --- | --- | --- |
| `actionName` | as above | exactly one |
| `query` | The SPARQL SELECT query. | exactly one |
| `uriBinding` | The name of a request parameter whose value should be bound in the query as a URI. | zero or more | 
| `literalBinding` | The name of a request parameter whose value should be bound in the query as a plain literal. | zero or more | 

**Request parameters**

| Parameter | Meaning |
| --- | --- |
| _as named in the configuration_ | Values to be bound as URIs in the SPARQL query. |
| _as named in the configuration_ | Values to be bound as plain literals in the SPARQL query. |

**Example configuration:** 

```
@prefix      : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:sample_select_from_content_distributor
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.SelectFromContentDistributor> ;
    :actionName "sampleAction" ;
    :query """
      PREFIX foo: &lt;http://some.silly.domain/foo#&gt;
      SELECT ?article
      WHERE {
        ?person foo:isAuthor ?article .
        ?article foo:hasTopic ?topic .
      }
    """ ;
    :uriBinding "person" ;
    :literalBinding "topic" .
```

**Example request:** 

```
http://localhost:8080/vivo/api//sampleAction?person=http%3A%2F%2Fmy.domain.edu%2Findividual%2Fn1234&topic=Oncology
```

----------------------------------------------

### `RdfGraphDistributor`

![UML for RdfGraphDistributor](images/uml_RdfGraphDistributor.png)

**Description**

Executes one or more [`GraphBuilder`](#ardfgraphbuilder_package) instances. Merges the results into an
internal RDF graph, and returns that graph in Turtle format: `text/turtle`.

The `GraphBuilder` instances may accept configuration parameters, as specified for the class of each `GraphBuilder`.

**Configuration properties**

| Parameter | Meaning | How many? |
| --- | --- | --- |
| `actionName` | Associates the instance with an HTTP request. | exactly one |
| `graphBuilder` | Creates the internal RDF graph. | one or more | 
 
**Request parameters**

| Parameter | Meaning |
| --- | --- |
| _as permitted for graph builders_ | See the description of the relevant `GraphBuilder`(s). |

**Example configuration:** 

```
@prefix      : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:data_distributor_mapping
    a   <java:edu.cornell.mannlib.vitro.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.mannlib.vitro.webapp.controller.api.distribute.rdf.RdfGraphDistributor> ;
    :actionName "mapper" ;
    :graphBuilder :mapping_graph_builder ;
    
:mapping_graph_builder
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :uriBinding "person" ;
    :constructQuery """
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX vivo: <http://vivoweb.org/ontology/core#>
      CONSTRUCT {
        ?pub rdfs:label ?articleName .
      } WHERE {
        ?person vivo:relatedBy ?auth .
        ?auth a vivo:Authorship .
        ?auth vivo:relates ?pub .
        ?pub rdfs:label ?articleName .
      }
    """ .
```

**Example request:** 

```
http://localhost:8080/vivo/api/mapper?person=http%3A%2F%2Fmy.domain.edu%2Findividual%2Fn1234
```

----------------------------------------------

### `SelectFromGraphDistributor`

![UML for SelectFromGraphDistributor](images/uml_SelectFromGraphDistributor.png)

**Description**

Executes a SPARQL SELECT query against an internal RDF graph. The graph is created by merging the outputs of one or more `GraphBuilder` instances. The results are returned in JSON format: application/sparql-results+json.

This is commonly used as a way to issue a complicated SELECT query against
a subset of the content triple-store, when querying the full triple-store
would take too long. The internal graph is built from one or more CONSTRUCT
queries, and the SELECT query is executed on the resulting model.

However, it is not restricted to such use. The internal graph may be built
from any source of data (see the [`GraphBuilder`](#ardfgraphbuilder_package) classes, below).

You may bind values from the HTTP request to variables in the SELECT query as specified in [`SelectFromContentDistributor`](#SelectFromContentDistributor). The `GraphBuilder` instances may also accept configuration parameters, as specified for the class of each `GraphBuilder`.

**Configuration properties**

| Parameter | Meaning | How many? |
| --- | --- | --- |
| `actionName` | Associates the instance with an HTTP request. | exactly one |
| `query` | The SPARQL SELECT query. | exactly one |
| `uriBinding` | See [`SelectFromContentDistributor`](#SelectFromContentDistributor). | zero or more | 
| `literalBinding` | See [`SelectFromContentDistributor`](#SelectFromContentDistributor). | zero or more | 
| `graphBuilder` | Creates the local RDF graph, which the SPARQL SELECT query will run against. | one or more | 

**Request parameters**

| Parameter | Meaning |
| --- | --- |
| _as named in the configuration_ | See [`SelectFromContentDistributor`](#SelectFromContentDistributor). |
| _as permitted for graph builders_ | See the description of the relevant `GraphBuilder`(s). |

**Example configuration:** 

```
@prefix      : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:from_graph_distributor
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.SelectFromGraphDistributor> ;
    :actionName "fromGraph" ;
    :query """
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      SELECT ?person ?article
      WHERE {
        ?article rdfs:label ?articleName .
      }
    """ ;
    :uriBinding "person" ;
    :literalBinding "topic" ;
    :graphBuilder :from_graph_builder .
    
:from_graph_builder
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :uriBinding "person" ;
    :constructQuery """
      PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX vivo: <http://vivoweb.org/ontology/core#>
      CONSTRUCT {
        ?pub rdfs:label ?articleName .
      } WHERE {
        ?person vivo:relatedBy ?auth .
        ?auth a vivo:Authorship .
        ?auth vivo:relates ?pub .
        ?pub rdfs:label ?articleName .
      }
    """ .
```

**Example request:** 

```
http://localhost:8080/vivo/api/fromGraph?person=http%3A%2F%2Fmy.domain.edu%2Findividual%2Fn1234
```

----------------------------------------------


## /rdf/graphbuilder package

As shown in the previous section, some `DataDistributor` instances are 
configured with one or more `GraphBuilder` instances, to build an RDF graph which 
is used in building the HTTP response.

### EmptyGraphBuilder

![UML for EmptyGraphBuilder](images/uml_EmptyGraphBuilder.png)

**Description**

Creates an RDF graph containing no triples. Used for tests, examples, or placeholders.

**Configuration properties**

| Parameter | Meaning | How many? |
| --- | --- | --- |
| _none_ | | |

**Request parameters**

| Parameter | Meaning |
| --- | --- |
| _none_ | |

**Example configuration:** 

```
@prefix      : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:placeholder
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.EmptyGraphBuilder> .
```

----------------------------------------------

### ConstructQueryGraphBuilder

![UML for ConstructQueryGraphBuilder](images/uml_ConstructQueryGraphBuilder.png)

**Description**

Executes SPARQL CONSTRUCT query(s) against Vitro's content triple-store, and 
returns a model that contains the (merged) results.

Bindings for URI values or plain literal values are available as described above
for [`SelectFromContentDistributor`](#SelectFromContentDistributor).


**Configuration properties**

| Parameter | Meaning | How many? |
| --- | --- | --- |
| `constructQuery` | The SPARQL CONSTRUCT query. | one or more |
| `uriBinding` | See [`SelectFromContentDistributor`](#SelectFromContentDistributor). | zero or more | 
| `literalBinding` | See [`SelectFromContentDistributor`](#SelectFromContentDistributor). | zero or more | 

**Request parameters**

| Parameter | Meaning |
| --- | --- |
| _as named in the configuration_ | See [`SelectFromContentDistributor`](#SelectFromContentDistributor). |

**Example configuration:** 

```
@prefix      : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:authors_graph_builder
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :uriBinding "person" ;
    :constructQuery """
      PREFIX vivo: <http://vivoweb.org/ontology/core#>
      CONSTRUCT {
        ?person vivo:relatedBy ?auth .
        ?auth a vivo:Authorship .
        ?auth vivo:relates ?pub .
      } WHERE {
        ?person vivo:relatedBy ?auth .
        ?auth a vivo:Authorship .
        ?auth vivo:relates ?pub .
      }
    """ .

```

----------------------------------------------

### IteratingGraphBuilder

![UML for IteratingGraphBuilder](images/uml_IteratingGraphBuilder.png)

**Description**

A `GraphBuilder` decorator that runs one or more "child" builders multiple times,
each time providing a different value for a specified request parameter. 
The results of all queries are merged into a local RDF graph.

This permits the easy execution of multiple SPARQL queries that differ only
in the value of a bound variable.

**Configuration properties**

| Parameter | Meaning | How many? |
| --- | --- | --- |
| `parameterName` | the request parameter that will be augmented when running the `childGraphBuilder`(s). | exactly one |
| `parameterValue` | the values that will be added to the named parameter, one value for each run of the `childGraphBuilder`(s). | one or more |
| `childGraphBuilder` | The "decorated" `GraphBuilder` instance(s), which will produce the RDF graph. | one or more |

**Request parameters**

| Parameter | Meaning |
| --- | --- |
| _none_ | |

**Example configuration:** 

```
@prefix      : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:iterating_graph_builder
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.IteratingGraphBuilder> ;
    :parameterName "subjectArea" ;
    :parameterValue "cancer", "tissues", "echidnae" ;
    :childGraphBuilder :subordinate_graph_builder .

:subordinate_graph_builder
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :literalBinding "subjectArea" ;
    :constructQuery """
      PREFIX foo: &lt;http://some.silly.domain/foo#&gt;
      CONSTRUCT {
        ?article foo:hasKeyword ?subjectArea .
      }
      WHERE {
        ?article foo:hasKeyword ?subjectArea .
      }
    """ .
```

----------------------------------------------

### DrillDownGraphBuilder

![UML for DrillDownGraphBuilder](images/uml_DrillDownGraphBuilder.png)

**Description**

A `GraphBuilder` decorator that runs one or more "child" builders multiple times,
each time providing a different value for a specified request parameter. 
The results of all queries are merged into a local RDF graph.

A top-level `GraphBuilder` is run, and a SPARQL SELECT query is run against the 
resulting graph, to obtain a list of values. This list of values is then used 
to iterate over the child `GraphBuilder` instances.

This is similar to [`IteratingGraphBuilder`](#IteratingGraphBuilder), but
the values for the request parameter are not specified in the configuration. 
Instead they are discovered, using the top-level `GraphBuilder` and the 
configured SPARQL SELECT query.

Note that a SPARQL SELECT query may return more than one named value for each result.
In that case, each set of values will be used in turn to augment the request
parameters as seen by the child `GraphBuilder`(s).

**Configuration properties**

| Parameter | Meaning | How many? |
| --- | --- | --- |
| `topLevelGraphBuilder` | The source of the top-level graph, against which the `drillDownQuery` is run. | exactly one |
| `drillDownQuery` | Discovers the values that will be passed to the child `GraphBuilder` instance(s). | exactly one |
| `childGraphBuilder` | The "decorated" `GraphBuilder` instance(s), which will produce the RDF graph. | one or more |
| `uriBinding` | See [`SelectFromContentDistributor`](#SelectFromContentDistributor). These bindings will be applied to the `drillDownQuery`. They are not passed on to the `childGraphBuilder` instance(s). | zero or more | 
| `literalBinding` | See [`SelectFromContentDistributor`](#SelectFromContentDistributor). These bindings will be applied to the `drillDownQuery`. They are not passed on to the `childGraphBuilder` instance(s). | zero or more | 

**Request parameters**

| Parameter | Meaning |
| --- | --- |
| _as named in the configuration_ | See [`SelectFromContentDistributor`](#SelectFromContentDistributor). |

**Example configuration:** 

```
@prefix      : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:drilldown_graph_builder
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.DrillDownGraphBuilder> ;
    :topLevelGraphBuilder :top_level_builder ;
    :childGraphBuilder :child_graph_builder ;
    :drillDownQuery """
      PREFIX foo: &lt;http://some.silly.domain/foo#&gt;
      SELECT ?article ?subjectArea
      WHERE {
        foo:author_123 foo:writes ?article ;
        ?article foo:pertainsTo ?subjectArea .
      }
    """ .

:top_level_graph_builder
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :literalBinding "subjectArea" ;
    :constructQuery """
      PREFIX foo: &lt;http://some.silly.domain/foo#&gt;
      CONSTRUCT {
        ?author foo:writes ?article ;
        ?article foo:pertainsTo ?subjectArea .
      }
      WHERE {
        ?author foo:writes ?article ;
        ?article ?property ?subjectArea .
      }
    """ .
    
:child_graph_builder
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :literalBinding "subjectArea" ;
    :constructQuery """
      PREFIX foo: &lt;http://some.silly.domain/foo#&gt;
      CONSTRUCT {
        ?article ?property ?subjectArea .
      }
      WHERE {
        ?article ?property ?subjectArea .
      }
    """ .
```

----------------------------------------------

## /decorator package

### `JavaScriptTransformDistributor`
![UML for JavaScriptTransformDistributor](images/uml_JavaScriptTransformDistributor.png)

**Description**

A wrapper that uses a JavaScript function to transform the output of the wrapped distributor.

JavaScript code is provided by an inline script and optional script files loaded from the webapp. There must be a `transform()` function that accepts a String as argument (the output of the wrapped distributor), and returns a String as result.

A `org.apache.commons.logging.Log` Java object is created and bound to a global JavaScript variable called `logger`. The script can write to the VIVO log file by calling `logger.error()`, `logger.debug()`, etc. 
The category of the `Log` is `edu.cornell.library.scholars.webapp.controller.api.distribute.decorator.JavaScriptTransformDistributor.[actionName]`. The logging level can be set by the usual mechanisms.

**Configuration properties**

| Property | Meaning | How many? |
| --- | --- | --- |
| `actionName` | Associates the instance with an HTTP request. | exactly one |
| `contentType` | The MIME type to be sent in the HTTP response header. | exactly one |
| `script` | A string of JavaScript to be interpreted and executed. It must contain a function named `transform` which accepts a String as argument and returns a String as result. | exactly one |
| `supportingScript` | The path to a JavaScript file in the webapp. The path must be as specified for [`ServletContext.getResource()`](https://docs.oracle.com/javaee/7/api/javax/servlet/ServletContext.html#getResource-java.lang.String-). That is, it must begin with a '/' and is interpreted as relative to the context root of the webapp. | zero or more |
| `child` | The "wrapped" Data Distributor. | exactly one |

**Request parameters**

| Parameter | Meaning |
| --- | --- |
| _as required by the child distributor_ | The request context will be passed to the child. |

**Example configuration:** 

In this example, the "Hello, World" distributor is wrapped with a script that will add exclamation marks to the response as emphasis.

Note that 

* The wrapper must have an `actionName` that matches the request. 
* The child must also have an `actionName`, but it is arbitrary, since the child will be invoked by its `child` relationship to the wrapper.

```  
@prefix : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .

:enthusiastic_wrapper_distributor
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.decorator.JavaScriptTransformDistributor> ;
    :actionName "hello" ;
    :contentType "text/plain" ;
    :child :hello_distributor ;
    :script """
        function transform(rawOutput) {
            return rawOutput + "!!! OMG!!!"
        }
    """ .

:hello_distributor
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.examples.HelloDistributor> ;
    :actionName "" .
```

**Example request:** 

```
http://localhost:8080/vivo/api/dataRequest/hello?name=World
```

----------------------------------------------
