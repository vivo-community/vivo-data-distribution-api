# Using in VIVO 1.10

VIVO 1.10 comes with the Data Distributor API already installed!
You will still need to configure your distributors before using the API.
Here is an example of how to configure a simple distributor.

In these instructions, `{VIVO}` is used to indicate the path to the main directory of the VIVO distribution. 

## Configure a distributor
Create a configuration file for the example distributor. The file will contain RDF data that tells the Data Distributor controller how to respond to requests. The file must be created in your VIVO distribution (or your third tier) in `{VIVO}/home/src/main/resources/rdf/display/everytime`.

For this example, create a file named `HelloDistributorConfig.ttl`:

```
@prefix : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .
  
:data_distributor_hello
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.examples.HelloDistributor> ;
    :actionName "hello" .
```

## Build VIVO
Run the Maven install script as you would normally.

# Does it work?
Start your VIVO running and direct your browser to `http://localhost:8080/vivo/api/dataRequest/hello?name=World`.
Your browser should display the following message:

```
Hello, World! 
```
