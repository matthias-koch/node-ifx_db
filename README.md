ifx_db: IBM Informix native NodeJS driver
-----------------------------------------

An asynchronous/synchronous interface for node.js to IBM Informix.

install
-------

```bash
SET ENV FOR CSDK_HOME to CSDK install location  
Eg:  
SET CSDK_HOME=C:\mycsdk  
or  
export CSDK_HOME=/work/mycsdk  
  
npm install ifx_db
```

Note:  
The current version of Informix native node driver (ifx_db@4.0.3) is being compiled with Node.JS v4.4.5 LTS libraries. The driver is expected to work node.js version 4x.   
  
Unix/Linux (non Windows) platforms:  
CSDK_HOME environment variable must be set on the shell that you are trying to issue installation command.  
The CSDK_HOME should point to a valid Informix Client SDK distribution with same bit architecture as the NodeJS engine.  
export CSDK_HOME=/work/csdk410x5  



Local Build Prerequisite 
------------------------
* **Informix Client SDK 410 xC2 or above**
* **Node-gyp**
* **Python 2.7.x  (3.x is not supported yet)**

https://www.python.org/   
npm install -g node-gyp  

Local Linux Build 
--------------------
FYI:  
make sure bit architecture matches for all binary components  
If you are using 64bit nodejs make sure you are using 64bit Informix Client-SDK as well.
  
\#Complile time environment setting  
export CSDK_HOME=/work/csdk410x5  
export PATH=/work/nodejs/bin:$PATH  

\#Runtime environment setting  
export INFORMIXDIR=${CSDK_HOME}  
export LD_LIBRARY_PATH=${INFORMIXDIR}/lib/esql:${INFORMIXDIR}/lib/cli  

\#local build  
\#cd ifx_db package directory, say:   
cd /work/user1/node_modules/ifx_db  
  
rm -rf ./build  
node-gyp configure -v  
node-gyp build -v  
  
check the build output, if all right then the driver binary is    
./build/Release/ifx_node_bind.node
  
To run your nodejs JavaScript program  
cd /work/user1  
node SampleApp1.js

Local Windows Build 
----------------------
FYI:  
make sure bit architecture matches for all binary components  
If you are using 64bit nodejs make sure you are using 64bit Informix Client-SDK as well.

Set CSDK_HOME environment variable pointing to Informix Client SDK installation.  
CSDK_HOME=C:\MyCsdk410xC5

Build node.lib:  
The node.lib is needed for compiling native addon,  
one of the way to get node.lib is to build it from NodeJS source (you may either try Node-gyp).  
https://github.com/joyent/node/wiki/installation#installing-on-windows  
Download NodeJS source and do a local build.  

Build ifx_db native addon module:  
set NODE_SRC pointing to NodeJS source  
SET NODE_SRC=C:\njs\Src445  
Yo u may use the Visual Studio 2015 Solution to build from source  
  
or  

\#Command line build  
node-gyp configure  
node-gyp build  
  


Connection String
-----------------

```javascript
var dbobj = require('ifx_db');
var ConnectionString = "SERVER=<IDS ServerName>;DATABASE=<dbname>;HOST=<myhost>;PROTOCOL=<Protocol>;SERVICE=<IDS SQLI Port#>;UID=<UserName>;PWD=<password>;";
//Eg: "SERVER=ids1;DATABASE=mydb1;HOST=BlueGene.ibm.com;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;"
```



Example
-------

```javascript

var dbobj = require('ifx_db');

function DirExec( conn, ErrIgn, sql )
{
	try
	{
		var result = conn.querySync( sql );
		console.log( sql  );
	}
    catch (e) 
	{
		console.log( "--- " + sql  );
		if( ErrIgn != 1 )
		{
			console.log(e);
			console.log();
		}
    }
}

function DoSomeWork(err, conn)
{
    if (err) 
	{
        return console.log(err);
    }
	
	DirExec( conn, 1, "drop table t1" );
	DirExec( conn, 0, "create table t1 ( c1 int, c2 char(20) ) " );
	DirExec( conn, 0, "insert into t1 values( 1, 'val-1' )" );
	DirExec( conn, 0, "insert into t1 values( 2, 'val-2' )" );
	DirExec( conn, 0, "insert into t1 values( 3, 'val-3' )" );
	DirExec( conn, 0, "insert into t1 values( 4, 'val-4' )" );
	DirExec( conn, 0, "insert into t1 values( 5, 'val-5' )" );
  
  console.log(" --- SELECT * FROM t1 ------ " );
  // blocks until the query is completed and all data has been acquired
  var rows = conn.querySync( "SELECT * FROM t1" );
  console.log();
  console.log(rows);
};


var MyAsynchronousTask = function (err, conn)
{
	DoSomeWork(err, conn);
	conn.close();
}

function ifx_db_Open(ConStr) 
{
	console.log(" --- MyAsynchronousTask Starting....." );
	dbobj.open( ConStr, MyAsynchronousTask );
	console.log(" --- Check the sequence printed!" );
}

function ifx_db_OpenSync(ConStr) 
{
	console.log(" --- Executing ifx_db.openSync() ...." );
	var conn;
	try 
	{
	  conn = dbobj.openSync(ConStr);
	}
	catch(e) 
	{
	  console.log(e);
	  return;
	}
	
	DoSomeWork(0, conn);
	
	try 
	{
	    conn.closeSync();
	}
	catch(e) 
	{
	  console.log(e);
	}
	console.log(" --- End ifx_db.openSync()" );
}

function main_func()
{
	//  Make sure the port is IDS SQLI port.
	var ConnectionString = "SERVER=ids1;DATABASE=mydb1;HOST=BlueGene.ibm.com;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";
		
	//Synchronous Execution 
	ifx_db_OpenSync(ConnectionString);
	
	//Asynchronous Execution
	ifx_db_Open(ConnectionString);
}

main_func();

```



Discussion Forums
-----------------
To start a discussion or need help you can post a topic on http://stackoverflow.com/questions/tagged/informix

api
---

### Database

The simple api is based on instances of the `Database` class. You may get an 
instance in one of the following ways:

```javascript
require("ifx_db").open(connectionString, function (err, conn){
  //conn is already open now if err is falsy
});
```

or by using the helper function:

```javascript
var ibmdb = require("ifx_db")();
``` 

or by creating an instance with the constructor function:

```javascript
var Database = require("ifx_db").Database
  , ibmdb = new Database();
```

#### .open(connectionString, [options,] callback)

Open a connection to a database.

* **connectionString** - The connection string for your database
* **options** - _OPTIONAL_ - Object type. Can be used to avoid multiple 
    loading of native ODBC library for each call of `.open`.
* **callback** - `callback (err, conn)`

```javascript
var ibmdb = require("ifx_db");

ibmdb.open(connectionString, function (err, connection) {
    if (err) 
    {
      console.log(err);
      return;
    }
    connection.query("select 1 from mytab1", function (err1, rows) 
    {
      if (err1) console.log(err1);
      else console.log(rows);
      connection.close(function(err2) 
      { 
        if(err2) console.log(err2);
      });
    });
};

```

#### .openSync(connectionString)

Synchronously open a connection to a database.

* **connectionString** - The connection string for your database

```javascript
var ibmdb = require("ifx_db"),
	connString = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

try {
	var conn = ibmdb.openSync(connString);
	conn.query("select * from customers fetch first 10 rows only", function (err, rows, moreResultSets) {
		if (err) {
			console.log(err);
		} else {
		  console.log(rows);
		}
		conn.close();	
	});
} catch (e) {
	console.log(e.message);
}
```

#### .query(sqlQuery [, bindingParameters], callback)

Issue an asynchronous SQL query to the database which is currently open.

* **sqlQuery** - The SQL query to be executed.
* **bindingParameters** - _OPTIONAL_ - An array of values that will be bound to
    any '?' characters in `sqlQuery`.
* **callback** - `callback (err, rows, moreResultSets)`

```javascript
var ibmdb = require("ifx_db"),
   cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

ibmdb.open(cn, function (err, conn) {
	if (err) {
		return console.log(err);
	}

	//we now have an open connection to the database
	//so lets get some data
	conn.query("select * from customers fetch first 10 rows only", function (err, rows, moreResultSets) {
		if (err) {
			console.log(err);
		} else {
		
		  console.log(rows);
		}

		//if moreResultSets is truthy, then this callback function will be called
		//again with the next set of rows.
	});
});
```

#### .querySync(sqlQuery [, bindingParameters])

Synchronously issue a SQL query to the database that is currently open.

* **sqlQuery** - The SQL query to be executed.
* **bindingParameters** - _OPTIONAL_ - An array of values that will be bound to
    any '?' characters in `sqlQuery`.

```javascript
var ibmdb = require("ifx_db"),
   cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

ibmdb.open(cn, function(err, conn){

    //blocks until the query is completed and all data has been acquired
    var rows = conn.querySync("select * from customers fetch first 10 rows only");

    console.log(rows);
})
```

#### .close(callback)

Close the currently opened database.

* **callback** - `callback (err)`

```javascript
var ibmdb = require("ifx_db"),
   cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

ibmdb.open(cn, function (err, conn) {
	if (err) {
		return console.log(err);
	}
	
	//we now have an open connection to the database
	
	conn.close(function (err) {
		console.log("the database connection is now closed");
	});
});
```

#### .closeSync()

Synchronously close the currently opened database.

```javascript
var ibmdb = require("ifx_db")(),
   cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

//Blocks until the connection is open
var conn = ibmdb.openSync(cn);

//Blocks until the connection is closed
conn.closeSync();
```

#### .prepare(sql, callback)

Prepare a statement for execution.

* **sql** - SQL string to prepare
* **callback** - `callback (err, stmt)`

Returns a `Statement` object via the callback

```javascript
var ibmdb = require("ifx_db"),
   cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

ibmdb.open(cn,function(err,conn){
  conn.prepare("insert into hits (col1, col2) VALUES (?, ?)", function (err, stmt) {
    if (err) {
      //could not prepare for some reason
      console.log(err);
      return conn.closeSync();
    }

    //Bind and Execute the statment asynchronously
    stmt.execute(['something', 42], function (err, result) {
      if( err ) console.log(err);  
      else result.closeSync();

      //Close the connection
	  conn.close(function(err){}));
    });
  });
});
```

#### .prepareSync(sql)

Synchronously prepare a statement for execution.

* **sql** - SQL string to prepare

Returns a `Statement` object

```javascript
var ibmdb = require("ifx_db"),
   cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

ibmdb.open(cn,function(err,conn){
  var stmt = conn.prepareSync("insert into hits (col1, col2) VALUES (?, ?)");

  //Bind and Execute the statment asynchronously
  stmt.execute(['something', 42], function (err, result) {
    result.closeSync();

    //Close the connection
	conn.close(function(err){}));
  });
});
```

#### .beginTransaction(callback)

Begin a transaction

* **callback** - `callback (err)`

#### .beginTransactionSync()

Synchronously begin a transaction

#### .commitTransaction(callback)

Commit a transaction

* **callback** - `callback (err)`

```javascript
var ibmdb = require("ifx_db"),
   cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

ibmdb.open(cn, function(err,conn) {

  conn.beginTransaction(function (err) {
    if (err) {
      //could not begin a transaction for some reason.
      console.log(err);
      return conn.closeSync();
    }

    var result = conn.querySync("insert into customer (customerCode) values ('stevedave')");

    conn.commitTransaction(function (err) {
      if (err) {
        //error during commit
        console.log(err);
        return conn.closeSync();
      }

    console.log(conn.querySync("select * from customer where customerCode = 'stevedave'"));

     //Close the connection
     conn.closeSync();
    });
  });
});
```

#### .commitTransactionSync()

Synchronously commit a transaction

```javascript
var ibmdb = require("ifx_db"),
   cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

ibmdb.open(cn, function(err,conn) {

  conn.beginTransaction(function (err) {
    if (err) {
      //could not begin a transaction for some reason.
      console.log(err);
      return conn.closeSync();
    }

    var result = conn.querySync("insert into customer (customerCode) values ('stevedave')");

    conn.commitTransactionSync();

    console.log(conn.querySync("select * from customer where customerCode = 'stevedave'"));

     //Close the connection
    conn.closeSync();
  });
});
```

#### .rollbackTransaction(callback)

Rollback a transaction

* **callback** - `callback (err)`

```javascript
var ibmdb = require("ifx_db"),
   cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

ibmdb.open(cn, function(err,conn) {

  conn.beginTransaction(function (err) {
    if (err) {
      //could not begin a transaction for some reason.
      console.log(err);
      return conn.closeSync();
    }

    var result = conn.querySync("insert into customer (customerCode) values ('stevedave')");

    conn.rollbackTransaction(function (err) {
      if (err) {
        //error during rollback
        console.log(err);
        return conn.closeSync();
      }

    console.log(conn.querySync("select * from customer where customerCode = 'stevedave'"));

     //Close the connection
     conn.closeSync();
    });
  });
});
```

#### .rollbackTransactionSync()

Synchronously rollback a transaction

```javascript
var ibmdb = require("ifx_db")
   cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

ibmdb.open(cn, function(err,conn) {

  conn.beginTransaction(function (err) {
    if (err) {
      //could not begin a transaction for some reason.
      console.log(err);
      return conn.closeSync();
    }

    var result = conn.querySync("insert into customer (customerCode) values ('stevedave')");

    conn.rollbackTransactionSync();

    console.log(conn.querySync("select * from customer where customerCode = 'stevedave'"));

     //Close the connection
    conn.closeSync();
  });
});
```

----------

### Pool

Rudimentary support, rework in progress...... 


#### .open(connectionString, callback)

Get a `Database` instance which is already connected to `connectionString`

* **connectionString** - The connection string for your database
* **callback** - `callback (err, db)`

```javascript
var Pool = require("ifx_db").Pool
	, pool = new Pool()
    , cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

pool.open(cn, function (err, db) {
	if (err) {
		return console.log(err);
	}

	//db is now an open database connection and can be used like normal
	//if we run some queries with db.query(...) and then call db.close();
	//a connection to `cn` will be re-opened silently behind the scense
	//and will be ready the next time we do `pool.open(cn)`
});
```

#### .close(callback)

Close all connections in the `Pool` instance

* **callback** - `callback (err)`

```javascript
var Pool = require("ifx_db").Pool
	, pool = new Pool()
    , cn = "SERVER=ids1;DATABASE=mydb1;HOST=9.25.140.10;PROTOCOL=onsoctcp;SERVICE=5550;UID=user1;PWD=xyz;";

pool.open(cn, function (err, db) {
	if (err) {
		return console.log(err);
	}

	//db is now an open database connection and can be used like normal
	//but all we will do now is close the whole pool
	
	pool.close(function () {
		console.log("all connections in the pool are closed");
	});
});
```
build options
-------------

### Debug

If you would like to enable debugging messages to be displayed you can add the 
flag `DEBUG` to the defines section of the `binding.gyp` file and then execute 
`node-gyp rebuild`.

```javascript
<snip>
'defines' : [
  "DEBUG"
],
<snip>
```
### Unicode

The driver has support for UTF8 strings


```javascript
<snip>
'defines' : [
  "UNICODE"
],
<snip>
```




tips
----
### Using node < v0.10 on Linux

Be aware that through node v0.9 the uv_queue_work function, which is used to 
execute the ODBC functions on a separate thread, uses libeio for its thread 
pool. This thread pool by default is limited to 4 threads.

This means that if you have long running queries spread across multiple 
instances of ifx_db.Database() or using odbc.Pool(), you will only be able to 
have 4 concurrent queries.

You can increase the thread pool size by using @developmentseed's [node-eio]
(https://github.com/developmentseed/node-eio).

#### install: 
```bash
npm install eio
```

#### usage:
```javascript
var eio = require('eio'); 
eio.setMinParallel(threadCount);
```

contributors
------
* IBM
* Sathyanesh Krishnan (msatyan@gmail.com)
* Dan VerWeire (dverweire@gmail.com)
* Lee Smith (notwink@gmail.com)
* Bruno Bigras
* Christian Ensel
* Yorick
* Joachim Kainz
* Oleg Efimov
* paulhendrix


license
-------
Copyright (c) 2015 Sathyanesh Krishnan <msatyan@gmail.com>

Copyright (c) 2014 IBM Corporation <opendev@us.ibm.com>

Copyright (c) 2013 Dan VerWeire <dverweire@gmail.com>

Copyright (c) 2010 Lee Smith <notwink@gmail.com>


Permission is hereby granted, free of charge, to any person obtaining a copy of 
this software and associated documentation files (the "Software"), to deal in 
the Software without restriction, including without limitation the rights to 
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
the Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR 
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR 
COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER 
IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
