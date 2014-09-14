[![Build Status](https://secure.travis-ci.org/Belphemur/node-json-db.png?branch=master)](http://travis-ci.org/Belphemur/node-json-db) [![Coverage Status](https://img.shields.io/coveralls/Belphemur/node-json-db.svg)](https://coveralls.io/r/Belphemur/node-json-db?branch=master)

[![NPM](https://nodei.co/npm/node-json-db.png?downloads=true&stars=true)](https://nodei.co/npm/node-json-db/)

> A simple "database" that use JSON file for Node.JS.

## Installation
Add `node-json-db` to your existing Node.js project.
```bash
npm install node-json-db --save
```
## Inner Working

###Data
The module store the data using JavaScript Object directly into a JSON file. You can easily traverse the data to reach 
directly the interesting property using the DataPath. The principle of DataPath is the same as XMLPath.

###Example
```javascript
{
    test: {
        data1 : {
            array : ['test','array']
        },
        data2 : 5
    }
}
```
If you want to fet the value of array, the DataPath is **/test/data1/array**
To reach the value of data2 : **/test/data2**
You can of course get also the full object **test** : **/test**
Or even the root : **/**
## Usage
See [test](https://github.com/Belphemur/node-json-db/tree/master/test) for more usage details.


```javascript
var JsonDB = require('node-json-db');
//The second argument is used to tell the DB to save after each push
//If you put false, you'll have to call the save() method.
var db = new JsonDB("myDataBase", true);

//Pushing the data into the database
//With the wanted DataPath
//By default the push will override the old value
db.push("/test1","super test");

//It also create automatically the hierarchy when pushing new data for a DataPath that doesn't exists
db.push("/test2/my/test/",5);

//You can also push directly objects
db.push("/test3", {test:"test", json: {test:["test"]}});

//If you don't want to override the data but to merge them
//The merge is recursive and work with Object and Array.
db.push("/test3", {new:"cool", json: {important : 5}}, false);
/*
This give you this results :
{
   "test":"test",
   "json":{
      "test":[
         "test"
      ],
      "important":5
   },
   "new":"cool"
}
*/
//You can't merge primitive.
//If you do this:
db.push("/test2/my/test/",10,false);
//the data will be overriden

//Get the data from the root
var data = db.getData("/");

//From a particular DataPath
var data = db.getData("/test1");

//If you try to get some data from a DataPath that doesn't exists
//You'll get an Error
try {
var data = db.getData("/test1/test/dont/work");
} catch(error) {
//The error will tell you where the DataPath stopped. In this case test1
//Since /test1/test does't exist.
    console.error(error);
}

//Deleting data
db.delete("/test1");

//Save the data (useful if you disable the saveOnPush)
db.save();

//In case you have a exterior change to the databse file and want to reload it
//use this method
db.reload();

```
### Exception/Error
JsonDB use 2 type of Error/Exception :

| Error         |                   Explanation                                    |
| ------------- |:----------------------------------------------------------------:|
| DataError     | When the error is linked to the Data Given                       | 
| DatabaseError | Linked to a problem with the loading or saving of the Database.  |

####"The Data Path can't be empty" *(DataError)*
The Database expect to minimum receive the root **/** as DataPath.

####"Can't find dataPath: /" + dataPath.join("/") + ". Stopped at " + property *(DataError)*
When the full hierarchy of the DataPath given is not present in the Database.
It tells you until where it's valid. This error can happen when using *getData*
and *delete*

####"Can't merge another type of data with an Array" *(DataError)*
If you chose to not override the data (merging) when pushing and the new data is an array
but the current data isn't an array (an Object by example).

####"Can't merge an Array with an Object" *(DataError)*
Same idea as the previous message. You have an array as current data and ask to 
merge it with an Object.

####"Can't Load Database: " + err *(DatabaseError)*
JsonDB can't load the database for "err" reason.
You can find the nested error in **error.inner**

####"Can't save the database: " + err *(DatabaseError)*
JsonDB can't save the database for "err" reason.
You can find the nested error in **error.inner**

####"DataBase not loaded. Can't write" *(DatabaseError)*
Since the database hasn't been loaded correctly, the module won't let you save the data to avoid erasing your database.



