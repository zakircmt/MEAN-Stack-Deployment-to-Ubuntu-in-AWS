# Introduction

The MEAN stack is a popular JavaScript stack used for building web applications. It stands for MongoDB, Express.js, AngularJS (or Angular), and Node.js. Here's a brief overview of each component:

__MongoDB:__ MongoDB is a NoSQL database that stores data in a flexible, JSON-like format. It is a popular choice for web applications because of its scalability, flexibility, and performance.

__Express.js:__ Express.js is a web application framework for Node.js. It provides a set of features for building web applications and APIs, including routing, middleware support, and templating engines. Express.js simplifies the process of building web applications with Node.js by providing a high-level abstraction over the HTTP protocol.

__AngularJS (or Angular):__ AngularJS is a JavaScript framework maintained by Google for building dynamic web applications. It extends HTML with additional attributes and provides a set of built-in directives for creating interactive user interfaces. AngularJS follows the Model-View-Controller (MVC) architecture, making it easy to organize code and build complex web applications.

__Node.js:__ Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. It allows developers to run JavaScript code outside of a web browser, making it possible to build server-side applications with JavaScript. Node.js is known for its event-driven, non-blocking I/O model, which makes it well-suited for building scalable and high-performance web applications.

##  __Note:__ Follow Issue-Solving Notes for Smooth Project Completion
For a smooth and successful project completion, please ensure to follow the notes I have documented on issue-solving. These steps outline the necessary actions to address potential challenges and ensure the project progresses without delays.

Refer to the notes for detailed instructions and proceed accordingly..

## Step 0: Prerequisites
##  __Note:__ When starting my last (MERN) project, I initially launched an EC2 instance with a t2.micro configuration. However, this setup proved insufficient due to the higher memory and CPU requirements of Node.js. As a result, I upgraded the instance to t2.medium, which resolved the performance issues, and the project began running smoothly. I recommend ensuring that your EC2 instance has adequate resources to meet the official requirements of Node.js and your specific project before deployment. Additionally, be mindful that when migrating to a different instance type, your public IP address may change, requiring updates to any relevant configurations.

__1.__ EC2 Instance of t2.medium type and Ubuntu 22.04 LTS (HVM) was lunched in the us-east-1 region using the AWS console.

![Lunch Instance](./images/1.png)
![Lunch Instance](./images/2.png)
![Lunch Instance](./images/3.png)
![Lunch Instance](./images/4.png)
![Lunch Instance](./images/5.png)
![Lunch Instance](./images/6.png)
![Lunch Instance](./images/7.png)

__2.__ Attached SSH key named __my-server-key__ to access the instance on port 22

__3.__ The security group was configured with the following inbound rules:

- Allow traffic on port 80 (HTTP) with source from anywhere on the internet.

- Allow traffic on port 443 (HTTPS) with source from anywhere on the internet.

- Allow traffic on port 22 (SSH) with source from any IP address. This is opened by default.

- Allow traffic on port 3300 (Custom TCP) with source from anywhere.

![Security Rules](./images/24.png)

__4.__ The private ssh key permission was changed for the private key file and then used to connect to the instance by running
```bash
chmod 400 my-server-key.pem
```
```bash
ssh -i "my-server-key.pem" ubuntu@44.201.165.6
```
Where __username=ubuntu__ and __public ip address=44.201.165.6__

![Connect to instance](./images/8.png)


## Step 1 - Install Nodejs

Node.js is a JavaScript runtime built on Chrome’s V8 JavaScript engine. Node.js is used in this tutorial to set up the Express routes and AngularJS controllers.

__1.__ __Update and Upgrade ubuntu__

```bash
sudo apt update -y
sudo apt upgrade -y
```
![Update ubuntu](./images/9.png)
![Update ubuntu](./images/10.png)
![Update ubuntu](./images/11.png)

__2.__ __Add certificates__

```bash
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates
```
![Ade cert.](./images/12.png)


```bash
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
```
![Add cert.](./images/13.png)

__.3__ __Install NodeJS__

```bash
sudo apt-get install -y nodejs
```
![Install nodejs](./images/14.png)

##  __Note:__ A new kernel version has been loaded on your EC2 instance. To ensure the system is running on the latest kernel, a restart of the instance is required. Pleaese check your diagonstics message.

## Step 2 - Install MongoDB

For this application, Book records were added to MongoDB that contain book name, isbn number, author, and number of pages.

__1.__ __Download the MongoDB public GPG key__

```bash
sudo apt-get install -y gnupg curl 
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg  --dearmor

```
![Mongodb public gpg key](./images/15.png)
![Mongodb public gpg key](./images/14-1.png)

__2.__ __Add the MongoDB repository__

```bash
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```
![Mongo public key](./images/16.png)


__3.__ __Update the package database and install MongoDB__

##  __Note:__ Before installing MongoDB, ensure to run the following command to update the repository list: (#apt-get update -y) Skipping this step may cause issues during the installation of mongodb-org, as the system may not have the latest MongoDB repository information.

Proceed with the MongoDB installation only after updating the repository to avoid potential errors
```bash
sudo apt-get update -y
```

```bash
sudo apt-get install -y mongodb-org
```
![Install mongodb](./images/17.png)
![Install mongodb](./images/18.png)
![Install mongodb](./images/19.png)


__4.__ __Start and enable MongoDB__

```bash
sudo systemctl start mongod
```
```bash
sudo systemctl enable mongod
```
```bash
sudo systemctl status mongod
```
![Start Mongodb](./images/20.png)

__5.__ __Install body-parser package__

__body-parser__ package is needed to help process JSON files passed in requests to the server.

```bash
sudo npm install body-parser
```
![Install body-parser](./images/21.png)

__6.__ __Create the project root folder named ‘Books’__

```bash
mkdir Books && cd Books
```

Initialize the root folder
```bash
npm init
```
![init folder](./images/22.png)

__Add file named server.js to Books folder__
```bash
vi server.js
```
Copy and paste the web server code below into the server.js file.

```bash
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose'); // Make sure mongoose is installed and required
const path = require('path'); // To handle static file serving
const app = express();

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/test', { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error('MongoDB connection error:', err));

// Middleware
app.use(bodyParser.json());
app.use(express.static(path.join(__dirname, 'public')));

// Routes
require('./apps/routes')(app);

// Start the server
app.set('port', 3300);
app.listen(app.get('port'), () => {
  console.log('Server up: http://localhost:' + app.get('port'));
});
```
![Server.js](./images/23.png)


## Step 3 - Install Express and set up routes to the server

Express was used to pass book information to and from our MongoDB database.
Mongoose package provides a straightforward schema-based solution to model the application data. Mongoose was used to establish a schema for the database to store data of the book register.

__1.__ __Install express and mongoose__

```bash
sudo npm install express mongoose
```
![Install express](./images/25.png)

__2.__ __In Books folder, create a folder named ‘apps’__

```bash
mkdir apps && cd apps
```
__In apps, create a file named routes.js__

```bash
vi routes.js
```

Copy and paste the code below into routes.js

```bash
const Book = require('./models/book');
const path = require('path');

module.exports = function(app) {
  // Get all books
  app.get('/book', async (req, res) => {
    try {
      const books = await Book.find({});
      res.json(books);
    } catch (err) {
      console.error(err);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  });

  // Add a new book
  app.post('/book', async (req, res) => {
    try {
      const book = new Book({
        name: req.body.name,
        isbn: req.body.isbn,
        author: req.body.author,
        pages: req.body.pages
      });
      const result = await book.save();
      res.json({
        message: "Successfully added book",
        book: result
      });
    } catch (err) {
      console.error(err);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  });

  // Update a book
  app.put('/book/:isbn', async (req, res) => {
    try {
      const updatedBook = await Book.findOneAndUpdate(
        { isbn: req.params.isbn },
        req.body,
        { new: true }
      );
      if (!updatedBook) {
        return res.status(404).json({ error: 'Book not found' });
      }
      res.json({
        message: "Successfully updated the book",
        book: updatedBook
      });
    } catch (err) {
      console.error(err);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  });

  // Delete a book
  app.delete('/book/:isbn', async (req, res) => {
    try {
      const result = await Book.findOneAndRemove({ isbn: req.params.isbn });
      if (!result) {
        return res.status(404).json({ error: 'Book not found' });
      }
      res.json({
        message: "Successfully deleted the book",
        book: result
      });
    } catch (err) {
      console.error(err);
      res.status(500).json({ error: 'Internal Server Error' });
    }
  });

  // Serve static files
  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, '../public', 'index.html'));
  });
};
```
![create "apps"folder](./images/26.png)
![Routes](./images/27.png)
![Routes](./images/28.png)

__3.__ __In the ‘apps’ folder, create a folder named models__

```bash
mkdir models && cd models
```
__In models, create a file named book.js__

```bash
vi book.js
```
![create "models"folder](./images/29.png)


Copy and paste the code below into book.js

```bash
const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
  name: { type: String, required: true },
  isbn: { type: String, required: true, unique: true },
  author: { type: String, required: true },
  pages: { type: Number, required: true }
});

module.exports = mongoose.model('Book', bookSchema);
```
![books](./images/30.png)


## Step 4 - Access the routes with AngularJS

In this project, AngularJS was used to connect the web page with Express and perform actions on the book register.

__1.__ __Change the directory back to ‘Books’ and create a folder named ‘public’__

```bash
cd ../..

mkdir public && cd public
```
__Add a file named script.js into public folder__

```bash
vi script.js
```
![create "public"folder](./images/31.png)

Copy and paste the code below (controller configuration defined) into the script.js file.

```bash
var app = angular.module('myApp', []);

app.controller('myCtrl', function($scope, $http) {
  // Get all books
  function getAllBooks() {
    $http({
      method: 'GET',
      url: '/book'
    }).then(function successCallback(response) {
      $scope.books = response.data;
    }, function errorCallback(response) {
      console.log('Error: ' + response.data);
    });
  }

  // Initial load of books
  getAllBooks();

  // Add a new book
  $scope.add_book = function() {
    var body = {
      name: $scope.Name,
      isbn: $scope.Isbn,
      author: $scope.Author,
      pages: $scope.Pages
    };
    $http({
      method: 'POST',
      url: '/book',
      data: body
    }).then(function successCallback(response) {
      console.log(response.data);
      getAllBooks();  // Refresh the book list
      // Clear the input fields
      $scope.Name = '';
      $scope.Isbn = '';
      $scope.Author = '';
      $scope.Pages = '';
    }, function errorCallback(response) {
      console.log('Error: ' + response.data);
    });
  };

  // Update a book
  $scope.update_book = function(book) {
    var body = {
      name: book.name,
      isbn: book.isbn,
      author: book.author,
      pages: book.pages
    };
    $http({
      method: 'PUT',
      url: '/book/' + book.isbn,
      data: body
    }).then(function successCallback(response) {
      console.log(response.data);
      getAllBooks();  // Refresh the book list
    }, function errorCallback(response) {
      console.log('Error: ' + response.data);
    });
  };

  // Delete a book
  $scope.delete_book = function(isbn) {
    $http({
      method: 'DELETE',
      url: '/book/' + isbn
    }).then(function successCallback(response) {
      console.log(response.data);
      getAllBooks();  // Refresh the book list
    }, function errorCallback(response) {
      console.log('Error: ' + response.data);
    });
  };
});
```
![Script.js](./images/32.png)
![Script.js](./images/33.png)


__2.__ __In ‘public’ folder, create a file named index.html__

```bash
vi index.html
```
Copy and paste the code below into index.html file

```bash
<!DOCTYPE html>
<html ng-app="myApp" ng-controller="myCtrl">
<head>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
  <script src="script.js"></script>
  <style>
    /* Add your custom CSS styles here */
  </style>
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
    <div ng-if="successMessage">{{ successMessage }}</div>
    <div ng-if="errorMessage">{{ errorMessage }}</div>
  </div>
  <hr>
  <div>
    <table>
      <tr>
        <th>Name</th>
        <th>Isbn</th>
        <th>Author</th>
        <th>Page</th>
        <th>Action</th>
      </tr>
      <tr ng-repeat="book in books">
        <td>{{ book.name }}</td>
        <td>{{ book.isbn }}</td>
        <td>{{ book.author }}</td>
        <td>{{ book.pages }}</td>
        <td><button ng-click="del_book(book)">Delete</button></td>
      </tr>
    </table>
  </div>
</body>
</html>
```
![HTML](./images/34.png)
![HTML](./images/35.png)

__3.__ __Change the directory back up to ‘Books’ and start the server__

```bash
cd ..
```
```bash
node server.js
```
![Run server](./images/36.png)

The server is now up and running, Connection to it is via port 3300. A separate Putty/terminal or SSH console to test what curl command returns locally can be launched.

![curl anothr putty](./images/38.png)
![curl anothr putty](./images/37.png)

The Book Register web application can now be accessed from the internet with a browser using the Public IP address or Public DNS name.

![Book register](./images/39.png)
![Book register](./images/40.png)

Add & more books to the register

![Book register](./images/41.png)
![Book register](./images/42.png)

Get the json view

![Book register json view](./images/43.png)

## Conclusion

The MEAN stack—comprising MongoDB, Express.js, AngularJS (or Angular), and Node.js—provides a powerful and cohesive set of technologies for building modern web applications.

Together, these technologies allow developers to use JavaScript throughout the entire development process, from front-end to back-end, promoting a unified and streamlined development workflow.

