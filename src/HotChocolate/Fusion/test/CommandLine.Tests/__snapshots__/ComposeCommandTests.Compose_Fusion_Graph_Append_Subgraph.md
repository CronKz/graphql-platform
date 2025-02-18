# Compose_Fusion_Graph_Append_Subgraph

## Schema Document

```graphql
schema {
  query: Query
  mutation: Mutation
  subscription: Subscription
}

type Query {
  errorField: String
  productById(id: ID!): Product
  reviewById(id: ID!): Review
  reviewOrAuthor: ReviewOrAuthor!
  reviews: [Review!]!
  userById(id: ID!): User
  users: [User!]!
  usersById(ids: [ID!]!): [User!]!
  viewer: Viewer!
}

type Mutation {
  addReview(input: AddReviewInput!): AddReviewPayload!
  addUser(input: AddUserInput!): AddUserPayload!
}

type Subscription {
  onNewReview: Review!
  onNewReviewError: Review!
}

type AddReviewPayload {
  review: Review
}

type AddUserPayload {
  user: User
}

type Product {
  id: ID!
  reviews: [Review!]!
}

type Review implements Node {
  author: User!
  body: String!
  errorField: String
  id: ID!
  product: Product!
}

type SomeData {
  accountValue: String!
  reviewsValue: String!
}

"The user who wrote the review."
type User implements Node {
  birthdate: Date!
  errorField: String
  id: ID!
  name: String!
  reviews: [Review!]!
  username: String!
}

type Viewer {
  data: SomeData!
  latestReview: Review
  user: User
}

"The node interface is implemented by entities that have a global unique identifier."
interface Node {
  id: ID!
}

union ReviewOrAuthor = User | Review

input AddReviewInput {
  authorId: Int!
  body: String!
  upc: Int!
}

input AddUserInput {
  birthdate: Date!
  name: String!
  username: String!
}

"The `Date` scalar represents an ISO-8601 compliant date type."
scalar Date
```

## Fusion Graph Document

```graphql
schema @fusion(version: 1) @transport(subgraph: "Accounts", group: "Fusion", location: "http:\/\/localhost:5000\/graphql", kind: "HTTP") @transport(subgraph: "Accounts", group: "Fusion", location: "ws:\/\/localhost:5000\/graphql", kind: "WebSocket") @transport(subgraph: "Reviews2", group: "Fusion", location: "http:\/\/localhost:5000\/graphql", kind: "HTTP") @transport(subgraph: "Reviews2", group: "Fusion", location: "ws:\/\/localhost:5000\/graphql", kind: "WebSocket") {
  query: Query
  mutation: Mutation
  subscription: Subscription
}

type Query {
  errorField: String @resolver(subgraph: "Accounts", select: "{ errorField }")
  productById(id: ID!): Product @variable(subgraph: "Reviews2", name: "id", argument: "id") @resolver(subgraph: "Reviews2", select: "{ productById(id: $id) }", arguments: [ { name: "id", type: "ID!" } ])
  reviewById(id: ID!): Review @variable(subgraph: "Reviews2", name: "id", argument: "id") @resolver(subgraph: "Reviews2", select: "{ reviewById(id: $id) }", arguments: [ { name: "id", type: "ID!" } ])
  reviewOrAuthor: ReviewOrAuthor! @resolver(subgraph: "Reviews2", select: "{ reviewOrAuthor }")
  reviews: [Review!]! @resolver(subgraph: "Reviews2", select: "{ reviews }")
  userById(id: ID!): User @variable(subgraph: "Accounts", name: "id", argument: "id") @resolver(subgraph: "Accounts", select: "{ userById(id: $id) }", arguments: [ { name: "id", type: "ID!" } ]) @variable(subgraph: "Reviews2", name: "id", argument: "id") @resolver(subgraph: "Reviews2", select: "{ authorById(id: $id) }", arguments: [ { name: "id", type: "ID!" } ])
  users: [User!]! @resolver(subgraph: "Accounts", select: "{ users }")
  usersById(ids: [ID!]!): [User!]! @variable(subgraph: "Accounts", name: "ids", argument: "ids") @resolver(subgraph: "Accounts", select: "{ usersById(ids: $ids) }", arguments: [ { name: "ids", type: "[ID!]!" } ])
  viewer: Viewer! @resolver(subgraph: "Accounts", select: "{ viewer }") @resolver(subgraph: "Reviews2", select: "{ viewer }")
}

type Mutation {
  addReview(input: AddReviewInput!): AddReviewPayload! @variable(subgraph: "Reviews2", name: "input", argument: "input") @resolver(subgraph: "Reviews2", select: "{ addReview(input: $input) }", arguments: [ { name: "input", type: "AddReviewInput!" } ])
  addUser(input: AddUserInput!): AddUserPayload! @variable(subgraph: "Accounts", name: "input", argument: "input") @resolver(subgraph: "Accounts", select: "{ addUser(input: $input) }", arguments: [ { name: "input", type: "AddUserInput!" } ])
}

type Subscription {
  onNewReview: Review! @resolver(subgraph: "Reviews2", select: "{ onNewReview }", kind: "SUBSCRIBE")
  onNewReviewError: Review! @resolver(subgraph: "Reviews2", select: "{ onNewReviewError }", kind: "SUBSCRIBE")
}

type AddReviewPayload {
  review: Review @source(subgraph: "Reviews2")
}

type AddUserPayload {
  user: User @source(subgraph: "Accounts")
}

type Product @variable(subgraph: "Reviews2", name: "Product_id", select: "id") @resolver(subgraph: "Reviews2", select: "{ productById(id: $Product_id) }", arguments: [ { name: "Product_id", type: "ID!" } ]) {
  id: ID! @source(subgraph: "Reviews2")
  reviews: [Review!]! @source(subgraph: "Reviews2")
}

type Review implements Node @variable(subgraph: "Reviews2", name: "Review_id", select: "id") @resolver(subgraph: "Reviews2", select: "{ reviewById(id: $Review_id) }", arguments: [ { name: "Review_id", type: "ID!" } ]) @resolver(subgraph: "Reviews2", select: "{ nodes(ids: $Review_id) { ... on Review { ... Review } } }", arguments: [ { name: "Review_id", type: "[ID!]!" } ], kind: "BATCH") {
  author: User! @source(subgraph: "Reviews2")
  body: String! @source(subgraph: "Reviews2")
  errorField: String @source(subgraph: "Reviews2")
  id: ID! @source(subgraph: "Reviews2")
  product: Product! @source(subgraph: "Reviews2")
}

type SomeData {
  accountValue: String! @source(subgraph: "Accounts")
  reviewsValue: String! @source(subgraph: "Reviews2")
}

"The user who wrote the review."
type User implements Node @variable(subgraph: "Accounts", name: "User_id", select: "id") @variable(subgraph: "Reviews2", name: "User_id", select: "id") @resolver(subgraph: "Accounts", select: "{ userById(id: $User_id) }", arguments: [ { name: "User_id", type: "ID!" } ]) @resolver(subgraph: "Accounts", select: "{ usersById(ids: $User_id) }", arguments: [ { name: "User_id", type: "[ID!]!" } ], kind: "BATCH") @resolver(subgraph: "Reviews2", select: "{ authorById(id: $User_id) }", arguments: [ { name: "User_id", type: "ID!" } ]) @resolver(subgraph: "Reviews2", select: "{ nodes(ids: $User_id) { ... on User { ... User } } }", arguments: [ { name: "User_id", type: "[ID!]!" } ], kind: "BATCH") {
  birthdate: Date! @source(subgraph: "Accounts")
  errorField: String @source(subgraph: "Accounts")
  id: ID! @source(subgraph: "Accounts") @source(subgraph: "Reviews2")
  name: String! @source(subgraph: "Accounts") @source(subgraph: "Reviews2")
  reviews: [Review!]! @source(subgraph: "Reviews2")
  username: String! @source(subgraph: "Accounts")
}

type Viewer {
  data: SomeData! @source(subgraph: "Accounts") @source(subgraph: "Reviews2")
  latestReview: Review @source(subgraph: "Reviews2")
  user: User @source(subgraph: "Accounts")
}

"The node interface is implemented by entities that have a global unique identifier."
interface Node {
  id: ID!
}

union ReviewOrAuthor = User | Review

input AddReviewInput {
  authorId: Int!
  body: String!
  upc: Int!
}

input AddUserInput {
  birthdate: Date!
  name: String!
  username: String!
}

"The `Date` scalar represents an ISO-8601 compliant date type."
scalar Date
```

## Accounts Subgraph Configuration

```json
{
  "Name": "Accounts",
  "Schema": "schema {\n  query: Query\n  mutation: Mutation\n}\n\n\"The node interface is implemented by entities that have a global unique identifier.\"\ninterface Node {\n  id: ID!\n}\n\ntype Query {\n  \"Fetches an object given its ID.\"\n  node(\"ID of the object.\" id: ID!): Node\n  \"Lookup nodes by a list of IDs.\"\n  nodes(\"The list of node IDs.\" ids: [ID!]!): [Node]!\n  users: [User!]!\n  userById(id: ID!): User\n  usersById(ids: [ID!]!): [User!]!\n  errorField: String\n  viewer: Viewer!\n}\n\ntype Mutation {\n  addUser(input: AddUserInput!): AddUserPayload!\n}\n\n\"The `Date` scalar represents an ISO-8601 compliant date type.\"\nscalar Date\n\ntype Viewer {\n  user: User\n  data: SomeData!\n}\n\ntype User implements Node {\n  errorField: String\n  id: ID!\n  name: String!\n  birthdate: Date!\n  username: String!\n}\n\ntype SomeData {\n  accountValue: String!\n}\n\ninput AddUserInput {\n  name: String!\n  username: String!\n  birthdate: Date!\n}\n\ntype AddUserPayload {\n  user: User\n}",
  "Extensions": [
    "extend type Query {\n  userById(id: ID!\n    @is(field: \"id\")): User!\n  usersById(ids: [ID!]!\n    @is(field: \"id\")): [User!]!\n}"
  ],
  "Clients": [
    {
      "ClientName": null,
      "BaseAddress": "http://localhost:5000/graphql"
    },
    {
      "ClientName": null,
      "BaseAddress": "ws://localhost:5000/graphql"
    }
  ],
  "ConfigurationExtensions": null
}
```

## Reviews2 Subgraph Configuration

```json
{
  "Name": "Reviews2",
  "Schema": "schema {\n  query: Query\n  mutation: Mutation\n  subscription: Subscription\n}\n\n\"The node interface is implemented by entities that have a global unique identifier.\"\ninterface Node {\n  id: ID!\n}\n\ntype Query {\n  \"Fetches an object given its ID.\"\n  node(\"ID of the object.\" id: ID!): Node\n  \"Lookup nodes by a list of IDs.\"\n  nodes(\"The list of node IDs.\" ids: [ID!]!): [Node]!\n  reviews: [Review!]!\n  reviewById(id: ID!): Review\n  authorById(id: ID!): User\n  productById(id: ID!): Product\n  reviewOrAuthor: ReviewOrAuthor!\n  viewer: Viewer!\n}\n\ntype Mutation {\n  addReview(input: AddReviewInput!): AddReviewPayload!\n}\n\ntype Subscription {\n  onNewReview: Review!\n  onNewReviewError: Review!\n}\n\ntype Product {\n  reviews: [Review!]!\n  id: ID!\n}\n\n\"The user who wrote the review.\"\ntype User implements Node {\n  reviews: [Review!]!\n  id: ID!\n  name: String!\n}\n\ntype Review implements Node {\n  errorField: String\n  id: ID!\n  author: User!\n  product: Product!\n  body: String!\n}\n\nunion ReviewOrAuthor = User | Review\n\ntype Viewer {\n  latestReview: Review\n  data: SomeData!\n}\n\ntype SomeData {\n  reviewsValue: String!\n}\n\ninput AddReviewInput {\n  body: String!\n  authorId: Int!\n  upc: Int!\n}\n\ntype AddReviewPayload {\n  review: Review\n}",
  "Extensions": [
    "extend type Query {\n  authorById(id: ID!\n    @is(field: \"id\")): User\n  productById(id: ID!\n    @is(field: \"id\")): Product\n}\n\nschema\n  @rename(coordinate: \"Query.authorById\", newName: \"userById\") {\n\n}"
  ],
  "Clients": [
    {
      "ClientName": null,
      "BaseAddress": "http://localhost:5000/graphql"
    },
    {
      "ClientName": null,
      "BaseAddress": "ws://localhost:5000/graphql"
    }
  ],
  "ConfigurationExtensions": null
}
```

