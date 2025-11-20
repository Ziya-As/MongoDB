# jsonSchema in MongoDB

- [jsonSchema in MongoDB](#jsonschema-in-mongodb)
  - [Some Useful Links](#some-useful-links)
  - [Intro](#intro)
  - [Creating a collection with a schema](#creating-a-collection-with-a-schema)
  - [Adding a schema to an existing collection](#adding-a-schema-to-an-existing-collection)
  - [Specify Allowed Field Values](#specify-allowed-field-values)
  - [Specifying that no other fields should be added to a document](#specifying-that-no-other-fields-should-be-added-to-a-document)
  - [Validation for `null` Field Values](#validation-for-null-field-values)
  - [Validation for numbers](#validation-for-numbers)
    - [`minimum` and `maximum`](#minimum-and-maximum)
  - [Validate `array` bson type](#validate-array-bson-type)
  - [Setting a validation for sub-documents](#setting-a-validation-for-sub-documents)
  - [Viewing a collection's validation rules](#viewing-a-collections-validation-rules)
  - [Dependencies](#dependencies)
    - [Property Dependencies](#property-dependencies)
    - [Schema Dependencies](#schema-dependencies)

## Some Useful Links

- https://www.mongodb.com/blog/post/json-schema-validation--locking-down-your-model-the-smart-way
- https://www.digitalocean.com/community/tutorials/how-to-use-schema-validation-in-mongodb
- https://json-schema.org/learn/getting-started-step-by-step.html
- https://www.percona.com/blog/mongodb-how-to-use-json-schema-validator/
- https://www.mongodb.com/basics/json-schema-examples
- https://www.youtube.com/watch?v=dtLl37W68g8
- https://www.youtube.com/watch?v=DdvhZj7SsEM
- For a full list of keywords, visit [json-schema.org](https://json-schema.org/draft/2020-12/json-schema-validation). The link explains the keywords that you can use in your schema and their purpose.

## Intro

Although having a flexible database could be good, we might want to make our database rigid. If we set that documents in a collection should have some required fields, and the data types should be one of the specified data types, this could help us to avoid unexpected entries into our database.

In MongoDB, schema validation works on individual collections by assigning a JSON Schema document to the collection. JSON Schema is an open standard that allows you to define and validate the structure of JSON documents. You do this by creating a schema definition that lists a set of requirements that documents in the given collection must follow to be considered valid.

## Creating a collection with a schema

This example shows how to create a collection using the MongoDB shell. To create a collection, we use the `createCollection` method.

- The first argument to this method is the name of a collection.
- If we want to set a schema, then as a second argument, we provide an object with specific properties:
  - The object should have `validator` property set to an object:
    - The `validator` object should `$jsonSchema` property set to an object
      - Finally, inside the `$jsonSchema` object, we set our schema (validation)

To specify what type of data a document should have we use `bsonType`. BSON is a binary serialization format used to store documents and make remote procedure calls in MongoDB. There are many BSON types, such as `object`, and `string`, that could be used.

To specify what fields of a document are required fields, we use the `required` property and set it to an array. The array should include the names of fields that are going to be required.

You can use `title` and `description` fields to provide an explanation of validation rules when the rules are not immediately clear. When a document fails validation, MongoDB includes these fields in the error output.

```shell
db.createCollection("collName", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "email", "address"],
      title: "Collection Object Validation",
      properties: {
        name: {
          bsonType: "string",
          description: "Name is required",
        },
        email: {
          bsonType: "string",
          description: "Email is required",
        },
        address: {
          bsonType: "object",
          description: "Address is required",
          properties: {
            street: {
              bsonType: "string",
            },
            city: {
              bsonType: "string",
            },
            country: {
              bsonType: "string",
            },
          },
        },
      },
    },
  },
});
```

Now, we can test if we can insert some documents and if our validation works:

This gets inserted

```shell
db.collName.insertOne({
  name: "John",
  email: "john@test.com",
  address: {
    street: "abc",
    city: "London",
    country: "UK",
  },
});
```

This fails validation and an error is thrown

```shell
db.collName.insertOne({
  name: 123,
  email: "john@test.com",
  address: {
    street: "abc",
    city: "London",
    country: "UK",
  },
});
```

## Adding a schema to an existing collection

`runCommand` should be used to change the json schema validation of a collection that has already been created.

`collMod` in the below code stands for collection modifier. We specify which collection needs to be modified with `collMod`.

Here we set the `validationLevel` to `strict`. We can also set it to `moderate` or `off`.

| `validationLevel` | Description                                                                                                                                |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `"off"`           | No validation for inserts or updates.                                                                                                      |
| `"strict"`        | Default. Apply validation rules to all inserts and all updates.                                                                            |
| `"moderate"`      | Apply validation rules to inserts and to updates on existing valid documents. Do not apply rules to updates on existing invalid documents. |

We also set the `validationAction` to `warn`. We can also set it to `error`. The `validationAction` option determines whether to error on invalid documents or just warn about the violations but allow invalid documents.

```shell
db.runCommand({
  collMod: "collName",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "email", "address"],
      properties: {
        name: {
          bsonType: "string",
          description: "Name is required",
        },
        email: {
          bsonType: "string",
          description: "Email is required",
        },
        address: {
          bsonType: "object",
          description: "Address is required",
          properties: {
            street: {
              bsonType: "string",
            },
            city: {
              bsonType: "string",
            },
            country: {
              bsonType: "string",
            },
          },
        },
      },
    },
  },
  validationLevel: "strict",
  validationAction: "warn",
});
```

Note that we can use the `runCommand` and `collMod` to update the `$jsonSchema` as well.

## Specify Allowed Field Values

When you create a JSON Schema, you can specify what values are allowed in a particular field.

To specify a list of allowed values, use the `enum` keyword in your JSON schema. The `enum` keyword means "enumerate", and is used to list possible values of a field.

```shell
db.createCollection("shipping", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      title: "Shipping Country Validation",
      properties: {
        _id: {
          bsonType: "objectId",
        },
        country: {
          enum: ["UK", "Azerbaijan"],
          description: "Must be either UK, or Azerbaijan",
        },
      },
    },
  },
});
```

This gets inserted

```shell
db.shipping.insertOne({
  country: "Azerbaijan",
});
```

This throws an error

```shell
db.shipping.insertOne({
  country:'Germany'
})
```

## Specifying that no other fields should be added to a document

When you specify `additionalProperties: false` in your JSON schema, MongoDB rejects documents that contain fields not included in your schema's properties object.

```shell
db.createCollection("shipping", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      title: "Shipping Country Validation",
      properties: {
        _id: {
          bsonType: "objectId",
        },
        country: {
          enum: ["UK", "Azerbaijan"],
          description: "Must be either UK, or Azerbaijan",
        },
      },
      additionalProperties: false,
    },
  },
});
```

This gets inserted

```shell
db.shipping.insertOne({
  country: "Azerbaijan",
});
```

This throws an error

```shell
db.shipping.insertOne({
  country: "Azerbaijan",
  region: 'Caucasus'
});
```

## Validation for `null` Field Values

If your schema validates data types for a field, to insert documents with a `null` value for that field, you must explicitly allow `null` as a valid BSON type.

For example, this schema validation does not allow documents where `storeLocation` is `null`:

```shell
db.createCollection("sales", {
  validator: {
    $jsonSchema: {
      properties: {
        storeLocation: { bsonType: "string" },
      },
    },
  },
});
```

Alternatively, this schema validation allows `null` values for `storeLocation`:

```shell
db.createCollection("store", {
  validator: {
    $jsonSchema: {
      properties: {
        storeLocation: { bsonType: ["null", "string"] },
      },
    },
  },
});
```

We can now run this without an issue:

```shell
db.store.insertOne( { storeLocation: null } )
```

## Validation for numbers

There are various BSON types for a number. For example, `int`, `double`, etc. There are specific validation rules that could be set for these types of data.

### `minimum` and `maximum`

We can specify the `minimum` and the `maximum` number that a field can accept.

In the below example, we create a collection of mountain peaks, and setting the minimum and the maximum heights that a peak must have:

```shell
db.createCollection("peaks", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      description: "Document describing a mountain peak",
      required: ["name", "height"],
      properties: {
        name: {
          bsonType: "string",
          description: "Name must be a string and is required",
        },
        height: {
          bsonType: "int",
          description:
            "Height must be a number between 100 and 10000 and is required",
          minimum: 100,
          maximum: 10000,
        },
      },
    },
  },
});
```

## Validate `array` bson type

Arrays have their own specific validation keywords as well:

- `minItems` - validates that the array must contain at least some number of elements.
- `uniqueItems` - ensures that elements within the array will be unique.
- `items` - helps to define a validation schema for each individual array item.

Now, we'll change our collection of mountain peaks to have a new `location` field. This field will be an `array` BSON type, will have at least 1 item, all the items will have to be unique, and each item must be a string:

```shell
db.runCommand({
  collMod: "peaks",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      description: "Document describing a mountain peak",
      required: ["name", "height", "location"],
      properties: {
        name: {
          bsonType: "string",
          description: "Name must be a string and is required",
        },
        height: {
          bsonType: "number",
          description:
            "Height must be a number between 100 and 10000 and is required",
          minimum: 100,
          maximum: 10000,
        },
        location: {
          bsonType: "array",
          description: "Location must be an array of strings",
          minItems: 1,
          uniqueItems: true,
          items: {
            bsonType: "string",
          },
        },
      },
    },
  },
});
```

## Setting a validation for sub-documents

Imagine that we want to add number of ascents, which describes successful attempts at summiting each peak, to our collection of mountain peaks.

After this change, a document in our collection might look like this:

```json
{
  "name": "Everest",
  "height": 8848,
  "location": ["Nepal", "China"],
  "ascents": {
    "first": {
      "year": 1953
    },
    "first_winter": {
      "year": 1980
    },
    "total": 5656
  }
}
```

The `ascents` sub-document must contain a field for total number of attempts. But the fields for `first` attempt or `first_winter` attempt should be optional; maybe a peak wasn't ascented yet.

We define the validation for a sub-document, the way how we defined the validation for our document.

Here is an example:

```shell
db.runCommand({
  collMod: "peaks",
  validator: {
    $jsonSchema: {
      bsonType: "object",
      description: "Document describing a mountain peak",
      required: ["name", "height", "location", "ascents"],
      properties: {
        name: {
          bsonType: "string",
          description: "Name must be a string and is required",
        },
        height: {
          bsonType: "number",
          description:
            "Height must be a number between 100 and 10000 and is required",
          minimum: 100,
          maximum: 10000,
        },
        location: {
          bsonType: "array",
          description: "Location must be an array of strings",
          minItems: 1,
          uniqueItems: true,
          items: {
            bsonType: "string",
          },
        },
        ascents: {
          bsonType: "object",
          description: "Ascent attempts information",
          required: ["total"],
          required: ["total"],
          properties: {
            total: {
              bsonType: "number",
              description: "Total number of ascents must be 0 or higher",
              minimum: 0,
            },
          },
        },
      },
    },
  },
});
```

## Viewing a collection's validation rules

To view a collection's validation rules, use the `db.getCollectionInfos()` method. Alternatively, you can use the MongoDB Compass.

## Dependencies

We can make properties in a schema dependent on other properties.

There are two forms of dependencies we should look at:

- Property dependencies which state that if a specified property is present, other properties must be present as well, and
- Schema dependencies which define a change in the schema when a given property is present.

### Property Dependencies

An example use case of property dependencies would be students.
If a student has graduated from a program, we want to stay in touch with them and want their mailing address to be a required field. If they haven’t graduated, it isn’t necessary.

```shell
db.createCollection("students", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name"],
      properties: {
        _id: {},
        name: {
          bsonType: ["string"],
          description: "'name' is a required string",
        },
        graduated: {
          bsonType: ["bool"],
          description: "'graduated' is an optional boolean value",
        },
      },
      dependencies: {
        graduated: ["mailing_address"],
      },
    },
  },
});
```

With the addition of the `dependencies` keyword, we’ve defined that if `graduated` exists, so must `mailing_address`. One thing to note here is that this dependency is not bidirectional, therefore doing this insert would still work:

```shell
db.students.insertOne({name: "Andrei Popov", mailing_address: "456 Oak Street"})
```

In order to make the dependencies bi-directional, we would need to explicitly define them:

```shell
db.createCollection("students", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name"],
      properties: {
        _id: {},
        name: {
          bsonType: ["string"],
          description: "'name' is a required string",
        },
        graduated: {
          bsonType: ["bool"],
          description: "'graduated' is an optional boolean value",
        },
      },
      dependencies: {
        "graduated": ["mailing_address"],
        "mailing_address": ["graduated"]
      },
    },
  },
});
```

With those bi-directional dependencies in place, neither of these inserts would work:

```shell
db.students.insertOne({name: "Jamal Aattou", graduated: true})
```

```shell
db.students.insertOne({name: "Courtney DeSaja", mailing_address: "789 Broadway"})
```

### Schema Dependencies

Schema dependencies extend the schema to have additional constraints. If we used the schema dependency technique instead of the single direction property dependency technique we could achieve the same results as follows:

```shell
db.createCollection("students", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name"],
      properties: {
        _id: {},
        name: {
          bsonType: ["string"],
          description: "'name' is a required string",
        },
        graduated: {
          bsonType: ["bool"],
          description: "'graduated' is an optional boolean value",
        },
      },
      dependencies: {
        graduated: {
          required: ["mailing_address"],
          properties: {
            mailing_address: {
              bsonType: ["string"],
            },
          },
        },
      },
    },
  },
});
```

If we try a few sample inserts into the `students` collection with those rules defined:

```shell
db.students.insertOne({name: "Alena Weber",
        graduated: true,
        mailing_address: "123 Main Street"})
```

```shell
db.students.insertOne({name: "Jamal Aattou",
        graduated: true})
```

```shell
db.students.insertOne({name: "Chris T. Barker",
        mailing_address: "987 Pine St NE"})
```

In these instances, example 1 and 3 both will be successfully inserted into the database, while example 2 will fail because of a lack of the required mailing_address field.
