# Part 6: A working GraphQL Server

Let's build a real server which can parse GraphQL queries sent over HTTP.

First, install `babel` (to use ES2015 awesomeness), `restify` and of course, `graphql`

```
npm i babel graphql restify
```

Create three files- `index.js`, `server.js` and `schema.js`.

```
touch index.js server.js schema.js
```

`index.js` file will be our entry point. We'll just register babel here so that the rest of our application can use ES2015 code.

```
//index.js
require('babel/register');
require('./server');
```

Let's build our schema now. We'll use dummy data here for the sake of brevity, but as you'll see using something like `Bookshelf.js`/`Knexjs`/`Sequelize` will be trivial. We'll pass the ID parameter with our query like `{contributor (id: "1")}` to get the GraphQL contributor's github username.

Here's the code in `schema.js`:

```
//schema.js
import {
  graphql,
  GraphQLSchema,
  GraphQLObjectType,
  GraphQLString
} from 'graphql';

let dummyData = {
  '1': 'leebyron',
  '2': 'enaqx',
  '3': 'schrockn',
  '4': 'andimarek'
};

export var schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'RootQueryType',
    fields: {
      contributor: {
        type: GraphQLString,
        args: {
          id: { type: GraphQLString }
        },
        //Using Destructuring of ES2015 to assign value to id
        resolve: (root, {id}) => {
          return dummyData[id];
        }
      }
    }
  })
});
```
You are probably already familiar with `koa`/`express`/`restify`.  This part is easiest for you then:

```
//server.js
import restify from 'restify';
import { graphql } from 'graphql';
import { schema } from './schema.js';

var server = restify.createServer({
  name: 'GraphQL Demo'
});

server.use(restify.acceptParser(server.acceptable));
server.use(restify.queryParser());
server.use(restify.bodyParser());

server.post('/', (req, res, next) => {
  graphql(schema, req.body).then((result) => {
    res.send(result);
  });
});

server.get('/', (req, res, next) => {
  let instruction = 'POST GraphQL queries to' + server.url + '. Sample query: {contributor (id: "1")}';
  res.send(instruction);
});

server.listen(process.env.PORT || 8080, () => {
  console.log('%s listening at %s', server.name, server.url);
});
```

In order to run the server:

```
node index.js
```

The server will start at http://localhost:8080 address.

Now we can use `Postman` or just simple `CURL` to test if it is working:

```
curl -XPOST -d '{contributor (id: "1")}' http://localhost:8080/
```

If you got `{"data":{"contributor":"leebyron"}}` in return, congratulations for making it this far!
