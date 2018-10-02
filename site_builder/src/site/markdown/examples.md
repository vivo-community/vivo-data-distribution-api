# Examples from the wild

## [A simple CONSTRUCT query](./example_simpleRdfGraph.html)

An `RdfGraphdistributor` with a `ConstructQueryGraphBuilder`.

## [Three sets of queries with slight variations: total of 12 queries](./example_iteratedQueries.html)

A `SelectFromGraphDistributor`, with three (or four) `IteratingGraphBuilder`s wrapping three 
`ConstructQueryGraphBuilder`s.

## [Drilldown with multiple queries, wrapped in a Javascript formatter](./example_drilldownAndFormatting.html)

An `RdfGraphDistributor` with a `DrillDownGraphBuilder` and two `ConstructQueryGraphBuilder`s, 
all wrapped in a `JavaScriptTransformDistributor`.

## [Serve a JSON file for each department](./example_selectedJsonFiles.html)

A `SelectingFileDistributor`

## [Cacheing the output when performance is unacceptable](./example_cacheingForPerformance.html)

A `FileDistributor`