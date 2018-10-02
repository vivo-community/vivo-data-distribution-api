# Cacheing the output when performance is unacceptable

[Three sets of queries with slight variations: total of 12 queries](./example_iteratedQueries.html) describes
a collection of queries that analyze all of the Grants and Contracts in the triple-store, and that take
more than 45 seconds to run.

To avoid this, two data distributors were created. 

* The first was invoked by a scheduled job, which executed the distributor and stored the output in a file. 
* The second was used interactively, and was merely a `FileDistributor` which served the cached output of the scheduled job.

Here is the beginning of the configuration for these distributors:

```
# 
# Serves the cached results.
#
:data_distributor_grants_page_cached
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.file.FileDistributor> ;
    :actionName "grants_bubble_chart" ;
    :path "visualizationData/grants_select_results.json" ;
    :contentType "application/sparql-results+json" .

#
# Run this in the background to create the cached result file.
#
:data_distributor_grants_page_iterate_select
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.SelectFromGraphDistributor> ;
    :actionName "grants_bubble_chart_cache" ;
    :graphBuilder :grants_page_graph_builder_A ;
    :graphBuilder :grants_page_graph_builder_B ;
    :graphBuilder :grants_page_graph_builder_C ;
    :query """
      PREFIX local:          <http://scholars.cornell.edu/individual/>
      PREFIX obo:            <http://purl.obolibrary.org/obo/>
      
...

```

This solution is quite satisfactory for two reasons:

* The data applies to the entire university. If we tried this technique on data for each individual faculty member, we would have thousands of cached files. This might constitute a problem.
* The data is relatively stable. It is enough to refresh this data daily.