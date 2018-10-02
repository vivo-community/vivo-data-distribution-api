# Three sets of queries with slight variations

The graphs visualization in Scholars@Cornell required a complex set of data.

* This could have been expressed as a twelve separate queries, consisting of three groups of almost identical queries; a maintenance headache.

* Or, it could have been expressed as three queries with several UNION clauses among them.
This could have been a maintenance headache or a performance headache, depending on the location of the UNIONs.

* Instead it uses three `IteratingGraphBuilder` instances. Actually four, because two are nested together.

An `IteratingGraphBuilder` is configured to create multiple instances of a `GraphBuilder`, each using the same query, but with one value changed in the parameter map. So, for example, `builder_B` will create three instances of `builder_B1`. 

* The first instance receives a parameter map in which `grantClass` is set to `vivo:Grant`. 
* The second instance receives a parameter map in which `grantClass` is set to `vivo:Contract`. 
* The third instance receives a parameter map in which `grantClass` is set to `vivo:CooperativeInvestigation`. 

`builder_C` does the same with three instances of `builder_C1`.

`builder_A` takes this one step further, using three instances of `builder_A1` (another `IteratingGraphBuilder`)
to create six instances of `builder_A2`. These six instances receive parameter maps with the following values (in addition to those from the request)

| instance | value of `grantClass` | value of `investigatorRoleClass`|
| --- | --- | --- |
| 1 | `vivo:Grant` | `vivo:PrincipalInvestigatorRole` |
| 2 | `vivo:Grant` | `vivo:CoPrincipalInvestigatorRole` |
| 3 | `vivo:Contract` | `vivo:PrincipalInvestigatorRole` |
| 4 | `vivo:Contract` | `vivo:CoPrincipalInvestigatorRole` |
| 5 | `vivo:CooperativeInvestigation` | `vivo:PrincipalInvestigatorRole` |
| 6 | `vivo:CooperativeInvestigation` | `vivo:CoPrincipalInvestigatorRole` |

The results of all of these queries are merged into a single in-memory model, and the `SelectFromGraphDistributor`
runs a SELECT query against that model, and serves the result.

![Instances and data flow](images/example_iteratedQueries.png)

## Equivalent options

The configuration could have been restructured as shown below. 
This is effectively equivalent, both in terms of performance and maintenance.
There is no obvious reason to prefer one over the other, except for clarity, which is subject to debate.

![Instances and data flow](images/example_iteratedQueries_again.png)

## Performance

Even when broken down into simple queries, the time taken by this distributor was prohibitive. In a development environment, the query ran for more than 45 seconds. It was never deployed to production.

Instead, the query was run in a batch job, and the results were cached, as described in 
[Cacheing for Performance](./example_cacheingForPerformance.html).

## The configuration

```
@prefix : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .

#
# Grants data:
#
# Twelve construct queries and a select query. 
#
# The first six construct queries handle the combinations of 
# Grant/Contract/CooperativeInvestigation and PrincipalInvestigator/CoPrincipalInvestigator. 
# The six separate queries run much more quickly than a single query using UNIONs or FILTERs. 
# 
# The remaining six construct queries provide two sets of optional data for 
# Grant/Contract/CooperativeInvestigation.
#
# The select query creates a result set from the graph built by the construct queries.
#

:ddgrants
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.SelectFromGraphDistributor> ;
    :actionName "grants_bubble_chart_cache" ;
    :graphBuilder :builder_A ;
    :graphBuilder :builder_B ;
    :graphBuilder :builder_C ;
    :query """
      PREFIX local:          <http://scholars.cornell.edu/individual/>
      PREFIX obo:            <http://purl.obolibrary.org/obo/>
      PREFIX rdfs:           <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX scholars-grant: <http://scholars.cornell.edu/ontology/grant.owl#>
      PREFIX scholars-hr:    <http://scholars.cornell.edu/ontology/hr.owl#>
      PREFIX vivo:           <http://vivoweb.org/ontology/core#>
      
      SELECT ?grant ?grantTitle ?grantId ?type
             ?amount ?fundingOrg ?fundingOrgName
             ?person ?personName ?personNetid ?role
             ?dept ?deptName
             ?startdt ?enddt
      
      WHERE {
        ?grant a ?type .
        ?grant rdfs:label ?grantTitle .
        ?grant vivo:localAwardId ?grantId .
        ?grant vivo:totalAwardAmount ?amount .
        OPTIONAL {
          ?grant vivo:assignedBy ?fundingOrg .
          ?fundingOrg rdfs:label ?fundingOrgName .
        }
       
        ?grant vivo:relates ?node1 .
        ?node1 a ?role .
        ?node1 obo:RO_0000052 ?person .
        ?person rdfs:label ?personName .
        ?person scholars-hr:netId ?personNetid .
      
        OPTIONAL {
          ?grant vivo:relates ?node2 .
          ?node2 a vivo:AdministratorRole .
          ?node2 obo:RO_0000052 ?dept .
          ?dept rdfs:label ?deptName .
        }
        
        ?grant vivo:dateTimeInterval ?dti .
        ?dti vivo:end ?end .
        ?end vivo:dateTime ?enddt .
        ?dti vivo:start ?start .
        ?start vivo:dateTime ?startdt .
      }
      """ .
      
:builder_A
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.IteratingGraphBuilder> ;
    :parameterName "grantClass" ;
    :parameterValue 
        "http://vivoweb.org/ontology/core#Grant", 
        "http://vivoweb.org/ontology/core#Contract", 
        "http://vivoweb.org/ontology/core#CooperativeInvestigation" ;
    :childGraphBuilder :builder_A1 .

:builder_A1
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.IteratingGraphBuilder> ;
    :parameterName "investigatorRoleClass" ;
    :parameterValue 
        "http://vivoweb.org/ontology/core#PrincipalInvestigatorRole", 
        "http://vivoweb.org/ontology/core#CoPrincipalInvestigatorRole";
    :childGraphBuilder :builder_A2 .

:builder_A2
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :uriBinding "grantClass" ;
    :uriBinding "investigatorRoleClass" ;
    :constructQuery """
      PREFIX obo:            <http://purl.obolibrary.org/obo/>
      PREFIX rdfs:           <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX scholars-grant: <http://scholars.cornell.edu/ontology/grant.owl#>
      PREFIX scholars-hr:    <http://scholars.cornell.edu/ontology/hr.owl#>
      PREFIX vivo:           <http://vivoweb.org/ontology/core#>
 
      CONSTRUCT {     
        ?grant a ?grantClass .
        ?grant vivo:relates ?roleNode .
        ?roleNode a ?investigatorRoleClass .

        ?grant rdfs:label ?grantTitle .
        ?grant vivo:localAwardId ?grantId .
        ?grant vivo:totalAwardAmount ?amount .
       
        ?roleNode obo:RO_0000052 ?person .
        ?person rdfs:label ?personName .
        ?person scholars-hr:netId ?personNetid .
       
        ?grant vivo:dateTimeInterval ?dti .
        ?dti vivo:end ?end .
        ?end vivo:dateTime ?enddt .
        ?dti vivo:start ?start .
        ?start vivo:dateTime ?startdt .
      } 
      WHERE {
        ?grant a ?grantClass .
        ?grant vivo:relates ?roleNode .
        ?roleNode a ?investigatorRoleClass .

        ?grant rdfs:label ?grantTitle .
        ?grant vivo:localAwardId ?grantId .
        ?grant vivo:totalAwardAmount ?amount .
       
        ?roleNode obo:RO_0000052 ?person .
        ?person rdfs:label ?personName .
        ?person scholars-hr:netId ?personNetid .
       
        ?grant vivo:dateTimeInterval ?dti .
        ?dti vivo:end ?end .
        ?end vivo:dateTime ?enddt .
        ?dti vivo:start ?start .
        ?start vivo:dateTime ?startdt .
      }
      """ .
      
:builder_B
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.IteratingGraphBuilder> ;
    :parameterName "grantClass" ;
    :parameterValue 
        "http://vivoweb.org/ontology/core#Grant", 
        "http://vivoweb.org/ontology/core#Contract", 
        "http://vivoweb.org/ontology/core#CooperativeInvestigation" ;
    :childGraphBuilder :builder_B1 .

:builder_B1
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :uriBinding "grantClass" ;
    :constructQuery """
      PREFIX rdfs:           <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX vivo:           <http://vivoweb.org/ontology/core#>
 
      CONSTRUCT {     
        ?grant a ?grantClass .
        ?grant vivo:assignedBy ?fundingOrg .
        ?fundingOrg rdfs:label ?fundingOrgName .
      } 
      WHERE {
        ?grant a ?grantClass .
        ?grant vivo:assignedBy ?fundingOrg .
        ?fundingOrg rdfs:label ?fundingOrgName .
      }
      """ .
      
:builder_C
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.IteratingGraphBuilder> ;
    :parameterName "grantClass" ;
    :parameterValue 
        "http://vivoweb.org/ontology/core#Grant", 
        "http://vivoweb.org/ontology/core#Contract", 
        "http://vivoweb.org/ontology/core#CooperativeInvestigation" ;
    :childGraphBuilder :builder_C1 .

:builder_C1
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.GraphBuilder> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.rdf.graphbuilder.ConstructQueryGraphBuilder> ;
    :uriBinding "grantClass" ;
    :constructQuery """
      PREFIX obo:            <http://purl.obolibrary.org/obo/>
      PREFIX rdfs:           <http://www.w3.org/2000/01/rdf-schema#>
      PREFIX vivo:           <http://vivoweb.org/ontology/core#>
 
      CONSTRUCT {     
        ?grant a ?grantClass .
        ?grant vivo:relates ?node2 .
        ?node2 a vivo:AdministratorRole .
        ?node2 obo:RO_0000052 ?dept .
        ?dept rdfs:label ?deptName .
      } 
      WHERE {
        ?grant a ?grantClass .
        ?grant vivo:relates ?node2 .
        ?node2 a vivo:AdministratorRole .
        ?node2 obo:RO_0000052 ?dept .
        ?dept rdfs:label ?deptName .
      }
      """ .

```