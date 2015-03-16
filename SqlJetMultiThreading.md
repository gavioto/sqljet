# Introduction #

This page provides short description of multi-threading support in SQLJet library. There are following ways one could use SQLJet in multi-threaded environment:

  * One database connection (SqlJetDb instance) shared by multiple threads.
  * One dedicated database connection for each thread.
  * One dedicated database connection for each thread with synchronization.

# Using one shared connection (SqlJetDb instanсe) #

How does it work: SqlJetDb class instance has internal monitor on which all API calls are synchronized. Both read and write access is synchronized on the same object monitor.

Benefits: No special code needed to enable thread-safeness.

Disadvantages: Only one thread could work with the database in the same time. Synchronization only synchronizes threads of the same JVM process, other processes might work with the same databases not respecting synchronization rules. Poor performance because every operation is exclusive.

# One connection (SqlJetDb instance) per thread #

How does it work: All SqlJetDb instances (those that work with the same database) shares read and write locks in file system. There could be either multiple simultaneous "readers" or single "writer" at the same time. Contrary to the first approach that uses Java object monitor, in this approach "readers" and "writers" poll lock periodically to check if read or write operation could be started.

Benefits: Multiple read operations in the same time are allowed. Locks are file system based, so synchronization covers all processes accessing database, not only Java threads of the same JVM process.

Disadvantages: Special exceptions handling is needed - users should handle BUSY exception when lock poll time exceeds certain configurable limit. Higher CPU usage due to the lock polling.

# One connection (SqlJetDb instance) per thread with synchronization #

How does it work: Same as above, but additionally threads in the same JVM process are synchronized.

Benefits: Same as above, plus less lock polling within the same JVM process.

Disadvantages: Same as above, but performance will degrade less.