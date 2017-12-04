# Cross-site distributions

## Simple requests

It is possible to receive distributed data by entering the appropriate URL into your 
browser's address bar. This is like the example used in the installation process, 
to prove that the installation was successful:

```
    http://localhost:8080/vivo/api/dataRequest/hello?name=World
```

This will also work with Unix tools like `curl` and `wget`.

## JavaScript requests

When a browser uses a JavaScript call to request data from a different origin, the 
call will not complete successfully unless the response complies with the 
[CORS Standard](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).

Specifically, if a request is made to a Data Distribution API at a different origin 
than the requesting script, the response must contain an appropriate `Access-Control-Allow-Origin` 
header, or the browser will reject the response.

### Ad hoc solution

For now, the Data Distribution API appends a CORS-compliant header to all responses.

```
    Access-Control-Allow-Origin: *
```

In future releases, we may add the ability to configure CORS restrictions. In that 
case, the default action may be to omit the CORS headers, unless enabled by the configuration.
