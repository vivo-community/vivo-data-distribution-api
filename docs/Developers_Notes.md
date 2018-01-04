# Questions

  * Do I need to make manual updates to the pom.xml after each release
    * This would be to change the commit comments for the artifacts. But is it enough to 
      keep the earlier Maven POM from disappearing?
  
# To do:

* Create a deploy profile. Only require signing if that profile is used.
	* Can we make it so `deploy` is the default target for that profile?

* Move the site deployment from `deploy` to `site-deploy`. 
	* Then, can I safely do these commands?
		* `mvn clean site`
		* `mvn clean deploy`
		* `mvn clean site-deploy`

* CORS 
	* Configure individual data distributors
		* Requires enhancement to the ConfigurationBeanLoader
	* Put headers on error responses also
		* The goal is that 404 should be reported as such by the browser, not as 400.
	* Support preflight requests 
		* At least enough to do basic authentication
	* Reference: [Cross-Origin Resource Sharing][corsReference]
 	


# Build, test, deploy

## Build and install locally
Use this when you want to test changes, but don't want to deploy to the OSS respository

```
cd {top-level-directory}
mvn clean install
```

* Compile the code, test, create the JARs.
* Update the code to the local repository

## Test the site pages
Use this when editing the site pages, to see what the site will look like.

```
cd {top-level-directory}/site_builder
mvn site:run
```

* Assemble the site pages
* Serve the pages at [http://localhost:9000](http://localhost:9000).
	* Note that the Javadoc pages are not created until you ask for them, so there may be some delay.

## Build and deploy
Use this when you are ready to deploy to the OSS repository

```
cd {top-level-directory}
mvn clean site deploy
```

* Compile the code, test, create the JARs.
* Update the code to the snapshot repository at Sonatype
* Assemble the site pages
* Deploy the site pages to GitHub.

**DO NOT run `install` and `deploy` in the same command, or you wind up with double signatures.**

## Update the site
Use this to deploy changes to the site pages without updating the OSS repository

```
cd {top-level-directory}/site_builder
mvn site deploy
```

* Assemble the site pages
* Deploy the site pages to GitHub.

## Change the version numbers
Use this when changing from SNAPSHOT to a release version, or vice versa

```
mvn release:update-versions -DautoVersionSubmodules=true
```

* Prompt you for a version number, and set it in the main project and sub-projects.

## Issue a release

Build and deploy with a non-SNAPSHOT version number.

Go through the OSS release process

* Reference: [Releasing the Deployment][ossReleaseDoc]

## Note: GPG-signatures

**The Maven script prompts for a passphrase that will be used in signing the artifacts.**

* The key used to sign the artifacts is the default key for the account where the build occurs. The public key must be published so people can verify the signatures.
* Check out the page at [http://central.sonatype.org/pages/working-with-pgp-signatures.html](http://central.sonatype.org/pages/working-with-pgp-signatures.html)

## Note: pause in the build script

**It takes some time to build the site.**

The build script will appear to pause at this message:

```
[INFO] --- site-maven-plugin:0.12:site (siteGithubPages) @ data-distribution-api-site ---
[INFO] Creating 160 blobs
```

On my machine, the entire build takes about 9 minutes

## Use the snapshot

If the project version ends in "SNAPSHOT" then the code was deployed to [https://oss.sonatype.org/content/repositories/snapshots][ossSnapshotRepo]

You can see it by directly visiting [the snapshot repository][ossSnapshotRepoProject] or by going to the [repository browser][ossRepositoryBrowser], logging in, and choosing _Snapshots_ from the list at _Repositories_.

You can use it as described in the installation instructions.

## Use the release
If the project version does not end in "SNAPSHOT", then the code is deployed to BOGUS BOGUS
You can see it by BOGUS BOGUS

This is a staging repository. To move to the full repository, you need to close and release the code.
Close it by BOGUS BOGUS
Release it by BOGUS BOGUS

## References
[http://central.sonatype.org/pages/ossrh-guide.html](http://central.sonatype.org/pages/ossrh-guide.html)

[https://www.youtube.com/watch?v=dXR4pJ_zS-0&feature=youtu.be](https://www.youtube.com/watch?v=dXR4pJ_zS-0&feature=youtu.be)

    
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

## Reference
* [Stack Overflow: maven-add-a-dependency-to-a-jar-by-relative-path][stackOverflow1]

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



[corsReference]:          https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
[ossReleaseDoc]:          http://central.sonatype.org/pages/releasing-the-deployment.html
[ossRepositoryBrowser]:   https://oss.sonatype.org/
[ossSnapshotRepo]:        https://oss.sonatype.org/content/repositories/snapshots
[ossSnapshotRepoProject]: https://oss.sonatype.org/content/repositories/snapshots/edu/cornell/library/scholars/
[stackOverflow1]:         https://stackoverflow.com/questions/2229757/maven-add-a-dependency-to-a-jar-by-relative-path/2230464#2230464
