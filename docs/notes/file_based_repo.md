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

## Reference
* [Stack Overflow: maven-add-a-dependency-to-a-jar-by-relative-path][stackOverflow1]

[stackOverflow1]:         https://stackoverflow.com/questions/2229757/maven-add-a-dependency-to-a-jar-by-relative-path/2230464#2230464
