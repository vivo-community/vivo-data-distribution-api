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
 	
# Relationship with Maven Central
Handled through this [JIRA ticket](https://issues.sonatype.org/browse/OSSRH-35717).

# Build, test, deploy

## Build and install locally
Use this when you want to test changes, but don't want to deploy to the OSS respository

```
cd {top-level-directory}
mvn clean install
```

* What does it do?
	* Compiles the code, runs the tests, creates the JARs.
	* Updates the code to the **local** repository

## Test the site pages
Use this when editing the site pages, to see what the site will look like.

```
cd {top-level-directory}/site_builder
mvn site:run
```

* What does it do?
	* Assembles the site pages
	* Serves the pages at [http://localhost:9000](http://localhost:9000).
		* **Note:** the Javadoc pages are not created until you ask for them, 
		  so there may be some delay when you click on the link.
		* To stop serving, press ^c

## Build and deploy

**Before you deploy a release (even a snapshot) for public use, you must have:**

* An account at [Sonatype's JIRA system](https://issues.sonatype.org)
* Authorization to post on behalf of `edu.cornell.library`. 
  The authorizations are maintained on an [open JIRA ticket][sonatypeJiraTicket].
* A published key-pair, for signing the release artifacts. See the **Note** below regarding signatures.

Use this when you are ready to deploy to the OSS repository

```
cd {top-level-directory}
mvn clean site deploy -P release
```

* What does it do?
	* Compile the code, test, create the JARs.
	* Attach Javadoc and source files.
	* Create signature files for the release artifacts.
	* Upload the artifacts and signatures to the repository at Sonatype.
	* Assemble [the site pages][githubPages]
	* Deploy the site pages to GitHub.

**DO NOT run `install` and `deploy` in the same command, or you wind up with double signatures.**

If you are deploying a release (not a snapshot), then you will need to follow 
[these instructions](http://central.sonatype.org/pages/releasing-the-deployment.html) 
to make it publicly accessible.

Note the following sections on __Use the snapshot__ or __Use the release__, as appropriate.

### Note: pause in the build script

**It takes some time to build the site.**

The build script will appear to pause at this message:

```
[INFO] --- site-maven-plugin:0.12:site (siteGithubPages) @ data-distribution-api-site ---
[INFO] Creating 165 blobs
```

On my machine, the entire build takes about 9 minutes

### Note: GPG-signatures

**The Maven script prompts for a passphrase that will be used in signing the artifacts.**

* The key used to sign the artifacts is the default key for the account where the build occurs. 
* You must create a key-pair on your account
* You must publish the public key so Sonatype (and users) can verify the signatures.
* Check out Sonatype's page on [working with PGP signatures][ossSignaturesDoc], 
  and their [accompanying video][videoSignatures].

## Update the site pages
Deploy changes to [the site pages][githubPages] without updating the OSS repository:

```
cd {top-level-directory}/site_builder
mvn site deploy
```

* What does it do?
	* Assemble [the site pages][githubPages]
	* Deploy the site pages to GitHub.

## Change the version numbers
Use this when changing from SNAPSHOT to a release version, or vice versa

```
mvn release:update-versions -DautoVersionSubmodules=true
```

* What does it do?
	* Prompt you for a version number, and set it in the main project and sub-projects.

## Use the snapshot

If the project version ends in "SNAPSHOT" then the code was deployed to [the OSS snapshot repository][ossSnapshotRepo]

You can see it by directly visiting [the snapshot repository][ossSnapshotRepoProject] 
or by going to the [repository browser][ossRepositoryBrowser], logging in, 
and choosing _Snapshots_ from the list at _Repositories_.

You can use it as described in the installation instructions.

Watch the video: [05 - First Deployments - Easy Publishing to Central Repository][videoDeployment]

## Use the release
If the project version does not end in "SNAPSHOT", then the code is deployed to [the OSS staging repository][ossStagingRepo]

You cannot see this by directly visiting a URL, as with the snapshot repository. 
It is only available through the [repository browser][ossRepositoryBrowser]. 
Log in, look on the left for the _Build Promotion_ section, choose _Staging Repositories_, 
and scroll down through the list of repositories to find one that starts with `educornell`...

This is a staging repository. To move to the full repository, you need to close and release the code.
Close it by selecting `Close` from the menu bar (right below the tab header).
Release it by selecting `Release` from the menu bar.
Note that each of these operations takes some time, 
and then Sonatype takes a while to post the artifacts to Maven Central.

You can use it as described in the installation instructions.

[Read about it][ossReleaseDoc] and [Watch the video][videoDeployment].

## References
[http://central.sonatype.org/pages/ossrh-guide.html](http://central.sonatype.org/pages/ossrh-guide.html)

    
# UML Diagrams    
`Modeling.mdj` is a file created using StarUML. It contains the model and diagrams
that were used to create the UML diagrams in the site pages.

# Other Topics
### Putting VIVO 1.8 classes into a local file-based repository 
`docs/notes/file_based_repo.md`
### "Borrowing" source code from VIVO 1.10
`docs/notes/borrowing_source_code.md`




[corsReference]:          https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
[githubPages]:            https://cul-it.github.io/vivo-data-distribution-api/
[ossReleaseDoc]:          http://central.sonatype.org/pages/releasing-the-deployment.html
[ossSignaturesDoc]:       http://central.sonatype.org/pages/working-with-pgp-signatures.html
[ossRepositoryBrowser]:   https://oss.sonatype.org/
[ossSnapshotRepo]:        https://oss.sonatype.org/content/repositories/snapshots
[ossSnapshotRepoProject]: https://oss.sonatype.org/content/repositories/snapshots/edu/cornell/library/scholars/
[ossStagingRepo]:         https://oss.sonatype.org/service/local/staging/deploy/maven2
[sonatypeJiraTicket]:     https://issues.sonatype.org/browse/OSSRH-35717
[videoDeployment]:        https://www.youtube.com/watch?v=dXR4pJ_zS-0&feature=youtu.be
[videoSignatures]:        https://www.youtube.com/watch?v=HeQ70mRSSGE&feature=youtu.be

