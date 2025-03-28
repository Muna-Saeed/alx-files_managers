# 0x04. Files Manager

## Project Overview

This project is a back-end application to manage file uploads and views. It involves implementing authentication, file handling, and background processing using various technologies. The project was completed in a team of two (Muna Mohammed) and ran from June 27, 2024, 6:00 AM to July 4, 2024, 6:00 AM.

## Technologies Used

- **Programming Language**: JavaScript (ES6)
- **Database**: MongoDB (NoSQL)
- **Caching**: Redis
- **Server**: Node.js, Express.js
- **Background Processing**: Kue

## Objective

To build a platform that allows users to:

- Authenticate via token
- List all files
- Upload new files
- Change file permissions
- View files
- Generate thumbnails for images

## Learning Objectives

By the end of this project, you should be able to:

- Create an API with Express
- Authenticate a user
- Store data in MongoDB
- Store temporary data in Redis
- Set up and use a background worker

## Project Structure

The project is organized into the following main parts:

- **utils/**: Contains utility classes for Redis and MongoDB.
- **routes/**: Defines API endpoints.
- **controllers/**: Implements the logic for handling requests.
- **`utils/redis.js`**: Redis client utility.
- **`utils/db.js`**: MongoDB client utility.
- **`server.js`**: Express server setup.
- **`routes/index.js`**: API route definitions.
- **`controllers/AppController.js`**: Controller for status and stats endpoints.
- **`controllers/UsersController.js`**: Controller for user management.
- **`controllers/AuthController.js`**: Controller for authentication.
- **`controllers/FilesController.js`**: Controller for file management.

## Requirements

- Allowed editors: vi, vim, emacs, Visual Studio Code
- Node version: 12.x.x
- All files should end with a new line
- Code must adhere to ESLint rules
- A `README.md` file is mandatory

## Provided Files

- `package.json`
- `.eslintrc.js`
- `babel.config.js`

**Don’t forget to run** `$ npm install **when you have the package.json.**

## Resources

- [Node.js Getting Started](https://nodejs.org/en/docs/guides/getting-started-guide/)
- [Express Getting Started](https://expressjs.com/en/starter/installing.html)
- [Mocha Documentation](https://mochajs.org/)
- [Nodemon Documentation](https://nodemon.io/)
- [MongoDB Documentation](https://docs.mongodb.com/)
- [Bull](https://www.npmjs.com/package/bull)
- [Image Thumbnail](https://www.npmjs.com/package/image-thumbnail)
- [Mime-Types](https://www.npmjs.com/package/mime-types)
- [Redis](https://redis.io/documentation)
- [ESLint](https://eslint.org/)

### Setup Instructions

1. **Clone the Repository**

   ```bash
   git clone https://github.com/<your-username>/alx-files_manager.git
   cd alx-files_manager
   ```

2. **Install Dependencies**

   ```bash
   npm install
   ```

3. **Set Up Environment Variables**

   Create a `.env` file in the root directory and set the required environment variables:

   ```bash
   DB_HOST=localhost
   DB_PORT=27017
   DB_DATABASE=files_manager
   PORT=5000
   FOLDER_PATH=/tmp/files_manager
   ```

4. **Start the Server**

   ```bash
   npm start
   ```

5. **Run the Tests**

   Use the following command to run tests:

   ```bash
   npm test
   ```

### Endpoints

- **GET /status**: Check the status of Redis and MongoDB.
- **GET /stats**: Get the number of users and files.
- **POST /users**: Create a new user.
- **GET /connect**: Authenticate a user and generate a token.
- **GET /disconnect**: Sign out a user based on the token.
- **GET /users/me**: Retrieve user information based on the token.
- **POST /files**: Upload a new file.

### Tasks

#### 0. Redis utils
- **Score**: 100.0% (Checks completed: 100.0%)

Inside the folder `utils`, create a file `redis.js` that contains the class `RedisClient`.

`RedisClient` should have:
- A constructor that creates a client to Redis:
  - Any error of the redis client must be displayed in the console (you should use `on('error')` of the redis client)
- A function `isAlive` that returns `true` when the connection to Redis is a success; otherwise, `false`
- An asynchronous function `get` that takes a string `key` as an argument and returns the Redis value stored for this key
- An asynchronous function `set` that takes a string `key`, a `value`, and a `duration` in seconds as arguments to store it in Redis (with an expiration set by the duration argument)
- An asynchronous function `del` that takes a string `key` as an argument and removes the value in Redis for this key

After the class definition, create and export an instance of `RedisClient` called `redisClient`.

```js
import redisClient from './utils/redis';

(async () => {
    console.log(redisClient.isAlive());
    console.log(await redisClient.get('myKey'));
    await redisClient.set('myKey', 12, 5);
    console.log(await redisClient.get('myKey'));

    setTimeout(async () => {
        console.log(await redisClient.get('myKey'));
    }, 1000 * 10)
})();
```

**Repo**:
- GitHub repository: `alx-files_manager`
- File: `utils/redis.js`

#### 1. MongoDB utils

Inside the folder `utils`, create a file `db.js` that contains the class `DBClient`.

`DBClient` should have:
- A constructor that creates a client to MongoDB:
  - `host`: from the environment variable `DB_HOST` or default: `localhost`
  - `port`: from the environment variable `DB_PORT` or default: `27017`
  - `database`: from the environment variable `DB_DATABASE` or default: `files_manager`
- A function `isAlive` that returns `true` when the connection to MongoDB is a success; otherwise, `false`
- An asynchronous function `nbUsers` that returns the number of documents in the collection `users`
- An asynchronous function `nbFiles` that returns the number of documents in the collection `files`

After the class definition, create and export an instance of `DBClient` called `dbClient`.

```js
import dbClient from './utils/db';

const waitConnection = () => {
    return new Promise((resolve, reject) => {
        let i = 0;
        const repeatFct = async () => {
            await setTimeout(() => {
                i += 1;
                if (i >= 10) {
                    reject()
                }
                else if (!dbClient.isAlive()) {
                    repeatFct()
                }
                else {
                    resolve()
                }
            }, 1000);
        };
        repeatFct();
    })
};

(async () => {
    console.log(dbClient.isAlive());
    await waitConnection();
    console.log(dbClient.isAlive());
    console.log(await dbClient.nbUsers());
    console.log(await dbClient.nbFiles());
})();
```

**Repo**:
- GitHub repository: `alx-files_manager`
- File: `utils/db.js`

#### 2. First API

Inside `server.js`, create the Express server:
- It should listen on the port set by the environment variable `PORT` or by default `5000`
- It should load all routes from the file `routes/index.js`

Inside the folder `routes`, create a file `index.js` that contains all endpoints of our API:
- `GET /status` => `AppController.getStatus`
- `GET /stats` => `AppController.getStats`

Inside the folder `controllers`, create a file `AppController.js` that contains the definition of the 2 endpoints:
- `GET /status` should return if Redis is alive and if the DB is alive too by using the 2 utils created previously: `{ "redis": true, "db": true }` with a status code 200
- `GET /stats` should return the number of users and files in DB: `{ "users": 12, "files": 1231 }` with a status code 200

```bash
# Terminal 1:
npm run start-server
# Output:
Server running on port 5000
...
# Terminal 2:
curl 0.0.0.0:5000/status ; echo ""
# Output:
{"redis":true,"db":true}
curl 0.0.0.0:5000/stats ; echo ""
# Output:
{"users":4,"files":30}
```

**Repo**:
- GitHub repository: `alx-files_manager`
- Files: `server.js`, `routes/index.js`, `controllers/AppController.js`

#### 3. Create a new user

In the file `routes/index.js`, add a new endpoint:
- `POST /users` => `UsersController.postNew`

Inside `controllers`, create a file `UsersController.js` that contains the new endpoint:
- `POST /users` should create a new user in DB:
  - To create a user, you must specify an email and a password
  - If the email is missing, return an error `Missing email` with a status code 400
  - If the password is missing, return an error `Missing password` with a status code 400
  - If the email already exists in DB, return an error `Already exist` with a status code 400
  - The password must be stored after being hashed in `SHA1`
  - The endpoint is returning the new user with only the email and the id (auto generated by MongoDB) with a status code 201
  - The new user must be saved in the collection `users`:
    - `email`: same as the value received
    - `password`: `SHA1` value of the value received

```bash
curl 0.0.0.0:5000/users -XPOST -H "Content-Type: application/json" -d '{ "email": "bob@dylan.com", "password": "toto1234!" }' ; echo ""
# Output:
{"id":"5f1e7d35c7ba06511e683b21","email":"bob@dylan.com"}
echo 'db.users.find()' | mongo files_manager
# Output:
{ "_id" : ObjectId("5f1e7d35c7ba06511e683b21"), "email" : "bob@dylan.com", "password" : "89cad29e3ebc1035b29b1478a8e70854f25fa2b2" }
curl 0.0.0.0:5000/users -XPOST -H "Content-Type: application/json" -d '{ "email": "bob@dylan.com", "password": "toto1234!" }' ; echo ""
# Output:
{"error":"Already exist"}
curl 0.0.0.0:5000/users -XPOST -H "Content-Type: application/json" -d '{ "email": "bob@dylan.com" }' ; echo ""
# Output:
{"error":"Missing password"}
```

**Repo**:
- GitHub repository: `alx-files_manager`
- Files: `utils/`, `routes/index.js`, `controllers/UsersController.js`

#### 4. Authenticate a user

In the file `routes/index.js`, add 3 new endpoints:
- `GET /connect` => `AuthController.getConnect`
- `GET /disconnect` => `AuthController.getDisconnect`
- `GET /users/me` => `UserController.getMe`

Inside `controllers`, create a file `AuthController.js` that contains new endpoints:
- `GET /connect` should sign-in the user by generating a new authentication token:
  - By using the header `Authorization` and the technique of the Basic auth (Base64 of the `<email>:<password>`), find the user associated with this email and with this password (reminder: we are storing the SHA1 of the password)
  - If no user has been found, return an error `Unauthorized` with a status code 401
  - Otherwise:
    - Generate a random string (using `uuidv4`) as token
    - Create a key: `auth_<token>`
    - Use this key for storing in Redis (by using the redisClient created previously) the user ID for 24 hours
    - Return this token: `{ "token": "155342df-2399-41da-9e8c-458b6ac52a0c" }` with a status code 200

Inside the file `controllers/UsersController.js`, add a new endpoint:
- `GET /users/me` should retrieve the user based on the token used:
  - Retrieve the user based on the token:
  - If not found, return an error `Unauthorized` with a status code 401
  - Otherwise, return the user object (email and id only)

```bash
curl 0.

0.0.0:5000/users -XPOST -H "Content-Type: application/json" -d '{ "email": "bob@dylan.com", "password": "toto1234!" }' ; echo ""
# Output:
{"id":"5f1e7d35c7ba06511e683b21","email":"bob@dylan.com"}
curl -v 0.0.0.0:5000/connect -u "bob@dylan.com:toto1234!" ; echo ""
# Output:
{ "token": "155342df-2399-41da-9e8c-458b6ac52a0c" }
curl 0.0.0.0:5000/users/me -H "X-Token: 155342df-2399-41da-9e8c-458b6ac52a0c" ; echo ""
# Output:
{ "id": "5f1e7d35c7ba06511e683b21", "email": "bob@dylan.com" }
curl 0.0.0.0:5000/disconnect -H "X-Token: 155342df-2399-41da-9e8c-458b6ac52a0c" ; echo ""
# Output:
{}
curl 0.0.0.0:5000/users/me -H "X-Token: 155342df-2399-41da-9e8c-458b6ac52a0c" ; echo ""
# Output:
{"error":"Unauthorized"}
```

**Repo**:
- GitHub repository: `alx-files_manager`
- Files: `utils/`, `routes/index.js`, `controllers/AuthController.js`, `controllers/UsersController.js`

### 5. First file
In the file `routes/index.js`, add a new endpoint:

POST `/files` => `FilesController.postUpload`
Inside `controllers`, add a file `FilesController.js` that contains the new endpoint:

POST `/files` should create a new file in DB and in disk:

- Retrieve the user based on the token:
  - If not found, return an error `Unauthorized` with a status code 401
- To create a file, you must specify:
  - `name`: as filename
  - `type`: either folder, file or image
  - `parentId`: (optional) as ID of the parent (default: 0 -> the root)
  - `isPublic`: (optional) as boolean to define if the file is public or not (default: false)
  - `data`: (only for type=file|image) as Base64 of the file content
  - If the name is missing, return an error `Missing name` with a status code 400
  - If the type is missing or not part of the list of accepted type, return an error `Missing type` with a status code 400
  - If the data is missing and type != folder, return an error `Missing data` with a status code 400
- If the parentId is set:
  - If no file is present in DB for this parentId, return an error `Parent not found` with a status code 400
  - If the file present in DB for this parentId is not of type folder, return an error `Parent is not a folder` with a status code 400
- The user ID should be added to the document saved in DB - as owner of a file
- If the type is folder, add the new file document in the DB and return the new file with a status code 201
- Otherwise:
  - All file will be stored locally in a folder (to create automatically if not present):
    - The relative path of this folder is given by the environment variable `FOLDER_PATH`
    - If this variable is not present or empty, use `/tmp/files_manager` as storing folder path
  - Create a local path in the storing folder with filename a UUID
  - Store the file in clear (reminder: data contains the Base64 of the file) in this local path
  - Add the new file document in the collection files with these attributes:
    - `userId`: ID of the owner document (owner from the authentication)
    - `name`: same as the value received
    - `type`: same as the value received
    - `isPublic`: same as the value received
    - `parentId`: same as the value received - if not present: 0
    - `localPath`: for a type=file|image, the absolute path to the file save in local
- Return the new file with a status code 201

```
bob@dylan:~$ curl 0.0.0.0:5000/connect -H "Authorization: Basic Ym9iQGR5bGFuLmNvbTp0b3RvMTIzNCE=" ; echo ""
{"token":"f21fb953-16f9-46ed-8d9c-84c6450ec80f"}
bob@dylan:~$ 
bob@dylan:~$ curl -XPOST 0.0.0.0:5000/files -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" -H "Content-Type: application/json" -d '{ "name": "myText.txt", "type": "file", "data": "SGVsbG8gV2Vic3RhY2shCg==" }' ; echo ""
{"id":"5f1e879ec7ba06511e683b22","userId":"5f1e7cda04a394508232559d","name":"myText.txt","type":"file","isPublic":false,"parentId":0}
bob@dylan:~$
bob@dylan:~$ ls /tmp/files_manager/
2a1f4fc3-687b-491a-a3d2-5808a02942c9
bob@dylan:~$
bob@dylan:~$ cat /tmp/files_manager/2a1f4fc3-687b-491a-a3d2-5808a02942c9 
Hello Webstack!
bob@dylan:~$
bob@dylan:~$ curl -XPOST 0.0.0.0:5000/files -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" -H "Content-Type: application/json" -d '{ "name": "images", "type": "folder" }' ; echo ""
{"id":"5f1e881cc7ba06511e683b23","userId":"5f1e7cda04a394508232559d","name":"images","type":"folder","isPublic":false,"parentId":0}
bob@dylan:~$
bob@dylan:~$ cat image_upload.py
import base64
import requests
import sys

file_path = sys.argv[1]
file_name = file_path.split('/')[-1]

file_encoded = None
with open(file_path, "rb") as image_file:
    file_encoded = base64.b64encode(image_file.read()).decode('utf-8')

r_json = { 'name': file_name, 'type': 'image', 'isPublic': True, 'data': file_encoded, 'parentId': sys.argv[3] }
r_headers = { 'X-Token': sys.argv[2] }

r = requests.post("http://0.0.0.0:5000/files", json=r_json, headers=r_headers)
print(r.json())

bob@dylan:~$
bob@dylan:~$ python image_upload.py image.png f21fb953-16f9-46ed-8d9c-84c6450ec80f 5f1e881cc7ba06511e683b23
{'id': '5f1e8896c7ba06511e683b25', 'userId': '5f1e7cda04a394508232559d', 'name': 'image.png', 'type': 'image', 'isPublic': True, 'parentId': '5f1e881cc7ba06511e683b23'}
bob@dylan:~$
bob@dylan:~$ echo 'db.files.find()' | mongo files_manager
{ "_id" : ObjectId("5f1e881cc7ba06511e683b23"), "userId" : ObjectId("5f1e7cda04a394508232559d"), "name" : "images", "type" : "folder", "parentId" : "0" }
{ "_id" : ObjectId("5f1e879ec7ba06511e683b22"), "userId" : ObjectId("5f1e7cda04a394508232559d"), "name" : "myText.txt", "type" : "file", "parentId" : "0", "isPublic" : false, "localPath" : "/tmp/files_manager/2a1f4fc3-687b-491a-a3d2-5808a02942c9" }
{ "_id" : ObjectId("5f1e8896c7ba06511e683b25"), "userId" : ObjectId("5f1e7cda04a394508232559d"), "name" : "image.png", "type" : "image", "parentId" : ObjectId("5f1e881cc7ba06511e683b23"), "isPublic" : true, "localPath" : "/tmp/files_manager/51997b88-5c42-42c2-901e-e7f4e71bdc47" }
bob@dylan:~$
bob@dylan:~$ ls /tmp/files_manager/
2a1f4fc3-687b-491a-a3d2-5808a02942c9   51997b88-5c42-42c2-901e-e7f4e71bdc47
bob@dylan:~$
```
**Repo:**

GitHub repository: alx-files_manager
File: `utils/`, `routes/index.js`, `controllers/FilesController.js`

## 6. Get and List File

#### In the file `routes/index.js`, add 2 new endpoints:
- `GET /files/:id` => `FilesController.getShow`
- `GET /files` => `FilesController.getIndex`

#### In the file `controllers/FilesController.js`, add the 2 new endpoints:
- `GET /files/:id` should retrieve the file document based on the ID:
  - Retrieve the user based on the token:
    - If not found, return an error `Unauthorized` with a status code 401
    - If no file document is linked to the user and the ID passed as parameter, return an error `Not found` with a status code 404
    - Otherwise, return the file document

- `GET /files` should retrieve all users' file documents for a specific `parentId` and with pagination:
  - Retrieve the user based on the token:
    - If not found, return an error `Unauthorized` with a status code 401
    - Based on the query parameters `parentId` and `page`, return the list of file documents:
      - `parentId`:
        - No validation of `parentId` needed - if the `parentId` is not linked to any user folder, return an empty list
        - By default, `parentId` is equal to 0 (the root)
      - Pagination:
        - Each page should be 20 items max
        - `page` query parameter starts at 0 for the first page. If equals to 1, it means it’s the second page (from the 20th to the 40th), etc…
        - Pagination can be done directly by the aggregate of MongoDB

```sh
bob@dylan:~$ curl 0.0.0.0:5000/connect -H "Authorization: Basic Ym9iQGR5bGFuLmNvbTp0b3RvMTIzNCE=" ; echo ""
{"token":"f21fb953-16f9-46ed-8d9c-84c6450ec80f"}
bob@dylan:~$ 
bob@dylan:~$ curl -XGET 0.0.0.0:5000/files -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
[{"id":"5f1e879ec7ba06511e683b22","userId":"5f1e7cda04a394508232559d","name":"myText.txt","type":"file","isPublic":false,"parentId":0},{"id":"5f1e881cc7ba06511e683b23","userId":"5f1e7cda04a394508232559d","name":"images","type":"folder","isPublic":false,"parentId":0},{"id":"5f1e8896c7ba06511e683b25","userId":"5f1e7cda04a394508232559d","name":"image.png","type":"image","isPublic":true,"parentId":"5f1e881cc7ba06511e683b23"}]
bob@dylan:~$
bob@dylan:~$ curl -XGET 0.0.0.0:5000/files?parentId=5f1e881cc7ba06511e683b23 -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
[{"id":"5f1e8896c7ba06511e683b25","userId":"5f1e7cda04a394508232559d","name":"image.png","type":"image","isPublic":true,"parentId":"5f1e881cc7ba06511e683b23"}]
bob@dylan:~$
bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e8896c7ba06511e683b25 -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"id":"5f1e8896c7ba06511e683b25","userId":"5f1e7cda04a394508232559d","name":"image.png","type":"image","isPublic":true,"parentId":"5f1e881cc7ba06511e683b23"}
bob@dylan:~$
```

**Repo:**

GitHub repository: `alx-files_manager`
File: `utils/`, `routes/index.js`, `controllers/FilesController.js`

### 7. File publish/unpublish  
In the file routes/index.js, add 2 new endpoints:

- PUT /files/:id/publish => FilesController.putPublish
- PUT /files/:id/unpublish => FilesController.putUnpublish

In the file controllers/FilesController.js, add the 2 new endpoints:

- **PUT /files/:id/publish** should set `isPublic` to `true` on the file document based on the ID:

  - Retrieve the user based on the token:
    - If not found, return an error Unauthorized with a status code 401
    - If no file document is linked to the user and the ID passed as parameter, return an error Not found with a status code 404
    - Otherwise:
      - Update the value of `isPublic` to `true`
      - And return the file document with a status code 200

- **PUT /files/:id/unpublish** should set `isPublic` to `false` on the file document based on the ID:

  - Retrieve the user based on the token:
    - If not found, return an error Unauthorized with a status code 401
    - If no file document is linked to the user and the ID passed as parameter, return an error Not found with a status code 404
    - Otherwise:
      - Update the value of `isPublic` to `false`
      - And return the file document with a status code 200

```bash
bob@dylan:~$ curl 0.0.0.0:5000/connect -H "Authorization: Basic Ym9iQGR5bGFuLmNvbTp0b3RvMTIzNCE=" ; echo ""
{"token":"f21fb953-16f9-46ed-8d9c-84c6450ec80f"}

bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e8896c7ba06511e683b25 -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"id":"5f1e8896c7ba06511e683b25","userId":"5f1e7cda04a394508232559d","name":"image.png","type":"image","isPublic":false,"parentId":"5f1e881cc7ba06511e683b23"}

bob@dylan:~$ curl -XPUT 0.0.0.0:5000/files/5f1e8896c7ba06511e683b25/publish -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"id":"5f1e8896c7ba06511e683b25","userId":"5f1e7cda04a394508232559d","name":"image.png","type":"image","isPublic":true,"parentId":"5f1e881cc7ba06511e683b23"}

bob@dylan:~$ curl -XPUT 0.0.0.0:5000/files/5f1e8896c7ba06511e683b25/unpublish -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"id":"5f1e8896c7ba06511e683b25","userId":"5f1e7cda04a394508232559d","name":"image.png","type":"image","isPublic":false,"parentId":"5f1e881cc7ba06511e683b23"}
```

**Repo:**

GitHub repository: alx-files_manager  
File: utils/, routes/index.js, controllers/FilesController.js

---

### 8. File data  
In the file routes/index.js, add one new endpoint:

- GET /files/:id/data => FilesController.getFile

In the file controllers/FilesController.js, add the new endpoint:

- **GET /files/:id/data** should return the content of the file document based on the ID:

  - If no file document is linked to the ID passed as parameter, return an error Not found with a status code 404
  - If the file document (folder or file) is not public (`isPublic: false`) and no user is authenticated or not the owner of the file, return an error Not found with a status code 404
  - If the type of the file document is folder, return an error A folder doesn't have content with a status code 400
  - If the file is not locally present, return an error Not found with a status code 404
  - Otherwise:
    - By using the module mime-types, get the MIME-type based on the name of the file
    - Return the content of the file with the correct MIME-type

```bash
bob@dylan:~$ curl 0.0.0.0:5000/connect -H "Authorization: Basic Ym9iQGR5bGFuLmNvbTp0b3RvMTIzNCE=" ; echo ""
{"token":"f21fb953-16f9-46ed-8d9c-84c6450ec80f"}

bob@dylan:~$ curl -XPUT 0.0.0.0:5000/files/5f1e879ec7ba06511e683b22/unpublish -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"id":"5f1e879ec7ba06511e683b22","userId":"5f1e7cda04a394508232559d","name":"myText.txt","type":"file","isPublic":false,"parentId":0"}

bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e879ec7ba06511e683b22/data -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
Hello Webstack!

bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e879ec7ba06511e683b22/data ; echo ""
{"error":"Not found"}

bob@dylan:~$ curl -XPUT 0.0.0.0:5000/files/5f1e879ec7ba06511e683b22/publish -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"id":"5f1e879ec7ba06511e683b22","userId":"5f1e7cda04a394508232559d","name":"myText.txt","type":"file","isPublic":true,"parentId":0"}

bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e879ec7ba06511e683b22/data ; echo ""
Hello Webstack!
```

**Repo:**

GitHub repository: alx-files_manager  
File: utils/, routes/index.js, controllers/FilesController.js

---

### 9. Image Thumbnails
Update the endpoint POST /files endpoint to start a background processing for generating thumbnails for a file of type image:

- Create a Bull queue `fileQueue`
- When a new image is stored (in local and in DB), add a job to this queue with the `userId` and `fileId`
- Create a file `worker.js`:
  - By using the module Bull, create a queue `fileQueue`
  - Process this queue:
    - If `fileId` is not present in the job, raise an error Missing fileId
    - If `userId` is not present in the job, raise an error Missing userId
    - If no document is found in DB based on the `fileId` and `userId`, raise an error File not found
    - By using the module `image-thumbnail`, generate 3 thumbnails with width = 500, 250 and 100 - store each result on the same location of the original file by appending `_`<width size>
- Update the endpoint GET /files/:id/data to accept a query parameter `size`:
  - `size` can be 500, 250 or 100
  - Based on size, return the correct local file
  - If the local file doesn’t exist, return an error Not found with a status code 404

```bash
bob@dylan:~$ npm run start-worker
...

bob@dylan:~$ curl 0.0.0.0:5000/connect -H "Authorization: Basic Ym9iQGR5bGFuLmNvbTp0b3RvMTIzNCE=" ; echo ""
{"token":"f21fb953-16f9-46ed-8d9c

-84c6450ec80f"}

bob@dylan:~$ curl -XPOST 0.0.0.0:5000/files -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" -F "file=@tests/images/large_image.png" ; echo ""
{"id":"5f1e8f97c7ba06511e683b29","userId":"5f1e7cda04a394508232559d","name":"large_image.png","type":"image","isPublic":false,"parentId":0"}

bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e8f97c7ba06511e683b29/data?size=500 -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"error":"Not found"}

bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e8f97c7ba06511e683b29/data?size=250 -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"error":"Not found"}

bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e8f97c7ba06511e683b29/data?size=100 -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"error":"Not found"}

bob@dylan:~$ curl -XPOST 0.0.0.0:5000/files -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" -F "file=@tests/images/small_image.png" ; echo ""
{"id":"5f1e8f97c7ba06511e683b2a","userId":"5f1e7cda04a394508232559d","name":"small_image.png","type":"image","isPublic":false,"parentId":0"}

bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e8f97c7ba06511e683b2a/data?size=500 -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"error":"Not found"}

bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e8f97c7ba06511e683b2a/data?size=250 -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"error":"Not found"}

bob@dylan:~$ curl -XGET 0.0.0.0:5000/files/5f1e8f97c7ba06511e683b2a/data?size=100 -H "X-Token: f21fb953-16f9-46ed-8d9c-84c6450ec80f" ; echo ""
{"error":"Not found"}
```

**Repo:**

GitHub repository: alx-files_manager
File: utils/, controllers/FilesController.js, worker.js

### 10. Tests!
Of course, a strong and stable project cannot be good without tests.

Create tests for `redisClient` and `dbClient`.

Create tests for each endpoint:
- `GET /status`
- `GET /stats`
- `POST /users`
- `GET /connect`
- `GET /disconnect`
- `GET /users/me`
- `POST /files`
- `GET /files/:id`
- `GET /files` (don’t forget the pagination)
- `PUT /files/:id/publish`
- `PUT /files/:id/unpublish`
- `GET /files/:id/data`

**Repo:**
- GitHub repository: `alx-files_manager`
- File: `tests/`

### 11. New User - Welcome Email
Update the endpoint `POST /users` to start background processing for sending a “Welcome email” to the user:

- Create a Bull queue `userQueue`
- When a new user is stored (in DB), add a job to this queue with the `userId`

Update the file `worker.js`:

- By using the module Bull, create a queue `userQueue`
- Process this queue:
  - If `userId` is not present in the job, raise an error `Missing userId`
  - If no document is found in DB based on the `userId`, raise an error `User not found`
  - Print in the console `Welcome <email>!`

In real life, you can use a third-party service like Mailgun to send real email. These APIs are slow, (sending via SMTP is worse!) and sending emails via a background job is important to optimize the API endpoint.

**Repo:**
- GitHub repository: `alx-files_manager`
- File: `utils/`, `worker.js`, `controllers/UsersController.js`
File: utils/, routes/index.js, controllers/FilesController.js, worker.js
