# Questions

  * how can I put a 1.1 Snapshot version on the Maven repo without clobbering the 1.0 version?
    * Should I start putting older versions onto Maven central?
  * Do I need to make manual updates to the pom.xml after each release
    * This would be to change the commit comments for the artifacts. But is it enough to 
      keep the earlier Maven POM from disappearing?
  * How to handle Cross-site distribution: Cross-Origin Resource Sharing (CORS)
	  * Reference: [Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
	  * For now, 
		  * Only simple requests are supported. Preflight requests are not supported.
		  * all HTTP responses will include the header `Access-Control-Allow-Origin: *`
	  * It would be better to 




# To build and deploy:

    mvn clean site deploy

This will:

* compile and build the code.
* update the code to the Maven repository on GitHub
* assemble the site pages, including creating the JavaDoc and converting the MarkDown
* deploy the site pages to GitHub.
    
# UML Diagrams    
`Modeling.mdj` is a file created using StarUML. It contains the model and diagrams
that were used to create the UML diagrams in the site pages.

# Putting VIVO 1.8 classes into a local file-based repository

## Create a JAR file of VIVO 1.8 classes

* Create a `build.properties` file in the VIVO distribution area

* Run a partial build of VIVO to compile all of the source
    
    ```
    cd [vivo-path]
    ant distribute
    ```
* JAR it up

    ```
    cd .build/main/webapp/WEB-INF/classes
    jar -cf [temp-path]/vivo_classes.jar *
    cd .build/main/testClasses
    jar -uf [temp-path]/vivo_classes.jar *
    ```
    
## Create a file-based repository within the project

* Make the repository

    ```
    mkdir [project_path]/vivo-and-vitro-dependencies-repo
    ```
    
* Populate it

    ```
    mvn deploy:deploy-file \
      -Durl=file://[project-path]/vivo-and-vitro-dependencies-repo \
      -DgroupId=edu.cornell.library.scholars \
      -DartifactId=vivo_classes \
      -Dpackaging=jar \
      -Dversion=1.8 \
      -Dfile=[temp-path]/vivo_1.8_classes.jar
    ```

## Declare it in the POM

* Declare the repository:

    ```
    <repository>
	   <id>vivo-and-vitro-dependencies-repo</id>
	   <url>file://${basedir}/vivo-and-vitro-dependencies-repo/</url>
	 </repository>

    ```

* Declare the dependency:

    ```
    <dependency>
      <groupId>edu.cornell.library.scholars</groupId>
      <artifactId>vivo_classes</artifactId>
      <version>1.8</version>
    </dependency>

    ```

### Reference
* [Stack Overflow: maven-add-a-dependency-to-a-jar-by-relative-path](https://stackoverflow.com/questions/2229757/maven-add-a-dependency-to-a-jar-by-relative-path/2230464#2230464)

# "Borrowing" source code from VIVO 1.10
The versions for VIVO 1.9 and VIVO 1.8 need to get the DDAPI source code and the source for some utility packages from VIVO 1.10. All of this source code must be modified to work with the older VIVO dependencies.

## Share the DDAPI source code
The source code and the tests are part of the `vivo_1.10` sub-project. They are copied as part of the filtering operation.

## Manually get the supporting "utils" packages
Like the JAR file of Vitro classes (above), this will be a one-time operation, since we are tracking a version of Vitro that is not being maintained.

Copy the source code from the Vitro distribution into this project. From:

    [Vitro]/api/src/main/java/edu/cornell/mannlib/vitro/webapp/utils/configuration
    [Vitro]/api/src/main/java/edu/cornell/mannlib/vitro/webapp/utils/sparqlrunner
    
to

    src_from_vitro-1.10/main/java/edu/cornell/mannlib/vitro/webapp/utils/configuration
    src_from_vitro-1.10/main/java/edu/cornell/mannlib/vitro/webapp/utils/sparqlrunner

    
## Filter the source so it will compile correctly
The imported sources were written as part of VIVO 1.10. They will not compile as part of VIVO 1.8 or 1.9 without some changes. These are:

* Change references to Apache Commons Language 3 (VIVO 1.10) to version 2 (VIVO 1.8, 1.9).
* Change references to Jena 3 (VIVO 1.10) to Jena 2 (VIVO 1.8, 1.9).
	* Also, remove reference to `RDFLangString`, which is not present in Jena 2.
* The `edu.cornell.mannlib.vitro.webapp.utils.sparqlrunner` package is not present in VIVO 1.8, and is incomplete in VIVO 1.9. Move this to the `vitro_1_10.edu.cornell.mannlib.vitro.webapp.utils.sparqlrunner` package. Change all references to point to the new location.
* The `edu.cornell.mannlib.vitro.webapp.utils.configuration` package is more advanced in VIVO 1.10 than in VIVO 1.8 or VIVO 1.9. Move this to the `vitro_1_10.edu.cornell.mannlib.vitro.webapp.utils.configuration` package. Change all references to point to the new location.

Filter the main source and test source from the Data Distribution API 1.10. 

Filter the borrowed Vitro 1.10 source.

## Add the source and tests to the project

Use the `build-helper-maven-plugin` to make Maven aware of these source directories and test directory.