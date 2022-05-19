
#### SIMPLE TO-DO APPLICATION ON MERN WEB STACK

##### MERN Web stack consists of following components:

* MongoDB: A document-based, No-SQL database used to store application data in a form of documents.

* ExpressJS: A server side Web Application framework for Node.js.

* ReactJS: A frontend framework developed by Facebook. It is based on JavaScript, used to build User Interface (UI) components.

* Node.js: A JavaScript runtime environment. It is used to run JavaScript on a machine rather than in a browser.

<img width="1135" alt="Screenshot 2022-05-19 at 11 12 00 PM" src="https://user-images.githubusercontent.com/105562242/169363758-e09250fe-a446-432c-8747-ff6874989b60.png">

##### As shown on the illustration above, a user interacts with the ReactJS UI components at the application front-end residing in the browser. This frontend is served by the application backend residing in a server, through ExpressJS running on top of NodeJS.

##### Any interaction that causes a data change request is sent to the NodeJS based Express server, which grabs data from the MongoDB database if required, and returns the data to the frontend of the application, which is then presented to the user.

#### STEP 1 – BACKEND CONFIGURATION

* Update ubuntu
```
sudo apt update
```
* Upgrade ubuntu
```
sudo apt upgrade
```
* Lets get the location of Node.js software from Ubuntu repositories.

```
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```
* Install Node.js on the server
```
sudo apt-get install -y nodejs
```
* Note: The command above installs both nodejs and npm. NPM is a package manager for Node like apt for Ubuntu, it is used to install Node modules & packages and to manage dependency conflicts.

* Verify the node installation with the command below
```
node -v 
npm -v 
```
<img width="888" alt="Screenshot 2022-05-19 at 11 20 36 PM" src="https://user-images.githubusercontent.com/105562242/169365229-ef5e6b6b-d9d2-4579-b88c-38e5e07d006b.png">


* Application Code Setup
* Create a new directory for your To-Do project:
```
mkdir Todo-list
```
* Now change your current directory to the newly created one:

* Next, we will use the command npm init to initialise project, so that a new file named package.json will be created. This file will normally contain information about  application and the dependencies that it needs to run. Follow the prompts after running the command. we can press Enter several times to accept default values, then accept to write out the package.json file by typing yes.

```
npm init
```
<img width="1114" alt="Screenshot 2022-05-19 at 11 29 48 PM" src="https://user-images.githubusercontent.com/105562242/169367513-c8c07e8b-1f73-4d11-a49e-04228c7365eb.png">

#### INSTALL EXPRESSJS

##### Install ExpressJS
* Remember that Express is a framework for Node.js, therefore a lot of things developers would have programmed is already taken care of out of the box. Therefore it simplifies development, and abstracts a lot of low level details. For example, Express helps to define routes of application based on HTTP methods and URLs.
* To use express, install it using npm:
```
npm install express
```
* Now create a file index.js with the command below
```
touch index.js
```
* Install the dotenv module
```
npm install dotenv
```
* Open the index.js file with the command below
```
vim index.js
```
* Type the code below into it and save

```
const express = require('express');
require('dotenv').config();

const app = express();

const port = process.env.PORT || 5000;

app.use((req, res, next) => {
res.header("Access-Control-Allow-Origin", "\*");
res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
next();
});

app.use((req, res, next) => {
res.send('Welcome to Express');
});

app.listen(port, () => {
console.log(`Server running on port ${port}`)
});
```

* Notice that we have specified to use port 5000 in the code. This will be required later when we go on the browser.Use :w to save in vim and use :qa to exit vim
* Now it is time to start our server to see if it works. Open your terminal in the same directory as your index.js file and type:
```
node index.js
```
* If every thing goes well, you should see Server running on port 5000 in your terminal.
* Now we need to open below ports in EC2 Security Groups. 

<img width="1025" alt="Screenshot 2022-05-19 at 11 44 45 PM" src="https://user-images.githubusercontent.com/105562242/169371652-a75dc1f5-a436-4900-ad99-246869c346c5.png">

* Open up  browser and try to access your server’s Public IP or Public DNS name followed by port 5000:
```
http://<PublicIP-or-PublicDNS>:5000
```
##### Routes

##### There are three actions that our To-Do application needs to be able to do:

* Create a new task
* Display list of all tasks
* Delete a completed task
* Each task will be associated with some particular endpoint and will use different standard HTTP request methods: POST, GET, DELETE.

* For each task, we need to create routes that will define various endpoints that the To-do app will depend on. So let us create a folder routes

```
mkdir routes
```
* Change directory to routes folder.
```
cd routes
```
* Now, create a file api.js with the command below
```
touch api.js
```
* Open the file with the command below
```
vim api.js
```
* Copy below code in the file.

<img width="1133" alt="Screenshot 2022-05-19 at 11 51 16 PM" src="https://user-images.githubusercontent.com/105562242/169372630-00d93da7-f5e6-4591-b2d9-861a15c24062.png">

#### MODELS
##### Now comes the interesting part, since the app is going to make use of Mongodb which is a NoSQL database, we need to create a model.

##### A model is at the heart of JavaScript based applications, and it is what makes it interactive.

##### We will also use models to define the database schema . This is important so that we will be able to define the fields stored in each Mongodb document. (Seems like a lot of information, but not to worry, everything will become clear to you over time. I promise!!!)

##### In essence, the Schema is a blueprint of how the database will be constructed, including other data fields that may not be required to be stored in the database. These are known as virtual properties

##### To create a Schema and a model, install mongoose which is a Node.js package that makes working with mongodb easier.

##### Change directory back Todo folder with cd .. and install Mongoose
```
npm install mongoose
```
##### Create a new folder models :
```
mkdir models
```
##### Change directory into the newly created ‘models’ folder with
```
cd models
```
##### Inside the models folder, create a file and name it todo.js
```
touch todo.js
```
##### Tip: All three commands above can be defined in one line to be executed consequently with help of && operator, like this:
```
mkdir models && cd models && touch todo.js
```
##### Open the file created with vim todo.js then paste the code below in the file:

<img width="975" alt="Screenshot 2022-05-20 at 12 02 27 AM" src="https://user-images.githubusercontent.com/105562242/169374484-e9582fa9-06d5-4c41-b920-26b4ae174be6.png">

##### Now we need to update our routes from the file api.js in ‘routes’ directory to make use of the new model.

##### In Routes directory, open api.js with vim api.js, delete the code inside with :%d command and paste there code below into it then save and exit
<img width="825" alt="Screenshot 2022-05-20 at 12 05 06 AM" src="https://user-images.githubusercontent.com/105562242/169375031-21088926-a6d0-4129-845d-25f4f8a0b160.png">

#### MONGODB DATABASE
##### MongoDB Database
##### We need a database where we will store our data. For this we will make use of mLab. mLab provides MongoDB database as a service solution (DBaaS), so to make life easy, you will need to sign up for a shared clusters free account, which is ideal for our use case. Sign up here. Follow the sign up process, select AWS as the cloud provider, and choose a region near you.

<img width="1699" alt="Screenshot 2022-05-20 at 12 10 56 AM" src="https://user-images.githubusercontent.com/105562242/169376183-6dc05e26-d4e1-48c2-ba34-4dda82104cdd.png">

##### Now we need to update the index.js to reflect the use of .env so that Node.js can connect to the database.

##### Simply delete existing content in the file, and update it with the entire code below.

##### To do that using vim, follow below steps

* Open the file with vim index.js
* Press esc
* Type :
* Type %d
* Hit ‘Enter’
* The entire content will be deleted, then,

* Press i to enter the insert mode in vim
* Now, paste the entire code below in the file.

<img width="1409" alt="Screenshot 2022-05-20 at 12 16 32 AM" src="https://user-images.githubusercontent.com/105562242/169377080-8a59f24f-3992-4e4e-9461-bd7a1a7d2cbf.png">

* Using environment variables to store information is considered more secure and best practice to separate configuration and secret data from the application, instead of writing connection strings directly inside the index.js application file.

* Start  server using the command:
```
node index.js
```
##### We shall see a message ‘Database connected successfully’, if so – we have our backend configured. Now we are going to test it.

* Testing Backend Code without Frontend using RESTful API
So far we have written backend part of our To-Do application, and configured a database, but we do not have a frontend UI yet. We need ReactJS code to achieve that. But during development, we will need a way to test our code using RESTfulL API. Therefore, we will need to make use of some API development client to test our code. In this project, we will use Postman to test our API.


<img width="885" alt="Screenshot 2022-05-20 at 12 45 10 AM" src="https://user-images.githubusercontent.com/105562242/169385013-a894bfe0-657a-46ac-b43e-f9cbe56e3770.png">


<img width="876" alt="Screenshot 2022-05-20 at 12 46 18 AM" src="https://user-images.githubusercontent.com/105562242/169385168-ab441cf1-79fc-48cb-afe9-f6cd1985e5c7.png">

<img width="856" alt="Screenshot 2022-05-20 at 12 49 27 AM" src="https://user-images.githubusercontent.com/105562242/169385720-0288324a-36b9-42e5-a81b-1947ff9e6a3e.png">


<img width="1207" alt="Screenshot 2022-05-20 at 12 50 10 AM" src="https://user-images.githubusercontent.com/105562242/169385819-1628d984-8d28-43ad-b68d-ba0caeccc952.png">


#### Step 2 – Frontend creation
##### Since we are done with the functionality we want from our backend and API, it is time to create a user interface for a Web client (browser) to interact with the application via API. To start out with the frontend of the To-do app, we will use the create-react-app command to scaffold our app.

##### In the same root directory as your backend code, which is the Todo directory, run:
```
 npx create-react-app client
 ```
##### This will create a new folder in your Todo directory called client, where you will add all the react code.

##### Running a React App
 * Before testing the react app, there are some dependencies that need to be installed.

 * Install concurrently. It is used to run more than one command simultaneously from the same terminal window.
 ```
npm install concurrently --save-dev
```
 * Install nodemon. It is used to run and monitor the server. If there is any change in the server code, nodemon will restart it automatically and load the new changes.
 ```
npm install nodemon --save-dev
``` 
* In Todo folder open the package.json file.  replace scripts portion with the code below.
```
"scripts": {
"start": "node index.js",
"start-watch": "nodemon index.js",
"dev": "concurrently \"npm run start-watch\" \"cd client && npm start\""
},
```
##### Configure Proxy in package.json
```
Change directory to ‘client’
cd client
Open the package.json file
vi package.json
Add the key value pair in the package.json file "proxy": "http://localhost:5000".
```
##### The whole purpose of adding the proxy configuration in number 3 above is to make it possible to access the application directly from the browser by simply calling the server url like http://localhost:5000 rather than always including the entire path like http://localhost:5000/api/todos

##### Now, ensure you are inside the Todo directory, and simply do:
```
npm run dev
Your app should open and start running on localhost:3000
```

<img width="1180" alt="Screenshot 2022-05-20 at 1 00 22 AM" src="https://user-images.githubusercontent.com/105562242/169387478-97170f73-bbf1-43c1-91aa-8e4de47e4fee.png">

#### Creating your React Components

##### One of the advantages of react is that it makes use of components, which are reusable and also makes code modular. For our Todo app, there will be two stateful components and one stateless component.

##### From your Todo directory run
```
cd client
move to the src directory

cd src
Inside your src folder create another folder called components

mkdir components
Move into the components directory with

cd components
Inside ‘components’ directory create three files Input.js, ListTodo.js and Todo.js.

touch Input.js ListTodo.js Todo.js
Open Input.js file

vi Input.js

```
<img width="908" alt="Screenshot 2022-05-20 at 1 04 00 AM" src="https://user-images.githubusercontent.com/105562242/169388062-ceb519fc-df72-4105-92c4-58c891f03209.png">

##### To make use of Axios, which is a Promise based HTTP client for the browser and node.js, you need to cd into your client from your terminal and run yarn add axios or npm install axios.
```
Move to the src folder

cd ..
Move to clients folder

cd ..
Install Axios

npm install axios
```
* Go to ‘components’ directory
```
cd src/components
After that open  ListTodo.js

vi ListTodo.js
```
* in the ListTodo.js copy and paste the following code


<img width="1049" alt="Screenshot 2022-05-20 at 1 07 08 AM" src="https://user-images.githubusercontent.com/105562242/169388567-7c467bf7-ba0e-44ee-a47f-6003b963f0f1.png">

* Then in  Todo.js file write the following code

<img width="882" alt="Screenshot 2022-05-20 at 1 08 50 AM" src="https://user-images.githubusercontent.com/105562242/169388844-f7bec457-346b-41b1-ae65-e6286b043e88.png">

* Update App.js as below 

<img width="753" alt="Screenshot 2022-05-20 at 1 10 21 AM" src="https://user-images.githubusercontent.com/105562242/169389094-297275bd-3d52-470b-bfa5-caaa7e17afb7.png">

* Update App.css as below 

```
.App {
text-align: center;
font-size: calc(10px + 2vmin);
width: 60%;
margin-left: auto;
margin-right: auto;
}

input {
height: 40px;
width: 50%;
border: none;
border-bottom: 2px #101113 solid;
background: none;
font-size: 1.5rem;
color: #787a80;
}

input:focus {
outline: none;
}

button {
width: 25%;
height: 45px;
border: none;
margin-left: 10px;
font-size: 25px;
background: #101113;
border-radius: 5px;
color: #787a80;
cursor: pointer;
}

button:focus {
outline: none;
}

ul {
list-style: none;
text-align: left;
padding: 15px;
background: #171a1f;
border-radius: 5px;
}

li {
padding: 15px;
font-size: 1.5rem;
margin-bottom: 15px;
background: #282c34;
border-radius: 5px;
overflow-wrap: break-word;
cursor: pointer;
}

@media only screen and (min-width: 300px) {
.App {
width: 80%;
}

input {
width: 100%
}

button {
width: 100%;
margin-top: 15px;
margin-left: 0;
}
}

@media only screen and (min-width: 640px) {
.App {
width: 60%;
}

input {
width: 50%;
}

button {
width: 30%;
margin-left: 10px;
margin-top: 0;
}
}
Exit

In the src directory open the index.css

vim index.css
Copy and paste the code below:

body {
margin: 0;
padding: 0;
font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen",
"Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue",
sans-serif;
-webkit-font-smoothing: antialiased;
-moz-osx-font-smoothing: grayscale;
box-sizing: border-box;
background-color: #282c34;
color: #787a80;
}

code {
font-family: source-code-pro, Menlo, Monaco, Consolas, "Courier New",
monospace;
}
```
* Go to the Todo directory
```
cd ../..
When we are in the Todo directory run:

npm run dev
```
<img width="1716" alt="Screenshot 2022-05-20 at 1 13 39 AM" src="https://user-images.githubusercontent.com/105562242/169389611-2520bd3c-12a4-4cf4-88af-f9ca5796f197.png">

<img width="1288" alt="Screenshot 2022-05-20 at 1 14 34 AM" src="https://user-images.githubusercontent.com/105562242/169389776-b37297da-4b95-4c9a-9759-2c6a3256d710.png">
<img width="1261" alt="Screenshot 2022-05-20 at 1 14 43 AM" src="https://user-images.githubusercontent.com/105562242/169389789-465cd044-34e3-4fef-b227-d21156f859d3.png">


<img width="1216" alt="Screenshot 2022-05-20 at 1 15 18 AM" src="https://user-images.githubusercontent.com/105562242/169389850-bb99be38-f902-4b3d-83f8-a85ace20c333.png">





