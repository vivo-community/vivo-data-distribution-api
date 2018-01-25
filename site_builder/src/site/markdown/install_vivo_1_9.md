# Installing in VIVO 1.9
Install the API in VIVO by following these steps:

* Edit `{Vitro}/dependencies/pom.xml`, adding the Data Distribution API as a dependency, from its own repository.
* Configure a sample distributor.
* Build VIVO as you would normally.

In these instructions, `{VIVO}` and `{Vitro}` are used to indicate the paths to the directories of the VIVO distribution. 

More details are in the sections below.

_**Note: These instructions describe how to install a stable release of the API. 
To install a (non stable) SNAPSHOT, use 
[these instructions](installing_the_latest_snapshot.html#vivo1.9) 
instead.**_

## Edit `{Vitro}/dependencies/pom.xml`

There are many `<dependency>` tags within the `<dependencies>` section of `pom.xml`. Add another dependency:

```
  <!-- Data Distribution API -->
  <dependency>
    <groupId>edu.cornell.library.scholars</groupId>
    <artifactId>data-distribution-api-vivo_1_09</artifactId>
    <version>1.1</version>
    <type>jar</type>
  </dependency>
```

Before editing, you might see this at the end of `pom.xml`:

![pom.xml before editing](images/pom_xml_vivo1.9_before.png)

After editing, you would see this:

![pom.xml after editing](images/pom_xml_vivo1.9_after_release.png)

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
