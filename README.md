# MongoLab
The MongoLab is a wrapper class for the [MongoLab](mongolab.com) REST API. MongoLab is a hosted MongoDB as a service, and allows you a variety of options in terms of size, redundancy, etc.

For information about getting started with MongoLab and acquiring your MongoLab API key, see the [REST API for MongoLab](http://docs.mongolab.com/restapi/) documentation.

### Callbacks

All methods in this class that interact with the database take an optional callback parameter. The callback takes three parameters: *err, response, data*. The *err* parameter will be null if the request was successful, or contain an *err* object with a `statuscode` and `body` if an error was encountered during the request. The *resp* object is the [response table](https://electricimp.com/docs/api/httprequest/sendasync/) returned from the request, and *data* will contain the decoded data from the request.

```squirrel
// Asyncronous request (recommended method)
mongo.getDatabases(function(err, resp, databases) {
    if (err != null) {
        server.error(err.body);
        return;
    }

    foreach(database in databases) {
        server.log(database);
    }
});
```

## constructor(apiKey, [db])
The create a new MongoDB object you will need your API key, you can also supply an optional database parameter to connect to. If no database is selected, all methods that require a collection will invoke the callback with an error until a database is selected with **.use()**.

```
db <- MongoLab("<-- API_KEY -->", "<-- DATABASE_NAME -->");
```

## db.use(db)
The `use` method can be used to change the active database (i.e. what database we're running our queries against). This method should not be required in the majority of applications:

```
db.use("someOtherDatabase");
```

## db.getDatabases([callback])
The `getDatabases` method lists all of the databases attached to the account associated with the API key:

```squirrel
db.getDatabases(function(err, resp, databases) {
    if (err != null) {
        server.error(err.body);
        return;
    }

    foreach(database in databases) {
        server.log(database);
    }
});
```

## db.getCollections([callback])
The `getCollection` method lists all collections (tables) in the active database. If no database is selected, this method will invoke the callback with an error.

```squirrel
db.getCollections(function(err, resp, collections) {
    if (err != null) {
        server.error(err.body);
        return;
    }

    foreach(collection in collections) {
      server.log(collection);
    }
});
```

## db.find(collection, query, [callback])
The `find` method invokes a query against the specified collection in the active database. If no database is selected, this method will invoke the callback with an error.

If an empty table is passed to the *query* parameter, find will return all records in the collection. Information about building query strings can be found in [MongoLab's documentation](http://docs.mongodb.org/v2.6/reference/operator/query/).

```squirrel
// Return all users
db.find("users", {}, function(err, resp, users) {
    if (err != null) {
        server.error(err.body);
        return;
    }

    foreach(user in users) {
      server.log(http.jsonencode(user));
    }
});
```

```squirrel
// Return all records in the users collection that
// don't have a verified key (i.e. unverified users)
db.find("users", { "verified": { "$exists": false } }, function(err, resp, users) {
    if (err != null) {
        server.error(err.body);
        return;
    }

    foreach(user in users) {
      server.log(user._id["$oid"] + ": " + user.username);
    }
});
```

## db.insert(collection, record, [callback])
The `insert` method inserts a new record into the specified collection.

```squirrel
device.on("data", function(data) {
    local record = {
        ts = data.ts,
        temp = data.temp
    };

  db.insert("sensor_readings", record, function(err, resp, record) {
      if (err != null) {
          server.log(err.message);
          return;
      }

      server.log("Created record with id: " + record._id["$oid"]);
  });
});
```

## db.update(collection, multi, q, record, [callback])
The `update` method updates records in the specified collection collection matching the specified query. If the *multi* parameter is set to `false`, MongoLab will only update a single record, if the *multi* parameter is set to `true`, MongoLab will update all matching records. The *record* parameter can either be an object or contain an [Update Action](http://docs.mongodb.org/v2.6/reference/method/db.collection.update/#update-method-examples). If the *multi* flag is set to `true`, *record* MUST contain an update action.

```squirrel
// Update a single record:

local query = {
    "_id": { "$oid": "54d505afe4b05f6282260adf" }
};

local newRecord = {
    "username": "newUserName",
    "verified": false
};

db.update("users", false, query, newRecord, function(err, resp, data) {
    if (err != null) {
        server.log(err.message);
        return;
    }

    if (data.n == 0) {
        server.log("No matching records found!");
        return;
    }

    server.log("Success!");
});
```

```squirrel
// Find all records that are missing the verified flag, and add it
local query = {
    "verified": false
};

// Add the verified flag, and set it to false
local updateAction = {
    "$set": { "verified": true }
};

db.update("users", true, query, updateAction, function(err, resp, data) {
    if (err != null) {
        server.log(err.message);
        return;
    }

    if (data.n == 0) {
        server.log("No matching records found!");
        return;
    }

    server.log("Updated " + data.n + " records!");
});
```

## db.remove(collection, id, [callback])
The `remove` method removes the specified record from the specified collection.

```squirrel
// remove record with _id: { $oid: "54d505afe4b05f6282260adf" }
db.remove("users", "54d505afe4b05f6282260adf", function(err, resp, result) {
    if (err != null) {
        server.log(err.message);
        return;
    }

    server.log("Success!");
});
```

# License
This code is licensed under the [MIT License](./LICENSE).
