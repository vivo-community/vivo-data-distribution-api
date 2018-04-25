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

