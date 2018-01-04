# Why another API in VIVO?

This API was developed at Cornell University as part of the Scholars@Cornell 
project. Scholars@Cornell is a local customization of VIVO.

One of the goals of Scholars is to deliver visualizations that are:

* attractive and engaging,
* inherently informative,
* rich with navigation links.

In addition, the visualizations must be 

* responsive, so as not to interrupt the user experience,
* easy to develop, to fit in with the frequent iterations of Scholars development.

The Data Distribution API began with these requirements, but has since grown to provide a more general-purpose solution for communicating with VIVO.

## The philosophy of the API
Today's computing environment is ideally suited to interactive visualizations, and distributed use of data.

The Data Distribution API takes advantage of the fact that modern browsers contain optimized JavaScript engines, running mature and expressive JavaScript frameworks on very powerful processors.

In light of that, the responsibility of the server is to quickly provide data when requested, and to let the client device use that data as it sees fit.

## Challenges for the API

### Making development easier

The Data Distribution API will work with any Java object that implements the `DataDistributor` interface. But maintaining Java code in VIVO can be difficult. Many VIVO maintainers are not familiar with the internal structure of the VIVO code base. Some are not familiar with Java.

The problem is compounded by the need to rebuild VIVO every time the Java code changes. It can take several minutes to demonstrate even the simplest Java change in VIVO.

#### The solution

One of the basic strategies of the Data Distribution API has been to create general-purpose Java code that can be configured for a wide variety of use cases. The configuration resides in the content triple-store of the VIVO instance.

As a result, a change to the Data Distribution API usually requires no Java coding, and no rebuild. Instead, the maintainer will modify a file of RDF configuration data and restart Tomcat.

### Limiting the data window

VIVO already supports a SPARQL query API. By default, this API is disabled, 
because it is dangerous to let users issue arbitrary SPARQL requests.

* Relatively simple SPARQL requests can severely affect VIVO's performance.
* All of VIVO's content has the same protection level
	* if a user can see any of the data, they can see all of it.
* None of VIVO's configuration data is available, regardless of the user's authorization level.

#### The solution

With the Data Distribution API, a VIVO administrator creates a configuration file describing how to satisfy each request. 
The requests may be parameterized for flexibility, but the form of the request is fixed. The user knows only:

* The name of the configured request.
* The parameters that the request will accept.
* The format of the response.

The Data Distribution API provides a hook into VIVO's existing authorization API, permitting granular authorization to data.

### Serving data from many sources

VIVO provides many ways to serve data from the content models and the search index. In most cases, these are special-purpose connections, implemented on a one-off basis without much thought for code re-use or standard calling sequences. In many cases, there is no documentation.

#### The solution

The Data Distribution API provides a mechanism to serve data from:

* The content models,
* The Solr index,
* Static files that were created off-line,
* Data obtained in real-time from other sites.

The interface is general purpose, extensible, and well-documented.
