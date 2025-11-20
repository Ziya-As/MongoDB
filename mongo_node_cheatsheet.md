# Mongo Node Cheatsheet

- [Mongo Node Cheatsheet](#mongo-node-cheatsheet)
  - [Installing, and setting up connection to Mongo DB](#installing-and-setting-up-connection-to-mongo-db)
  - [Ping a database](#ping-a-database)
  - [Listing databases](#listing-databases)
  - [Connecting to a specific database](#connecting-to-a-specific-database)
  - [List of all the collections in a specific db](#list-of-all-the-collections-in-a-specific-db)
  - [Connecting to a specific collection](#connecting-to-a-specific-collection)
  - [Creating a collection](#creating-a-collection)
  - [Cursor and iterating through it](#cursor-and-iterating-through-it)
  - [Inserting documents](#inserting-documents)
  - [Querying a db](#querying-a-db)
    - [`$elemMatch`, `$gt`, `$lt`](#elemmatch-gt-lt)
    - [`$or`](#or)
    - [`$and`, `$nor`](#and-nor)
    - [`$ne`](#ne)
    - [`$in`](#in)
    - [`$all`](#all)
    - [Querying a document in a specific position of an array](#querying-a-document-in-a-specific-position-of-an-array)
    - [`$exists`](#exists)
    - [`$type`](#type)
    - [`$mod`](#mod)
    - [Projections in a query](#projections-in-a-query)
  - [`sort()`, and `limit()` methods](#sort-and-limit-methods)
  - [`skip()` method](#skip-method)
  - [Aggregation](#aggregation)
    - [`$unwind`, `$group`, `$avg`](#unwind-group-avg)
    - [`$out`](#out)
    - [`$count`](#count)
    - [`$addFields`](#addfields)
    - [`$add`](#add)
    - [`$multiply`](#multiply)
    - [`$lookup`](#lookup)
    - [`$facet`](#facet)
  - [Updating documents](#updating-documents)
    - [`updateOne()`](#updateone)
    - [`$push`](#push)
      - [`$push` with `$each`](#push-with-each)
    - [`$pop`](#pop)
    - [`$currentDate`](#currentdate)
    - [`$currentDate` with `timestamp`](#currentdate-with-timestamp)
    - [`$inc`](#inc)
    - [`$rename`](#rename)
    - [`$unset`](#unset)
  - [Deleting documents, collections, and db](#deleting-documents-collections-and-db)
    - [Deleting a document](#deleting-a-document)
    - [Deleting a collection](#deleting-a-collection)
    - [Deleting a db](#deleting-a-db)

<hr>

## Installing, and setting up connection to Mongo DB

Install MongoDB:

```
npm i mongodb
```

Import MongoClient:

```js
const { MongoClient } = require("mongodb");
```

It's better to create a uri variable to hold the connection string, and use that variable when needed, instead of putting connection string directly to places where we'll need it:

```js
// this is to connect to the localhost, default port of 27017
const uri = "mongodb://127.0.0.1:27017";
```

Now that we have our URI, we need create an instance of MongoClient to be able to connect to the database.

```js
const client = new MongoClient(uri);
```

Now we're ready to use MongoClient to connect to our cluster. `client.connect()` will return a promise. We will use the `await` keyword when we call `client.connect()` to indicate that we should block further execution until that operation has completed.

```js
await client.connect();
```

<hr>

It's better to wrap our calls to functions that interact with the database in a `try... catch` statement so that we handle any unexpected errors. We also want to be sure we close the connection to our cluster, so we’ll end our `try... catch` with a `finally` statement.

We can also combine the whole connection operation in an `async` function:

```js
async function main() {
  try {
    await client.connect();

    console.log("Successfully connected");
  } catch (e) {
    console.error(e);
  } finally {
    await client.close();
  }
}

main().catch(console.error);
```

<hr>

## Ping a database

We can send a ping to the database to confirm the successful connection using the `.command({ ping: 1 })` method:

```js
async function main() {
  try {
    await client.connect();

    // Send a ping to confirm a successful connection
    await client.db("admin").command({ ping: 1 });
  } catch (e) {
    console.error(e);
  } finally {
    await client.close();
  }
}

main().catch(console.error);
```

<hr>

## Listing databases

We can list the whole databases in our cluster with the `listDatabases()` method:

```js
async function main() {
  try {
    await client.connect();

    databasesList = await client.db().admin().listDatabases();
    console.log(databasesList);
  } catch (e) {
    console.error(e);
  } finally {
    await client.close();
  }
}

main().catch(console.error);
```

<hr>

## Connecting to a specific database

To connect to a specific database, we use this syntax:

```js
await client.db("<database_name>");
```

<hr>

## List of all the collections in a specific db

To get the list of all the collections in a database, we can use `.listCollections()` method:

```js
async function main() {
  try {
    await client.connect();

    const collections = await client.db("restaurants").listCollections();

    for await (let col of collection) {
      console.log(col);
    }
  } catch (e) {
    console.error(e);
  } finally {
    await client.close();
  }
}

main().catch(console.error);
```

<hr>

## Connecting to a specific collection

To connect to a specific collection, we use this syntax:

```js
await client.db("<database_name>").collection("<collection_name>");
```

<hr>

## Creating a collection

To create a collection, we can use `createCollection()` method:

```js
await client.db("<db>").createCollection("collectionName");
```

<hr>

## Cursor and iterating through it

There are methods that return not the document that we want or a result for our action on a database, but a cursor, which includes a batch of documents. To access the actual documents within cursor batches, we can use several ways.

One way to access documents in a cursor is to use `for await...of`:

```js
const cursor = <collection>.find({});
console.log("async");

for await (const doc of cursor) {
  console.log(doc);
}
```

Another way to check documents in a cursor is to use `hasNext()` method to check if a cursor can provide additional data, and then use the `next()` method to retrieve the next document:

```js
const cursor = <collection>.find({});

while (await cursor.hasNext()) {
  console.log(await cursor.next());
}
```

One more way to retrieve documents from a cursor is to use `toArray()` method:

```js
const cursor = <collection>.find({});
const allValues = await cursor.toArray();
```

To reset a cursor to its initial position in the set of returned documents, use `rewind()`.

```js
await cursor.rewind();
```

<hr>

## Inserting documents

To insert a document, we can use the `insertOne()` method:

```js
async function run() {
  try {
    const database = client.db("restaurants");
    const restaurants = database.collection("restaurants");

    // create a document to insert
    const doc = {
      name: "Some Restaurant Name",
      address: {
        street: "Some Street",
      },
    };

    const result = await restaurants.insertOne(doc);

    console.log(`A document was inserted with the _id: ${result.insertedId}`);
  } finally {
    await client.close();
  }
}

run().catch(console.dir);
```

To insert more than one document, we can use the `insertMany()` method:

```js
async function run() {
  try {
    const database = client.db("restaurants");
    const restaurants = database.collection("restaurants");

    // create an array of documents to insert
    const docs = [
      { name: "Some Restaurant Name 2", address: { street: "Some Street 2" } },
      { name: "Some Restaurant Name 3", address: { street: "Some Street 3" } },
      { name: "Some Restaurant Name 4", address: { street: "Some Street 4" } },
    ];

    // this option prevents additional documents from being inserted if one fails
    const options = { ordered: true };

    const result = await restaurants.insertMany(docs, options);
    console.log(`${result.insertedCount} documents were inserted`);
  } finally {
    await client.close();
  }
}

run().catch(console.dir);
```

<hr>

## Querying a db

To query a database, we can use the `find()` method:

```js
async function run() {
  try {
    const database = client.db("restaurants");
    const restaurants = database.collection("restaurants");

    const cursor = restaurants.find({});

    // print a message if no documents were found
    if ((await restaurants.countDocuments({})) === 0) {
      console.log("No documents found!");
    }

    for await (const doc of cursor) {
      console.dir(doc);
    }
  } finally {
    await client.close();
  }
}
run().catch(console.dir);
```

### `$elemMatch`, `$gt`, `$lt`

We can specify our query to find documents matching our filters:

```js
async function run() {
  try {
    const database = client.db("restaurants");
    const restaurants = database.collection("restaurants");

    // query for restaurants that have a runtime less than 15 minutes
    const query = { grades: { $elemMatch: { score: { $gt: 80, $lt: 100 } } } };

    const cursor = restaurants.find(query);

    // print a message if no documents were found
    if ((await restaurants.countDocuments(query)) === 0) {
      console.log("No documents found!");
    }

    for await (const doc of cursor) {
      console.dir(doc);
    }
  } finally {
    await client.close();
  }
}
run().catch(console.dir);
```

<hr>

### `$or`

Use logical operators, such as `$or`:

```js
const query = {
  borough: "Bronx",
  $or: [{ cuisine: "American " }, { cuisine: "Chinese" }],
};

const cursor = myColl.find(query);

for await (const doc of cursor) {
  console.dir(doc);
}
```

<hr>

### `$and`, `$nor`

Use logical `$and` operator. `$nor` returns documents that doesn't equal to any one of the specified values. It's the opposite of `$or`:

```js
const query = {
  $and: [
    { $or: [{ borough: "Manhattan" }, { borough: "Brooklyn" }] },
    { $nor: [{ cuisine: "American " }, { cuisine: "Chinese" }] },
    { grades: { $elemMatch: { score: { $lt: 5 } } } },
  ],
};

const cursor = myColl.find(query);

for await (const doc of cursor) {
  console.dir(doc);
}
```

<hr>

### `$ne`

`$ne` operator stands for "not equal to":

```js
const query = {
  $and: [
    { cuisine: { $ne: "American " } },
    { "grades.score": { $gt: 70 } },
    { "address.coord": { $lt: -65.754168 } },
  ],
};

const cursor = myColl.find(query);

for await (const doc of cursor) {
  console.dir(doc);
}
```

<hr>

### `$in`

To return documents that matches any value in a provided array, use `$in` operator:

```js
const query = {
  borough: { $in: ["Staten Island", "Queens", "Bronx", "Brooklyn"] },
};

const cursor = myColl.find(query);

for await (const doc of cursor) {
  console.dir(doc);
}
```

<hr>

### `$all`

`$all` operator checks if a document has all the values in a specified array of values:

```js
const query = {
  $and: [
    { borough: { $in: ["Manhattan", "Brooklyn"] } },
    { "grades.score": { $all: [2, 6] } },
    { cuisine: { $ne: "American" } },
  ],
};

const cursor = myColl.find(query);

for await (const doc of cursor) {
  console.dir(doc);
}
```

<hr>

### Querying a document in a specific position of an array

If an embedded document has an array of documents, and we want to query an item on a specific position, we can do it by specifying the position in an array:

```js
const query = {
  "grades.1.date": ISODate("2014-08-11T00:00:00Z"),
  "grades.1.grade": "A",
  "grades.1.score": 9,
};

const cursor = myColl.find(query);

for await (const doc of cursor) {
  console.dir(doc);
}
```

<hr>

### `$exists`

`$exists` operator returns documents if they contain a specified field:

```js
const query = {
  "address.street": { $exists: true },
};

const cursor = myColl.find(query);

for await (const doc of cursor) {
  console.dir(doc);
}
```

<hr>

### `$type`

`$type` operator returns documents if they have the specified type of data:

```js
const query = { "address.coord": { $type: 1 } };

const cursor = myColl.find(query);

for await (const doc of cursor) {
  console.dir(doc);
}
```

<hr>

### `$mod`

`$mod` operator is used to check if a value divided by a specified divisor gives a specified remainder:

```js
const query = {
  "grades.score": { $mod: [7, 0] },
};

const cursor = myColl.find(query);

for await (const doc of cursor) {
  console.dir(doc);
}
```

<hr>

### Projections in a query

We can also provide options object to the `find()` method, which would help us sort and retrieve only the fields that we want using projections.

```js
async function run() {
  try {
    const database = client.db("restaurants");
    const restaurants = database.collection("restaurants");

    // query for restaurants that have a runtime less than 15 minutes
    const query = { grades: { $elemMatch: { score: { $gt: 80, $lt: 100 } } } };

    const options = {
      // sort returned documents in ascending order by name (A->Z)
      sort: { name: 1 },

      // Include only the `name` and `cuisine` fields in each returned document
      projection: { _id: 0, name: 1, cuisine: 1 },
    };

    const cursor = restaurants.find(query, options);

    // print a message if no documents were found
    if ((await restaurants.countDocuments(query)) === 0) {
      console.log("No documents found!");
    }

    for await (const doc of cursor) {
      console.dir(doc);
    }
  } finally {
    await client.close();
  }
}
run().catch(console.dir);
```

<hr>

## `sort()`, and `limit()` methods

Instead of providing sort inside options, we can use `sort()` method. We can also use `limit()` method to retrieve limited number of documents:

```js
// sort in descending (-1) order by name
const sort = { name: -1 };
const limit = 3;

const cursor = myColl.find({}).sort(sort).limit(limit);

for await (const doc of cursor) {
  console.dir(doc);
}
```

## `skip()` method

To skip several documents from retrieved documents, we can use `skip()` method:

```js
// sort in descending (-1) order by name
const sort = { name: -1 };
const limit = 3;
const skip = 3;

const cursor = myColl.find({}).sort(sort).limit(limit).skip(skip);

for await (const doc of cursor) {
  console.dir(doc);
}
```

<hr>

## Aggregation

### `$unwind`, `$group`, `$avg`

`$unwind` aggregation stage deconstructs an array field from the input documents to output a document for each element. We need to provide a field name to it. When you specify the field path, prefix the field name with a dollar sign `$` and enclose in quotes.:

`$group` aggregation stage groups input documents by a specified identifier expression and applies the accumulator expression(s), if specified, to each group. We have to provide a group key with the `_id` field for `$group` to work.

`$avg` is an aggregation pipeline operator. It returns an average of numerical values. Ignores non-numeric values.

```js
const database = client.db("restaurants");
const restaurants = database.collection("restaurants");

const pipeline = [
  { $unwind: "$grades" },
  {
    $group: {
      _id: "$name",
      avgScore: { $avg: "$grades.score" },
    },
  },
];

const aggCursor = await restaurants.aggregate(pipeline);
for await (const doc of aggCursor) {
  console.log(doc);
}
```

### `$out`

`$out` takes the documents returned by the aggregation pipeline and writes them to a specified collection. The `$out` stage must be the last stage in the pipeline. If the collection specified by the `$out` operation already exists, then upon completion of the aggregation, the `$out` stage atomically replaces the existing collection with the new results collection.

```js
const pipeline = [
  { $match: { borough: "Bronx" } },
  { $group: { _id: "$cuisine", count: { $sum: 1 } } },
  { $out: "bronx_cuisine_count" },
];

const aggCursor = await restaurants.aggregate(pipeline);

console.dir(await db.collection("bronx_cuisine_count").find({}).toArray());
```

### `$count`

`$count` passes a document to the next stage that contains a count of the number of documents input to the stage. The `$count` stage takes a string argument that represents the name of the field where the count will be stored.

```js
const pipeline = [
  { $match: { borough: "Bronx" } },
  { $count: "total_restaurants_in_Bronx" },
];

const aggCursor = await restaurants.aggregate(pipeline);

for await (const doc of aggCursor) {
  console.log(doc);
}
```

<hr>

### `$addFields`

`$addFields` appends new fields to existing documents. You can include one or more `$addFields` stages in an aggregation operation.

```js
const pipeline = [{ $addFields: { totalScore: { $sum: "$grades.score" } } }];

const aggCursor = await restaurants.aggregate(pipeline);

for await (const doc of aggCursor) {
  console.log(doc);
}
```

### `$add`

`$add` Adds numbers together or adds numbers and a date. If one of the arguments is a date, `$add` treats the other arguments as milliseconds to add to the date.

```js
const pipeline = [{ $addFields: { fifteen: { $add: [10, 5] } } }];

const aggCursor = await restaurants.aggregate(pipeline);

for await (const doc of aggCursor) {
  console.log(doc);
}
```

### `$multiply`

`$multiply` multiplies numbers together and returns the result. Pass the arguments to `$multiply` in an array.

```js
const pipeline = [
  {
    $addFields: {
      totalRevenue: { $multiply: ["$averagePrice", "$totalCustomers"] },
    },
  },
];

const aggCursor = await restaurants.aggregate(pipeline);

for await (const doc of aggCursor) {
  console.log(doc);
}
```

<hr>

### `$lookup`

`$lookup` performs a left outer join to a collection in the same database to filter in documents from the "joined" collection for processing. The `$lookup` stage adds a new array field to each input document. The new array field contains the matching documents from the "joined" collection. The `$lookup` stage passes these reshaped documents to the next stage.

```js
const pipeline = [
  {
    $lookup: {
      from: "reviews",
      localField: "restaurant_id",
      foreignField: "restaurant_id",
      as: "reviews",
    },
  },
];

const aggCursor = await restaurants.aggregate(pipeline);

for await (const doc of aggCursor) {
  console.log(doc);
}
```

<hr>

### `$facet`

The `$facet` processes multiple aggregation pipelines within a single stage on the same set of input documents. It allows you to compute multiple facets of data in parallel.

```js
const pipeline = [
  {
    $facet: {
      totalScore: [{ $group: { _id: null, total: { $sum: "$grades.score" } } }],
      averageScore: [{ $group: { _id: null, avg: { $avg: "$grades.score" } } }],
    },
  },
];

const aggCursor = await restaurants.aggregate(pipeline);

for await (const doc of aggCursor) {
  console.log(doc);
}
```

<hr>

## Updating documents

### `updateOne()`

Update a document with `updateOne()`:

```js
async function run() {
  try {
    const database = client.db("restaurants");
    const restaurants = database.collection("restaurants");

    // create a filter for a restaurant to update
    const filter = { name: "Some Restaurant" };

    // create a document that sets the borough of the restaurant
    const updateDoc = { $set: { borough: "Brooklyn" } };

    const result = await restaurants.updateOne(filter, updateDoc);

    console.log(
      `${result.matchedCount} document(s) matched the filter, updated ${result.modifiedCount} document(s)`
    );
  } finally {
    await client.close();
  }
}
run().catch(console.dir);
```

### `$push`

The `$push` operator appends a specified value to an array.

```js
const query = { name: "Morris Park Bake Shop" };

const updateDoc = {
  $push: { grades: { date: new Date(), grade: "A", score: 8 } },
};

const restaurants = await client.db("restaurants").collections("restaurants");

const result = await restaurants.updateOne(query, updateDoc);
```

#### `$push` with `$each`

Use `$push` with `$each` modifier to append multiple values to the array field.

```js
const query = { name: "Morris Park Bake Shop" };

const updateDoc = {
  $push: {
    grades: {
      $each: [
        { date: new Date(), grade: "A", score: 8 },
        { date: new Date(), grade: "B", score: 6 },
      ],
    },
  },
};

const restaurants = await client.db("restaurants").collections("restaurants");

const result = await restaurants.updateOne(query, updateDoc);
```

### `$pop`

The `$pop` operator removes the first or last element of an array. Pass `$pop` a value of `-1` to remove the first element of an array and `1` to remove the last element in an array.

```js
const query = { name: "Morris Park Bake Shop" };

const updateDoc = { $pop: { grades: 1 } };

const restaurants = await client.db("restaurants").collections("restaurants");

const result = await restaurants.updateOne(query, updateDoc);
```

### `$currentDate`

The `$currentDate` operator sets the value of a field to the current date, either as a `Date` or a `timestamp`. The default type is `Date`. If the field does not exist, `$currentDate` adds the field to a document.

```js
const query = { name: "Morris Park Bake Shop" };

const updateDoc = { $currentDate: { lastModified: true } };

const result = await restaurants.updateOne(query, updateDoc);
```

### `$currentDate` with `timestamp`

Here is an example with `timestamp`:

```js
const query = { name: "Morris Park Bake Shop" };

const updateDoc = { $currentDate: { lastModified: { $type: "timestamp" } } };

const result = await restaurants.updateOne(query, updateDoc);
```

### `$inc`

The `$inc` operator is used to increment the value of a field by a specified amount. If the field does not exist, `$inc` creates the field and sets the field to the specified value. Use of the `$inc` operator on a field with a `null` value will generate an error.

```js
const query = { name: "Morris Park Bake Shop" };
const updateDoc = { $inc: { score: 1 } };

const result = await restaurants.updateOne(query, updateDoc);
```

### `$rename`

`$rename` is used to rename a field within a document:

```js
const query = { name: "Morris Park Bake Shop" };
const updateDoc = { $rename: { score: "rating" } };

const result = await restaurants.updateOne(query, updateDoc);
```

### `$unset`

`$unset` operator is used to remove a field from a document:

```js
const query = { name: "Morris Park Bake Shop" };
const updateDoc = { $unset: { rating: "" } };

const result = await restaurants.updateOne(query, updateDoc);
```

<hr>

## Deleting documents, collections, and db

### Deleting a document

Delete a document with `deleteOne()`:

```js
async function run() {
  try {
    const database = client.db("restaurants");
    const restaurants = database.collection("restaurants");

    // Query for a restaurant that has title "Some Restaurant 2"
    const query = { name: "Some Restaurant 2" };

    const result = await restaurants.deleteOne(query);

    if (result.deletedCount === 1) {
      console.log("Successfully deleted one document.");
    } else {
      console.log("No documents matched the query. Deleted 0 documents.");
    }
  } finally {
    await client.close();
  }
}
run().catch(console.dir);
```

<hr>

### Deleting a collection

To delete a collection, we can use the `drop()` method:

```js
await client.db("<db>").collection("<coll>").drop();
```

<hr>

### Deleting a db

To delete a database, we can use the `dropDatabase()` method:

```js
await client.db("newDb").dropDatabase();
```
