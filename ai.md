# Some tools

- [workik javascript code generator](https://workik.com/javascript-code-generator)- produces quite nice express apps, e.g. I asked it for `create express application which can make requests for list of public github repositories for a given user. Use graphql, es modules  and typescript type annotations` and it produced me

```typescript

import express, { Request, Response } from 'express';
import { graphqlHTTP } from 'express-graphql';
import { buildSchema, GraphQLSchema, GraphQLObjectType, GraphQLString, GraphQLList } from 'graphql';
import axios from 'axios';

// Define a schema
const schema: GraphQLSchema = buildSchema(`
  type Repository {
    name: String
    description: String
    url: String
  }

  type Query {
    githubRepositories(username: String!): [Repository]
  }
`);

// Define resolvers
const root = {
  githubRepositories: async ({ username }: { username: string }) => {
    try {
      const response = await axios.post('https://api.github.com/graphql', {
        query: `
          query {
            user(login: "${username}") {
              repositories(first: 10, isFork: false) {
                nodes {
                  name
                  description
                  url
                }
              }
            }
          }
        `
      }, {
        headers: {
          Authorization: `Bearer YOUR_GITHUB_ACCESS_TOKEN`
        }
      });

      return response.data.data.user.repositories.nodes;
    } catch (error) {
      throw new Error('Failed to fetch GitHub repositories');
    }
  },
};

// Create an instance of Express
const app = express();

// Define a route for GraphQL endpoint
app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: root,
  graphiql: true, // Enable GraphiQL for testing
}));

// Start the server on port 3000
app.listen(3000, () => {
  console.log('Server is running on http://localhost:3000/graphql');
});
```
with comment
```
In this TypeScript example, we use ES modules and type annotations to create an Express server with GraphQL integration. We define a GraphQL schema with a `Repository` type and a `Query` type that includes a `githubRepositories` field to fetch repositories for a given GitHub username. We use Axios to make a POST request to the GitHub GraphQL API endpoint with a GraphQL query to fetch the user's repositories.

Make sure to replace `YOUR_GITHUB_ACCESS_TOKEN` with your actual GitHub personal access token to authenticate the request to the GitHub API.

To run this code, save it in a TypeScript file (e.g., `server.ts`) and compile it using a TypeScript compiler (e.g., `tsc server.ts`). Then run the compiled JavaScript file using the command `node server.js`. You can access the GraphQL endpoint at `http://localhost:3000/graphql` and test the `githubRepositories` query with a username parameter to fetch the list of repositories for that user from the GitHub API.
```
so as you can see you can quite nicely play with it to learn e.g. graphql.
When you visit `http://localhost:3000/graphql` then you can e.g. ask for (remember about the github token)
```
{
  githubRepositories(username:"lemkis") {
    name   
  }
}
```
