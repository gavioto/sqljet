# File Format #
SQLJet is fully compatible with native implementation on database file format level. Files created by SQLJet may be read by SQLite and vice versa.

TODO: do we support 'pragma legacy\_file\_format'?

# File Locks #
Database file lock is obtained for each read and write transaction which corresponds to 'pragma locking\_mode=normal'.

TODO: do we support 'pragma locking\_mode=exclusive'?

# Concurrency #
SQLJet supports 'multithreaded' mode of SQLite.