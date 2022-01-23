# fakebeard-amplify
Repo to store notes for amplify

Notes when following along with [Amplify tutorial](https://docs.amplify.aws/start/getting-started/installation/q/integration/js/)

# Prerequisites
Create a free tier or payed AWS account if you dont have one.

npm install -g @aws-amplify/cli

amplify configure

Create an IAM user for Amplify

Specify the AWS Region
? region:  # Your preferred region
Specify the username of the new IAM user:
? user name:  # User name for Amplify IAM user
Complete the user creation using the AWS console

Once the user is created, Amplify CLI will ask you to provide the accessKeyId and the secretAccessKey to connect Amplify CLI with your newly created IAM user.

Enter the access key of the newly created user:
? accessKeyId:  # YOUR_ACCESS_KEY_ID
? secretAccessKey:  # YOUR_SECRET_ACCESS_KEY
This would update/create the AWS Profile in your local machine
? Profile Name:  # (default)
Note: You WONT see the secretAccessKey when you paste it in.

# Create the app structure
amplify-js-app
├── index.html
├── package.json
├── src
│   └── app.js
└── webpack.config

add to package.json
{
  "name": "amplify-js-app",
  "version": "1.0.0",
  "description": "Amplify JavaScript Example",
  "dependencies": {
    "aws-amplify": "latest"
  },
  "devDependencies": {
    "copy-webpack-plugin": "^6.1.0",
    "webpack": "^4.46.0",
    "webpack-cli": "^4.9.1",
    "webpack-dev-server": "^4.4.0"
  },
  "scripts": {
    "start": "webpack && webpack-dev-server --mode development",
    "build": "webpack"
  }
}

run npm install

copy this into index.html

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Amplify Framework</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style>
      html, body { font-family: "Amazon Ember", "Helvetica", "sans-serif"; margin: 0; }
      a { color: #ff9900; }
      h1 { font-weight: 300; }
      hr { height: 1px; background: lightgray; border: none; }
      .app { width: 100%; }
      .app-header { color: white; text-align: center; background: linear-gradient(30deg, #f90 55%, #ffc300); width: 100%; margin: 0 0 1em 0; padding: 3em 0 3em 0; box-shadow: 1px 2px 4px rgba(0, 0, 0, 0.3); }
      .app-logo { width: 126px; margin: 0 auto; }
      .app-body { width: 400px; margin: 0 auto; text-align: center; }
      .app-body button { background-color: #ff9900; font-size: 14px; color: white; text-transform: uppercase; padding: 1em; border: none; }
      .app-body button:hover { opacity: 0.8; }
    </style>
  </head>

  <body>
    <div class="app">
      <div class="app-header">
        <div class="app-logo">
          <img
            src="https://aws-amplify.github.io/images/Logos/Amplify-Logo-White.svg"
            alt="AWS Amplify"
          />
        </div>
        <h1>Welcome to the Amplify Framework</h1>
      </div>
      <div class="app-body">
        <h1>Mutation Results</h1>
        <button id="MutationEventButton">Add data</button>
        <div id="MutationResult"></div>
        <hr />

        <h1>Query Results</h1>
        <div id="QueryResult"></div>
        <hr />

        <h1>Subscription Results</h1>
        <div id="SubscriptionResult"></div>
      </div>
    </div>
    <script src="main.bundle.js"></script>
  </body>
</html>

run npm start

browse to localhost:8080 (http)

run amplify init

accept the defaults and press enter

when asked about authentication use profile and choose your profile

Review the note below and then explore the directories

When you initialize a new Amplify project, a few things happen:

It creates a top level directory called amplify that stores your backend definition. During the tutorial you'll add capabilities such as authentication, GraphQL API, storage, and set up authorization rules for the API. As you add features, the amplify folder will grow with infrastructure-as-code templates that define your backend stack. Infrastructure-as-code is a best practice way to create a replicable backend stack.
It creates a file called aws-exports.js in the src directory that holds all the configuration for the services you create with Amplify. This is how the Amplify client is able to get the necessary information about your backend services.
It modifies the .gitignore file, adding some generated files to the ignore list
A cloud project is created for you in the AWS Amplify Console that can be accessed by running amplify console. The Console provides a list of backend environments, deep links to provisioned resources per Amplify category, status of recent deployments, and instructions on how to promote, clone, pull, and delete backend resources

run amplify console and choose AWS console to see your project

# connect api and database to the app
amplify add api
choose Graphql and notice schema now

type Todo @model {
  id: ID!
  name: String!
  description: String
}

amplify push

? Do you want to generate code for your newly created GraphQL API (Yes)
? Choose the code generation language target (javascript)
? Enter the file name pattern of graphql queries, mutations and subscriptions (src/graphql/**/*.js)
? Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions (Yes)
? Enter maximum statement depth [increase from default if your schema is deeply nested] (2)

amplify status

amplify console api to view the GraphQL database

update front end

update src/app.js with code below

import Amplify, { API, graphqlOperation } from "aws-amplify";

import awsconfig from "./aws-exports";
import { createTodo } from "./graphql/mutations";

Amplify.configure(awsconfig);

async function createNewTodo() {
  const todo = {
    name: "Use AppSync",
    description: `Realtime and Offline (${new Date().toLocaleString()})`,
  };

  return await API.graphql(graphqlOperation(createTodo, { input: todo }));
}

const MutationButton = document.getElementById("MutationEventButton");
const MutationResult = document.getElementById("MutationResult");

MutationButton.addEventListener("click", (evt) => {
  createNewTodo().then((evt) => {
    MutationResult.innerHTML += `<p>${evt.data.createTodo.name} - ${evt.data.createTodo.description}</p>`;
  });
});

run npm start

finish  by updating the app.js as below

import Amplify, { API, graphqlOperation } from "aws-amplify";

 import awsconfig from "./aws-exports";
 import { createTodo } from "./graphql/mutations";
 import { listTodos } from "./graphql/queries";
 import { onCreateTodo } from "./graphql/subscriptions";

 Amplify.configure(awsconfig);

 async function createNewTodo() {
   const todo = {
     name: "Use AppSync",
     description: `Realtime and Offline (${new Date().toLocaleString()})`,
   };

   return await API.graphql(graphqlOperation(createTodo, { input: todo }));
 }

 async function getData() {
   API.graphql(graphqlOperation(listTodos)).then((evt) => {
     evt.data.listTodos.items.map((todo, i) => {
       QueryResult.innerHTML += `<p>${todo.name} - ${todo.description}</p>`;
     });
   });
 }

 const MutationButton = document.getElementById("MutationEventButton");
 const MutationResult = document.getElementById("MutationResult");
 const QueryResult = document.getElementById("QueryResult");
 const SubscriptionResult = document.getElementById("SubscriptionResult");

 MutationButton.addEventListener("click", (evt) => {
   createNewTodo().then((evt) => {
     MutationResult.innerHTML += `<p>${evt.data.createTodo.name} - ${evt.data.createTodo.description}</p>`;
   });
 });

API.graphql(graphqlOperation(onCreateTodo)).subscribe({
  next: (evt) => {
    const todo = evt.value.data.onCreateTodo;
    SubscriptionResult.innerHTML += `<p>${todo.name} - ${todo.description}</p>`;
  },
});

 getData();

 #  add hosting
 amplify add hosting

 hit enter then choose manual hosting

 amplify publish

 amplify delete to clean up
