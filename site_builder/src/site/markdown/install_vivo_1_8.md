# Installing in VIVO 1.8
Install the API in VIVO by following these steps:

* Download the JAR file and add it to your Vitro distribution directory.
* Edit `web.xml` in VIVO, or in your third tier, if you have one, to establish a `servlet-mapping` for the Data Distribution API.
* Configure a sample distributor.
* Build VIVO as you would normally.

In these instructions, `{VIVO}` and `{Vitro}` are used to indicate the paths to the directories of the VIVO distribution. 

More details are in the sections below.

_**Note: These instructions describe how to install a stable release of the API. 
To install a (non stable) SNAPSHOT, use 
[these instructions](installing_the_latest_snapshot.html#vivo1.8) 
instead.**_

## Download the JAR file and add it to your Vitro distribution directory.

The JAR file is located in the [Snapshots repository at Sonotype.org](https://oss.sonatype.org/content/repositories/snapshots). You can browse through the directory structure to find the most recent snapshot for the API, and download the JAR file. 

Alternatively, if you have Maven installed on your machine, you can use this command to fetch the most recent snapshot to the local repository on your machine:

```
mvn org.apache.maven.plugins:maven-dependency-plugin:get \
    -DgroupId=edu.cornell.library.scholars \
    -DartifactId=data-distribution-api-vivo_1_08 \
    -Dversion=1.1 \
    -Dtransitive=false
```

When maven completes, you will find the JAR file here:

```
~/.m2/repository/edu/cornell/library/scholars/data-distribution-api-vivo_1_08/1.1/data-distribution-api-vivo_1_08-1.1.jar
```

Copy or move the JAR file to your project's top-level `lib` directory.

* If you are using a standard VIVO distribution, add the JAR file to `[VIVO]/lib`
* If you have added a third tier, add the JAR file to the `lib` directory of your third tier.

## Edit `web.xml`

You may have several files named `web.xml` in your project. Each tier overrides the previous tier, so:

* if you are using a standard VIVO distribution, you will edit `{VIVO}/webapp/src/main/webapp/web.xml`. 
* If you have added a third tier, with its own `web.xml` file, you should edit that one instead.
* If you have added a third tier, but it does not contain `web.xml`, you should edit the `web.xml` in VIVO, as above.

Add a `<servlet>` tag and a `<servlet-mapping>` tag to associate the 
`api/dataRequest` with the `DistributeDataApiController`, like this:

```
  <!-- Data Distribution API -->
  <servlet>
    <servlet-name>DistributeDataApi</servlet-name>
    <servlet-class>edu.cornell.library.scholars.webapp.controller.api.DistributeDataApiController</servlet-class>
  </servlet>

  <!-- Data Distribution API -->
  <servlet-mapping>
    <servlet-name>DistributeDataApi</servlet-name>
    <url-pattern>/api/dataRequest/*</url-pattern>
  </servlet-mapping>
```
These tags can be placed anywhere among the other servlet-related tags.

Before editing, you might see this in `web.xml`:

![web.xml before editing](images/web_xml_before.png)

After editing, you would see this:

![web.xml after editing](images/web_xml_after.png)

## Configure a distributor

Create a configuration file for the example distributor. The file will contain RDF data that tells the Data Distributor 
controller how to respond to requests. The file must be created in your VIVO distribution (or your third tier) 
in `{VIVO}/rdf/display/everytime`.

For this example, create a file named `HelloDistributorConfig.ttl`:

```
@prefix : <http://vitro.mannlib.cornell.edu/ns/vitro/ApplicationSetup#> .
  
:data_distributor_hello
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.examples.HelloDistributor> ;
    :actionName "hello" .
```

## Build VIVO

Run the Ant install script as you would normally.

# Does it work?
Start your VIVO running and direct your browser to `http://localhost:8080/vivo/api/dataRequest/hello?name=World`.
Your browser should display the following message:

```
Hello, World! 
```

