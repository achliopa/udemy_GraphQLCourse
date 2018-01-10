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

# Section 7 - Gotchas w/ Queries in React

## Lecture 39 - Handling Pending Queries

* grapgql library creates the data object in the component props
* our job is to render a list of songs in the component
* we add a hlper method to extract songs from props and inject it in jsx code. but when we render songs is undefined.  its not yet available, query is not complete.
* we use the loading flag.
* we add the id of the song in the query to satisfy react unique key requirement in list items
* we add some materialcss styling

## Lecture 41 - Architecture Review

* The data flow detailed is GraphQl Server -> Apollo Store -> Apollo Provider -> Root React Component -> React Component w/Query attached -> React Subcomponents
* Usually queries are centralized in a components and the data returned are oassed to child presentational components
* same approach as reedux containers
* we will use react router to navigate between components

## Lecture 42 - Add React Router

* we add several helpers from react-router.
* we use hashHistory as graphQL server uses hashHistory instead of browserhistory (?!?)
* we put IndexROute=SongList in Route=App.. Therefore in App component contstructor we pass children components as props. If Route decides to show IndexRoute component this will be passed as a prop "children"
* we use component state to track changes in form

# Section 8 - Frontend Mutations

## Lecture 44 - Mutations in React

* we add onSubmit event handler in form and we add the workaround  of react binding it to this object. 
* onSubmit gets passed the submit event and we prtevent default behaviour to avoid page refresh.
* we need to add a mutation to persist the data in the backend using graphQL
* following the aforementioned development flow we test our mutation in graphiql
* then we ad graphql and gql in the react component. we add the mutation script in the gql like queries using backquotes and we wrap the component with graphql passing the mutation. the problem we are facing it how to pass the title in the gql from the component state object )this.state.title)

## Lecture 45 - Query Params

* we need a way to pass param into mutations. graphql has provision for this into its standar spec.
* the way to pass params is the same for mutations and queries. we use a special syntax. we name the mutation or query passing the parameter as an argument (sort of function style) : the syntax (for mutation) is: mutation MutationName($param: ParamType) {
	addSOng(title: $param)
}
* we pass the query variable as
{
	"param": "Some Value"
}
* graphiql supports it so we can test it.

## Lecture 46 - Query Variables in React

* we modify the mutation script in gql to be ready to get the query variable
mutation AddSong($title: String){
  addSong(title: $title) {
    title
  }
}
* in the graphql helper that wraps the component we pass mutation
* we log the props in onSubmint event handler and we trigger it in browser. we see that a mutate function has been added to props. this is the way to trigger the mutation from our component. we see that it accepts an arguments object. this we will use to pass the title query variable. 
* the syntax is like:

		this.props.mutate({
			variables: {
				title: this.state.title
			}
		});

## Lecture 46 - navigating on Successful Mutation

* we add React Links to navigate back and forth the pages
* we need to make sure that we dont navigate before the mutation finishes. graphql queries throw promises so we add then after mutation call to redirect our user back to '/' on suceessful completion. this is done by hashHistory.push('/') method

## Lecture 48 - Troubleshoot List Fetching

* we see that when we forcefully redirect after addSong mutation the songlist is not updated. we need to refresh the songlist page (an rerun the query) to see the new song.
* this is a apollo behaviour. when a query is run apollostore is populated with query results whicha are structured. when we mutate and our mutation affects the data. maybe more data are added in the store but are not automatically included in the data structures bound to the react components. 
* so we rerun the query to fix this (Non elegant)

## Lecture 49 - Refetch Queries

* apollo addresses that by adding another attribute in the mutate function passed in the props. apart from variables we can pass refetchQueries: whish takes an array of objects of the type { query:, variables: } query takes a gql script and variables query variables like the mutation. the syntax for us is 

this.props.mutate({
			variables: { title: this.state.title },
			refetchQueries: [{ query }]
		}).then(()=> hashHistory.push('/'));

## Lecture 50 - Deletion Mutation

* our script tested in graphiql is: 

mutation DeleteSong($id: ID) {
  deleteSong(id:$id) {
    id
  }
}

* we add a variable in SongList component wrapping it up in gql format
* we bind it to the component by adding one more graphql(mutation) helper which wraps the graphql(query)(SongList). apollo does not natively support bing mutlitple graphql operations to the component so this is a workaround
* we add an icon to li song list element, we set an onClick handler. in the handler we call the mutate() function from props. we need to refetchquery to update the list after the mutation like before.
* finished mutate call: 
this.props.mutate({ variables: { id }})
	}

# Section 9 - Automatic Data Caching

## Lecture 53 - Refetching a query

* we use an alternative way to refetch data after mutation by chaining
this.props.data.refetch() in the promise return callback arrow func
* we prefer this approach as query is already associated with the component so there is no need to re assosiate it by passing it in the object paramater

## Lecture 54 - CSS Styling

* we want to style the delete icon. we add a stylesheet and we import it.
* we use flexbot

## Lecture 55 - Show a Particual Song (SongDetail React Component)

* we add a class based component, import it in index.js and add it to the Routes component as ROute. we use restful routing naming conv. wildcard :id is passed in the Component in props.

## Lecture 56 - Fetching Indiv. Records

* we test our query in graphiql
* our query is 

query SongQuery($id: ID!){ 
  song(id: $id) {
    id
  	title
	}
}

* ! after Type says that it is required to provide this argument.
* we create a new query file wrapping it up in gql and import it in React Detail Comp.

# Section 10 - React router & GraphQL

## Lecture 57 - Integration React Router with GraphQL

* wildcard :id from to property of LInk / react component passes in the linked component( ReactDetail) in the this.props.params object as a string with key id
* our query needs some query variable to be passed into. unlike the mutation with is manually called query is automatic called. 
* the solution is to add an object as a second argument to the graphql component wrapper after the query itself. this object takes key option: and passes an arrow function which takes props from graphql and return a query variable extracting the id from the graphql props. graphql intercepts the flow between parent component and child getting the component props and then passing them to the child (ReactDetails). the syntax is like: 

export default graphql(query, {
	options: (props) => { return { variables: { id: props.params.id } } }
})(SongDetail);

## Lecture 58 - Watching for Data

* as usual query rfesult is passed in the components props object. we use them to render the title on screen
* we add a back link and make songs in list clickable(react router + sytling)

## Lecture 60 -  Lyric Creation Form

* we follow the development flow like song create, two points of attention are passing props from component to child component. also reseting the ste using this.setState to clear input

## Lecture 63 - Showing a List of Lyrics

* we create anew class based react component named LyricList and we insert it to SongDetail
* we have two possible paths . one to add a query to fetch lyrics from backend and then list them. the second is to modify the existing fetchSong adding the lyrics to it. we go for the second solution
* we modify the query fetchSong adding lyrics{id, title}
* we pass song.lyrics as a prop from SongDetail to LyricList
 in the same way like SOngList we render the list of lyrics in LyricList with array map funtion
 * int he sonddetail page when we create alyric its not automaticly shown in the list.only with page refresh.we could refetch the Query after mutation like in SOngCreate but we will do it different this tume.

 ## Lecture 66 - Identifying Records

 * Apollo store keeps internal records of data specified in graphql. these data are added removed and edited but apollo has no knowledge of the ids of the data he has in its containers. 
 * even thogh a mutation in its reply shows the new state of a data container  apollo doesn know what data are they to trigger react rerender.
 * to solve this we need to add ids to the container items in the apollo store. this is done in apollo configuration. then when data change apollo will trigger rerender to the react components that use these data.
 * we pass 	a transforming arrow function dataIdFromObject: o => o.id in the apolloclient config object
 * this adds id to all objects in store.
 to make it work we need to use ids to all params and results od queries and mutations. so we modify AddLyricToSong mutation and it workds
 * we can fix song list in that way modifying the add song mutation adding id. we beed to remove refetchQueries from mutate() params
 * we add thunb_up icon to lyric list items and an onClick event handler where we pass the lytric.id to increase the like counter

 # Section 11 - More CLient Side mutations

 ## Lecture 68 - The like mutation

 * we form the mutation in graphiql and test it

 mutation LikeLyric($id: ID!){ 
  likeLyric(id: $id) {
  	id
    likes
  }
}

* we follow the drill adding mutation lyric list. also we modify fetchsong query adding likes to return values and weextract this value along the others in lyric list
* we wrap thumbs and likes in a div and style them with flexbox

## Lecture 71 - Optimistic UI Updates

* to give an instant update making our site appear realtime we use optimistic responses feature from apollo. what actually happens is:
we call mutation -> guess response -> update ui .... get response -> update ui
* to implement it in the configuration object we pass in the mutate method we add an extra key optimisticResponse: which contains the mutation with the prediction on the value change. to acheive that we need to add likes as a second argument to the event handler.
* syntax: 

			optimisticResponse: {
				__typename: 'Mutation',
				likeLyric: {
					id,
					__typename: 'LyricType',
					likes: likes + 1
				}
			}

## Lecture 73 - BugFix

* LyricCreate mutation doesnt ask for lyrics likes. but likes are requested in the LyricList props. this gives an error when we create a lyric in a song that has already other rendered. 
* to fix this we add likes in the lyric returned in the mutation.

# Section 12 - Building from Scratch  an End to End GraphQL Application

## Lecture 75 - App Overview

* we will implement authentication with graphql
** mutiple pages -> react router
** user data storage -> mongo db
** authentication -> passport JS
** restrict access to data -> ???
** input validation -> ???
integrate passport to graphql -> ????

## Lecture 77 - Boilerplate Setup

* git clone git@github.com:StephenGrider/auth-graphql-starter.git
* npm install
* complete boilerplate
** client react root comp
** server mongo conn
** server graphql setup
** server authent. setup (session token)
** user.js mongoose model w/ authentication
** server/auth.js passport authentication service + patches to glue it to graphql (read the comments)

## Lecture 78 - Authentication approach

* 2 approaches: Coupled and Decoupled
** in coupled Appoach graphql is used as a thin layer.
** it takes all requests (queries/mutations). if they are 
authentication related it parses them to passport and then forwards the reply making it graphql compatible
** in that way graphql is used as it meant to be . abstraction layer
** in decoupled approach passport and graphql are separate. passport handles authentications and once user is authenticated all requests are handled by graphql
** app is split in 2 parts in decoupled , easier to implement. no interaction between them, no integration issues.
* we will go with coupled for learning purposes. the hard way.


## Lecture 79 - Mlab setup

* the usual drill. we put the db path in MONGO_URI in backend server.js adding an admin user 

## Lecture 80 - User Type

* before we move on with the implementation we need to define schema, types (User) and mutations (signup, login , logout)
* to define UserType we check our mongoose user model. we dont expose the original password in anycase in graphql. so we dont put it in UserType

## Lecture 81 - Signup Mutation

* we wont place any authentication logic in mutations. graphql is an abstraction thin layer. the logic will be handled by backend helper functions/objects
* we can do password confirmation of signup entrirely on the frontend with validation.
* in our resolve function of signup we have 3rd parameter called request. this is the http request object from express.
* from the resolve function we call the alreday implemented signup method from the auth service (passportjs) in auth.js
* signup is async returning a promise . we return it to propagate it to graphql resolve() function which is asynchronous as well
* we form our mutation in graphiql and test it

mutation {
  signup(email: "test@test.com", password: "password") {
    email
  }
}

* the reply is successful 

{
  "data": {
    "signup": {
      "email": "test@test.com"
    }
  }
}

* we check in mlab and see the collection being populated

## Lecture 84 - The logout Mutation

*  according to passport logout() doc the user the function will logout is inside the express request object in user attribute, so we dont need any search in backend. but we need to extract user first call backend and then return user to satisfy graphql spec regarding return something after an operation

mutation {
  logout{
    email
  }
}

* reply (as user is in a session toke because of passportjs)

{
  "data": {
    "logout": {
      "email": "test@test.com"
    }
  }
}

* login mutation is almost the same like signup

## Lecture 86 - Checking Authentication Status

* we implement a user field in root_type to be able to test authentication using graphql query on user. the resolve function makes use of express request object which contians user object if loged in.
* we test it in graphiql

{
  user {
    email
  }
}

{
  "data": {
    "user": {
      "email": "test@test.com"
    }
  }
}

* to test authentication we run logout mutation and we rerun the query. the reply is now

{
  "data": {
    "user": null
  }
}

# Section 13 - Moving Client Side

## Lecture 87 - Client Side Setup

* react app. start react router and apollo client in index.js
* we initalize apollo client adding the dataIdFromObject propery to identify records in store.
* we id id field in our usertye in backend as requirement for record identification
* we wrap our root jsx in ApolloProvider React Component passing the apollo client (Redux style)
* we structure our app with react router on top wraping Route / (App) that wraping header componet and the component that changes depending on the children prop passed to it.

* when header is loaded depending on the auth status (graphql root query type)
we show the logout or login buttons. we form our query in graphiql and we place it in gql in a separate file. we folow the drill binding the query to the header compoennt and we console log the result (this.props.data). we see that user is null although we loged in in /graphql  graphiql client.

## Lecture 90 - Including Cookies in graphql requests

* according to graphiql we are authenicated. according to apollo-react e are not.
* graphiql by default attach queries to request. containing the cookie.
* in graphql queries sent through the app , apollo by default does not attach the cookies.
* passport is cookie based authentication
* to solve this we make use of apollo-client createNetworkInterface method, with defines a custom network interface.

const createNetworkInterface = createNetworkInterface({
	uri: '/graphql',
	opts: {
		credentials: 'same-origin'
	}
});

* the config object sets uri  and a credentials param.
* we insert the networkInterface to ApolloClient config object and test again with success the authentication query

## Lecture 91 - Authentication State

* we implement the header adding some condition logic based on the query result
* we add Links and event handlers to implement authenticcation mutations and react routing. we implement logout mutation using the normal workflow. however we need to rerender to change header state after mutation completes.

## Lecture 94 - Automatic Component Renderers

* we add refetchQueries in the mutation params object to rerun the auth query trigerring a rerender as react rerenders after a query (refetchQueries: [{ query }])

## Lecture 95 - Login Form Design

* Login Form and SignUp Form are almost identical they will wrap a common react component the AuthForm at render()
* loginform is passed in the router as a route inside a route (APP), this chains the to properties (assembling the route path) and passed the component for rendering to the parrent component App as a childrens object in props.
* AuthForm has state to handle the inputs
* we make login mutation into a parametrical one and we test it in graphiql
* we bind it to the LoginForm and add the mutate() function into onSubmit event handler. we pass this function as a callback into AuthForm props.
* in AuthFOrm we decalre its own onSubmit event handler where we call the callback onSubmit passed through react component props.
* we need to refetch query to trigger a rerender to header so that it reflects the change in auth status.

## Lecture 100 - Error handling in GraphQl

* mutation replies with error message when inserting wrong credentials. this reply is injected in component props at props.data.errors[0].message. we want to extract it and show it on the form to the user
* we dont use the props but we go straight to the mutation to solve it the graphql way. we know mutations are async methods returning a promise. we look for errors so we chain a catch(res => {}) in there we launch debugger; to look into this error.
we see that message is in res.graphQLErrors[0].message
* if we have multiple errorswe get more entres in the array so we use array.map method to capture them all
* to communicate the rror list from the mutation back to the componenbt we use the same way we used to pass query variables the other way. component state
* we initialize it as an empty array as map throws error on null
* we populate the array in mutation catch function pass it to AuthForm thast does the rendering of the form as a prop

## Lecture 102 - Signup Form

* i copy paste loginform changing only the mutation script and the component name
