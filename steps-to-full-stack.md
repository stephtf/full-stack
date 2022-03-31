# Steps to creating a full-stack app

## Description 
These are my notes on how to create a full stack web application. We will be using a SQL database, RESTful APIs with Express, and Handlebars for the front end. These steps could be modified for React for a more modern UI. These steps are meant for beginners. If you have any questions, please reach out to me at stfajardo@gmail.com. Feel free to check out my work at [stephfajardo.com](https://www.stephfajardo.com).

# STEPS

## 1. Create your repo in Github 
- make sure to add a readme, license, and gitignore 

## 2. Install npm and create your package.json
- npm init -y 
- npm i mysql2 express sequelize dotenv

## 3. Update the gitignore file
- add node_modules, .DS_store, and .env

## 4. Create your .env file 
- database name
- password
- user 

```
DB_NAME=
DB_USER=root
DB_PASSWORD= 
```

## 5. Create your server.js file (this will be empty at this point)
- this is a good time to push to github

## 6. Create your db folder and schema.sql file
- actually create the database by going into mysql 
- mysql -u root -p 

``` 
DROP DATABASE IF EXISTS database_db;
CREATE DATABASE database_db;
```

## 7. Start your server 
- add the necessary code into the server.js file 

```
const PORT = process.env.PORT || 3001;

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

app.listen(PORT, () => {
    console.log(`relax! everything is working over at port ${PORT}`);
})
```

## 8. Create your config folder 
- create the folder called "config"
- create a document within the folder called "connection.js" 
- and the necessary code to connect to the sequelize library 

``` 
const Sequelize = require('sequelize'); 
require('dotenv').config(); 
    
const sequelize = new Sequelize(
    process.env.DB_NAME,
    process.env.DB_USER,
    process.env.DB_PASSWORD,
    {
        host: 'localhost',
        dialect: 'mysql',
        port: 3306    }
); 
```

## 9. Export out the Sequelize object 
- from the connnection.js you need to export out the sequelize object that you created 

``` 
module.exports = sequelize; 
```

## 10. Import in the Sequelize library
- add the connection into the server.js file 
- make sure that the path is correct 

```
const sequelize = require('./config/connection');
```
- make sure to also add sequelize.sync, then take the code you wrote earlier for starting the server, and put it after the .then portion

```
sequelize.sync({ force : false }).then(() => {
    app.listen(PORT, () => {
        console.log(`relax! everything is working over at port ${PORT}`);
    });
});
```
- make sure to check that your server still works by testing node server.js 
- when you see "Executing (default): SELECT 1+1 AS result" this means that your sequelize connection is working as it should and you are connected to the database 
- now is a good time to push to github 

## 11. Make your tables 
- make the models folder, which will house each javascript file for the tables
- make one javascript file for each of your tables 
    - don't forget to capitalize the first letter of your javscript tables files since these are classes 
- connect your javascript table files to the database, use object de-structuring to connect to sequelize.model and sequelize.datatypes 
- then, connect to sequelize so that the newly created tables are also connected to your database 

``` 
const { Model, DataTypes } = require('sequelize'); 

const sequelize = require('../config/connection'); 
```
- then, declare the class that you want (the example below is a User table) and create its extension from Model 

``` 
class User extends Model {} 
```
- then, create the object that will house all the data about the new User 

``` 
User.init({},{});
```

- the first parameter is an object that represents the columns of your table
- the second parameter is another object that allows you to define what the table name is and its configurations 

```
{
    sequelize, 
    timestamps: true,
    underscored: true, 
    freezeTableName: true, 
    modelName: 'user',  
}
```
- then you can just copy and paste the first table you made for the rest of them so you don't have to re-create the structure, just change out the name and the details to make it fit for subsequent tables 
- if there are relationships to define, do that now for each table 
- don't forget to put "module.exports" and export it out at the bottom of each document  
- after creating the javascript files for each table, you will also want to create an index.js file within the models folder, which will define the relationships between the tables 
- example: 

``` 
const User = require('./User');
const Book = require('./Book');

User.hasMany(Book, {
    foreignKey: 'user_id',
    onDelete: 'CASCADE', 
}); 

Book.belongsTo(User, {
    foreignKey: 'user_id',
});

module.exports = { User, Book };
```

## 12. Import your tables in the server.js file 
- make sure to do this for every table or it won't work 

``` 
const User = require('./models/User');
```

## 13. Run the server to add the tables to database
- nodemon server.js 
- if you run into issues, check: 
    - are there any typos with the word dataTypes? capitalizations need to be accurate 
    - did you write "INTEGER" rather than "INT"? (should be INTEGER)
- now is a good time to commit to github

## 14. Create your API routes 
- create a controllers folder 
- inside the controllers folder, make another folder for api routes 
- create your route javascript files 

## 15. Connect your API routes to the server.js file 
- make sure that your server.js file has access to the controllers folder 
- if you have created a separate folder within controllers for api routes, you want to link those here too 
- important: make sure your paths are correct! below is just an example of what it might look like
```
const routes = require('./controllers/');
const apiRoutes = require('./controllers/api');
```
- also make sure that you have app.use in server.js so that you can use the routes (doesn't need to be app, just whatever variable you created to require express)

``` 
app.use('/', routes);
app.use('/api', apiRoutes);
```

## 16. Define the URLs for your API routes and home routes
- for API routes, you can do this in an index.js file located in the controllers/api folder 
    - require express 
    - require the routes javascript files 
    - app.use to define them 
    - don't forget module.exports 
- here is an example of the index.js file: 

``` 
const app = require('express').Router(); 
const userRoutes = require('./userRoutes');
const bookRoutes = require('./bookRoutes');

// the url is localhost:3001/api/users or books
app.use('/users', userRoutes);
app.use('/books', bookRoutes); 

module.exports = app; 
``` 

## 17. Write your home routes 
- now you can define your home routes in the js document within controllers (not api)
    - require express 
    - app.get 
    - don't forget to add module.exports! 

- example: 

``` 
const app = require('express').Router(); 

// the url for this is localhost:3001 (login)
app.get('/', (req, res) => {
	res.send('Hello this the route to login page!');
	});


module.exports = app;
``` 

## 18. Begin writing your API routes 
- the router method allows you to chain URLs together
- start by requiring express 
```
const app = require('express').Router(); 
```
- then, make sure to require the models you need for your routes 

```
const User = require('../../models/User');
```
- don't forget to export out the app! 

```
module.exports = app;
```

- now would be a good time to test your route connections by writing a tester route 
- for example: 

```
app.get('/', (req, res) => {
	res.send('Hello this is Steph from this route!');
	});
```

- if everything goes well with the test, go ahead and write the rest of your API routes! 


## 19. Make some seed data 
- create a seeds folder, and within that folder, create an index.js file and .json files to house your data 
- in this index.js file (for the seed data) you will need: 
    - require sequelize connection 
    - require the tables from the models folder 
    - require the .json files that house the seed data 
    - create a function for seeding the database 
- example: 

``` 
const sequelize = require('../config/connection');
const User = require('../models/User');
const Book = require('../models/Book');

const userData = require('./userData.json');
const bookData = require('./bookData.json');

const seedDatabase = async () => {
  await sequelize.sync({ force: true });

  await User.bulkCreate(userData);
  await Book.bulkCreate(bookData);

  process.exit(0);
};

seedDatabase();
``` 
- when you are done, now you can plant the seeds with node seeds/index.js 
- watch the computer do its magic, executing...

## 20. Test out your routes with the seed data using Insomnia
- run your server and open insomnia to test the routes you created 


## 21. Create your front end folders and documents 
- create a views folder
- inside views add: 
    - layouts: 
        - main.handlebars 
    - add the rest of your handlebars pages 
- create a public folder
- inside the public folder: 
    - create a style.css document and put it in a css folder 
    - create an index.js document and put it in a js folder
    - add an image file and put it in an img folder 

## 22. Connect handlebars to server 
- go into your server.js file and make sure that you require express-handlebars 

``` 
const exphbs = require('express-handlebars');
const hbs = exphbs.create({});
```
- make sure that you also set handlebars as the default template engine (see unit 14 activity 3)

``` 
app.engine('handlebars', hbs.engine);
app.set('view engine', 'handlebars');
``` 

## 23. Connect your public js files to html
- you need to establish access to the public folder first. go back to server.js and ensure that you have the following code: 

``` 
app.use(express.static(path.join(__dirname, 'public')));
```
- you will also need to require 'path' 
```
const path = require('path');
```
- ^ this will allow you to not have to write out the entire pathway once you are ready to go to the html and write the script tag for the javascript file. you start at the public folder and then wherever your file is located, that's what you link from your html file. for example: 

``` 
<script src="js/index.js"></script>
```
- in the above, i did not need to write out ../../public/js/index.js because of the connection we established in the server.js file

## 24. Start creating the front-end 
- you have a full-stack app already! now you just need to create the front end with html, css, and then write the logic to fetch the data you need 
- remember, you will more than likely need more than one js file for each of the html pages. don't forget to add script tags to connect them to each html page
- once you establish the js-html connections, start writing code for front end 

## NOW DO A HAPPY DANCE BECAUSE YOU GOT THIS! YOU NOW HAVE A FULL-STACK APP THAT YOU CAN FURTHER DEVELOP





