## Contents ##

This page provides very simple tutorial on how to use SQLJet API to work with the database of SQLite format. The following operations are described:

  * Create new database and configure options
  * Create table and indices
  * Insert records
  * Select records in order specified by index
  * Lookup records matching scope or exact value
  * Update and delete records
  * Drop table and index

Full working source code of this example is available as part of SQLJet project in Subversion repository at [Tutorial.java](http://svn.sqljet.com/repos/sqljet/trunk/sqljet-examples/simple/src/org/tmatesoft/sqljet/examples/tutorial/Tutorial.java).

## Create new database ##

```
00  File dbFile = new File(DB_NAME);
01  dbFile.delete();
02        
03  SqlJetDb db = SqlJetDb.open(dbFile, true);
04  db.getOptions().setAutovacuum(true);
05  db.beginTransaction(SqlJetTransactionMode.WRITE);
06  try {
07    db.getOptions().setUserVersion(1);
08  } finally {
09    db.commit();
10  }
```

For the sake of atomicity this example always creates new empty data base (_lines 0,1_), then `SqlJetDb` object is created for that file (_line 3_), opened for writing. As file does not yet exist it will be created.

Sqlite format supports number of options. Some of these options have to be set before anything is changed in the database, even before first transaction is started (_line 4_), because exact way transaction is executed depends on these very options. Other options should be set inside "write" transaction (_line 7_).

There are basically two ways to execute certain code as a transaction. First is described above (_lines 5 and 9_) - write transaction is started and then committed. To roll back a transaction one should call `db.rollback()` instead of `db.commit()`, for instance in case exception is thrown from the try/catch block.

Another way is to subclass `SqlJetTransaction` class and run it with `SqlJetDb.runTransaction(...)` method:

```
 Object result = db.runTransaction(new ISqlJetTransaction() {
   public Object run(SqlJetDb db) throws SqlJetException {
     db.getOptions().setUserVersion(1);
     return true;
   }
 } SqlJetTransactionMode.WRITE);
```

Above method is more convenient in a sense that transaction will be automatically rolled back in case exception is thrown from the `run` method or committed in case there were no exceptions. On the other side usage of anonymous or inner classes might be inconvenient and then one could prefer the first way to run transaction. In this example we will use first way to save on indentation and curly brackets.

Note, that when you no longer need to work with the database it makes sense to _close_ it by calling `SqlJetDb.close()` method:

```
  SqlJetDb db = SqlJetDb.open(dbFile, true);
  try {
    ...
    ...
  } finally {
    db.close();
  }
```

## Create table and indices ##

We will create one table with three fields and two indices. Third index (for primary key field) will be created automatically. In SQLite format database schema is stored as plain SQL statements and similar statements are used to create tables and indices.

We are using the following statements:

```
CREATE TABLE employees (second_name TEXT NOT NULL PRIMARY KEY , first_name TEXT NOT NULL, date_of_birth INTEGER NOT NULL)
CREATE INDEX full_name_index ON employees(first_name,second_name)
CREATE INDEX dob_index ON employees(date_of_birth)
```

And the following code:

```
  db.beginTransaction(SqlJetTransactionMode.WRITE);
  try {            
    db.createTable(createTableQuery);
    db.createIndex(createFirstNameIndexQuery);
    db.createIndex(createDateIndexQuery);
  } finally {
    db.commit();
  }
```

First index, `full_name_index` is a composite one - it indexes rows by values of two fields - first\_name and second\_name. This means that searching using two values (first name and second name) will use this index and will work fast.

`dob_index` is a simple index of integer type field. SQLite "integer" is always represented as signed long in Java. Here we use long value type to store dates.

Finally, SQLJet will create one more index, because one of the table fields (`second_name`) is declared as `PRIMARY KEY`. This index will be names `sqlite_autoindex_employees_1` and this name will be available later, so that we will use this index as well.

Note, that database schema is created in a write transaction.

## Insert records ##

Now let fill our `employees` table we've just created:

```
  Calendar calendar = Calendar.getInstance();
  calendar.clear();

  db.beginTransaction(SqlJetTransactionMode.WRITE);
  try {
    ISqlJetTable table = db.getTable(TABLE_NAME);
    calendar.set(1981, 4, 19);
    table.insert("Prochaskova", "Elena", calendar.getTimeInMillis());
    calendar.set(1967, 5, 19);
    table.insert("Scherbina", "Sergei", calendar.getTimeInMillis());
    calendar.set(1987, 6, 19);
    table.insert("Vadishev", "Semen", calendar.getTimeInMillis());
    calendar.set(1982, 7, 19);
    table.insert("Sinjushkin", "Alexander", calendar.getTimeInMillis());
    calendar.set(1979, 8, 19);
    table.insert("Stadnik", "Dmitry", calendar.getTimeInMillis());
    calendar.set(1977, 9, 19);
    table.insert("Kitaev", "Alexander", calendar.getTimeInMillis());
  } finally {
    db.commit();
  }
```

Code above is pretty straightforward: we fetch table by name (`employees`), then call  `table.insert(...)` method passing values of all fields for each row. These fields are `second_name`, `first_name` and finally `date_of_birth`.

SQLJet updates indices automatically on any modifications done to the tables, so there is no need to call other methods.

## Select records in order specified by index ##

Before looking at the code that selects records from the table, lets introduce utility method that simplifies displaying of those records. This method accepts `ISqlJetCursor` - object of iterator type that represents ordered set of rows - and prints out those rows:

```
  private static void printRecords(ISqlJetCursor cursor) throws SqlJetException {
    try {
      if (!cursor.eof()) {
        do {
          System.out.println(cursor.getRowId() + " : " + 
                             cursor.getString(FIRST_NAME_FIELD) + " " + 
                             cursor.getString(SECOND_NAME_FIELD) + " was born on " + 
                             formatDate(cursor.getInteger(DOB_FIELD)));
         } while(cursor.next());
      }
    } finally {
      cursor.close();
    }
  }
```

This utility method iterates over ordered row set using `cursor.next()` method until cursor points behind the last row in the ordered set - `cursor.next()` return `false` and `cursor.eof()` returns `true`.

At every particular moment of its lifetime, cursor points to one of the rows in the ordered set it represents (or to `null` row in case end of the ordered row set has been reached) and allows user to get fields values for the very row it points to. Additionally to the fields defined by schema every row has `rowId` - unique long integer which is, by default, equal to the row number (1-based).

When cursor is no longer needed, `cursor.close()` method will free associated resources and will make that cursor instance invalid.

Now, when utility method has been introduced it is easy to write code that selects records and print then out:

```
  db.beginTransaction(SqlJetTransactionMode.READ_ONLY);
  try {
    printRecords(table.order(table.getPrimaryKeyIndexName()));
  } finally {
    db.commit();
  }
```

Method `table.order(String indexName)` returns all rows in the table in order defined by the index specified. In this case we use index that has been automatically created for the primary key field.

Note, that we run above code in a `READ_ONLY` transaction. This helps us to make sure that no concurrent write operation influence our row set.

Other examples on how rows might be selected:

```
  db.beginTransaction(SqlJetTransactionMode.READ_ONLY);
  try {
    printRecords(table.order(FULL_NAME_INDEX));
  } finally {
    db.commit();
  }
```
- in order defined by a composite index, i.e. sorted by a concatenation of a first\_name and second\_name field values.

```
  db.beginTransaction(SqlJetTransactionMode.READ_ONLY);
  try {
    printRecords(table.open());
  } finally {
    db.commit();
  }
```
- in order defined by the `rowId`, i.e. sorted in order rows were added to the table.

```
  db.beginTransaction(SqlJetTransactionMode.READ_ONLY);
  try {
    printRecords(table.order(DOB_INDEX).reverse());
  } finally {
    db.commit();
  }
```
- from records with less value in `date_of_birth` field to those with greater values.

Note the use of `ISqlJetCursor.reverse()` method - it 'reverses' the cursor returning its mirrored copy that will iterate rows in the opposite order. 'Reversed' cursor wraps original cursor, so that later changes its position when former is iterated. It is enough to close reversed cursor to close original one as well.

## Lookup records matching scope or exact value ##

Similar cursor-based approach is used to select only certain records - those that match certain criteria. The difference is that `table.lookup(indexName, ...)` method is used instead of `table.order(indexName, ...)`.

`table.lookup(...)` method accepts index name and field values to select records. It is easy to understand this looking at examples:

```
  db.beginTransaction(SqlJetTransactionMode.READ_ONLY);
  try {
    printRecords(table.lookup(FULL_NAME_INDEX, "Alexander"));
  } finally {
    db.commit();
  }
```
- gets all records with first part of  `full_name_index` equal to 'Alexander'. This prints out two records:
```
6 : Alexander Kitaev was born on Oct 19, 1977
4 : Alexander Sinjushkin was born on Aug 19, 1982
```

And with stricter criteria:

```
  db.beginTransaction(SqlJetTransactionMode.READ_ONLY);
  try {
    printRecords(table.lookup(FULL_NAME_INDEX, "Alexander", "Kitaev"));
  } finally {
    db.commit();
  }
```
- gets all records with both parts of `full_name_index` specified. This prints out single record:
```
6 : Alexander Kitaev was born on Oct 19, 1977
```

Note, that currently SQLJet only allows to search for a string (specifying beginning of it) using indices, not _inside_ the string (specifying part of it or regular expression). This functionality will be available in the next versions of SQLJet.

Other way to select records matching criteria is to specify `scope`, not exact field values. It is possible to do with the help of `table.scope(...)` method that takes index name, range start and end values and returns our old friend `cursor`:

```
  db.beginTransaction(SqlJetTransactionMode.READ_ONLY);
  try {
    printRecords(table.scope(FULL_NAME_INDEX, new Object[] {"B"}, new Object[] {"I"}));
 } finally {
    db.commit();
  }
```

- prints all records with `full_name_index` (which is a composite of `first_name` and `second_name`) in range from B to I inclusive. Here it means that all employees with first name starting with letter B to I will be selected.

```
  Calendar calendar = Calendar.getInstance();
  calendar.setTime(new Date(System.currentTimeMillis()));
  calendar.add(Calendar.YEAR, -30);

  db.beginTransaction(SqlJetTransactionMode.READ_ONLY);
  try {
    printRecords(table.scope(DOB_INDEX, 
                     new Object[] {Long.MIN_VALUE}, 
                     new Object[] {calendar.getTimeInMillis()}));
 } finally {
    db.commit();
 }
```

- prints all records with `date_of_birth` value in scope between `Long.MIN_VALUE` and data thirty years ago from now.

## Update and delete records ##

To modify (update) or delete records SQLJet uses the following algorithm:

  1. Start `WRITE` transaction.
  1. Select rows you'd like to modify or delete, in other words get a `cursor`.
  1. Iterate over cursor updating or deleting rows as you go.

Example below deletes records of all employees who are older than thirty years old (_lines 07:13_). Then it adds one more record (_line 16_)and changes `date_of_birth` field value for all records in the table (_lines 18:27_)):

```
00  Calendar calendar = Calendar.getInstance();
01  calendar.setTime(new Date(System.currentTimeMillis()));
02  calendar.add(Calendar.YEAR, -30);
03
04  db.beginTransaction(SqlJetTransactionMode.WRITE);
05  try {
06    // delete
07    ISqlJetCursor deleteCursor = table.scope(DOB_INDEX, 
08                                   new Object[] {Long.MIN_VALUE}, 
09                                   new Object[] {calendar.getTimeInMillis()});
10    while (!deleteCursor.eof()) {
11      deleteCursor.delete();
12    }
13    deleteCursor.close();
14
15    // insert
16    table.insert("Smith", "John", 0);
17
18    // update
19    calendar.setTime(new Date(System.currentTimeMillis()));
20    ISqlJetCursor updateCursor = table.open();
21    do {
22       updateCursor.update(
23             updateCursor.getValue(SECOND_NAME_FIELD), 
24             updateCursor.getValue(FIRST_NAME_FIELD), 
25             calendar.getTimeInMillis());
26    } while(updateCursor.next());
27    updateCursor.close();
28  } finally {
29    db.commit();
30  }
```

Code above is ran, of course, as a `WRITE` transaction and similar to `table.insert(...)`, delete and update methods does all necessary updates to indices.

## Drop table and indices ##

To drop (delete) table and indices use `SqlJetDb.dropTable(String tableName)` and `SqlJetDb.dropIndex(String indexName)` methods.

This is pretty clear and more interesting is how to figure out what tables and indices are contained in particular database. SQLJet provides an API to read database schema and fetching names of all tables and indices is easy:

```
  db.beginTransaction(SqlJetTransactionMode.WRITE);
  try {      
   Set<String> indices = db.getSchema().getIndexNames();
   Set<String> tables = db.getSchema().getTableNames();
   for (String tableName : tables) {
      ISqlJetTableDef tableDef = db.getSchema().getTable(tableName);
      Set<ISqlJetIndexDef> tableIndices = db.getSchema().getIndexes(tableName);
      for (ISqlJetIndexDef indexDef : tableIndices) {
         if (!indexDef.isImplicit()) {
           db.dropIndex(indexDef.getName());
         }
      }
      db.dropTable(tableName);
    }
  } finally {
    db.commit();
  }
```

Above code gets names of all tables stored in the database and list of indices for each table, then drops those indices and tables. It is not necessary to drop first indices and then table - dropping table deletes indices automatically. `ISqlJetTableDef` and `ISqlJetIndexDef` objects provides detailed information of table and index including all names, fields and their types.