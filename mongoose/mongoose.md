# Mongoose

- [Mongoose](#mongoose)
  - [Some useful links](#some-useful-links)
  - [Installation](#installation)
  - [Connect to database](#connect-to-database)
  - [Schema](#schema)
  - [Model](#model)
  - [The `save` method](#the-save-method)
  - [The `create` method](#the-create-method)
  - [Adding more Schema types to our project](#adding-more-schema-types-to-our-project)
  - [Defining more specificities for our fields](#defining-more-specificities-for-our-fields)
    - [Making a field `required`, `lowercase`, or `uppercase`](#making-a-field-required-lowercase-or-uppercase)
    - [Specifying a `default` value for a field](#specifying-a-default-value-for-a-field)
    - [Making a field `immutable`](#making-a-field-immutable)
    - [Setting `min` and `max` values for `Number` type](#setting-min-and-max-values-for-number-type)
    - [Setting `minLength` and `maxLength` values for `String` type](#setting-minlength-and-maxlength-values-for-string-type)
    - [Specifying custom validation](#specifying-custom-validation)
  - [The issue with validation](#the-issue-with-validation)
  - [Querying with mongoose](#querying-with-mongoose)
    - [`findById`](#findbyid)
    - [`find`](#find)
    - [`findOne`](#findone)
    - [`exists`](#exists)
    - [`deleteOne`](#deleteone)
    - [`where`](#where)
    - [`limit`](#limit)
    - [`select`](#select)
    - [`ref` and `populate`](#ref-and-populate)
  - [Schema methods](#schema-methods)
    - [Methods on each document instance in a model](#methods-on-each-document-instance-in-a-model)
    - [Static methods](#static-methods)
    - [Adding methods to a query](#adding-methods-to-a-query)
  - [Virtuals](#virtuals)
  - [Middleware](#middleware)

## Some useful links

- https://www.youtube.com/watch?v=DZBGEVgL2eE
- [Getting Started with MongoDB & Mongoose](https://www.mongodb.com/developer/languages/javascript/getting-started-with-mongodb-and-mongoose/)
- [Review the docs page on how to get started with Mongoose.](https://mongoosejs.com/docs/index.html)

<hr>

## Installation

Mongoose is a wrapper around MongoDB. All the queries, and commands available in MongoDB are also available in Mongoose.

To install mongoose, use the below code:

```
npm i mongoose
```

## Connect to database

To connect to the database, we need to use the `connect` method.

To handle initial connection errors, you should use `.catch()` or `try`/`catch` with `async`/`await`.

```js
// index.js
const mongoose = require("mongoose");

mongoose
  .connect("mongodb://localhost/nameOfDb")
  .then(() => console.log("connected"))
  .catch((err) => console.error(err));

// use `await mongoose.connect('mongodb://user:password@127.0.0.1:27017/nameOfDb');` if your database has auth enabled
```

or :

```js
// index.js
const mongoose = require("mongoose");

try {
  mongoose.connect("mongodb://localhost/nameOfDb");
  console.log("connected");
} catch (err) {
  console.error(err);
}
```

By default, localhost is "127.0.0.1:27017" in mongoDB. So, this also works, and this is preferable:

```js
// index.js
mongoose
  .connect("mongodb://127.0.0.1:27017/nameOfDb")
  .then(() => console.log("connected"))
  .catch((err) => console.error(err));
```

**nameOfDb**, in the above code, is the name of the database.

## Schema

Everything in Mongoose starts with a `Schema`. Each schema maps to a MongoDB collection and defines the shape of the documents within that collection.

To define a schema, we use the `new Schema` syntax. `Schema` accepts an object as an argument, and inside the object we define what properties and types of data a document should have.

The permitted SchemaTypes are:

- `String`
- `Number`
- `Date`
- `Buffer`
- `Boolean`
- `Mixed`
- `ObjectId`
- `Array`
- `Decimal128`
- `Map`
- `UUID`

Here is an example:

```js
const someSchema = new Schema({
  title: String, // String is shorthand for {type: String}
  author: String,
  body: String,
  comments: [{ body: String, date: Date }],
  date: { type: Date, default: Date.now },
  hidden: Boolean,
  meta: {
    votes: Number,
    favs: Number,
  },
});
```

Let's continue our project and learn setting the schema.

We create a new file named "User.js" and set our schema there:

```js
// User.js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
});
```

## Model

Models take your schema and apply it to each document in its collection. Therefore, specifying a schema is not enough. We should also use the `model` method. The `model` method takes 2 arguments:

- The first is a string which will become the name of a collection in a db.
  - the first argument passed to the model should be the singular form of your collection name. Mongoose automatically changes this to the plural form, transforms it to lowercase, and uses that for the database collection name.
- The second is the schema that we created.

Now, let's set our model in the "User.js" file:

```js
// User.js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
});

module.exports = mongoose.model("User", userSchema);
```

We gave `"User"` as the first argument of the `model` method. Our collection in mongoDB, will therefore be named `users` (with **s** added to the end of the string that we gave to the `model`).

## The `save` method

In Mongoose, a "document" generally means an instance of a model. So, if we want to save some data, in other words save a document in a db, then we can create a new instance of a model. When we want to save the document in a db, we use the `save` method.

Here is an example

- We import the model from "User.js",
- Connect to db,
- Create a new instance of the model, and
- Finally, save it in the db using the `save` method. The `save` method should be `await`ed.

```js
// index.js
const mongoose = require("mongoose");
const User = require("./User");

try {
  mongoose.connect("mongodb://127.0.0.1:27017/nameOfDb");
  console.log("connected");
} catch (err) {
  console.error(err);
}

async function run() {
  const user = new User({ name: "Kyle", age: 26 });
  await user.save();
  console.log(user);
}

run();
```

## The `create` method

Instead of initiating a new instance of a model, and then saving it using the `save` method, we can use the `create` method on the model. The `create` method creates a new document and saves it. This method also should be `await`ed.

```js
// index.js
// ...

async function run() {
  const user = await User.create({ name: "test", age: 26 });
  console.log(user);
}

run();
```

The `create` method also gives a handy way to update data. For example, here we add a document with the name "test", but right away, we update the name to "testing" and save the data. First, the document with the name "test" is create and then it gets updated to "testing".

```js
// index.js
// ...

async function run() {
  const user = await User.create({ name: "test2", age: 26 });
  user.name = "testing";
  await user.save();
  console.log(user);
}

run();
```

## Adding more Schema types to our project

Above we showed that there are many schema types that could be specified in mongoose. However, we didn't include them to our project. Let's now use more schema types for practice:

```js
// User.js
const mongoose = require("mongoose");

const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
  email: String,
  createdAt: Date,
  updatedAt: Date,
  bestFriend: mongoose.SchemaTypes.ObjectId,
  hobbies: [String],
  address: {
    street: String,
    city: String,
  },
});

module.exports = mongoose.model("User", userSchema);
```

- Pay attention to the `bestFriend` field. We specified that the value of that field should be the `ObjectId` of another item in our db. We'll come to that later.
- Also, note how we specified that `hobbies` should be the array of strings, and the `address` field should be an object.

There is another way to specify the `address` field. Here is an example:

```js
// User.js
const mongoose = require("mongoose");

const addressSchema = new mongoose.Schema({
  street: String,
  city: String,
});

const userSchema = new mongoose.Schema({
  name: String,
  age: Number,
  email: String,
  createdAt: Date,
  updatedAt: Date,
  bestFriend: mongoose.SchemaTypes.ObjectId,
  hobbies: [String],
  address: addressSchema,
});

module.exports = mongoose.model("User", userSchema);
```

The last example is almost the same as the previous one. The main differences are

- for the `address` field, we specified its own Schema, and
- when we add a document, mongoose will automatically add an `ObjectId` inside the object for the `address` field.

Although we specified what type of data a specific field should have in our Schema, this doesn't mean that these fields are required. For example, we can still run the below code, and save data to the db, even if there are various fields that this document lacks:

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.create({
      name: "test3",
      age: 26,
      hobbies: ["Weight Lifting", "Bowling"],
      address: {
        street: "Main St",
      },
    });
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

## Defining more specificities for our fields

### Making a field `required`, `lowercase`, or `uppercase`

What if we want to make a certain field required. The above way of defining what type of data a field should be is a shortcut. We can provide an object to our field, and specify a few more things for that field, instead of using the short way.

Here is an example:

- We make the `email` a `required` field,
- We also make sure to specify that it should always be saved into the database in `lowercase` (this doesn't mean that the user can't write the email in caps. We just specify that the email is saved to the db in lowercase).
- We also make the `name` field, `uppercase`, just for practice.

```js
// User.js
// ...

const userSchema = new mongoose.Schema({
  name: { type: String, uppercase: true },
  age: Number,
  email: {
    type: String,
    required: true,
    lowercase: true,
  },
  createdAt: Date,
  updatedAt: Date,
  bestFriend: mongoose.SchemaTypes.ObjectId,
  hobbies: [String],
  address: addressSchema,
});

module.exports = mongoose.model("User", userSchema);
```

Now, we have to specify the `email` to be able to save our documents in the user model (users collection). To access the error, if this validation fails, we can use `error.message`:

```js
// index.js
// ..

async function run() {
  try {
    const user = await User.create({
      name: "test4",
      age: 26,
      email: "test4@test.com",
      hobbies: ["Weight Lifting", "Bowling"],
      address: { street: "Main St" },
    });

    console.log(user);
  } catch (error) {
    console.error(error.message);
  }
}

run();
```

### Specifying a `default` value for a field

We can also specify a default value for our fields. Here we specify a `default` date for the `createdAt` property

```js
// User.js
// ...

const userSchema = new mongoose.Schema({
  name: { type: String, uppercase: true },
  age: Number,
  email: {
    type: String,
    required: true,
    lowercase: true,
  },
  createdAt: { type: Date, default: new Date() },
  updatedAt: Date,
  bestFriend: mongoose.SchemaTypes.ObjectId,
  hobbies: [String],
  address: addressSchema,
});

module.exports = mongoose.model("User", userSchema);
```

The above way of specifying a default date is not useful. Because mongoose will run the `new Date()` once and that date will become the default value for all `createdAt` fields. So, instead of using `new Date()`, it's better to provide a function that will run every time a new document is created. Here is an example:

```js
// User.js
// ...

const userSchema = new mongoose.Schema({
  name: { type: String, uppercase: true },
  age: Number,
  email: {
    type: String,
    required: true,
    lowercase: true,
  },
  createdAt: { type: Date, default: () => Date.now() },
  updatedAt: Date,
  bestFriend: mongoose.SchemaTypes.ObjectId,
  hobbies: [String],
  address: addressSchema,
});

module.exports = mongoose.model("User", userSchema);
```

### Making a field `immutable`

We can make sure that the value of a field won't be changed, after it's set once. We can use the `immutable` property for that. Here we make sure that the `createdAt` is going to be `immutable`:

```js
// User.js
// ...

const userSchema = new mongoose.Schema({
  name: { type: String, uppercase: true },
  age: Number,
  email: {
    type: String,
    required: true,
    lowercase: true,
  },
  createdAt: { type: Date, default: () => Date.now(), immutable: true },
  updatedAt: { type: Date, default: () => Date.now() },
  bestFriend: mongoose.SchemaTypes.ObjectId,
  hobbies: [String],
  address: addressSchema,
});

module.exports = mongoose.model("User", userSchema);
```

Now, even if we deliberately try to change the `createdAt` property, it won't change after being set the first time.

```js
// ...
async function run() {
  try {
    const user = await User.create({
      name: "test4",
      age: 26,
      email: "TEST3@test.com",
      hobbies: ["Weight Lifting", "Bowling"],
      address: {
        street: "Main St",
      },
    });

    // Trying to change the `createdAt` doesn't work as it's immutable
    user.createdAt = 5;
    await user.save();

    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

### Setting `min` and `max` values for `Number` type

We can specify minimum and maximum values for a field that has the Schema type of `Number`.

```js
// User.js
// ...

const userSchema = new mongoose.Schema({
  name: { type: String, uppercase: true },
  age: { type: Number, min: 1, max: 200 },
  email: {
    type: String,
    required: true,
    lowercase: true,
  },
  createdAt: { type: Date, default: () => Date.now(), immutable: true },
  updatedAt: { type: Date, default: () => Date.now() },
  bestFriend: mongoose.SchemaTypes.ObjectId,
  hobbies: [String],
  address: addressSchema,
});

module.exports = mongoose.model("User", userSchema);
```

Now, if we set the age 201, mongoose will throw an error.

### Setting `minLength` and `maxLength` values for `String` type

We can specify minimum and maximum length for a property that has the Schema type of `String`.

```js
// User.js
// ...

const userSchema = new mongoose.Schema({
  name: { type: String, uppercase: true },
  age: { type: Number, min: 1, max: 200 },
  email: {
    type: String,
    required: true,
    lowercase: true,
    minLength: 10,
    maxLength: 40,
  },
  createdAt: { type: Date, default: () => Date.now(), immutable: true },
  updatedAt: { type: Date, default: () => Date.now() },
  bestFriend: mongoose.SchemaTypes.ObjectId,
  hobbies: [String],
  address: addressSchema,
});

module.exports = mongoose.model("User", userSchema);
```

### Specifying custom validation

We can also write our own custom validation. Let's use the `age` property, to understand the custom validation:

- We use the `validate` property. This property is set to an object
  - The `validate` object accepts `validator` property. This will be set to a function which will run to validate the value of `age`.
    - That `validator` function accepts an argument which will be the value that is provided to the `age` field.
  - The `validate` object also accepts the `message` property. This will be set to a function that will run when the validation fails.
    - This function also has the access to the value of `age`, in addition to some additional data.

Here is an example:

```js
// User.js
// ...

const userSchema = new mongoose.Schema({
  name: { type: String, uppercase: true },
  age: {
    type: Number,
    min: 1,
    max: 200,

    validate: {
      validator: (value) => value % 2 === 0,
      message: (props) => `${props.value} is not an even number`,
    },
  },

  email: {
    type: String,
    required: true,
    lowercase: true,
    minLength: 10,
    maxLength: 40,
  },
  createdAt: { type: Date, default: () => Date.now(), immutable: true },
  updatedAt: { type: Date, default: () => Date.now() },
  bestFriend: mongoose.SchemaTypes.ObjectId,
  hobbies: [String],
  address: addressSchema,
});

module.exports = mongoose.model("User", userSchema);
```

## The issue with validation

There is an issue with validation, whether built-in or the custom one. The validation runs when we use the `create` or `save` methods. The reason why this is an issue is that there are many other mongoose methods that we can use and they don't go through this validation. They directly run on mongoDB.

One work around for this issue is to use such methods, which could still let us use the `save` method. For example, instead of using `findByIdAndUpdate` method, we can run `findById` and then append `save` method to the result: `findById(...).save()`

## Querying with mongoose

### `findById`

We can find a db item using the `findById` method:

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.findById("<someObjectId>").exec();
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

Notice that we use the `exec()` Mongoose function. This is technically optional and returns a promise. It’s better to use this function since it will prevent some issues.

> The thing is that Mongoose queries are not promises. They have a `.then()` function for **co** and `async`/`await` as a convenience. If you need a fully-fledged promise, use the `.exec()` function.

Just like with the standard MongoDB Node.js driver, we can project only the fields that we need. Let’s only get the `name`, `age`, and `email` fields.

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.findById("<someObjectId>", "name age email").exec();
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

The second parameter can be of type `Object`|`String`|`Array<String>` to specify which fields we would like to project. In this case, we used a `String`.

### `find`

A more generic method is `find`. It works like the `find` method in mongoDB:

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.find({ name: "test5" });
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

### `findOne`

`findOne` method returns the first db item that matches.

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.findOne({ name: "test5" });
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

### `exists`

`exists` returns the id of the first matched db item, if there is at least one match. Otherwise, it returns `null`.

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.exists({ name: "test4" });
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

### `deleteOne`

The `deleteOne` method deletes the first match.

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.deleteOne({ name: "Kyle" });
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

### `where`

Querying in mongoDB might be confusing due to its syntax. To help with that mongoose has `where` method, which we could use to query the database in a different way. For example, instead of this:

```js
const user = await User.find({ name: "test4" });
```

We can use this:

```js
const user = await User.where("name").equals("test4");
```

This also works, if you prefer querying this way:

```js
const user = await User.where({ name: "test4" });
```

We can specify more details when we query with the `where` method:

- We can use `gt` for "greater than`

```js
const user = await User.where("age").gt(20);
```

- We can use `lt` for "less than`

```js
const user = await User.where("age").lt(30);
```

- We can also chain queries with `where`:

```js
const user = await User.where("age")
  .gt(20)
  .lt(30)
  .where("name")
  .equals("test4");
```

### `limit`

We can limit the returned result to a set number of documents using the `limit` method:

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.where("age")
      .gt(20)
      .where("name")
      .equals("test4")
      .limit(2);
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

### `select`

To include projection when using the `where()` method, chain the `select()` method after your query. The `select` method takes only one argument. So, if you need more than one field, chain several `select` methods:

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.where("age")
      .gt(20)
      .where("name")
      .equals("test4")
      .limit(2)
      .select("age")
      .select("name");
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

If instead of selecting specific fields to retrieve, you want to exclude specific fields, put `-` before the field name, like this: `select(-<fieldName>)`.

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.where("age")
      .gt(20)
      .where("name")
      .equals("test4")
      .limit(2)
      .select("age")
      .select("-name");
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

### `ref` and `populate`

When we defined our schema, one of the fields in the schema was `bestFriend`. It was different than other fields, because its type was an `ObjectId`. However, we can use an additional property called `ref` to specifically state what model (collection) this `ObjectId` refers to. This also would help us to use a method called `populate`. First, let's change our **User** schema.

```js
// User.js
// ...

const userSchema = new mongoose.Schema({
  name: { type: String, uppercase: true },
  age: {
    type: Number,
    min: 1,
    max: 200,
    validate: {
      validator: (value) => value % 2 === 0,
      message: (props) => `${props.value} is not an even number`,
    },
  },
  email: {
    type: String,
    required: true,
    lowercase: true,
    minLength: 10,
    maxLength: 40,
  },
  createdAt: { type: Date, default: () => Date.now(), immutable: true },
  updatedAt: { type: Date, default: () => Date.now() },

  bestFriend: { type: mongoose.SchemaTypes.ObjectId, ref: "User" },

  hobbies: [String],
  address: addressSchema,
});

module.exports = mongoose.model("User", userSchema);
```

Now, first let's set a `bestFriend` for one user in our db.

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.where("age")
      .gt(20)
      .where("name")
      .equals("test3")
      .limit(1);

    const bestie = await User.where("name").equals("test4").limit(1);

    user[0].bestFriend = bestie[0]._id;
    await user[0].save();

    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

What we can do now is to use the `populate` method to bring all the data for that `bestFriend` that we specified. The `populate` method automatically populates the `bestFriend` field by joining the data of the matched user to the `bestFriend` field.

```js
// index.js

async function run() {
  try {
    const user = await User.where("age")
      .gt(20)
      .where("name")
      .equals("test3")
      .populate("bestFriend");

    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

Mongoose actually uses the MongoDB `$lookup` method behind the scenes.

## Schema methods

### Methods on each document instance in a model

Here is how to add a simple method to our **User** schema:

```js
// User.js
// ...

// method that could be called on a document
userSchema.methods.sayHi = function () {
  console.log(`Hi. My name is ${this.name}`);
};

module.exports = mongoose.model("User", userSchema);
```

Now, we can call that method on every document:

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.findOne({ name: "test4" });
    user.sayHi();
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

### Static methods

We can add methods to the model itself rather than to every document. This is called a **static** method. This way, instead of calling a method on a specific document, we can use the model called a method directly on it.

Here we create a static method to find users in our model by name:

```js
// User.js
// ...

// method that could be called on a document
userSchema.methods.sayHi = function () {
  console.log(`Hi. My name is ${this.name}`);
};

// method that could be called on a model itself
userSchema.statics.findByName = function (name) {
  return this.find({ name: new RegExp(name, "i") });
};

module.exports = mongoose.model("User", userSchema);
```

Now, we can use that method:

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.findByName("test4");
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

### Adding methods to a query

We can add methods to a returned query rather than to documents or models. This way, we can use a query such as `find`, and then chain our newly created method to that query.

Here, we define a method on a query called `byName`:

```js
// User.js
// ...

// method that could be called on a document
userSchema.methods.sayHi = function () {
  console.log(`Hi. My name is ${this.name}`);
};

// method that could be called on a model itself
userSchema.statics.findByName = function (name) {
  return this.find({ name: new RegExp(name, "i") });
};

// method that could be chained to a query
userSchema.query.byName = function (name) {
  return this.where({ name: new RegExp(name, "i") });
};

module.exports = mongoose.model("User", userSchema);
```

Notice below that we cannot directly use the `byName` on a model. We first, use the `find` query, and then chain the `byName` method onto the returned result.

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.find().byName("test4");
    console.log(user);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

## Virtuals

What if we want a certain value to be returned from a document, but we don't want to add another field to documents. We can add something called `virtual` to our schema. Here is how to add a `virtual`. Notice, we are using `get` with `virtual`:

```js
// User.js
// ...

userSchema.virtual("namedEmail").get(function () {
  return `${this.name} <${this.email}>`;
});

module.exports = mongoose.model("User", userSchema);
```

Now, we can refer to `namedEmail`, even if it's not present as a field in our documents.

```js
// index.js
// ...

async function run() {
  try {
    const user = await User.findOne({ name: "Test4" });
    console.log(user);
    console.log(user.namedEmail);
  } catch (error) {
    console.log(error.message);
  }
}

run();
```

## Middleware

In Mongoose, middleware are functions that run before and/or during the execution of asynchronous functions at the schema level.

We can specify if a middleware should run before removing, saving or validating or after. If we want the code to run before, we use `pre`, and for the after we use `post`:

- `<schemaInstance>.pre('save', function(next) {next()})`
- `<schemaInstance>.pre('remove', function(next) {next()})`
- `<schemaInstance>.pre('validate', function(next) {next()})`
- `<schemaInstance>.post('save', function(next) {next()})`
- `<schemaInstance>.post('remove', function(next) {next()})`
- `<schemaInstance>.post('validate', function(next) {next()})`

Here is an example:

```js
// User.js
// ...

userSchema.pre("save", function (next) {
  this.updatedAt = Date.now();
  next();
});

module.exports = mongoose.model("User", userSchema);
```

Here is an example with `post`. Notice the function accepts the document as the first argument.

```js
// User.js
// ...

userSchema.post("save", function (doc, next) {
  doc.sayHi();
  next();
});

module.exports = mongoose.model("User", userSchema);
```

The middleware function accepts `next` as an argument, and we need to call it for the code to continue execution. When we set a `pre` middleware before saving and don't call `next`, the item is not going to be saved to db. We can also throw an error based on certain conditions using the middleware.
