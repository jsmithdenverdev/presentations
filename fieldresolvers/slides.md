---
# try also 'default' to start simple
theme: seriph
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

# GraphQL and Field Resolvers

---

# Field Resolvers

A field resolver resolves a field in a graphql query.

---

# Questions?

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

- A type system for describing your API and its data.
- A query language to interact with the API.
- An execution engine to execute queries against the API.

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

Ours says we expose one query called `animal` which returns an `Animal` by `ID`.

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

`Animal` is an object type with scalar fields.

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

Making a query

GraphQL is just the specificiation. We need a library that implements the specificiation for our language.

We also need a way to make this implementation accessible, typically done by wrapping it with a publically available HTTP endpoint.

There are many client libraries for creating a GraphQL

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

```graphql {|3,10}
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

```graphql {|,9,12-15}
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

We can query those relationships

```graphql {|,9,12-15}
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

---

# GraphQL, an API panacea?

> /ˌpanəˈsēə/
>
> _noun_
>
> **a solution or remedy for all difficulties or diseases.**

## Absolutely not.

Engineering decision making involves tradeoffs, not perfect solutions.

GraphQL fits our domain well due to the hierarchical nature of our data. It allows us to build a flexible system that exposes data in an intuitive and performant way.

But that tradeoff comes with additional verbosity, complexity, and challenges such as keeping our schema backwards compatible.

With the right approaches we'll continue to build highly flexible systems.

---

```

```
