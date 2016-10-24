# Probo JWT

[Probo.ci](http://probo.ci) is an open source continuous integration and quality assurance tool. Probo.ci is built using microservices and we are leveraging JWT's to allow service to defer authorization in a coordinated, consistent, and standards compliant way.

### Schema

Needs to capture:

  - The user's Probo ID
  - A human readable representation of the user's id

``` json
{
  "key": "coordinator-1",
  "context": {
    "actingUser": {
      "id": "d0229483-18c5-4182-99ef-c38577f73961",
      "slug": "github:tizzo",
      "admin": true,
    }
  },
  "permissions": {
    "assetReceiver": {
      "backup": true,
      "read": {
        "buckets": [
          "3280f824-991d-4fb3-a48a-aed430a444f2"
        ],
      }
    },
    "loom": {
      "write": [
        "stream-build-3280f824-991d-4fb3-a48a-aed430a444f2"
      ]
    },
  }
}
```

``` javascript
app.get('/api/streams/:streamId', jwtHandler.createMiddleware('permissions.loom.write', 'req.params.streamId'), function(req,res) { /* do stuff. */});
```

``` javascript
var jwtHandler = require('probo-jwt');
var app = require('express')();
jwtHandler.configure({
  keys: {
    "coordinator-1": fs.readFileSync('/some/path/to/coordinator-1.pub'),
  },
});
app.use(jwtHandler.createMiddleware('permissions.assetReceiver.backup'));

// The above is short for:
app.use(function(req, res, next) {
  var token = parseToken(req.headers.bearer);
  var claims = jwt.verify(token)
  if (claims.context.actingUser.admin) {
    return next();
  }
  if (!claims.permissions || !claims.permissions.assetReceiver || !claims.permissions.assetReceiver.backup) {
    res.writeHead(403);
    res.end('Access denied');
    return;
  }
  next();
});
```
