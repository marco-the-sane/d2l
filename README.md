# What is d2l
data to ddl  - a command line tool deducting the CREATE TABLE statement from a CSV file.
#### How does that work?
Imagine this `titanic_passengers.csv` CSV file ...
```
PassengerId;Survived;Pclass;Name;Sex;Age;SibSp;Parch;Ticket;Fare;Cabin;Embarked
343;No;2;Collander, Mr. Erik Gustaf;male;28.0;0;0;248740;13.0;;S
76;No;3;Moen, Mr. Sigurd Hansen;male;25.0;0;0;348123;7.65;F G73;S
641;No;3;Jensen, Mr. Hans Peder;male;20.0;0;0;350050;7.8542000000000005;;S
672;No;1;Davidson, Mr. Thornton;male;31.0;1;0;F.C. 12750;52.0;B71;S
105;No;3;Gustafsson, Mr. Anders Vilhelm;male;37.0;2;0;3101276;7.925;;S
576;No;3;Patchett, Mr. George;male;19.0;0;0;358585;14.5;;S
382;Yes;3;"Nakid, Miss. Maria (""Mary"")";female;1.0;0;2;2653;15.7417;;C
228;No;3;"Lovell, Mr. John Hall (""Henry"")";male;20.5;0;0;A/5 21173;7.25;;S
```
And you need a table to exactly fit this data.

Try :
```
d2l -coldel:semicolon titanic_passengers.csv
```
To get:
```SQL
CREATE TABLE titanic_passengers (
  passengerid SMALLINT       NOT NULL
, survived    BOOLEAN        NOT NULL
, pclass      SMALLINT       NOT NULL
, name        VARCHAR(51)    NOT NULL
, sex         CHAR(6)        NOT NULL
, age         NUMERIC(3,1)   NOT NULL
, sibsp       SMALLINT       NOT NULL
, parch       SMALLINT       NOT NULL
, ticket      VARCHAR(10)    NOT NULL
, fare        NUMERIC(17,16) NOT NULL
, cabin       CHAR(5)       
, embarked    CHAR(1)        NOT NULL
);
```
`d2l` deducts, by default, the column names from the first line in the file; and from all following columns, it will deduct the most stringent and most effective data type, and, where applicable, the column length, the precision and the scale of numbers. It will also suggest a `DATE` for a datetime literal, like `1957-04-22 00:00:00`, for example, to create an exactly matching standard `CREATE TABLE`statement.
Standard means as close to the standard ANSI format as possible - you might want to replace `INTEGER` with `NUMBER(9)` and `SMALLINT` with `NUMBER(5)` in Oracle, for example.

You use whatever you like for column delimiter, record delimiter, string delimiter and decimal point.
You set them with switches:

 - `-chardel[:]<value>` (default: none, empty string), expecting it to be doubled within the string, for the character delimiter
 - `-coldel[:]<value>` (default: comma) for the column delimiter
 - `-recdel[:]<value>` (default: newline) for the record delimiter
 - `-decpoint[:]<value>` (default: dot) for the decimal point

In all cases, you can set `<value>` in several ways:

 - `SEMI[COLON]|COMMA|BAR|TAB|APO[STROPHE]|QUOTE` case-insensitive keywords
 - `x<hex number>|#<ascii number>|<unix literal>`
 - Just the character.

So you can set the column delimiter to bar in several ways, for example:
 - `-coldelbar`
 - `-coldel:x7c`
 - `-coldel:'\|'` (need to escape the bar character on a Linux command line)

It can assume column delimiters from extensions:
 - For `.bsv`, (bar separated values), the vertical bar
 - For `.ssv`, (semicolon separated values), the semicolon
 - For `.txt`, the horizontal tab `\t`
 - For `.csv`, (comma separated values), and while reading from stdin, and any other file extension, the comma

Other switches are:

 - `-notitle` to say there is no first line with the column names, and the columns will become `col0001, col0002`, etc.
 - `-tbname:<table_name>` to specify a table name differing from the in-file base name.
 - `-colcount:<number>` will parse exactly the specified number of columns, no matter how many column delimiters in the first line.
 - `-verbose` will print the parsed row count every 10,000 rows.
 - `-debug[:<number>]` will print an extended format of each line parsed. Either rather randomly, just `<number>` lines, or, with no number specified, all lines:
 ```
 $ d2l -coldelcomma -debug:1 people.csv 
line 2 of approx 2
id    : [ 42 ]
fname : [ 'Arthur' ]
lname : [ 'Dent' ]
sal   : [ 12500 ]
dob   : [ 2004-02-29 ]

CREATE TABLE people (
   id    SMALLINT NOT NULL
, fname CHAR(8)  NOT NULL
, lname CHAR(9)  NOT NULL
, sal   INTEGER  NOT NULL
, dob   DATE     NOT NULL
);
```
### How to compile
Linux:
```
gcc -O3 -fomit-frame-pointer -Wall -pedantic -std=c99 -D _GNU_SOURCE -o d2l d2l.c
```
Mac:
```
clang -Ofast -Wall -pedantic -o d2l d2l.c
```
