---
# try also 'default' to start simple
theme: default
# theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides, markdown enabled
title: Welcome to Slidev
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

# Field Resolvers in GraphQL

---

# Field Resolvers

<!-- An empty header formats the next line as standard slide text instead of a subheading -->

##

A field resolver resolves a field in a graphql query.

---

# Questions?

<!-- An empty header formats the next line as standard slide text instead of a subheading -->

##

Thank you for coming to my TED talk.

---

# Just kidding...

---

# Primers

Some things to review before diving in

- GraphQL refresher
- Graph data model
- Problems graphql attempts to solve
- An API panacea?

---

# Refresher

## What is GraphQL?

An open source query language and runtime for API's.

## Cool, what does that mean?

- A type system for describing your API and its data as a schema.
- A query language to interact with that API.
- An engine to execute queries against that API.

---

# Refresher

Schemas and types

A GraphQL API is backed by a schema, containing the types and operations for that API.

```graphql
type Query {
  animal(id: ID!): Animal
}

type Animal {
  id: ID!
  name: String!
  age: Int!
}
```

---

# Refresher

Schemas and types

All schemas have a root Query type that lists the queries the service supports.

Ours schema exposes one field in our query called `animal` which returns an `Animal` by `ID`.

```graphql {1-3|2}
type Query {
  animal(id: ID!): Animal
}

type Animal {
  id: ID!
  name: String!
  age: Int!
}
```

---

# Refresher

Schemas and types

GraphQL supports scalars (`String`, `Boolean`, `Int`, `ID`) as well as objects, lists, and more.

`Animal` is an object type with scalar fields `ID`, `String` and `Int`.

```graphql {5-9|6|7|8}
type Query {
  animal(id: ID!): Animal
}

type Animal {
  id: ID!
  name: String!
  age: Int!
}
```

---

# Refresher

Executing a query

GraphQL is just the specificiation. We need a library that implements the specificiation for our language.

We also need a way to make this implementation accessible, typically done by wrapping it with an HTTP handler.

There are many libraries for creating a GraphQL server.

<iframe src="https://graphql.org/code/#language-support" style="width: 100%; height: 250px;" />

---

# Refresher

Query

```graphql {|1|2,6,12,16|3-5,13-15}
query readAnimal {
  animal(id: "2ba43365-973a-41eb-87ff-b6033f332885") {
    id
    name
    age
  }
}

"""result
{
  "data": {
    "animal": {
      "id": "2ba43365-973a-41eb-87ff-b6033f332885",
      "name": "George",
      "age": 7
    }
  }
}
"""
```

---

# Refresher

Querying what we want

We can modify our query to return only the data we're interested in.

```graphql {|3,11}
query readAnimal {
  animal(id: "2ba43365-973a-41eb-87ff-b6033f332885") {
    name
  }
}

"""result
{
  "data": {
    "animal": {
      "name": "George"
    }
  }
}
"""
```

---

# Thinking in graphs

Our types can also express relationships (this is where the _Graph_ of GraphQL comes in)

```graphql {|,12-15|9}
type Query {
  animal(id: ID!): Animal
}

type Animal {
  id: ID!
  name: String!
  age: Int!
  owners: [Human!]
}

type Human {
  id: ID!
  name: String!
}
```

---

# Thinking in graphs

We can also query these relationships

```graphql {|,3-5,12-16}
query georgeOwners {
  animal(id: "2ba43365-973a-41eb-87ff-b6033f332885") {
    owners {
      name
    }
  }
}

""" result
{
  "data": {
    "owners": [
      {
        "name": "Jake"
      }
    ]
  }
}
"""
```

---

# What is GraphQL trying to solve?

- Allows us to model our types and their relationships as a graph
- Helps us fetch just the data we need
- Allows us to fetch this data from a single resource

---

# GraphQL, an API panacea?

> /ˌpanəˈsēə/
>
> _noun_
>
> **a solution or remedy for all difficulties or diseases.**

## Absolutely not

Engineering decision making involves tradeoffs, not perfect solutions.

GraphQL fits our domain well due to the hierarchical nature of our data. It allows us to build a flexible system that exposes data in an intuitive and performant way.

But this comes with additional verbosity, complexity, and challenges such as managing performance and keeping our schema backwards compatible.

With the right approaches we'll continue to build highly flexible systems.

---

# Recap

It's fields all the way down...

- Our schema has a query type
- This query type has fields for the various queries we support
- Those queries can return scalar values (strings, numbers, etc) or complex values like objects
- The objects we return have their own sets of fields, these can also be scalar values or complex types

And something new...

- Fields can be backed by a resolver
- This changes the behavior of the query
- That resolver is only executed if that field is requested

---

# Revisiting an earlier example

Our pet friends API

We have the following GraphQL schema

Our `Animal` type has an `owners` field which is a list of `Human` type.

```graphql {|9,12-15}
type Query {
  animal(id: ID!): Animal
}

type Animal {
  id: ID!
  name: String!
  age: Int!
  owners: [Human!]
}

type Human {
  id: ID!
  name: String!
}
```

---

# Let's look at some resolvers

_Disclaimer: we're about to enter psuedo code land, where things work simply because I want them to_

---

# Example 1

##

First, the simplest version of our query. It resolves the entire `Animal` in one call.

We query the database and use a sql join to retrieve the related owner data.

```typescript
const resolvers = {
  Query: {
    async animal({ params, db }): Promise<Animal> {
      const animal = await db.queryRow(
        `SELECT a.id, a.name, a.age, o.id, o.name FROM animals a
         INNER JOIN owners o ON o.animal_id = a.id
         WHERE a.id = $1`,
        params["id"]
      );
      return animal;
    },
  },
};
```

---

# Example 1

## Pros

- Simple
- Quick to implement
- For small models and relations (especially 1:1), performance should be fine
- This approach can work great initially or on smaller projects

## Cons

- May end up with additional relationships to fetch, leading to a larger and larger resolver
  - E.g., `Human` may have a `Contact` field, with data living in SalesForce (🤮)
  - Now we need to implement a SalesForce client just to continue fetching `Animal` data
- Query performance may suffer from loading more relations
- Still over fetching
  - Even if we aren't interested in `owners` data, the `animal` resolver is still fetching it

---

# Example 2

##

Now, we'll create a resolver for the `owners` field on the `Animal` type.

```typescript
const resolvers = {
  Query: {
    async animal({ parent, args, { db } }): Promise<Animal> {
      const animal = await db.queryRow(
        `SELECT id, name, age FROM animals WHERE id = $1`,
        args["id"]
      );
      return animal;
    },
  },
  Animal: {
    async owners({ parent, args, { db } }): Promise<Owner[]> {
      const owners = await db.query(
        `SELECT id, name FROM owners WHERE animal_id = $1`,
        parent["id"]
      );
      return owners;
    },
  },
};
```

---

# Example 2

## Pros

- Simpler resolver, fetches only the data needed to satisfy the scalar fields of an `Animal`
- Allows us to completely omit a DB query if we don't need `owners` data

## Cons

- Slightly more complex, requires a secondary resolver function
- Brings us to the GraphQL n+1 query problem

---

# The N+1 query problem

A bad day to be a field

The N+1 query problem arises when we leverage resolvers.

Resolvers are executed independently and don't know if similar data has been loaded or can be batched.

---

# An example of N+1

New requirements for our pet friends API

Let's imagine we have a new requirement to include `Contact` data in our API.

```graphql
type Query {
  animal(id: ID!): Animal
}

type Animal {
  id: ID!
  name: String!
  age: Int!
  owners: [Human!]
}

type Human {
  id: ID!
  name: String!
  contact: Contact!
}

type Contact {
  phone: String!
}
```

---

# An example of N+1

Things appear ok on the surface

But we can see issues. If we have a pet with 50 owners (weird) and we need the contact info, we're going to run the `contact` field resolver 50 times.

```typescript {|}{maxHeight: '70%'}
const resolvers = {
  Query: {
    async animal({ parent, args, { db }}): Promise<Animal> {
      const animal = await db.queryRow(
        `SELECT id, name, age FROM animals WHERE id = $1`,
        args["id"]
      );
      return animal;
    },
  },
  Animal: {
    async owners({ parent, args, { db }}): Promise<Owner[]> {
      const owners = await db.query(
        `SELECT id, name FROM owners WHERE animal_id = $1`,
        parent["id"]
      );
      return owners;
    },
  },
  Human: {
    async contact({ parent, args, { salesforce }}): Promise<Contact> {
      const contact = await salesforce.fetchContact(parent["id"]);
      return contact;
    }
  }
};
```

---

# Enter data loader

The batch and cache master

Instead of our resolvers directly calling the database, they call a data loader batching function which returns a promise.

Resolvers are typically called in serial, and when a promise is returned the runtime moves onto the next resolver.

This behavior allows the batching function to aggregate calls from each resolver, and submit a single query.
