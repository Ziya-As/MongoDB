# Cheatsheet

- [Cheatsheet](#cheatsheet)
  - [Create, use, and show databases](#create-use-and-show-databases)
  - [Create a collection](#create-a-collection)
  - [`mongoimport`](#mongoimport)
  - [Insert a document](#insert-a-document)
  - [Query documents](#query-documents)
    - [`$elemMatch`](#elemmatch)
    - [`$gt`, and `$lt` operators](#gt-and-lt-operators)
    - [`$or`](#or)
    - [`$and`, `$nor`](#and-nor)
    - [`$ne`](#ne)
    - [`$not`](#not)
    - [`$in` and `$nin`](#in-and-nin)
    - [`$all`](#all)
    - [Querying a document in a specific position of an array](#querying-a-document-in-a-specific-position-of-an-array)
    - [`$exists`](#exists)
    - [`$type`](#type)
    - [`$mod`](#mod)
    - [`$regex`](#regex)
  - [`limit()`](#limit)
  - [`skip()`](#skip)
  - [`sort()`](#sort)
  - [Aggregation](#aggregation)
    - [`$unwind`, `$group`, `$avg`](#unwind-group-avg)
    - [`$max`](#max)
    - [`$min`](#min)
    - [`$sum`](#sum)
    - [`$match`](#match)
    - [`$sort`, `$limit`, `$project`](#sort-limit-project)
    - [`$out`](#out)
    - [`$count`](#count)
    - [`$addFields`](#addfields)
    - [`$add`](#add)
    - [`$multiply`](#multiply)
    - [`$lookup`](#lookup)
    - [`$facet`](#facet)
  - [Update documents](#update-documents)
    - [`$push`](#push)
      - [`$push` with `$each`](#push-with-each)
    - [`$pull`](#pull)
    - [`$pop`](#pop)
    - [`$currentDate`](#currentdate)
      - [`$currentDate` with `timestamp`](#currentdate-with-timestamp)
    - [`$inc`](#inc)
    - [`$rename`](#rename)
    - [`$unset`](#unset)
    - [`bulkWrite()`](#bulkwrite)
  - [Delete documents](#delete-documents)
  - [Delete collection and databases](#delete-collection-and-databases)

<hr>

## Create, use, and show databases

To show the databases, we use this command:

```js
show dbs
```

To create or select a database:

```js
use <databaseName>
```

<hr>

## Create a collection

Create a collection:

```js
db.createCollection("collectionName");
```

<hr>

## `mongoimport`

To import a json file, use `mongoimport`. Run `mongoimport` from the system command line, not the mongo shell. Here is an example: `-d` indicates that the word after is the database, and `-c` indicates that the word after it is the collection name:

```js
mongoimport c:\data\books.json -d bookdb -c books
```

<hr>

## Insert a document

Insert a document with `insertOne()`:

```js
db.movies.insertOne({
  title: "Fight Club",
  writer: "Chuck Palahnick",
  year: 1999,
  actors: ["Brad Pitt", "Edward Norton"],
});
```

<hr>

## Query documents

Get all documents:

```js
db.restaurants.find({});
```

Specify only the fields that you'd like to retrieve:

```js
db.restaurants.find({}, { restaurant_id: 1, name: 1, borough: 1, cuisine: 1 });
```

Exclude the `_id` field:

```js
db.restaurants.find(
  {},
  { restaurant_id: 1, name: 1, borough: 1, cuisine: 1, _id: 0 }
);
```

Access an embedded document's field with a dot and inside double quotes:

```js
db.restaurants.find(
  {},
  { restaurant_id: 1, name: 1, borough: 1, "address.zipcode": 1, _id: 0 }
);
```

Filter documents by field values:

```js
db.restaurants.find({ borough: "Bronx" });
```

<hr>

### `$elemMatch`

If we want to find a value only if it's an element of an array, we can use `$elemMatch` operator:

```js
db.restaurants.find({
  grades: { $elemMatch: { score: { $gt: 90 } } },
});
```

<hr>

### `$gt`, and `$lt` operators

Use greater than (`$gt`) and less than (`$lt`) operators:

```js
db.restaurants.find({
  grades: { $elemMatch: { score: { $gt: 80, $lt: 100 } } },
});
```

<hr>

### `$or`

Use logical operators, such as `$or`:

```js
db.restaurants.find({
  borough: "Bronx",
  $or: [{ cuisine: "American " }, { cuisine: "Chinese" }],
});
```

### `$and`, `$nor`

Use logical `$and` operator. `$nor` returns documents that doesn't equal to any one of the specified values. It's the opposite of `$or`:

```js
db.restaurants.find({
  $and: [
    {
      $or: [{ borough: "Manhattan" }, { borough: "Brooklyn" }],
    },
    {
      $nor: [{ cuisine: "American" }, { cuisine: "Chinese" }],
    },
    {
      grades: {
        $elemMatch: { score: { $lt: 5 } },
      },
    },
  ],
});
```

### `$ne`

`$ne` operator stands for "not equal to":

```js
db.restaurants.find({
  $and: [
    { cuisine: { $ne: "American " } },
    { "grades.score": { $gt: 70 } },
    { "address.coord": { $lt: -65.754168 } },
  ],
});
```

### `$not`

`$not` operator inverts the effect of a query expression and returns documents that do not match the query expression:

```js
db.restaurants.find(
  {
    "grades.score": { $not: { $gt: 10 } },
  },
  {
    restaurant_id: 1,
    name: 1,
    borough: 1,
    cuisine: 1,
  }
);
```

<hr>

### `$in` and `$nin`

To return documents that matches any value in a provided array, use `$in` operator:

```js
db.restaurants.find(
  {
    borough: { $in: ["Staten Island", "Queens", "Bronx", "Brooklyn"] },
  },
  {
    restaurant_id: 1,
    name: 1,
    borough: 1,
    cuisine: 1,
  }
);
```

To return documents that doesn't have any value in a provided array, use `$nin` operator:

```js
db.restaurants.find(
  {
    borough: { $nin: ["Staten Island", "Queens", "Bronx", "Brooklyn"] },
  },
  {
    restaurant_id: 1,
    name: 1,
    borough: 1,
    cuisine: 1,
  }
);
```

<hr>

### `$all`

`$all` operator checks if a document has all the values in a specified array of values:

```js
db.restaurants.find({
  $and: [
    { borough: { $in: ["Manhattan", "Brooklyn"] } },
    { "grades.score": { $all: [2, 6] } },
    { cuisine: { $ne: "American" } },
  ],
});
```

<hr>

### Querying a document in a specific position of an array

If an embedded document has an array of documents, and we want to query an item on a specific position, we can do it by specifying the position in an array:

```js
db.restaurants.find(
  {
    "grades.1.date": ISODate("2014-08-11T00:00:00Z"),
    "grades.1.grade": "A",
    "grades.1.score": 9,
  },
  { restaurant_id: 1, name: 1, grades: 1 }
);
```

<hr>

### `$exists`

`$exists` operator returns documents if they contain a specified field:

```js
db.restaurants.find({
  "address.street": { $exists: true },
});
```

<hr>

### `$type`

`$type` operator returns documents if they have the specified type of data:

```js
db.restaurants.find({ "address.coord": { $type: 1 } });
```

or

```js
db.restaurants.find({ "address.coord": { $type: "double" } });
```

| Type                       | Number | Alias                 | Notes                      |
| -------------------------- | ------ | --------------------- | -------------------------- |
| Double                     | 1      | "double"              |                            |
| String                     | 2      | "string"              |                            |
| Object                     | 3      | "object"              |                            |
| Array                      | 4      | "array"               |                            |
| Binary data                | 5      | "binData"             |                            |
| Undefined                  | 6      | "undefined"           | Deprecated.                |
| ObjectId                   | 7      | "objectId"            |                            |
| Boolean                    | 8      | "bool"                |                            |
| Date                       | 9      | "date"                |                            |
| Null                       | 10     | "null"                |                            |
| Regular Expression         | 11     | "regex"               |                            |
| DBPointer                  | 12     | "dbPointer"           | Deprecated.                |
| JavaScript                 | 13     | "javascript"          |                            |
| Symbol                     | 14     | "symbol"              | Deprecated.                |
| JavaScript code with scope | 15     | "javascriptWithScope" | Deprecated in MongoDB 4.4. |
| 32-bit integer             | 16     | "int"                 |                            |
| Timestamp                  | 17     | "timestamp"           |                            |
| 64-bit integer             | 18     | "long"                |                            |
| Decimal128                 | 19     | "decimal"             |                            |
| Min key                    | -1     | "minKey"              |                            |
| Max key                    | 127    | "maxKey"              |                            |

<hr>

### `$mod`

`$mod` operator is used to check if a value divided by a specified divisor gives a specified remainder:

```js
db.restaurants.find(
  {
    "grades.score": { $mod: [7, 0] },
  },
  { restaurant_id: 1, name: 1, grades: 1 }
);
```

### `$regex`

Use `$regex` for a text search:

```js
db.movies.find({ synopsis: { $regex: "Bilbo" } });
```

<hr>

## `limit()`

Limit the output using the `limit()` method:

```js
db.restaurants.find({ borough: "Bronx" }).limit(5);
```

<hr>

## `skip()`

Skip several output documents with `skip()` method:

```js
db.restaurants.find({ borough: "Bronx" }).skip(5).limit(5);
```

<hr>

## `sort()`

Sort the output using the `sort()` method. `-1` is for a descending order:

```js
db.restaurants
  .find({
    cuisine: { $ne: "American" },
    "grades.grade": "A",
    borough: { $ne: "Brooklyn" },
  })
  .sort({ cuisine: -1 });
```

<hr>

## Aggregation

### `$unwind`, `$group`, `$avg`

`$unwind` aggregation stage deconstructs an array field from the input documents to output a document for each element. We need to provide a field name to it. When you specify the field path, prefix the field name with a dollar sign `$` and enclose in quotes.

`$group` aggregation stage groups input documents by a specified identifier expression and applies the accumulator expression(s), if specified, to each group. We have to provide a group key with the `_id` field for `$group` to work.

`$avg` is an aggregation pipeline operator. It returns an average of numerical values. Ignores non-numeric values.

```js
db.restaurants.aggregate([
  { $unwind: "$grades" },
  {
    $group: {
      _id: "$name",
      avgScore: { $avg: "$grades.score" },
    },
  },
]);
```

We can group based on more than one field:

```js
db.restaurants.aggregate([
  {
    $group: {
      _id: {
        cuisine: "$cuisine",
        borough: "$borough",
      },
      count: {
        $sum: 1,
      },
    },
  },
]);
```

### `$max`

`$max` is an aggregation pipeline operator. It returns the maximum of numerical values. The `$max` operator only considers the non-null and the non-missing values for the field.

```js
db.restaurants.aggregate([
  { $unwind: "$grades" },
  {
    $group: {
      _id: "$name",
      highest_score: { $max: "$grades.score" },
    },
  },
]);
```

### `$min`

`$min` is an aggregation pipeline operator. It returns the minimum of numerical values. The `$min` operator only considers the non-null and the non-missing values for the field.

```js
db.restaurants.aggregate([
  { $unwind: "$grades" },
  {
    $group: {
      _id: "$name",
      lowest_score: { $min: "$grades.score" },
    },
  },
]);
```

### `$sum`

`$sum` calculates and returns the collective sum of numeric values. `$sum` ignores non-numeric values.

```js
db.restaurants.aggregate([
  {
    $group: {
      _id: "$borough",
      count: {
        $sum: 1,
      },
    },
  },
]);
```

### `$match`

`$match` filters the document stream to allow only matching documents to pass unmodified into the next pipeline stage.

Remember that the order of your stages matters. Each stage only acts upon the documents that previous stages provide.

```js
db.restaurants.aggregate([
  {
    $unwind: "$grades",
  },
  {
    $match: { "grades.grade": "A" },
  },
  {
    $group: {
      _id: "$cuisine",
      count: { $sum: 1 },
    },
  },
]);
```

### `$sort`, `$limit`, `$project`

There are also `$sort`, `$limit`, and `$project` aggregation stages.

```js
db.restaurants.aggregate([
  { $unwind: "$grades" },
  { $sort: { "grades.date": -1 } },
  { $limit: 1 },
  { $project: { name: 1, "grades.date": 1, _id: 0 } },
]);
```

### `$out`

`$out` takes the documents returned by the aggregation pipeline and writes them to a specified collection. The `$out` stage must be the last stage in the pipeline. If the collection specified by the `$out` operation already exists, then upon completion of the aggregation, the `$out` stage atomically replaces the existing collection with the new results collection.

```js
db.restaurants.aggregate([
  { $match: { borough: "Bronx" } },
  { $group: { _id: "$cuisine", count: { $sum: 1 } } },
  { $out: "bronx_cuisine_count" },
]);
```

### `$count`

`$count` passes a document to the next stage that contains a count of the number of documents input to the stage. The `$count` stage takes a string argument that represents the name of the field where the count will be stored.

```js
db.restaurants.aggregate([
  { $match: { borough: "Bronx" } },
  { $count: "total_restaurants_in_Bronx" },
]);
```

### `$addFields`

`$addFields` appends new fields to existing documents. You can include one or more `$addFields` stages in an aggregation operation. It doesn't modify the original documents.

```js
db.restaurants.aggregate([
  { $addFields: { totalScore: { $sum: "$grades.score" } } },
]);
```

### `$add`

`$add` Adds numbers together or adds numbers and a date. If one of the arguments is a date, `$add` treats the other arguments as milliseconds to add to the date.

```js
db.restaurants.aggregate([
  { $addFields: { totalScore: { $add: ["$grades.score", 5] } } },
]);
```

### `$multiply`

`$multiply` multiplies numbers together and returns the result. Pass the arguments to `$multiply` in an array.

```js
db.restaurants.aggregate([
  {
    $addFields: {
      totalRevenue: { $multiply: ["$averagePrice", "$totalCustomers"] },
    },
  },
]);
```

<hr>

### `$lookup`

`$lookup` Performs a left outer join to a collection in the same database to filter in documents from the "joined" collection for processing. The `$lookup` stage adds a new array field to each input document. The new array field contains the matching documents from the "joined" collection. The `$lookup` stage passes these reshaped documents to the next stage.

```js
db.restaurants.aggregate([
  {
    $lookup: {
      from: "reviews",
      localField: "restaurant_id",
      foreignField: "restaurant_id",
      as: "reviews",
    },
  },
]);
```

<hr>

### `$facet`

`$facet` processes multiple aggregation pipelines within a single stage on the same set of input documents. It allows you to compute multiple facets of data in parallel.

```js
db.restaurants.aggregate([
  {
    $facet: {
      totalScore: [{ $group: { _id: null, total: { $sum: "$grades.score" } } }],
      averageRating: [{ $group: { _id: null, avg: { $avg: "$rating" } } }],
    },
  },
]);
```

<hr>

## Update documents

Update a document with `updateOne()`:

```js
db.movies.updateOne(
  { title: "The Hobbit: An unexpected Journey" },
  { $set: { synopsis: "The reluctant hobbit..." } }
);
```

To update all the matching documents, use `updateMany()`.

### `$push`

The `$push` operator appends a specified value to an array.

```js
db.restaurants.updateOne(
  { name: "Morris Park Bake Shop" },
  { $push: { grades: { date: new Date(), grade: "A", score: 8 } } }
);
```

#### `$push` with `$each`

Use `$push` with `$each` modifier to append multiple values to the array field.

```js
db.restaurants.updateOne(
  { name: "Morris Park Bake Shop" },
  {
    $push: {
      grades: {
        $each: [
          { date: new Date(), grade: "A", score: 8 },
          { date: new Date(), grade: "B", score: 6 },
        ],
      },
    },
  }
);
```

### `$pull`

The `$pull` operator removes from an existing array all instances of a value or values that match a specified condition.

```js
db.restaurants.updateOne(
  { name: "Morris Park Bake Shop" },
  { $pull: { grades: { grade: "A", score: 8 } } }
);
```

### `$pop`

The `$pop` operator removes the first or last element of an array. Pass `$pop` a value of `-1` to remove the first element of an array and `1` to remove the last element in an array.

```js
db.restaurants.updateOne(
  { name: "Morris Park Bake Shop" },
  { $pop: { grades: 1 } }
);
```

### `$currentDate`

The `$currentDate` operator sets the value of a field to the current date, either as a `Date` or a `timestamp`. The default type is `Date`. If the field does not exist, `$currentDate` adds the field to a document.

```js
db.restaurants.updateOne(
  { name: "Morris Park Bake Shop" },
  { $currentDate: { lastModified: true } }
);
```

#### `$currentDate` with `timestamp`

Here is an example with `timestamp`:

```js
db.restaurants.updateOne(
  { name: "Morris Park Bake Shop" },
  { $currentDate: { lastModified: { $type: "timestamp" } } }
);
```

### `$inc`

The `$inc` operator is used to increment the value of a field by a specified amount. If the field does not exist, `$inc` creates the field and sets the field to the specified value. Use of the `$inc` operator on a field with a null value will generate an error.

```js
db.restaurants.updateOne(
  { name: "Morris Park Bake Shop" },
  { $inc: { score: 1 } }
);
```

### `$rename`

`$rename` is used to rename a field within a document:

```js
db.restaurants.updateOne(
  { name: "Morris Park Bake Shop" },
  { $rename: { score: "rating" } }
);
```

### `$unset`

`$unset` operator is used to remove a field from a document:

```js
db.restaurants.updateOne(
  { name: "Morris Park Bake Shop" },
  { $unset: { rating: "" } }
);
```

### `bulkWrite()`

The `db.collection.bulkWrite()` method executes multiple write operations listed in an array.

<hr>

## Delete documents

Delete a document with `deleteOne()`:

```js
db.movies.deleteOne({ title: "Avatar" });
```

To delete all the matching documents, use `deleteMany()`.

<hr>

## Delete collection and databases

`<db>.<collection>.drop()` Removes a collection or view from the database. The method also removes any indexes associated with the dropped collection.

<hr>

To delete a database first run `use <db>`. After you are on the db that you want to delete, then run `db.dropDatabase()`. This removes the current database, deleting the associated data files.
