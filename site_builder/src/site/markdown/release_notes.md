# Release Notes

## version 1.1.1

* Use servlet annotations to simplify installation. No need to modify web.xml.

* Fix bug: translation from bytes to chars must always be UTF-8.

## version 1.1

* Unify the projects for VIVO 1.8, 1.9 and 1.10 into one project with multiple artifacts.
	* Change the names of the artifacts:
		* `data-distribution-api-parent`
		* `data-distribution-api-vivo_1_08`
		* `data-distribution-api-vivo_1_09`
		* `data-distribution-api-vivo_1_10`

* Move the source code to a new repository at 
[https://github.com/cul-it/vivo-data-distribution-api](https://github.com/cul-it/vivo-data-distribution-api).

* Make the artifacts available through Maven Central.
	* Browse the repository at [https://mvnrepository.com/artifact/edu.cornell.library.scholars](https://mvnrepository.com/artifact/edu.cornell.library.scholars).

* Improve the documentation site.
	* [https://cul-it.github.io/vivo-data-distribution-api/](https://cul-it.github.io/vivo-data-distribution-api/)