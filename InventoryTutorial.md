# Introduction #

Inventory is a simple warehouse management application. It keeps a list of items and their locations (room + shelf).

You can access the source code here: http://svn.sqljet.com/repos/sqljet/trunk/sqljet-examples/inventory/

Application runs as a web server so you can connect to it using any web browser. Server is multithreaded and each request is processed in its own worker thread. Each thread opens a connection to the inventory.db database if necessary so there may be several connections open at the same time.

There are two versions of the application: the first one allows to edit a list of items and the second one adds a list of users that may borrow items. You may change the version using InventoryDB.VERSION constant. When connection to the database is opened InventoryDB checks the current version and upgrades the schema if necessary.

# Version 1 #

Initially a list of items is displayed:

![http://sqljet.googlecode.com/svn/trunk/images/inventory/list-v1.png](http://sqljet.googlecode.com/svn/trunk/images/inventory/list-v1.png)

Form values are sent to server in query string:

![http://sqljet.googlecode.com/svn/trunk/images/inventory/add-v1.png](http://sqljet.googlecode.com/svn/trunk/images/inventory/add-v1.png)

All requests are logged to console. You may stop the server by typing any text and pressing 'Return'. Here is the log of the above session:

<pre>
Inventory is listening at http://127.0.0.1:8333<br>
Enter any text to shutdown.<br>
GET / HTTP/1.1<br>
GET /favicon.ico HTTP/1.1<br>
GET /add_item HTTP/1.1<br>
GET /add_item?name=Clippers&description=&room=10&shelf=20 HTTP/1.1<br>
GET / HTTP/1.1<br>
GET /edit_item?article=1 HTTP/1.1<br>
GET /edit_item?name=MacBook+Pro&description=Unibody+2GHz&room=7&shelf=23&article=1 HTTP/1.1<br>
GET / HTTP/1.1<br>
GET /?room=7&shelf=23 HTTP/1.1<br>
<br>
Exit.<br>
</pre>

# Version 2 #

List of items now has borrowers:

![http://sqljet.googlecode.com/svn/trunk/images/inventory/list-v2.png](http://sqljet.googlecode.com/svn/trunk/images/inventory/list-v2.png)

And we can see all users:

![http://sqljet.googlecode.com/svn/trunk/images/inventory/users-v2.png](http://sqljet.googlecode.com/svn/trunk/images/inventory/users-v2.png)

# Schema #

Class InventoryDB encapsulates connection to the database and provides convenient accessor methods:

```
public class InventoryDB {

	private static final int VERSION = 1;
	private static final String FILE_NAME = "inventory.db";
	private SqlJetDb db;

	public InventoryDB() throws SqlJetException {
		db = SqlJetDb.open(new File(FILE_NAME), true);
		upgrade(VERSION);
	}

	public void close() throws SqlJetException {
		db.close();
		db = null;
	}

	public int getVersion() throws SqlJetException {
		return db.getOptions().getUserVersion();
	}

	private void upgrade(int targetVersion) throws SqlJetException {
		if (targetVersion < 1) {
			return;
		}
		if (getVersion() < 1) {
			db.runWriteTransaction(new ISqlJetTransaction() {

				public Object run(SqlJetDb db) throws SqlJetException {
					db.createTable("create table items (article integer primary key, "
+ "name text not null, description blob, room int, shelf int, borrowed_from text, borrowed_to text)");
					db.createIndex("create index items_name on items (name asc)");
					db.createIndex("create index items_name_rev on items (name desc)");
					db.createIndex("create index items_location on items (room, shelf)");
					db.getOptions().setUserVersion(1);
					prefillItems();
					return null;
				}
			});
		}
		if (targetVersion < 2) {
			return;
		}
		if (getVersion() < 2) {
			db.runWriteTransaction(new ISqlJetTransaction() {

				public Object run(SqlJetDb db) throws SqlJetException {
					db.createTable("create table users (name text primary key, info text, rating real)");
					db.getOptions().setUserVersion(2);
					prefillUsers();
					return null;
				}
			});
		}
		if (targetVersion > 2) {
			throw new IllegalArgumentException("Unsupported version: " + targetVersion);
		}
	}

	private void prefillItems() throws SqlJetException {
		addItem(new InventoryItem(-1, "MacBook", "Unibody 2GHz", 7, 23, "Dmitry Stadnik", null));
		addItem(new InventoryItem(-1, "iPhone 3G", "8Mb", 7, 24, "Dmitry Stadnik", null));
		addItem(new InventoryItem(-1, "Cup", "Big & White", 3, 1, null, "MG"));
	}

	private void prefillUsers() throws SqlJetException {
		addUser(new InventoryUser("Dmitry Stadnik", "Prague", 0.99));
		addUser(new InventoryUser("James Bond", "Classified", 0));
		addUser(new InventoryUser("MG", null, 0.11));
	}

	// Items

	public ISqlJetCursor getAllItems() throws SqlJetException {...}
	public ISqlJetCursor getAllItemsInRoomOnShelf(long room, long shelf) throws SqlJetException {...}
	public ISqlJetCursor getAllItemsSortedByName(boolean asc) throws SqlJetException {...}
	public InventoryItem getItem(long article) throws SqlJetException {...}
	public long addItem(final InventoryItem item) throws SqlJetException {...}
	public void updateItem(final long article, final Map<String, Object> values) throws SqlJetException {...}
	public void removeItem(final long article) throws SqlJetException {...}

	// Users

	public ISqlJetCursor getAllUsers() throws SqlJetException {...}
	public InventoryUser getUser(String name) throws SqlJetException {...}
	public void addUser(final InventoryUser user) throws SqlJetException {...}
	public void updateUser(final String name, final Map<String, Object> values) throws SqlJetException {...}
	public void removeUser(final String name) throws SqlJetException {...}
}
```

User version is effectively a logical version of a database layout. It is written in database file and application may check it and upgrade database if necessary. In Inventory application method upgrade() does exactly this thing: desired database version is passed as argument and database is upgraded to this version.

Items table has article column declared as 'integer primary key'. This particular combination of 'integer' type and 'primary key' constraint allows SqlJet to use row ids as the column values. In other words for the items table cursor.getRowId() will always be equal to cursor.getValue("article").
```
db.createTable("create table items (article integer primary key, "
+ "name text not null, description blob, room int, shelf int, borrowed_from text, borrowed_to text)");
```

Two indexes 'items\_name' and 'items\_name\_rev' allow to sort items by name in ascending and descending order. These indexes do not require item names to be unique.
```
db.createIndex("create index items_name on items (name asc)");
db.createIndex("create index items_name_rev on items (name desc)");
```

Index 'items\_location' allows to find items in particular room on particular shelf.
```
db.createIndex("create index items_location on items (room, shelf)");
```