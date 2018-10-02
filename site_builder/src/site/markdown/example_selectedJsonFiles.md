# Example: serve a JSON file for each department

The `SelectingFileDistributor` serves the contents of a file. The file is found by combining
a parameter on the HTTP request and a template in the configuration. 

The `SelectingFileDistributor` will attempt to...

* Match a regular expression to the value of an HTTP parameter. 
* Substitute that match into the filepath template.
* Treat the path as absolute or relative to __[vitro-home]__.
* Serve the contents of the file.

If the expression does not match the value of the parameter, or if the file does not exist, 
the `SelectingFileDistributor` serves an "empty" response.


## The configuration

Here is an example:

```
:sunburst_inter_dd
    a   <java:edu.cornell.library.scholars.webapp.controller.api.distribute.DataDistributor> ,
        <java:edu.cornell.library.scholars.webapp.controller.api.distribute.file.SelectingFileDistributor> ;
    :actionName "interdepartmental_sunburst" ;
    :parameterName "department" ;                              # ?department=[department URI]
    :parameterPattern "[^/#]+$" ;                              # matches the localname of the department URI.
    :filepathTemplate "visualizationData/interdept-\\0.json" ; # substitute the localname into this path.
    :emptyResponse "{}" ;                                      # if file not found, return this.
    :contentType "application/json" .
```

* The selected HTTP parameter is `department`, as specified by `parameterName`.
* The `department` parameter will contain the URI of the selected academic department.
* The regular expression specified in `parameterPattern` will match just the localname of the URI.
* The localname is substituted into the `filepathTemplate`, 
  according to the usual rules for substituting with regular expressions. 
* The contents of the file are served, with a `contentType` of `application/json`.
* If there is no match, or if the file does not exist, the `emptyResponse` will be served instead.

## A specific case

The HTTP request specifies the URI of an academic department, like this 
(_line break inserted for readability_):

```
http://scholars.cornell.edu/api/dataRequest/interdepartmental_sunburst?
   department=http://scholars.cornell.edu/individual/org73341
```

The regular expression matches this string:

```
org73341
```

Applying this to the template yields this filepath:

```
visualizationData/interdept-org73341.json
```

Since this is a relative path, it is treated as

```
[vitro-home]/visualizationData/interdept-org73341.json
```

The contents of the file are served.