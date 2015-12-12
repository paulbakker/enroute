# enroute
restify route specification via JSON

# Usage Example
```javascript
enroute.createRoute({
            //restify server object
            server: server,
            //bunyan log object
            log: log,
            //array of route config object, the object includes route configuration file path
            //and an optional baseUrl String for example v0.0.1/tvui/test, it will be prepend
            //for every route path specified in route configuration file
            routeConf: [{filePath : './test/fixture/es/route.json',
                        baseUrl :
                        helper.getBaseUrl('./test/fixture/es/container.json')}],
            scriptPath : appRoot + '/test/fixture/es'
        }, function() {
            client.get('/0.0.1/website/test/gettest',
                function(err, req, res, obj) {
                if (err) {
                    done(err);
                }
                assert.equal(obj.data, 'Hello world!',
                          'Hello World endpoint response.');
                done();
            });
        });
    });
```

# Route Configuration file Example
```json
{
    "routes": {
        "acceptcookies": {
            "put": {
                "source": "./routes/putAcceptCookies.js"
            },
            "get": {
                "source": "./routes/getAcceptCookies.js"
            }
        },
        "accountInfo": {
            "get": {
                "source": "./routes/getAccountInfo.js"
            }
        },
        "cancelPlan": {
            "post": {
                "source": "./routes/postCancelPlan.js"
            }
        },
        "accountInfo/old": {
            "get": {
                "source": "./routes/getAccountInfoOld.js"
            }
        }
    }
}
```
Second level keys "acceptcookies", "accountInfo", "cancelPlan", "accountInfo/old" is url path.
Third level keys are HTTP method names.
The value of "source" is the route handler script's relative path from app root.

## Route Handle Script Definition
```javascript
'use strict';
// Node core modules
var url = require('url');


// Third party modules specified in package.json
var _ = require('lodash');
var falcor = require('falcor');
var verror = require('verror');

function accountInfo(req, res, next) {
    var queryparams = req.queryParams;
    // bunyan logger
    var log = req.log;
    // metrics client;
    var metrics = req.metrics;
    // api client
    var apiClient = req.apiClient;

    // Make some api requests

    // send back response
    res.send(200);

    return next();
}

module.exports = accountInfo;
```
Route definitions must export as single function or an array of functions where the signature conforms to f(req, res, next). This is identical to writing a restify handler function. The req object contains request state, the api client, logging and metric clients that can be leveraged by the user land code in the endpoint script. The res object is used to send back the response to the client, and next() must be invoked when the script has completed.

Instead of a single function, route definitions can export an array of chained function handlers; increasing the modularity of their endpoint scripts.

```javascript
'use strict';
// Node core modules
var url = require('url');


// Third party modules specified in metadata dependencies/floatDependencies fields
var _ = require('lodash');
var falcor = require('falcor');
var verror = require('verror');

var postHandler = require('somePostHandler');


function somePreHandler(req, res, next) {
    return next();
}

function accountInfo(req, res, next) {
    var queryparams = req.queryParams;
    // bunyan logger
    var log = req.log;
    // metrics client;
    var metrics = req.metrics;
    // api client
    var apiClient = req.apiClient;

    // Make some api requests

    // send back response
    res.send(200);

    return next();
}

module.exports = [somePreHandler, accountInfo, postHandler];
```

Notice that the functions can be both defined within the endpoint script itself, but can also be a module. Critically, every function in the chain must invoke ```next()```, otherwise the request will hang.

## Functions

<dl>
<dt><a href="#readFile">readFile(fpath, log, errMsg)</a> ⇒ <code>String</code></dt>
<dd><p>read file synchronously, throw Verror
on error, log the error message</p>
</dd>
<dt><a href="#installConductor">installConductor(options, cb)</a> ⇒ <code>undefined</code></dt>
<dd><p>take the handlers from a route config file and setup routes into
restify server</p>
</dd>
</dl>

<a name="readFile"></a>
## readFile(fpath, log, errMsg) ⇒ <code>String</code>
read file synchronously, throw Verror
on error, log the error message

**Kind**: global function
**Returns**: <code>String</code> - Content of the file
**Access:** public

| Param | Type | Description |
| --- | --- | --- |
| fpath | <code>String</code> | file path |
| log | <code>Object</code> | log object |
| errMsg | <code>String</code> | Custom error message |

<a name="installConductor"></a>
## installConductor(options, cb) ⇒ <code>undefined</code>
take the handlers from a route config file and setup routes into
restify server

**Kind**: global function
**Access:** public

| Param | Type | Description |
| --- | --- | --- |
| options | <code>Object</code> | required options.server, restify server object, required options.log bunyan log object, optional, options.preMiddleware a list of middleware called before route handler, optional, options.postMiddleware a list of middleware called after route handler, optional, options.routeConf, array of object contains route configuration file relative path to app root and optional base url path for example, one set of route configuration [{filePath : ./etc/routes/route.json, baseUrl : 'v1/myaccount/'} |
| cb | <code>function</code> | optional callback function |


