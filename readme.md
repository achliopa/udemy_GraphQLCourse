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

Section 4 - Fetching Data from Queries

## Lecture 17 - Nested Queries

* we add CompanyType to Schema much like UserType, having as a ref the resource definition to the json-server. it is a GraphQLObjectType
* we associate it with UserType. in GraphQL associations are treated in the same way as fields, so we add a field company: to link them.
* graphQL when a field in the Model Class and in the GrapthQL Type  class have the same name doesnt need to do any resolving. but when they are different e.g Model.companyId Type.company needs resolving. This holds mainly for data associations
* resolve to implement the association is the following
			resolve(parentValue, args) {
				return axios.get(`http://localhost:3000/companies/${parentValue.companyId}`)
					.then(res => res.data);
			}
* in type to type associations the foreign key is in parentValue
* we form the nested graphql query in the graphiql client
{
  user(id: "40") {
    firstName
    company {
      id
      name
      description
    }
  }
}

## Lecture 20 - Multiple Root Query Entry Points

* we cannot for a query asking directlly because our root point only to the UserType
* we add another field to the rRootType in schema. for company. the syntax is the same as for users

## Lecture 21 - Bidirectional Relations

* we want to get the users that work in a company.
* this relation in a json-server is implemented based on the companyId.
* we access it with /companies/1/users
* in schema we add users field in companyTYpe
* it is an 1:n relationship so we use GraphQLList Class.
* the resolve function trigger the path from json mentioned before.

## Lecture 23 - Resolve Circular References

* we try to use a variable before it is  defined
* this is circular reference
* graphQL solves it by wrapping the fields object in an anonymous arrow function

## Lecture 24 - Query Fragments

* in apps its useful to name our queries like:
query queryName { ... }
* we can ask for the same resource multiple times in a graphQL query. but we have to assigne a name to each subquery as the reply is a json object and key values must be distinct. e.g google: company(id: "2") {}
* to avoid repetition of query object attributes we can use query fragments. fragments are defined for a Type and thei are called in the query as ...fragment e.g.

fragment companyDetails on Company {
  id
  name
  description
}

    apple: company(id: "1") {
		...companyDetails
    users{
      firstName
    }

 ## Lecture 25 - Intro to Mutations

 * mutations allows to modify our datastore in graphQL server, add,delete, modify data
 * mutation sits along with query in the heart of the Schema. query is instance of RootQuery which uses Types to support queries. mutation instantiates Mutations which contains Type operators to to the CUD part of CRUD operations on the datastore. 
 * in graphQl we specify the fields we will use to create a data record in the store. the mutation syntax is similar to the TYpe defs


 ## Lecture 26 - NuNull Fields and Mutations

 * if we wnt to set a argument field as non null we wrap its type with GraphQLNonNull ype: new GraphQLNonNull(GraphQLInt)
 * resolve function gets its arguments from args (second input object)
 * our mutation with addUser field complete looks like: 

 const mutation = new GraphQLObjectType({
	name: 'Mutation',
	fields: {
		addUser: {
			type: UserType,
			args: {
				firstName: { type: new GraphQLNonNull(GraphQLString) },
				age: { type: new GraphQLNonNull(GraphQLInt) },
				companyId: { type: GraphQLString }
			},
			resolve(parentValue, { firstName, age }) {
				return axios.post('http://localhost:3000/users', { firstName, age })
					.then(res => res.data);
			}
		}
	}
});

* we add mutation to our Schema.
* we invoke mutation from client with

mutation {
  addUser(firstName: "Stephen", age: 26) {
    id
    firstName
    age
  }
}

* we always have to ask for at least one field of the created data type back. e.g id

 ## Lecture 27 - deleteUser  Mutation field

 * result: deleteUser: {
			type: UserType,
			args: {
				id: { type: new GraphQLNonNull(GraphQLString)},
			},
			resolve(parentvalue, { id }) {
				return axios.delete(`http://localhost:3000/users/${id}`)
					.then(res => res.data);
			}
		}
* json-server doesnt return the deleted record so graphql passes back null

 ## Lecture 28 - editUser  Mutation field

 * http PUT request replaces record
 * http PATCH request updates only the properties sent inthe request body

 * result: editUser: {
			type: UserType,
			args: {
				id: { type: new GraphQLNonNull(GraphQLString)},
				firstName: { type: GraphQLString },
				age: { type: GraphQLInt },
				companyId: { type: GraphQLString }
			},
			resolve(parentValue, args) {
				return axios.patch(`http://localhost:3000/users/${args.id}`, args)
					.then(res => res.data);
			}
		}
 * mutation: 

 mutation {
  editUser(id: "DBXhaPm", firstName: "Bob") {
    firstName
  }
}

# Section 5 - GraphQL Ecosystem

## Lecture 29 - Apollo vs Relay

* we would like to start integrating graphQL and integrate it with React Frontend
* graph is an evolving technology and libraries are changing
* to see the interface between clients and graphql server we investigate the http request message. the http request payload is: 
{"query":"{\n  user(id: \"40\") {\n    id,\n    firstName,\n    age\n  }\n}","variables":null,"operationName":null}
* the request header Content-Type: application/json
* the http response payload is {"data":{"user":{"id":"40","firstName":"Roulis","age":49}}}
* the React Client will  have a GraphQLCLient library implementing the GraphQLClient Query using http protocol
* there are 3 GraphQLClients
** Lokka: barebone, basic queries, mutations, simple caching
** Apollo Client: Produced by the team of Metero JS. good balance between feats and complexity
** Relay: Amazing performance for mobile. Insanely complex. by Facebook. Not for starups. too complex
* we will choose APollo client for the tradeoffs. but we dont use apollo backend graphql server. we use express-graphql.
* there is big difference on the syntax between express graphql and apollo. express uses the official syntax according to the Facebook spec.express-graphql is stable

# Section 6 - ClientSide GraphQL

## Lecture 31 - The Next App

* we git clone git@github.com:StephenGrider/Lyrical-GraphQL.git
* we cd Lyrical-GraphQL
* we get a ready express graphql server for testing
* backend offers a webpack stack for react client
* it uses a mongolab hosted mongodb server
* we setup our own db in mongolab and set a user
* we cp the connetion address to server.js mongo_uri string
* we npm isntall packages from boilerplate project
* we npm run dev the app and visit localhost:4000, also /graphql works as well

## Lecture 34 - Walk through the Schema

* we use graphiql docs to see the supported queries and mutations
* we use mutations to populate our mlab db. we check the data. we query the data

## Lecture 35 - Apollo CLient Setup

* we check the client side main js file which is a typical react boilerplate.
* Between our react app and the backend graphql server are two packages.
** the apollo provider that wraps the react app. is the glue logic between react app and the apollo store.
** the apollo store  that stands between frontend and backend. it is a repository of all data coming from the backend. apollo store doesnt know about react app. react provider does the comm.
* packages are already installed. we import 'apollo-client' and
extract ApolloProvider from 'react-apollo' in client side index.js
* we create a new APolloClient (Apollo Store!?!?!)
* we wrap our react jsx with apollo prover where we pass the client much like a redux store.
* apollo client constructor get a config object which we leave empty. if we deviate from defaults (like the uri for graphql server) we need to add configuration in the object
* apollo provider is a react component.

## Lecture 37 - GQL queries in React

* we add a SongList React component .
* we need to get data from the GQL server to the Component for rendering
* the flow is -> identify data required -> write query in grapiql for testing -> write query in React component file -> bond query w/ compoennt -> get the data
* our list needs only the titles of all songs.
* query tested on graphil
{ 
  songs {
  title
	}
}
* i cannot just throw the query in react component  code. its not javascript. i need a parser . like babel for jsx. 
* import graphql-taq library as gql in SongList component file
* out of class i define my query as gql`graphql query code`

const query = gql`
{ 
  songs {
  title
	}
}
`;

## Lecture 38 - Bonding Queries w/ Components

* to bond the query to react component we import graphql from react-apollo.
* we use redux style syntax to bond the query to component
export default graphql(query)(SongList);
* when component is rendered query is issued. when query completes component rerenders
* in react component render() func we consolelog this.props . we render the page and in browser console we see the props object. in .data in first render it has loading=true. in second render after query completes it has loading=false and a new atribute songs. with the query result. page desnot rerender all only what is changed


