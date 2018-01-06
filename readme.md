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


## Lecture 9 - Writing the  Schema

* we impor graphql 
* we decompose a number of  classes 
* we use GraphQLObjectType to instruct graphql about the presence of a user. we create a UserType instantiating the GraphQLObjectType. this Objecttype has 2 required properies. name and fields which contains an object with the properties (key:name value: objec with type)
* for types we dont use String like in mongoose but GraphQLString. same holds for others e.g.

const UserType = new GraphQLObjectType({
	name: 'User',
	fields: {
		id: { type: GraphQLString},
		firstName: { type: GraphQLString},
		age: { type: GraphQLInt}
	}
});

## Lecture 10 - Root Queries

* Root Query is the entry point to our data or application, the first part of the query. it is a GraphQLObjectType of name RootQueyType. it
has fields-properties. a field is user of type UserType and args an id of QraphQLString. the meaning is that it says to the app. i am the entry point if you are looking for users give me an id and i will return you a UserType => User
* a very important parameter is the resolve function. it gets args which are already define them. in our example it implements the way to give back users based on id. resolve brings the date where as attributes describe the data.

## Lecture 11 - Resolving with data

* in resolve we implement the logic to return a user based on id from an array (no DB yet)
* our finished RootQuery looks like: 
const RootQuery = new GraphQLObjectType({
	name: 'RootQueryType',
	fields: {
		user: {
			type: UserType,
			args: { id: { type: GraphQLString } },
			resolve(parentValue, args) {
				return _.find(users, { id: args.id });
			}
		}
	}
});
* we instantiate a GraphQLQuery passing as query the RootQuery and we export it to add it to the express.middleware  params: express-graphql
after importing it to server.js

## Lecture 12 - GraphiQL tool

* after exporting the schema and adding it to the middleware when we access /graphql route we see the  graphiQL client dev tool
* there we write our first graphQL query:

{
  user(id: "47") {
    id,
    firstName,
    age
  }
}

* and we get the result:

{
  "data": {
    "user": {
      "id": "47",
      "firstName": "Samantha",
      "age": 21
    }
  }
}

* the query looks like javascript but is not. the result looks like JSON
* the query goes to the RootQueryType object -> because we specify user in the query the rootquery goes to the user key in the fields obj. we specify the id which we provide. resolve function fetches the data to graphql which extracts the data we specify and populates the reply
* inside the resolve we dont have to be graphql specific . we reply raw data. graphql trasforms them after ito graphql types
* we can modify the query and see the result, removing attributes, or changin the search argument

## Lecture 13 - Realistic Data Source

* in real world apps express/graphql server gets data from different sources . outside db servers, apis etc

* we use json-server as an outside data server (github project): npm install --save json-server
* we add a file (db.json) in project root folder where we specify the data to be served
* json-server is a separate process on another port
* we add a run script for it in package.json: "json:server": "json-server --watch db.json"
* we npm run it and check the rest routes it offers in browser: e.g localhost:3000/users

## Lecture 14 - Async Resolve Functions

* resolve supports promises
* datafetching from external services is always asynchrnous
* to fetch data from json-server we can use fetch or axios: npm install --save axios
* we import axios in schema.js and modify resolve to fetch data from json-server with axios:
			resolve(parentValue, args) {
				return axios.get(`http://localhost:3000/users/${args.id}`)
					.then(resp => resp.data);
			}
* this code returns a promise but graphql can handle that. also axios response contains body in res.data so we use the then statement to extract that data and pass them to the next then.
* install nodemon and run server,js with it
* add script to pacakage.json     "dev": "nodemon server.js". npm run dev

## Lecture 16 - Company Defs

* we add a new collection in json-server db.json file: companies
* we add attributes id, name, description
* we add a companyId propery to users to link them with companies
* json-server resolves the relation and adds nested restful routes to the API e.g. /companies/1/users