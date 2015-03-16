# Benchmarks (trunk, [r622](https://code.google.com/p/sqljet/source/detail?r=622)) #

**JDK 1.6.0\_10 (32 bit), Windows Vista 64-bit:
```
[echoproperties] #Ant properties
[echoproperties] #Wed Sep 23 22:27:30 CEST 2009
[echoproperties] os.arch=x86
[echoproperties] os.name=Windows Vista
[echoproperties] os.version=6.0
[echoproperties] #Ant properties
[echoproperties] #Wed Sep 23 22:27:30 CEST 2009
[echoproperties] java.vm.info=mixed mode, sharing
[echoproperties] java.vm.name=Java HotSpot(TM) Client VM
[echoproperties] java.vm.specification.name=Java Virtual Machine Specification
[echoproperties] java.vm.specification.vendor=Sun Microsystems Inc.
[echoproperties] java.vm.specification.version=1.0
[echoproperties] java.vm.vendor=Sun Microsystems Inc.
[echoproperties] java.vm.version=11.0-b12
```**

```
sandbox-server-benchmarks:
    [junit] Testsuite: org.tmatesoft.sqljet.sandbox.memory.MemoryBuffersBenchmarks
    [junit] Tests run: 15, Failures: 0, Errors: 0, Time elapsed: 551.876 sec
    [junit] 
    [junit] Testcase: arrayByte took 2.829 sec
    [junit] Testcase: arrayInt took 11.143 sec
    [junit] Testcase: arrayLong took 56.2 sec
    [junit] Testcase: arrayByteUnsigned took 2.852 sec
    [junit] Testcase: arrayIntUnsigned took 10.432 sec
    [junit] Testcase: bufferByte took 2.841 sec
    [junit] Testcase: bufferInt took 41.722 sec
    [junit] Testcase: bufferLong took 83.625 sec
    [junit] Testcase: bufferByteUnsigned took 36.082 sec
    [junit] Testcase: bufferIntUnsigned took 46.119 sec
    [junit] Testcase: directByte took 37.304 sec
    [junit] Testcase: directInt took 54.861 sec
    [junit] Testcase: directLong took 69.677 sec
    [junit] Testcase: directByteUnsigned took 37.246 sec
    [junit] Testcase: directIntUnsigned took 58.861 sec
```

```
sandbox-client-benchmarks:
    [junit] Testsuite: org.tmatesoft.sqljet.sandbox.memory.MemoryBuffersBenchmarks
    [junit] Tests run: 15, Failures: 0, Errors: 0, Time elapsed: 2,030.957 sec
    [junit] 
    [junit] Testcase: arrayByte took 68.305 sec
    [junit] Testcase: arrayInt took 77.836 sec
    [junit] Testcase: arrayLong took 141.421 sec
    [junit] Testcase: arrayByteUnsigned took 73.644 sec
    [junit] Testcase: arrayIntUnsigned took 95.036 sec
    [junit] Testcase: bufferByte took 83.658 sec
    [junit] Testcase: bufferInt took 207.271 sec
    [junit] Testcase: bufferLong took 398.237 sec
    [junit] Testcase: bufferByteUnsigned took 90.013 sec
    [junit] Testcase: bufferIntUnsigned took 223.167 sec
    [junit] Testcase: directByte took 83.066 sec
    [junit] Testcase: directInt took 113.234 sec
    [junit] Testcase: directLong took 147.694 sec
    [junit] Testcase: directByteUnsigned took 95.897 sec
    [junit] Testcase: directIntUnsigned took 132.454 sec
```

**JDK 1.5.0\_15 (32 bit), Windows Vista 64-bit:**

```
sandbox-server-benchmarks:
    [junit] Testsuite: org.tmatesoft.sqljet.sandbox.memory.MemoryBuffersBenchmarks
    [junit] Tests run: 15, Failures: 0, Errors: 0, Time elapsed: 904.684 sec
    [junit] 
    [junit] Testcase: arrayByte took 40.37 sec
    [junit] Testcase: arrayInt took 42.654 sec
    [junit] Testcase: arrayLong took 83.939 sec
    [junit] Testcase: arrayByteUnsigned took 35.649 sec
    [junit] Testcase: arrayIntUnsigned took 39.239 sec
    [junit] Testcase: bufferByte took 22.596 sec
    [junit] Testcase: bufferInt took 54.389 sec
    [junit] Testcase: bufferLong took 150.572 sec
    [junit] Testcase: bufferByteUnsigned took 36.605 sec
    [junit] Testcase: bufferIntUnsigned took 34.788 sec
    [junit] Testcase: directByte took 51.85 sec
    [junit] Testcase: directInt took 64.105 sec
    [junit] Testcase: directLong took 108.248 sec
    [junit] Testcase: directByteUnsigned took 83.588 sec
    [junit] Testcase: directIntUnsigned took 56.061 sec
```

```
sandbox-client-benchmarks:
    [junit] Testsuite: org.tmatesoft.sqljet.sandbox.memory.MemoryBuffersBenchmarks
    [junit] Tests run: 15, Failures: 0, Errors: 0, Time elapsed: 2,102.869 sec
    [junit] 
    [junit] Testcase: arrayByte took 79.171 sec
    [junit] Testcase: arrayInt took 104.373 sec
    [junit] Testcase: arrayLong took 198.291 sec
    [junit] Testcase: arrayByteUnsigned took 94.673 sec
    [junit] Testcase: arrayIntUnsigned took 127.609 sec
    [junit] Testcase: bufferByte took 82.948 sec
    [junit] Testcase: bufferInt took 139.305 sec
    [junit] Testcase: bufferLong took 238.899 sec
    [junit] Testcase: bufferByteUnsigned took 99.98 sec
    [junit] Testcase: bufferIntUnsigned took 177.274 sec
    [junit] Testcase: directByte took 95.12 sec
    [junit] Testcase: directInt took 156.846 sec
    [junit] Testcase: directLong took 207.542 sec
    [junit] Testcase: directByteUnsigned took 117.974 sec
    [junit] Testcase: directIntUnsigned took 182.833 sec
```

**JDK 1.7.0 M4 (32 bit), Windows Vista 64 bit:**

```
sandbox-server-benchmarks:
    [junit] Testsuite: org.tmatesoft.sqljet.sandbox.memory.MemoryBuffersBenchmarks
    [junit] Tests run: 15, Failures: 0, Errors: 0, Time elapsed: 539.637 sec
    [junit] 
    [junit] Testcase: arrayByte took 2.705 sec
    [junit] Testcase: arrayInt took 10.315 sec
    [junit] Testcase: arrayLong took 54.332 sec
    [junit] Testcase: arrayByteUnsigned took 2.692 sec
    [junit] Testcase: arrayIntUnsigned took 10.133 sec
    [junit] Testcase: bufferByte took 2.786 sec
    [junit] Testcase: bufferInt took 48.879 sec
    [junit] Testcase: bufferLong took 81.556 sec
    [junit] Testcase: bufferByteUnsigned took 45.276 sec
    [junit] Testcase: bufferIntUnsigned took 43.187 sec
    [junit] Testcase: directByte took 37.553 sec
    [junit] Testcase: directInt took 49.746 sec
    [junit] Testcase: directLong took 67.723 sec
    [junit] Testcase: directByteUnsigned took 39.291 sec
    [junit] Testcase: directIntUnsigned took 43.373 sec
```

```
sandbox-client-benchmarks:
    [junit] Testsuite: org.tmatesoft.sqljet.sandbox.memory.MemoryBuffersBenchmarks
    [junit] Tests run: 15, Failures: 0, Errors: 0, Time elapsed: 2,016.326 sec
    [junit] 
    [junit] Testcase: arrayByte took 65.651 sec
    [junit] Testcase: arrayInt took 77.253 sec
    [junit] Testcase: arrayLong took 146.968 sec
    [junit] Testcase: arrayByteUnsigned took 74.694 sec
    [junit] Testcase: arrayIntUnsigned took 95.31 sec
    [junit] Testcase: bufferByte took 80.157 sec
    [junit] Testcase: bufferInt took 215.27 sec
    [junit] Testcase: bufferLong took 406.184 sec
    [junit] Testcase: bufferByteUnsigned took 91.82 sec
    [junit] Testcase: bufferIntUnsigned took 226.047 sec
    [junit] Testcase: directByte took 87.573 sec
    [junit] Testcase: directInt took 104.965 sec
    [junit] Testcase: directLong took 128.952 sec
    [junit] Testcase: directByteUnsigned took 100.067 sec
    [junit] Testcase: directIntUnsigned took 115.392 sec
```