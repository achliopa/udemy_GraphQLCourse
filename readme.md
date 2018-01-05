# Section 2 - REST-ful routing

## Lecture 3 - Review of RESTful Routing

* given a collection of record on a server, there should be a uniform URL and  HTTP req method to utilize that collection of records
** /<name> 		POST 	=> create a record
** /<name> 		GET  	=> fetch all records
** /<name>/:id 	GET	 	=> fetch record with given id
** /<name>/:id 	PUT	 	=> update record with given id
** /<name>/:id 	DELETE	=> delete record with given id
* nested resources use similar rule in cascate URL 
** example /<name>/:id/<nested>/:new_id/<morenested>/:another_id

## Lecture 4 - Shortcomings of RESTful Routing

* it shows its limitations even at 3 levels of nesting
* a lot of http requests to implement e.g user/23/friends/companies /user/23/friends/positions => user/23/friends_with_companies_and_positions (NOT RESTful)
* RESTful rules break in highlu related data
* also we return the WHOLE resource even thoug we dont need all the data (overserving data)

# Section 3 - On to GraphQL

## Lecture 5  - What is GraphQL

* GraphQL is based on graph modeling of data in applications. graphs are data structures of nodes and relations between them(edges)
* if we want to find the company names of the friends of a user we issue this query in graphQL:
query {
	user(id: '23') {
		friends {
			company {
				name
			}
		}
	}
}

## Lecture 6  - Working with GraphQL

* we need a datastore, an express/GraphQL Server and a Client issuing GraphQL queries (GraphiQL tool)
* mkdir users , npm init , npm install --save express express-graphql graphql lodash
* make new server.js file and add express boilerplate

## Lecture 7 - Registering GraphQl in Express

* when adding graphql express when gets a request it checks if it is asking for graphql, if yes it passes it to graphql which replies back to express to send the response. if no it handles it by-itself
* we import graphql const expressGraphQL = require('express-graphql');
* we add it to express as a middleware with app.use('/graphql') only for requests on the  route /graphql and children
app.use('/graphql', expressGraphQL({
	graphiql: true
}));
* graphiql is the client graphic client for testing in development
* if we call the route graphql we get an error message that asks for a shema with the options in middleware setup object

## Lecture 8 - GraphQL Schemas

* we need to inform graphql how data in our applicatin is arranged and interconnected
* we make a new folder /schema and we add a file schema.js
* schema must contain wat properties objects contain and how they are connected together
