# SiriDB-Node.js-Add-on
Node.js add-on (C++) for SiriDB

## Installation
```
node-gyp configure && node-gyp build
```

## Quick usage
```javascript
const sdbaddon = require('./build/Release/siridb');

var siridb = new sdbaddon.SiriDBClient(
    "iris", "siri", "dbtest", "localhost", 9000);

siridb.connect((err) => {
    if (err) {
        console.error(`Connection error: ${err}`);
    } else {
        siridb.close();
    }
});
```

### SiriDBClient Constructor
Create a new SiriDB Client. This creates a new client but `connect()` must be used to connect.
```javascript
var siridb = new sdbaddon.SiriDBClient(
    "iris",         // database user
    "siri",         // password
    "dbtest",       // database name
    "localhost",    // server address
    9000            // server port
);
```

### SiriDBClient.connect
Connect to SiriDB. A callback function can be used to check if the connect is successful.
```javascript
siridb.connect((err) => {
    // successful: err is null
    // error: err is a string with an error message
    if (err) {
        console.error("Connection error: ", err);
    }
});
```

### SiriDBClient.query
Query SiriDB. Requires a string containing the query and a callback function to capture the result.
The callback returns two arguments: 
 
 - first argument: A response Object
 - second argument: Number indicating the status. The status is 0 when successful or a negative value in case of an error.
   (see [Error Codes](#error-codes) for the possible status codes)
   
```javascript
siridb.query("select * from /.*series/", (resp, status) => {
    // successful: status is 0 and resp is an Object containing the data
    // error:       status < 0 and resp.error_msg contains a description about the error
    if (status) {
        console.error(`Query error: ${resp.error_msg} (${status})`);
    } else {
        console.log(resp);  // query data
    }
});
```

### Error Codes
When an error has occurred the `error_msg` in the response contains details about the error.
Sometimes its useful to act on a specific error, for example you might want to retry the request in case of `ERR_SERVER` while a
`ERR_INSERT` error indicates something is wrong with the data.

The following error codes can be returned:

- `sdbaddon.ERR_MSG` (-64) *General error code*
- `sdbaddon.ERR_QUERY` (-65) *Most likely a syntax error in the query*
- `sdbaddon.ERR_INSERT` (-66) *Most likely the data is invalid or corrupt*
- `sdbaddon.ERR_SERVER` (-67) *The server could not perform the request, you could try another SiriDB server*
- `sdbaddon.ERR_POOL` (-68) *At least one pool has no online SiriDB server*
- `sdbaddon.ERR_ACCESS` (-69) *The database user has not enough privileges to process the request*,
- `sdbaddon.ERR_RUNTIME` (-70) *Unexpected error has occurred, please check the SiriDB log*
- `sdbaddon.ERR_NOT_AUTHENTICATED` (-71) *The connection is not authenticated*
- `sdbaddon.ERR_CREDENTIALS` (-72) *Credentials are invalid*
- `sdbaddon.ERR_UNKNOWN_DB` (-73) *Trying to authenticate to an unknown database*
- `sdbaddon.ERR_LOADING_DB` (-74) *The database is loading*
