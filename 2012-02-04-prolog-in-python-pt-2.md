# Prolog in Python (pt. 2)


**Note:** I've uploaded a basic barebones project based on this series to a [GitHub repository](https://github.com/ryepdx/prolog-mysql-appraise) for your convenience.

[Last time](http://ryepdx.com/2011/09/prolog-in-python-pt-1/ "Prolog in Python
\(pt. 1\)" ) we figured out how to use SWI-Prolog routines in Python. Now we
will learn how to connect SWI-Prolog to MySQL and use it to extend Prolog's
built-in database.

### Connecting and Disconnecting

  
The _[odbc_connect](http://www.swi-
prolog.org/pldoc/doc_for?object=odbc_connect/3)_ predicate will connect you to
your MySQL database. Here I encapsulate it in another predicate for ease of
use throughout the rest of my code.

` connect :- odbc_connect('mysql._your_database_name_', _,  
[ user(_your_username_),  
alias(_database_symbol_),  
password(_your_password_),  
open(once)  
]).  
`

Obviously you will need to fill in your own database's name and your own
username and password. Also, replace _database_symbol_ with the name of the
global symbol you want to bind your database connection to.

The following predicate will disconnect you from your database:

`% Disconnects from the database.  
disconnect :- odbc_disconnect(_database_symbol_).  
`

### Running Queries

  
Now that we can open and close a connection to the database, we can start
passing raw queries to it and getting back values. Here's an example of
pulling data from a row lifted directly from the GitHub project mentioned
above. Assume that we used _db_ for our _database_symbol_.

` % Extracts the values from a row in the Value table.  
parse_value(Row, Thing1, Price, Thing2, Inferred) :- row(Thing1, Price,
Thing2, Inferred) = Row.`

` % Finds the value of Thing1 in terms of Thing2.  
generic_value(Thing1, Price, Thing2, Inferred) :-  
connect,  
odbc_prepare(db, 'select * from value where ((thing1 = ? and thing2 = ?) or
(thing2 = ? and thing1 = ?)) and inferred = ?', [default, default, default,
default, integer], Statement, [fetch(fetch), types([atom, float, atom,
integer])]),  
odbc_execute(Statement, [Thing1, Thing2, Thing2, Thing1, Inferred]),  
odbc_fetch(Statement, Row, next),  
parse_value(Row, _, Price, _, _),  
disconnect.`

The first predicate, _parse_value_, does a little bit of pattern matching to
split the returned row out into its constituent columns.

The second predicate, _generic_value_, does a few things:

1\. It opens the database connection.

2\. It creates a prepared query with _[odbc_prepare](http://www.swi-
prolog.org/pldoc/man?predicate=odbc_prepare%2f5)_. There are a lot of
parameters there, so I'll explain them a bit.

  * First is global symbol we defined in our connect predicate. In this case I used db, though you can use whatever you replaced database_symbol with.
  * Second is the actual SQL query with ? marking where we want our query parameters to go.
  * Third is a list of types for the values we'll be binding to our query parameters.
  * Fourth is the symbol we want to bind the prepared statement to. In this case I bound it to Statement.
  * Fifth is an optional parameter containing a list of options for our query. _fetch_ describes how I want the results to be returned: by default Prolog will grab all the rows returned. By passing in _fetch(fetch)_, I specify that I want only the first row to be returned. _types_ contains a list of types that I want the returned columns to be automatically coerced to. By default Prolog will try to guess.
  

  
3\. It executes the prepared statement with _[odbc_execute](http://www.swi-
prolog.org/pldoc/man?predicate=odbc_execute%2f2)_, which takes a prepared
statement and a list of values.

4\. It fetches the returned row with _[odbc_fetch](http://www.swi-
prolog.org/pldoc/man?predicate=odbc_fetch%2f3)_, which  takes a prepared
statement, binds the returned row to a specified symbol (in this case I used
_Row_), and takes a parameter specifying which row should be returned. In this
case I specified that the "next" row should be returned.

5\. It binds the second column returned to _Price_, discarding the values in
the other columns.

6\. It closes the database connection.

Simple, no? Inserting and deleting data are similarly easy. I'll let you
figure out these examples for yourself. If you have questions, feel free to
leave a comment.

` % Sets the value of Thing1 in terms of Thing2.  
% If inferred is not specified, defaults to true.  
set_value(Thing1, Price, Thing2) :- set_value(Thing1, Price, Thing2, 1).  
set_value(Thing1, RPrice, Thing2, Inferred) :-  
Price is float(RPrice),  
connect,  
odbc_prepare(db, 'insert into value(thing1, price, thing2, inferred) values
(?, ?, ?, ?)', [default, float > decimal, default, integer], Statement),  
odbc_execute(Statement, [Thing1, Price, Thing2, Inferred]),  
disconnect.`

`% Clears all inferred values from the database.  
clear_inferrences :-  
connect,  
odbc_query(db, 'delete from value where inferred'),  
disconnect.`

**Another note:** Did you notice that sneaky _float > decimal_ action up there? All that means is that the decimal value that Prolog will provide when the statement is executed corresponds to a float value in the MySQL table.

