# OrientJS driver

Official [orientdb](http://www.orientechnologies.com/orientdb/) driver for node.js. Fast, lightweight, uses the binary protocol.

[![Build Status](https://travis-ci.org/orientechnologies/orientjs.svg?branch=master)](https://travis-ci.org/orientechnologies/orientjs)


# Supported Versions

OrientJS aims to work with version 2.0.0 of OrientDB and later. While it may work with earlier versions, they are not currently supported, [pull requests are welcome!](./CONTRIBUTING.md)

> **IMPORTANT**: OrientJS does not currently support OrientDB's Tree Based [RIDBag](https://github.com/orientechnologies/orientdb/wiki/RidBag) feature because it relies on making additional network requests.
> This means that by default, the result of e.g. `JSON.stringify(record)` for a record with up to 119 edges will be very different from a record with 120+ edges.
> This can lead to very nasty surprises which may not manifest themselves during development but could appear at any time in production.
> There is an [open issue](https://github.com/orientechnologies/orientdb/issues/2315) for this in OrientDB, until that gets fixed, it is **strongly recommended** that you set `RID_BAG_EMBEDDED_TO_SBTREEBONSAI_THRESHOLD` to a very large value, e.g. 2147483647.
> Please see the [relevant section in the OrientDB manual](http://www.orientechnologies.com/docs/2.0/orientdb.wiki/RidBag.html#configuration) for more information.

# Installation

Install via npm.

```sh
npm install orientjs
```

To install OrientJS globally use the `-g` option:

```sh
npm install orientjs -g
```

# Running Tests

To run the test suite, first invoke the following command within the repo, installing the development dependencies:

```sh
npm install
```

Then run the tests:

```sh
npm test
```


# Features

- Tested with latest OrientDB (2.0.x and 2.1).
- Intuitive API, based on [bluebird](https://github.com/petkaantonov/bluebird) promises.
- Fast binary protocol parser.
- Access multiple databases via the same socket.
- Migration support.
- Simple CLI.
- Connection Pooling



# Table of Contents

* [Configuring the Client](#configuring-the-client)
* [Server API](#server-api)
* [Database API](#database-api)
  * [Record API](#record-api)
  * [Class API](#class-api)
  * [Query](#query)
  * [Query Builder](#query-builder)   




## Configuring the Client
```js
var OrientDB = require('orientjs');

var server = OrientDB({
  host: 'localhost',
  port: 2424,
  username: 'root',
  password: 'yourpassword'
});
```

```js
// CLOSE THE CONNECTION AT THE END
server.close();
```


## Server API

### Listing the databases on the server

```js
server.list()
.then(function (dbs) {
  console.log('There are ' + dbs.length + ' databases on the server.');
});
```

### Creating a new database

```js
server.create({
  name: 'mydb',
  type: 'graph',
  storage: 'plocal'
})
.then(function (db) {
  console.log('Created a database called ' + db.name);
});
```

### Using an existing database

```js
var db = server.use('mydb');
console.log('Using database: ' + db.name);

// CLOSE THE CONNECTION AT THE END
db.close();
```

### Using an existing database with credentials

```js
var db = server.use({
  name: 'mydb',
  username: 'admin',
  password: 'admin'
});
console.log('Using database: ' + db.name);
// CLOSE THE CONNECTION AT THE END
db.close();
```


## Database API

### Record API

#### Loading a record by RID.

```js
db.record.get('#1:1')
.then(function (record) {
  console.log('Loaded record:', record);
});
```

#### Deleting a record

```js
db.record.delete('#1:1')
.then(function () {
  console.log('Record deleted');
});
```
### Class API

#### Listing all the classes in the database

```js
db.class.list()
.then(function (classes) {
  console.log('There are ' + classes.length + ' classes in the db:', classes);
});
```

#### Creating a new class

```js
db.class.create('MyClass')
.then(function (MyClass) {
  console.log('Created class: ' + MyClass.name);
});
```

#### Creating a new class that extends another

```js
db.class.create('MyOtherClass', 'MyClass')
.then(function (MyOtherClass) {
  console.log('Created class: ' + MyOtherClass.name);
});
```

#### Getting an existing class

```js
db.class.get('MyClass')
.then(function (MyClass) {
  console.log('Got class: ' + MyClass.name);
});
```

#### Updating an existing class

```js
db.class.update({
  name: 'MyClass',
  superClass: 'V'
})
.then(function (MyClass) {
  console.log('Updated class: ' + MyClass.name + ' that extends ' + MyClass.superClass);
});
```

#### Listing properties in a class

```js
MyClass.property.list()
.then(function (properties) {
  console.log('The class has the following properties:', properties);
});
```

#### Adding a property to a class

```js
MyClass.property.create({
  name: 'name',
  type: 'String'
})
.then(function () {
  console.log('Property created.')
});
```

To add multiple properties, pass an array of objects. Example:

```js
MyClass.property.create([{
  name: 'name',
  type: 'String'
}, {
  name: 'surname',
  type: 'String'
}])
.then(function () {
  console.log('Property created.')
});
```

#### Deleting a property from a class

```js
MyClass.property.drop('myprop')
.then(function () {
  console.log('Property deleted.');
});
```

#### Renaming a property on a class

```js
MyClass.property.rename('myprop', 'mypropchanged');
.then(function () {
  console.log('Property renamed.');
});
```

#### Creating a record for a class

```js
MyClass.create({
  name: 'John McFakerton',
  email: 'fake@example.com'
})
.then(function (record) {
  console.log('Created record: ', record);
});
```

#### Listing records in a class

```js
MyClass.list()
.then(function (records) {
  console.log('Found ' + records.length + ' records:', records);
});
```

### Query

#### Execute an Insert Query

```js
db.query('insert into OUser (name, password, status) values (:name, :password, :status)',
  {
    params: {
      name: 'Radu',
      password: 'mypassword',
      status: 'active'
    }
  }
).then(function (response){
  console.log(response); //an Array of records inserted
});

```


#### Execute a Select Query with Params

```js
db.query('select from OUser where name=:name', {
  params: {
    name: 'Radu'
  },
  limit: 1
}).then(function (results){
  console.log(results);
});

```

##### Raw Execution of a Query String with Params

```js
db.exec('select from OUser where name=:name', {
  params: {
    name: 'Radu'
  }
}).then(function (response){
  console.log(response.results);
});

```

### Query Builder

#### Query Builder: Insert Record

```js
db.insert().into('OUser').set({name: 'demo', password: 'demo', status: 'ACTIVE'}).one()
.then(function (user) {
  console.log('created', user);
});
```

#### Query Builder: Update Record

```js
db.update('OUser').set({password: 'changed'}).where({name: 'demo'}).scalar()
.then(function (total) {
  console.log('updated', total, 'users');
});
```

#### Query Builder: Delete Record

```js
db.delete().from('OUser').where({name: 'demo'}).limit(1).scalar()
.then(function (total) {
  console.log('deleted', total, 'users');
});
```


#### Query Builder: Select Records

```js
db.select().from('OUser').where({status: 'ACTIVE'}).all()
.then(function (users) {
  console.log('active users', users);
});
```

#### Query Builder: Text Search

```js
db.select().from('OUser').containsText({name: 'er'}).all()
.then(function (users) {
  console.log('found users', users);
});
```

#### Query Builder: Select Records with Fetch Plan

```js
db.select().from('OUser').where({status: 'ACTIVE'}).fetch({role: 5}).all()
.then(function (users) {
  console.log('active users', users);
});
```

#### Query Builder: Select an expression

```js
db.select('count(*)').from('OUser').where({status: 'ACTIVE'}).scalar()
.then(function (total) {
  console.log('total active users', total);
});
```

#### Query Builder: Traverse Records

```js
db.traverse().from('OUser').where({name: 'guest'}).all()
.then(function (records) {
  console.log('found records', records);
});
```

#### Query Builder: Return a specific column

```js
db
.select('name')
.from('OUser')
.where({status: 'ACTIVE'})
.column('name')
.all()
.then(function (names) {
  console.log('active user names', names.join(', '));
});
```


#### Query Builder: Transform a field

```js
db
.select('name')
.from('OUser')
.where({status: 'ACTIVE'})
.transform({
  status: function (status) {
    return status.toLowerCase();
  }
})
.limit(1)
.one()
.then(function (user) {
  console.log('user status: ', user.status); // 'active'
});
```


#### Query Builder: Transform a record

```js
db
.select('name')
.from('OUser')
.where({status: 'ACTIVE'})
.transform(function (record) {
  return new User(record);
})
.limit(1)
.one()
.then(function (user) {
  console.log('user is an instance of User?', (user instanceof User)); // true
});
```


#### Query Builder: Specify default values

```js
db
.select('name')
.from('OUser')
.where({status: 'ACTIVE'})
.defaults({
  something: 123
})
.limit(1)
.one()
.then(function (user) {
  console.log(user.name, user.something);
});
```


#### Query Builder: Put a map entry into a map

```js
db
.update('#1:1')
.put('mapProperty', {
  key: 'value',
  foo: 'bar'
})
.scalar()
.then(function (total) {
  console.log('updated', total, 'records');
});
```





