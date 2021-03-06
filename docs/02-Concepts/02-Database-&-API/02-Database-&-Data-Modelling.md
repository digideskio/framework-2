---
alias: sai7aes3iv 
description: In introduction to Graphcool's database layer.
---


# Database & Data Modelling

## Graphcool Abstracts Away the Database Layer

In traditional backend development, a big chunk of work usually goes into the database layer. That includes setting up the database, configuring (and migrating) the schema as well as implementing a data access layer (or integrating an ORM) to retrieve the data from the database and mapping it to in-memory data structures.

When using Graphcool, the database layer is abstracted away:

* With Graphcool, you use the GraphQL SDL to define your data model. Graphcool then generates and manages the underlying database schema for you. (This also includes migrations.)
* Graphcool also manages the whole data access layer. The GraphQL engine takes care of retrieving required information from the database and maps it such that it can be returned by the API. 

![](https://imgur.com/XkyqWYg.png)

> Every Graphcool service comes with an [AWS Aurora](https://aws.amazon.com/rds/aurora/) instance that is backing the GraphQL server.


## Using GraphQL SDL to Define Your Data Model

When using Graphcool, you're leveraging the [GraphQL type system](http://graphql.org/learn/schema/#type-system) to define the data model that represents the entities of your application domain. All types in your data model are defined using the GraphQL SDL. 

Graphcool will take the definition of each type and generate the required CRUD operations that can be accessed through the GraphQL API. Whenever you add a new type or make another change to your database schema, Graphcool will migrate the underlying relational schema that backs the SQL database and make sure the changes are also reflected in the GraphQL API.


### GraphQL Schema vs Database Schema

When using Graphcool, there is an important distinction to make between the* GraphQL schema* that defines the capabilities of the actual API and the *database schema* that's used for modelling data.

The *database schema* only defines the *data model*, i.e. it is used to represent objects from your *application domain*. An example for this is the schema for the blogging application that we saw in the example above. `Person` and `Post` are part of the database schema and are called *model types*. 

Generally when using GraphQL though, the actual *GraphQL schema* has an even broader role and will contain additional definitions that don't directly model entities from the application domain, such as input types for mutations (to aggregate multiple scalar values into an object type) or special filter types. With Graphcool, the GraphQL schema is generated based on the model types that have been provided in the database schema.


## GraphQL Root Types

Every GraphQL schema can contain up to three *root types* that represent the entry-points to the API. These root types are called: `Query`, `Mutation` and `Subscription`. 

The fields of these root types correspond precisely to the *root fields* that can be used in queries, mutations and subscriptions that the server receives. So, if we wanted to make the three examples work that we've seen in the previous sections, we'd have to define the following schema:

```graphql
###################################################################
# Root Types: Define the entry points for the API
###################################################################

type Query {
  allPersons: [Person!]!
}

type Mutation {
  createPerson(name: String!): Person
}

type Subscription {
  Post(filter: PostSubscriptionFilter): PostSubscriptionPayload
}


###################################################################
# Model Types: Represent entities from the application domain
###################################################################

type Person {
  name: String!
  posts: [Post!]!
}

type Post {
  title: String!
  content: String
  author: Person!
}


###################################################################
# API Types: Remaining utility types to complete the API
###################################################################

type PostSubscriptionFilter {
  AND: [PostSubscriptionFilter!]
  OR: [PostSubscriptionFilter!]
  mutation_in: [_ModelMutationType!]
  node: PostSubscriptionFilterNode
}

type PostSubscriptionPayload {
  mutation: _ModelMutationType!
  node: Person
  updatedFields: [String!]
}

enum _ModelMutationType {
  CREATED
  UPDATED
  DELETED
}
```

As you can see, the schema not only contains the model types that represent the entities from the application domain (`Person` and `Post`), but also includes the mentioned root types, plus a number of API types to complete the API. 

Notice that each of the types has different implications with respect to being generated by Graphcool or managed by the developer:

* Model types: Written and managed by the developer
* Root types: Auto-generated based on model types (the `Query` and Mutation type can however be extended through [schema extensions](http:/#))
* API types: Auto-generated based on model types (developers can however add custom API types when using [schema extensions](http:/#))
