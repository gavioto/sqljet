# In-Memory Data Base with SQLJet #

There is sample code:

```
        SqlJetDb db = new SqlJetDb(SqlJetDb.IN_MEMORY, true);
        db.open();
        try {
            final ISqlJetTableDef tDef = db.createTable(
              "create table t(a integer primary key, b text);");

            final ISqlJetTable t = db.getTable(tDef.getName());

            t.insert(null, "hello");
            t.insert(null, "world");

            db.runReadTransaction(new ISqlJetTransaction() {
                public Object run(SqlJetDb db) throws SqlJetException {
                    final ISqlJetCursor c = t.open();
                    try {
                        while (!c.eof()) {

                            Logger.global.info(String.format("#%d: \"%s\"",
                              c.getInteger("a"), c.getString("b")));

                            c.next();
                        }
                    } finally {
                        c.close();
                    }
                    return null;
                }
            });
        } finally {
            db.close();
        }
```