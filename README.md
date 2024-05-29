# Types of Databases
* There are a few types of databases, all service different types of use-cases -
## 1. NoSQL databases 
* Store data in a schema-less fashion. Extremely lean and fast way to store data. 
* Examples - MongoDB, 
![alt text](<README IMG/nosql.png>)

## 2. Graph databases 
* Data is stored in the form of a graph. Specially useful in cases where relationships need to be stored (social networks).
* Examples - Neo4j
![alt text](<README IMG/graph.png>)

## 3. Vector databases
* Stores data in the form of vectors
* Useful in Machine learning
* Examples - Pinecone
![alt text](<README IMG/vector.webp>)

## 4. SQL databases
* Stores data in the form of rows
* Most full stack applications will use this
* Examples - MySQL, Postgres
![alt text](<README IMG/sql.png>)


# Why not NoSQL
* You might’ve used MongoDB. It’s schemaless properties make it ideal to for bootstraping a project fast. But as your app grows, this property makes it very easy for data to get curropted.

## What is schemaless?
* Different rows can have different schema (keys/types)
![alt text](<README IMG/whynoswl.png>)

## Problems?
1. Can lead to inconsistent database
2. Can cause runtime errors 
3. Is too flexible for an app that needs strictness
 
# Why SQL ?
* SQL databases have a strict schema. They require you to-
1. Define your schema
2. Put in data that follows that schema
3. Update the schema as your app changes and perform migrations
 
* So there are 4 parts when using an SQL database (not connecting it to Node.js, just running it and putting data in it)
1. Running the database.
2. Using a library that let’s you connect and put data in it.
3. Creating a table and defining it’s schema.
4. Run queries on the database to interact with the data (Insert/Update/Delete)
 

# Creating a table and defining it’s schema
* SQL stands for Structured query language. It is a language in which you can describe what/how you want to put data in the database.
To create a table, the command to run is - 
```.sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```
* There are a few parts of this SQL statement, let’s decode them one by one -
## 1. CREATE TABLE users
##### ⏺ CREATE TABLE users: This command initiates the creation of a new table in the database named users.

## 2. id SERIAL PRIMARY KEY
##### ⏺ id: The name of the first column in the users table, typically used as a unique identifier for each row (user). Similar to _id in mongodb
##### ⏺ SERIAL: A PostgreSQL-specific data type for creating an auto-incrementing integer. Every time a new row is inserted, this value automatically increments, ensuring each user has a unique id.
##### ⏺ PRIMARY KEY: This constraint specifies that the id column is the primary key for the table, meaning it uniquely identifies each row. Values in this column must be unique and not null.

## 3. email VARCHAR(255) UNIQUE NOT NULL,
##### ⏺ email: The name of the second column, intended to store the user's username.
##### ⏺ VARCHAR(50): A variable character string data type that can store up to 50 characters. It's used here to limit the length of the username.
##### ⏺ UNIQUE: This constraint ensures that all values in the username column are unique across the table. No two users can have the same username.
##### ⏺ NOT NULL: This constraint prevents null values from being inserted into the username column. Every row must have a username value.

## 4. password VARCHAR(255) NOT NUL
* Same as above, can be non uniqye

## 5. created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
##### ⏺ created_at: The name of the fifth column, intended to store the timestamp when the user was created.
##### ⏺ TIMESTAMP WITH TIME ZONE: This data type stores both a timestamp and a time zone, allowing for the precise tracking of when an event occurred, regardless of the user's or server's time zone.
##### ⏺ DEFAULT CURRENT_TIMESTAMP: This default value automatically sets the created_at column to the date and time at which the row is inserted into the table, using the current timestamp of the database server.


#  Interacting with the database
* There are 4 things you’d like to do with a database :

## 1. INSERT
```.sql
INSERT INTO users (username, email, password)
VALUES ('username_here', 'user@example.com', 'user_password');
```
*  you didn’t have to specify the id  because it auto increments

## 2. UPDATE
```.sql
UPDATE users
SET password = 'new_password'
WHERE email = 'user@example.com';
```

## 3. DELETE
```.sql
DELETE FROM users
WHERE id = 1;
```

## 4. Select
```.sql
SELECT * FROM users
WHERE id = 1;
```


#  How to do queries from a Node.js app ?
* In the end, postgres exposes a protocol that someone needs to talk to be able to send these commands (update, delete) to the database.
psql  is one such library that takes commands from your terminal and sends it over to the database.
To do the same in a Node.js , you can use one of many Postgres clients 

* Initialise an empty typescript project
```.sql
npm init -y
npx tsc --init
```
* Change the rootDir and outDir in tsconfig.json
```.ts
"rootDir": "./src",
"outDir": "./dist",
```
* Install the pg library and it’s types (because we’re using TS)
```.js
npm install pg
npm install @types/pg
```

##### Task write a function to create a users table in your database
```.ts
import {Client} from 'pg'
const client = new Client({
  host: 'localhost',
  port: 5432,
  database: 'postgres',
  user: 'postgres',
  password: 'user',
});

async function createUsersTable() {
  await client.connect()
  const result = await client.query(`
      CREATE TABLE users (
          id SERIAL PRIMARY KEY,
          username VARCHAR(50) UNIQUE NOT NULL,
          email VARCHAR(255) UNIQUE NOT NULL,
          password VARCHAR(255) NOT NULL,
          created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
      );
  `)
  console.log(result)
}

createUsersTable();
```

##### Task  - Create a function that let’s you insert data into a table. Make it async, make sure client.connect resolves before u do the insert :
```.ts
import { Client } from 'pg';

// Async function to insert data into a table
async function insertData() {
  const client = new Client({
    host: 'localhost',
    port: 5432,
    database: 'postgres',
    user: 'postgres',
    password: 'user',
  });

  try {
    await client.connect(); // Ensure client connection is established
    const insertQuery = "INSERT INTO users (username, email, password) VALUES ('username2', 'user3@example.com', 'user_password');";
    const res = await client.query(insertQuery);
    console.log('Insertion success:', res); // Output insertion result
  } catch (err) {
    console.error('Error during the insertion:', err);
  } finally {
    await client.end(); // Close the client connection
  }
}

insertData();
```
* This is an insecure way to store data in your tables. When you expose this functionality eventually via HTTP, someone can do an SQL INJECTION to get access to your data/delete your data. so the good way is given in the below:-
```.ts
import { Client } from 'pg';

// Async function to insert data into a table
async function insertData(username: string, email: string, password: string) {
  const client = new Client({
    host: 'localhost',
    port: 5432,
    database: 'postgres',
    user: 'postgres',
    password: 'user',
  });

  try {
    await client.connect(); // Ensure client connection is established
    // Use parameterized query to prevent SQL injection
    const insertQuery = "INSERT INTO users (username, email, password) VALUES ($1, $2, $3)";
    const values = [username, email, password];
    const res = await client.query(insertQuery, values);
    console.log('Insertion success:', res); // Output insertion result
  } catch (err) {
    console.error('Error during the insertion:', err);
  } finally {
    await client.end(); // Close the client connection
  }
}

// Example usage
insertData('username5', 'user5@example.com', 'user_password').catch(console.error);
```

##### Task -  Write a function getUser that lets you fetch data from the database given a email as input.
```.ts
import { Client } from 'pg';

// Async function to fetch user data from the database given an email
async function getUser(email: string) {
    const client = new Client({
        host: 'localhost',
        port: 5432,
        database: 'postgres',
        user: 'postgres',
        password: 'user',
    });
    

  try {
    await client.connect(); // Ensure client connection is established
    const query = 'SELECT * FROM users WHERE email = $1';
    const values = [email];
    const result = await client.query(query, values);
    
    if (result.rows.length > 0) {
      console.log('User found:', result.rows[0]); // Output user data
      return result.rows[0]; // Return the user data
    } else {
      console.log('No user found with the given email.');
      return null; // Return null if no user was found
    }
  } catch (err) {
    console.error('Error during fetching user:', err);
    throw err; // Rethrow or handle error appropriately
  } finally {
    await client.end(); // Close the client connection
  }
}

// Example usage
getUser('user3@example.com').catch(console.error);
```