# Introduction #

'Spending' is a simple application to manage personal expenses. It's based on a list of payments that user can browse and modify. All payments are stored in 'spending.db' database, in the 'payments' table.

![http://sqljet.googlecode.com/svn/trunk/images/Spending.png](http://sqljet.googlecode.com/svn/trunk/images/Spending.png)

You can access the source code here: http://code.google.com/p/sqljet/source/browse/#svn/trunk/spending

# Schema #

Each payment is composed from the following attributes:
  * **date** Date when money were spent or received.
  * **amount** Amount of received money; if this number is negative then money were spent.
  * **currency** Payment currency; three letters.
  * **info** What was bought or where the money came from.

Class 'Payment' represents a payment and has method `read()` to extract payment attributes from a database cursor:

```
public class Payment {

	public Date date;
	public long amount;
	public String currency, info;

	public Payment() {
	}

	public Payment(Date date, long amount, String currency, String info) {
		this.date = date;
		this.amount = amount;
		this.currency = currency;
		this.info = info;
	}

	public void read(ISqlJetCursor cursor) throws SqlJetException {
		date = readDate(cursor);
		amount = cursor.getInteger("amount");
		currency = cursor.getString("currency");
		info = cursor.getString("info");
	}

	private Date readDate(ISqlJetCursor cursor) throws SqlJetException {
		String value = cursor.getString("date");
		try {
			return SpendingDB.getDateFormat().parse(value);
		} catch (ParseException pe) {
			return null;
		}
	}
}
```

Record values may also be read by field indexes but using field names is preferred solution - this makes application more tolerant to changes in schema. Table 'payments' keeps these records.

Class 'SpendingDB' encapsulates connection to the 'spending.db' file. Call `open()` to establish connection when application is launched and `close()` before exit. Note that we open database for writing and if the 'payments' table does not exists then we create it. This happens when application is launched for the first time and there is no database yet so we have to recreate all the schema. Another approach for more complex applications is to version schema with numbers and store schema version in 'user version' option. It can be accessed as `SqlJetDB.getOptions().[get|set]UserVersion()` methods.

```
public class SpendingDB {

	private static final String FILE_NAME = "spending.db";
	private static final String DF_PATTERN = "yyyy-MM-dd";

	private static SqlJetDb db;

	public static DateFormat getDateFormat() {
		return new SimpleDateFormat(DF_PATTERN, Locale.ENGLISH);
	}

	public static void open() throws SqlJetException {
		db = SqlJetDb.open(new File(FILE_NAME), true);
		if (db.getSchema().getTable("payments") == null) {
			db.runWriteTransaction(new ISqlJetTransaction() {

				public Object run(SqlJetDb arg0) throws SqlJetException {
					db.getSchema().createTable(
"create table payments (date text not null, amount int not null, currency text, info text)");
					db.getSchema().createIndex(
"create index payments_date on payments (date)");
					prefillDB();
					return null;
				}
			});
		}
	}

	public static void close() throws SqlJetException {
		db.close();
		db = null;
	}

	// ...

	private static void prefillDB() throws SqlJetException {
		Calendar cal = Calendar.getInstance();
		cal.set(2009, 6, 27);
		addPayment(new Payment(cal.getTime(), -199, "USD", "New iPhone"));
		cal.set(2009, 6, 28);
		addPayment(new Payment(cal.getTime(), -999, "USD", "New MacBook"));
		cal.set(2009, 6, 29);
		addPayment(new Payment(cal.getTime(), 2000, "EUR", "Salary"));
	}
}
```

Other methods in 'SpendingDB' class allow to select payments and modify them. To retrieve all records from a table we call `open()` method for the table. Returned cursor must be closed by clients after use.

```
	public static ISqlJetCursor getAllPayments() throws SqlJetException {
		return db.getTable("payments").open();
	}
```

Method `lookup(indexName, values)` makes it possible to select records using index. The second argument - values - is a key for the search. We use 'payments\_date' index where the key is the payment date.

```
	public static ISqlJetCursor getPayments(Date date) throws SqlJetException {
		String dateString = getDateFormat().format(date);
		return db.getTable("payments").lookup("payments_date", dateString);
	}
```

Every record within a table has an implicit 'rowid' attribute that is a unique number associated with the record. You can get it from a cursor for the current record using `getRowId()` method. Here we provide a method to access a particular payment by its 'rowid'.

```
	public static Payment getPayment(long rowid) throws SqlJetException {
		ISqlJetCursor cursor = db.getTable("payments").open();
		try {
			if (cursor.goTo(rowid)) {
				Payment p = new Payment();
				p.read(cursor);
				return p;
			}
		} finally {
			cursor.close();
		}
		return null;
	}
```

Record may be added to a table using its `insert(values)` method that accepts a list of all field values. In order to update or remove a record we first position a cursor over the particular record by using its 'rowid' and then calling `update(values)` or `delete()`.

```
	public static long addPayment(final Payment p) throws SqlJetException {
		return (Long) db.runWriteTransaction(new ISqlJetTransaction() {

			public Object run(SqlJetDb db) throws SqlJetException {
				String dateString = getDateFormat().format(p.date);
				return db.getTable("payments").insert(dateString, p.amount,
						p.currency, p.info);
			}
		});
	}

	public static void updatePayment(final long rowid, final Payment p)
			throws SqlJetException {
		db.runWriteTransaction(new ISqlJetTransaction() {

			public Object run(SqlJetDb db) throws SqlJetException {
				ISqlJetCursor cursor = db.getTable("payments").open();
				try {
					if (cursor.goTo(rowid)) {
						String dateString = getDateFormat().format(p.date);
						cursor.update(dateString, p.amount, p.currency, p.info);
					}
				} finally {
					cursor.close();
				}
				return null;
			}
		});
	}

	public static void removePayment(final long rowid) throws SqlJetException {
		db.runWriteTransaction(new ISqlJetTransaction() {

			public Object run(SqlJetDb db) throws SqlJetException {
				ISqlJetCursor cursor = db.getTable("payments").open();
				try {
					if (cursor.goTo(rowid)) {
						cursor.delete();
					}
				} finally {
					cursor.close();
				}
				return null;
			}
		});
	}
```

Note that all methods that modify database do it within write transaction. There are two ways to do it:
  * call `beginTransaction()`, `commit()` and `rollback()` methods of 'SqlJetDB' class
  * wrap modifications within `runTransaction(op, mode)` or `runWriteTransaction(op)`
The first approach is more flexible while the second is simpler and will be appropriate in most cases.

# User Interface #

Class 'SpendingApp' declares the `main(args)` method that establishes connection with database and shows application window.

```
public class SpendingApp {

	public static void main(String[] args) {
		try {
			SpendingDB.open();
		} catch (SqlJetException e) {
			e.printStackTrace();
			return;
		}
		Display display = new Display();
		Shell shell = new Shell(display);
		shell.setText("Spending");
		PaymentsController pc = new PaymentsController();
		pc.createView(shell);
		shell.setSize(400, 400);
		shell.open();
		while (!shell.isDisposed()) {
			if (!display.readAndDispatch()) {
				display.sleep();
			}
		}
		display.dispose();
		try {
			SpendingDB.close();
		} catch (SqlJetException e) {
			e.printStackTrace();
		}
	}
}
```

Class 'PaymentsController' creates all UI controls to browse and edit payments. You may check the source code to see all the details. For example `refresh()` method is used to update list of payments:

```
	public void refresh() {
		table.removeAll();
		try {
			ISqlJetCursor cursor = showForToday ? SpendingDB
					.getPayments(new Date()) : SpendingDB.getAllPayments();
			try {
				DateFormat df = DateFormat.getDateInstance(DateFormat.MEDIUM);
				while (!cursor.eof()) {
					Payment p = new Payment();
					p.read(cursor);
					TableItem item = new TableItem(table, 0);
					item.setText(new String[] { df.format(p.date),
							String.valueOf(p.amount),
							p.currency == null ? "" : p.currency });
					item.setData("rowid", cursor.getRowId());
					item.setData("payment", p);
					cursor.next();
				}
			} finally {
				cursor.close();
			}
		} catch (SqlJetException e) {
			e.printStackTrace();
		}
		for (TableColumn col : table.getColumns()) {
			col.pack();
		}
	}
```

When user clicks on 'Add' button 'AddPaymentDialog' is used to collect 'amount' and 'currency' values.

![http://sqljet.googlecode.com/svn/trunk/images/AddPayment.png](http://sqljet.googlecode.com/svn/trunk/images/AddPayment.png)