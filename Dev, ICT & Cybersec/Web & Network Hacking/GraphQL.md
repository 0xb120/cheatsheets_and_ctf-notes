# GraphQL 101

>[!question] What is GraphQL?
> GraphQL is an API query language that is designed to facilitate efficient communication between clients and servers. It enables the user to specify exactly what data they want in the response, helping to avoid the large response objects and multiple calls that can sometimes be seen with REST APIs.

In GraphQL, clients send queries to a GraphQL server, which fetches data from the relevant places. GraphQL is platform-agnostic.

## How GraphQL works

GraphQL schemas define the structure of the service's data, listing the available objects (known as types), fields, and relationships.
Data can be manipulated using 3 different types of operation:
- **Queries**: fetch data
- **Mutations**: add, change or remove data
- **Subscriptions**: similar to queries, but set up a permanent connection by which a server can proactively push data to a client

All GraphQL operations use the same endpoint, and are generally sent as a POST request and the type and name of the operation define how the query is handled.

## Schemas

Schema represents a contract between the frontend and backend of the service. It defines the data available as a series of types, and these types can then be implemented by a service.

>[!info]
>Most of the types defined are object types, which define the objects available and the fields and arguments they have. Each field has its own type, which can either be another object or a scalar, enum, union, interface, or custom type.

```graphql
# Example schema definition

type Product {
	id: ID!
	name: String!
	description: String!
	price: Int
}
```

Schemas **must** also include at least one available **query**. Usually, they also contain details of available **mutations**.

## Queries

Queries retrieve data from the data store. They are roughly equivalent to GET requests in a REST API.

Key components:
- **`query` operation type** (optional) - explicitly tells the server that the incoming request is a query
- **query name** (optional) - can be anything you want
- **data structure** - the data that the query should return
- **arguments** (optional) - used to create queries that return details of a specific object

```graphql
# Example query

query myGetProductQuery {
	getProduct(id: 123) {
		name
		description
	}
}
```

>[!note]
>Note that the product type may contain more fields in the schema than those requested here. The ability to request only the data you need is a significant part of the flexibility of GraphQL (like query in SQL).

## Mutations

Mutations change data in some way, either adding, deleting, or editing it. They are roughly equivalent to a REST API's POST, PUT, and DELETE methods.

Like queries, mutations have an operation type, name, and structure for the returned data. However, **mutations always take an input** of some type. This can be an inline value, but in practice is generally provided as a variable.

```graphql
# Example mutation request

mutation {
	createProduct(name: "Flamin' Cocktail Glasses", listed: "yes") {
		id
		name
		listed
	}
}

# Example mutation response

{
	"data": {
		"createProduct": {
			"id": 123, # the service is configured to automatically assign an ID to new products
			"name": "Flamin' Cocktail Glasses",
			"listed": "yes"
		}
	}
}
```

## Components of queries and mutations

### Fields

All GraphQL types contain items of queryable data called fields. When you send a query or mutation, you specify which of the fields you want the API to return.

```graphql
# Request
## In this case, `id`, `name.firstname`, and `name.lastname` are the fields requested.

query myGetEmployeeQuery {
	getEmployees {
		id
		name {
			firstname
			lastname
		}
	}
}

#Response

{
	"data": {
		"getEmployees": [
			{
				"id": 1,
				"name" {
					"firstname": "Carlos",
					"lastname": "Montoya"
				}
			},
			{
				"id": 2,
				"name" {
					"firstname": "Peter",
					"lastname": "Wiener"
				}
			}
		]
	}
}
```

### Arguments

Arguments are values that are provided for specific fields. The arguments that can be accepted for a type are defined in the schema.
When you send a query or mutation that contains arguments, the GraphQL server determines how to respond based on its configuration.

```graphql
# Example query with arguments
## The example below shows a `getEmployee` request that takes an employee ID as an argument.

query myGetEmployeeQuery {
	getEmployees(id:1) {
		name {
			firstname
			lastname
		}
	}
}

# Response to query

{
	"data": {
		"getEmployees": [
		{
			"name" {
				"firstname": Carlos,
				"lastname": Montoya
				}
			}
		]
	}
}
```

### Variables

Variables enable you to pass dynamic arguments, rather than having arguments directly within the query itself.

Variable-based queries use the same structure as queries using inline arguments, but certain aspects of the query are taken from a separate JSON-based variables dictionary.

When building a query or mutation that uses variables, you need to:
- Declare the variable and type.
- Add the variable name in the appropriate place in the query.
- Pass the variable key and value from the variable dictionary.

```graphql
# Example query with variable
## In this example, the variable is declared in the first line with (`$id: ID!`)
## The `!` indicates that this is a required field for this query

query getEmployeeWithVariable($id: ID!) {
	getEmployees(id:$id) {
		name {
			firstname
			lastname
		}
	 }
}

Variables:
{
	"id": 1
}
```

### Aliases

GraphQL objects can't contain multiple properties with the same name. Aliases enable you to bypass this restriction by explicitly naming the properties you want the API to return.

```graphql
# Valid query using aliases

query getProductDetails {
	product1: getProduct(id: "1") {
		id
		name
	}
	product2: getProduct(id: "2") {
		id
		name
	}
}

# Response to query

{
	"data": {
		"product1": {
			"id": 1,
			"name": "Juice Extractor"
		 },
		"product2": {
			"id": 2,
			"name": "Fruit Overlays"
		}
	}
}
```

### Fragments

Fragments are reusable parts of queries or mutations. They contain a subset of the fields belonging to the associated type.

Once defined, they can be included in queries or mutations. If they are subsequently changed, the change is included in every query or mutation that calls the fragment.

```graphql
# Example fragment

fragment productInfo on Product {
	id
	name
	listed
}

#Query calling the fragment

query {
	getProduct(id: 1) {
		...productInfo
		stock
	}
}

# Response including fragment fields

{
	"data": {
		"getProduct": {
			"id": 1,
			"name": "Juice Extractor",
			"listed": "no",
			"stock": 5
		}
	}
}
```

## Subscriptions

Subscriptions are a special type of query. They enable clients to establish a long-lived connection with a server so that the server can then push real-time updates to the client without the need to continually poll for data.

As with regular queries and mutations, the subscription request defines the shape of the data to be returned. They are commonly implemented using [WebSockets](WebSockets.md).

## Introspection

>[!info] Introspection
>Introspection is a built-in GraphQL function that enables you to query a server for information about the schema.

Like regular queries, you can specify the fields and structure of the response you want to be returned. For example, you might want the response to only contain the names of available mutations.

Introspection can represent a serious [Information Disclosure](Information%20Disclosure.md) risk, as it can be used to access potentially sensitive information (such as field descriptions) and help an attacker to learn how they can interact with the API.

To use introspection to discover schema information, query the `__schema` field. This field is available on the root type of all queries. Like regular queries, you can specify the fields and structure of the response you want to be returned when running an introspection query. For example, you might want the response to contain only the names of available mutations.

```graphql
# Introspection probe request

{
	"query": "{__schema{queryType{name}}}"
}

# Full introspection query

query IntrospectionQuery {
	__schema {
		queryType {
			name
		}
		mutationType {
			name
		}
		subscriptionType {
			name
		}
		types {
		 ...FullType
		}
		directives {
			name
			description
			args {
				...InputValue
		}
		onOperation  #Often needs to be deleted to run query
		onFragment   #Often needs to be deleted to run query
		onField      #Often needs to be deleted to run query
		}
	}
}

fragment FullType on __Type {
	kind
	name
	description
	fields(includeDeprecated: true) {
		name
		description
		args {
			...InputValue
		}
		type {
			...TypeRef
		}
		isDeprecated
		deprecationReason
	}
	inputFields {
		...InputValue
	}
	interfaces {
		...TypeRef
	}
	enumValues(includeDeprecated: true) {
		name
		description
		isDeprecated
		deprecationReason
	}
	possibleTypes {
		...TypeRef
	}
}

fragment InputValue on __InputValue {
	name
	description
	type {
		...TypeRef
	}
	defaultValue
}

fragment TypeRef on __Type {
	kind
	name
	ofType {
		kind
		name
		ofType {
			kind
			name
			ofType {
				kind
				name
			}
		}
	}
}
```

>[!Warning]
>If introspection is enabled but the above query doesn't run, try removing the `onOperation`, `onFragment`, and `onField` directives from the query structure. Many endpoints do not accept these directives as part of an introspection query, and you can often have more success with introspection by removing them.]

### Suggestions

Even if introspection is entirely disabled, you can sometimes use suggestions to glean information on an API's structure.
Suggestions are a feature of the Apollo GraphQL platform in which the server can suggest query amendments in error messages.

These are generally used where a query is slightly incorrect but still recognizable (for example, `There is no entry for 'productInfo'. Did you mean 'productInformation' instead?`). Use [clairvoyance](https://github.com/nikitastupin/clairvoyance) to automatically enumerate those data.