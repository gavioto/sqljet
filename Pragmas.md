# Introduction #
SQLite defines a pragma statement to query database options and alter its behaviour. Full description is available at http://www.sqlite.org/pragma.html page. SQLJet supports a subset of these options via typed and statement APIs.

# Details #
Typed API can be accessed using `SqlJetDb.getOptions()`. It returns an ISqlJetOptions implementation that has methods to query and alter individual options that correspond to SQLite pragmas.

Method `SqlJetDb.pragma(String)` allows to execute individual pragma statements and is fully compatible with SQLite syntax. Internally this method delegates to the typed API.

Currently supported pragmas include:
  * auto\_vacuum
  * cache\_size
  * encoding
  * schema\_version
  * user\_version

There is one important thing: some pragmas can be changed only in new empty database and only without active transaction. Other pragmas contrariwise should be called in active transaction.

Pragmas which allowed to be changed only in new empty database and without transaction are:
  * auto\_vacuum
  * encoding

This restriction is related to the file format and changes of this pragmas in non empty database could damage database. And changes of this pragmas must be immediately committed to database (it performs automatically), therefore it should be never called in active transaction because this active transaction will be committed unwittingly (and other backside effects are possible).

Other pragmas have no these restrictions and should be called in active transaction. These pragmas are:
  * cache\_size
  * schema\_version
  * user\_version

# Interface #

```

public interface ISqlJetOptions {

    /**
     * File format of schema layer.
     * 
     * @return the fileFormat
     */
    int getFileFormat() throws SqlJetException;

    /**
     * Set file format. It's allowed only on new empty data base.
     * 
     * @param fileFormat
     * @throws SqlJetException
     */
    void setFileFormat(int fileFormat) throws SqlJetException;

    // Pragmas to modify library operation

    /**
     * Use freelist if false. Autovacuum if true.
     * 
     * @return the autovacuum
     */
    boolean isAutovacuum() throws SqlJetException;

    /**
     * Set autovacuum flag. It's allowed only on new empty data base.
     * 
     * @param autovacuum
     * @throws SqlJetException
     */
    void setAutovacuum(boolean autovacuum) throws SqlJetException;

    /**
     * Incremental-vacuum flag.
     * 
     * @return the incrementalVacuum
     */
    boolean isIncrementalVacuum() throws SqlJetException;

    /**
     * Set incremental vacuum flag. It's allowed only on new empty data base.
     * 
     * @param incrementalVacuum
     * @throws SqlJetException
     */
    void setIncrementalVacuum(boolean incrementalVacuum) throws SqlJetException;

    /**
     * Size of the page cache.
     * 
     * @return the pageCacheSize
     */
    int getCacheSize() throws SqlJetException;

    /**
     * Set page cache's size. It's allowed only on new empty data base.
     * 
     * @param pageCacheSize
     * @throws SqlJetException
     */
    void setCacheSize(int pageCacheSize) throws SqlJetException;

    // case_sensitive_like
    // count_changes
    // default_cache_size

    /**
     * Db text encoding.
     * 
     * @return the encoding
     */
    SqlJetEncoding getEncoding() throws SqlJetException;

    /**
     * Set encoding. It's allowed only on new empty data base.
     * 
     * @param encoding
     * @throws SqlJetException
     */
    void setEncoding(SqlJetEncoding encoding) throws SqlJetException;

    // full_column_names
    // fullfsync
    // incremental_vacuum
    // journal_mode
    // journal_size_limit
    // legacy_file_format
    // locking_mode
    // page_size
    // max_page_count
    // read_uncommitted
    // reverse_unordered_selects
    // short_column_names
    // synchronous
    // temp_store
    // temp_store_directory

    // Pragmas to query the database schema

    // collation_list
    // database_list
    // foreign_key_list
    // freelist_count
    // index_info
    // index_list
    // page_count
    // table_info

    // Pragmas to query/modify version values

    /**
     * Schema cookie. Changes with each schema change.
     * 
     * @return the schemaCookie
     */
    int getSchemaVersion() throws SqlJetException;

    void setSchemaVersion(int version) throws SqlJetException;

    /**
     * Change SchemaCookie.
     */
    void changeSchemaVersion() throws SqlJetException;

    /**
     * Verify schema cookie and return true if it is unchanged by other process.
     * If throwIfStale is true then throw exception if cookie is changed by
     * other process.
     * 
     * @param throwIfStale
     * @return
     * @throws SqlJetException
     */
    boolean verifySchemaVersion(boolean throwIfStale) throws SqlJetException;

    /**
     * The user cookie. Used by the application.
     * 
     * @return the userCookie
     */
    int getUserVersion() throws SqlJetException;

    /**
     * Set user's cookie.
     * 
     * @param userCookie
     * @throws SqlJetException
     */
    void setUserVersion(int userCookie) throws SqlJetException;

    // Pragmas to debug the library

    // integrity_check
    // quick_check
    // parser_trace
    // vdbe_trace
    // vdbe_listing
}

```