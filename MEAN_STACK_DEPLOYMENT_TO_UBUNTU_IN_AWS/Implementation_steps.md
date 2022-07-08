
## MEAN Stack is a combination of following components:
##### MongoDB (Document database) – Stores and allows to retrieve data.
##### Express (Back-end application framework) – Makes requests to Database for Reads and Writes.
##### Angular (Front-end application framework) – Handles Client and Server Requests
##### Node.js (JavaScript runtime environment) – Accepts requests and displays results to end user



##### First we will create ubuntu EC2 and do the basic configuration so that we can install necessary dependecies to complete this project

<img width="1479" alt="Screenshot 2022-07-08 at 5 51 12 PM" src="https://user-images.githubusercontent.com/105562242/177991151-1d6c3209-7392-4038-aff1-4410d2c17b8c.png">

##### We will Update and upgrade ubuntu. We will add certificates
```
sudo apt update
sudo apt upgrade

sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
```

##### We will install NodeJS

<img width="883" alt="Screenshot 2022-07-08 at 6 01 10 PM" src="https://user-images.githubusercontent.com/105562242/177992639-38944452-aaa7-481e-8093-541978085788.png">


##### We will install MongoDB
 ###### MongoDB stores data in flexible, JSON-like documents. Fields in a database can vary from document to document and data structure can be changed over time. For our example application, we are adding book records to MongoDB that contain book name, isbn number, author, and number of pages.

```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6

echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
# Install
sudo apt install -y mongodb
# Start the server
sudo service mongodb start
# Verify if server is running
sudo systemctl status mongodb
```

<img width="885" alt="Screenshot 2022-07-08 at 6 15 35 PM" src="https://user-images.githubusercontent.com/105562242/177994669-9fac0cb0-4685-45af-b4c9-4de3e4895549.png">

<img width="882" alt="Screenshot 2022-07-08 at 6 16 25 PM" src="https://user-images.githubusercontent.com/105562242/177994804-4479ef8a-cef5-4bda-b609-1be24eb3900d.png">

##### We will install npm – Node package manager

` sudo apt install -y npm `

##### We will install body-parser package. it will help us process JSON files passed in requests to the server.

` sudo npm install body-parser `

<img width="883" alt="Screenshot 2022-07-08 at 6 19 50 PM" src="https://user-images.githubusercontent.com/105562242/177995335-519fac44-3d99-48e6-bbbb-b262800e8f37.png">

##### Create a folder named ‘Books’. In the Books directory, Initialize npm project and Add a file to it named server.js

```
mkdir Books && cd Books
npm init
vim server.js

```

##### We need to copy and paste the web server code below into the server.js file.

<img width="885" alt="Screenshot 2022-07-08 at 6 23 34 PM" src="https://user-images.githubusercontent.com/105562242/177995905-ad14f2cd-0b8f-4bec-b111-c0cd8417f041.png">


##### We will install Express and set up routes to the server

###### Express is a minimal and flexible Node.js web application framework that provides features for web and mobile applications. We will use Express in to pass book information to and from our MongoDB database.

###### We also will use Mongoose package which provides a straight-forward, schema-based solution to model your application data. We will use Mongoose to establish a schema for the database to store data of our book register.

` sudo npm install express mongoose `

##### In ‘Books’ folder, We will create a folder named app and create a file named routes.js

```
mkdir apps && cd apps
vi routes.js

```
##### We need below code in routes.js

<img width="1008" alt="Screenshot 2022-07-08 at 7 09 28 PM" src="https://user-images.githubusercontent.com/105562242/178003293-96066c39-4f49-43a8-a1c5-1a1ec2d737c0.png">

##### In the ‘apps’ folder, we will create a folder named models and Create a file named book.js

```
mkdir models && cd models
vim book.js
```

<img width="858" alt="Screenshot 2022-07-08 at 7 12 29 PM" src="https://user-images.githubusercontent.com/105562242/178003733-a9856f38-bac2-4907-afbf-727314fa9e4a.png">

##### We will access the routes with AngularJS

###### AngularJS provides a web framework for creating dynamic views in your web applications. In this tutorial, we use AngularJS to connect our web page with Express and perform actions on our book register.

```
# Create a folder named public under Book folder
mkdir public && cd public
# Add a file named script.js
vim script.js
# Copy and paste the Code below (controller configuration defined) into the script.js file.

var app = angular.module('myApp', []);
app.controller('myCtrl', function($scope, $http) {
  $http( {
    method: 'GET',
    url: '/book'
  }).then(function successCallback(response) {
    $scope.books = response.data;
  }, function errorCallback(response) {
    console.log('Error: ' + response);
  });
  $scope.del_book = function(book) {
    $http( {
      method: 'DELETE',
      url: '/book/:isbn',
      params: {'isbn': book.isbn}
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
  $scope.add_book = function() {
    var body = '{ "name": "' + $scope.Name + 
    '", "isbn": "' + $scope.Isbn +
    '", "author": "' + $scope.Author + 
    '", "pages": "' + $scope.Pages + '" }';
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response);
    }, function errorCallback(response) {
      console.log('Error: ' + response);
    });
  };
});
In public folder, create a file named index.html;

vim index.html

# Cpoy and paste the code below into index.html file.

<!doctype html>
<html ng-app="myApp" ng-controller="myCtrl">
  <head>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
    <script src="script.js"></script>
  </head>
  <body>
    <div>
      <table>
        <tr>
          <td>Name:</td>
          <td><input type="text" ng-model="Name"></td>
        </tr>
        <tr>
          <td>Isbn:</td>
          <td><input type="text" ng-model="Isbn"></td>
        </tr>
        <tr>
          <td>Author:</td>
          <td><input type="text" ng-model="Author"></td>
        </tr>
        <tr>
          <td>Pages:</td>
          <td><input type="number" ng-model="Pages"></td>
        </tr>
      </table>
      <button ng-click="add_book()">Add</button>
    </div>
    <hr>
    <div>
      <table>
        <tr>
          <th>Name</th>
          <th>Isbn</th>
          <th>Author</th>
          <th>Pages</th>

        </tr>
        <tr ng-repeat="book in books">
          <td>{{book.name}}</td>
          <td>{{book.isbn}}</td>
          <td>{{book.author}}</td>
          <td>{{book.pages}}</td>

          <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
        </tr>
      </table>
    </div>
  </body>
</html>

# Change the directory back up to Books
cd ..

```

##### We will start the server by running this command:

`node server.js`

<img width="973" alt="Screenshot 2022-07-08 at 7 18 26 PM" src="https://user-images.githubusercontent.com/105562242/178004779-5dfd9c37-763b-473b-977c-ffa637f47868.png">

##### The server is now up and running, we can connect it via port 3300. You can launch a separate Putty or SSH console to test what curl command returns locally. to access port 3300 we need to add this port to security group 

<img width="1457" alt="Screenshot 2022-07-08 at 7 19 51 PM" src="https://user-images.githubusercontent.com/105562242/178005022-0be163b0-8aa9-47b3-b7c2-d6f22c58f036.png">

##### Now We can access our Book Register web application from the Internet with a browser using Public IP address or Public DNS name.

<img width="1197" alt="Screenshot 2022-07-08 at 7 23 29 PM" src="https://user-images.githubusercontent.com/105562242/178005649-39683edd-2411-4f7f-81de-265dfdc02fdb.png">

<img width="1078" alt="Screenshot 2022-07-08 at 7 22 11 PM" src="https://user-images.githubusercontent.com/105562242/178005419-bf212374-1f50-46ce-aff3-7c44c5f26253.png">

<img width="1026" alt="Screenshot 2022-07-08 at 7 24 52 PM" src="https://user-images.githubusercontent.com/105562242/178005888-6e91a1e6-50ba-4854-b1b5-6b1732c99527.png">

##### Here we can see database is getting updated with input information 

<img width="975" alt="Screenshot 2022-07-08 at 7 25 19 PM" src="https://user-images.githubusercontent.com/105562242/178006017-0d6b1a29-41bb-4b11-b468-4acecd590303.png">








