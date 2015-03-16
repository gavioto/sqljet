# Benchmarks #

Version 1.0.0 of [SQLJet](http://sqljet.com/) is compared to SQLiteJDBC (v056 NestedVM and v3.6.14.2 of native SQLite through JNI).

Summary (from fastest to slowest):

| SQLJet 1.0.1 | 8,287 sec |
|:-------------|:----------|
| SQLLite 3.6.14 | 10,123 sec |
| SQLJet 1.0.0 | 10,502 sec |
| NestedVM v056 | 76,59 sec |

# Benchmarks #

**Java platform**

```
JDK 1.6.0_14 -server, Windows Vista 32

[echoproperties] #Ant properties
[echoproperties] #Thu Sep 24 21:16:24 EEST 2009
[echoproperties] os.arch=x86
[echoproperties] os.name=Windows Vista
[echoproperties] os.version=6.0
[echoproperties] #Ant properties
[echoproperties] #Thu Sep 24 21:16:24 EEST 2009
[echoproperties] java.vm.info=mixed mode, sharing
[echoproperties] java.vm.name=Java HotSpot(TM) Client VM
[echoproperties] java.vm.specification.name=Java Virtual Machine Specification
[echoproperties] java.vm.specification.vendor=Sun Microsystems Inc.
[echoproperties] java.vm.specification.version=1.0
[echoproperties] java.vm.vendor=Sun Microsystems Inc.
[echoproperties] java.vm.version=14.0-b16

```

**[SQLJet](http://sqljet.com/) 1.0.0.blocal (trunk) -server**

```

    [junit] Testsuite: org.tmatesoft.sqljet.benchmarks.SqlJetBenchmark
    [junit] Tests run: 7, Failures: 0, Errors: 0, Time elapsed: 10,502 sec
    [junit] Testcase: clear took 0,266 sec
    [junit] Testcase: nothing took 0,016 sec
    [junit] Testcase: selectAll took 0,293 sec
    [junit] Testcase: updateAll took 1,397 sec
    [junit] Testcase: deleteAll took 0,333 sec
    [junit] Testcase: insertRandoms took 1,664 sec
    [junit] Testcase: locate took 1,998 sec

```


**[SQLJet](http://sqljet.com/) 1.0.1.blocal (trunk) -server**

```
      		
    [junit] Testsuite: org.tmatesoft.sqljet.benchmarks.SqlJetBenchmark
    [junit] Tests run: 7, Failures: 0, Errors: 0, Time elapsed: 8,287 sec
    [junit] Testcase: clear took 0,174 sec
    [junit] Testcase: nothing took 0,01 sec
    [junit] Testcase: selectAll took 0,247 sec
    [junit] Testcase: updateAll took 1,022 sec
    [junit] Testcase: deleteAll took 0,209 sec
    [junit] Testcase: insertRandoms took 1,387 sec
    [junit] Testcase: locate took 1,549 sec

```

**[SQLiteJDBC](http://www.zentus.com/sqlitejdbc/) v056 (SQLite 3.6.14.2 native) -server**

```

    [junit] Testsuite: org.tmatesoft.sqljet.benchmarks.SQLiteBenchmark
    [junit] Tests run: 7, Failures: 0, Errors: 0, Time elapsed: 10,123 sec
    [junit] Testcase: clear took 0,175 sec
    [junit] Testcase: nothing took 0,007 sec
    [junit] Testcase: selectAll took 0,218 sec
    [junit] Testcase: updateAll took 0,576 sec
    [junit] Testcase: deleteAll took 0,1 sec
    [junit] Testcase: insertRandoms took 0,955 sec
    [junit] Testcase: locate took 6,489 sec

```

**[SQLiteJDBC](http://www.zentus.com/sqlitejdbc/) v056 (NestedVM) -server**

```

    [junit] Testsuite: org.tmatesoft.sqljet.benchmarks.SQLiteBenchmark
    [junit] Tests run: 7, Failures: 0, Errors: 0, Time elapsed: 76,59 sec
    [junit] Testcase: clear took 0,601 sec
    [junit] Testcase: nothing took 0,016 sec
    [junit] Testcase: selectAll took 2,858 sec
    [junit] Testcase: updateAll took 4,188 sec
    [junit] Testcase: deleteAll took 0,947 sec
    [junit] Testcase: insertRandoms took 5,688 sec
    [junit] Testcase: locate took 48,223 sec

```