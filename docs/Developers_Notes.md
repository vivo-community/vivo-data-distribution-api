# Questions

  * Do I need to make manual updates to the pom.xml after each release
    * This would be to change the commit comments for the artifacts. But is it enough to 
      keep the earlier Maven POM from disappearing?
  
# To do:

* Create a deploy profile. Only require signing if that profile is used.
	* Can we make it so `deploy` is the default target for that profile?

* Create a written process for release, including site.

* Create a written process for starting a new SNAPSHOT.

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

* Compiles the code, runs the tests, creates the JARs.
* Updates the code to the **local** repository

## Test the site pages
Use this when editing the site pages, to see what the site will look like.

```
cd {top-level-directory}/site_builder
mvn site:run
```

* Assembles the site pages
* Serves the pages at [http://localhost:9000](http://localhost:9000).
	* **Note:** the Javadoc pages are not created until you ask for them, so there may be some delay.
	* To stop serving, press ^c

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

# Other Topics
### Putting VIVO 1.8 classes into a local file-based repository 
`docs/notes/file_based_repo.md`
### "Borrowing" source code from VIVO 1.10
`docs/notes/borrowing_source_code.md`




[corsReference]:          https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
[ossReleaseDoc]:          http://central.sonatype.org/pages/releasing-the-deployment.html
[ossRepositoryBrowser]:   https://oss.sonatype.org/
[ossSnapshotRepo]:        https://oss.sonatype.org/content/repositories/snapshots
[ossSnapshotRepoProject]: https://oss.sonatype.org/content/repositories/snapshots/edu/cornell/library/scholars/
