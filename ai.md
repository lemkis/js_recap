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

- [blackbox chat](https://www.blackbox.ai/chat/nYq7SRU) handles nicely e.g. curl request. E.g. when i asked for `create c++ application which fetches the list of pull requests for given public repository and given user` then i received (after small modifications
```cpp

#include <iostream>
#include <string>
#include <curl/curl.h>
#include <jsoncpp/json/json.h>
#include <print>

static size_t write_memory(void *ptr, size_t size, size_t nmemb, void *data)
{
    std::string &response = *static_cast<std::string*>(data);
    response = std::string(static_cast<const char *>(ptr), size * nmemb);
    return size * nmemb;
}

int main() {
    // Set the GitHub API endpoint and authentication token
    std::string endpoint = "https://api.github.com/repos/";
    std::string username = "IS-UMK"; // Replace with the username
    std::string repoName = "lemkis"; // Replace with the repository name
    std::string token = "GITHUB_TOKEN"; // Replace with a valid GitHub API
    token

    // Construct the API request URL
    std::string url = endpoint + username + "/" + repoName + "/pulls";

    // Set up the CURL request
    CURL *curl;
    CURLcode res;
    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();
        std::string response{};
    if(curl) {
         curl_easy_setopt(curl, CURLOPT_USERAGENT, "Dark Secret Ninja/1.0");

        curl_easy_setopt(curl, CURLOPT_URL, url.c_str());
        curl_slist *list = NULL;
        std::string authorization{std::string{"Authorization: "} + token};
         list = curl_slist_append(list, authorization.c_str());
         std::string content_type{"Content-Type: application/json"};
         list = curl_slist_append(list, content_type.c_str());
        curl_easy_setopt(curl, CURLOPT_HTTPHEADER, list);
        curl_easy_setopt(curl, CURLOPT_WRITEFUNCTION, write_memory);
        curl_easy_setopt(curl, CURLOPT_WRITEDATA, &response);
        res = curl_easy_perform(curl);
        if(res!= CURLE_OK) {
            std::cerr << "cURL error: " << curl_easy_strerror(res) <<
            std::endl;
        }
        curl_easy_cleanup(curl);
    }
    curl_global_cleanup();
    std::print("response = {}\n\n", response);
    // Parse the JSON response
    Json::Value root;
    Json::Reader reader;
    bool parsingSuccessful = reader.parse(response, root);
    if (!parsingSuccessful) {
        std::cerr << "Error parsing JSON response" << std::endl;
        return 1;
    }

    // Extract the pull request list
    Json::Value pulls = root["pulls"];
    for (int i = 0; i < pulls.size(); i++) {
        std::cout << "Pull Request " << i + 1 << ":" << std::endl;
        std::cout << "  Title: " << pulls[i]["title"] << std::endl;
        std::cout << "  URL: " << pulls[i]["html_url"] << std::endl;
        std::cout << "  State: " << pulls[i]["state"] << std::endl;
        std::cout << std::endl;
    }

    return 0;
}

```
Here you can find the graphql version of above
```cpp

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
