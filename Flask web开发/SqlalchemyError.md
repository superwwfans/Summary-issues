Here is a list of the errors sqlalchemy itself can raise, taken from `help(sqlalchemy.exc)` and `help(sqlalchemy.orm.exc)`:

- ```
  sqlalchemy.exc
  ```

  :

  - `ArgumentError` - Raised when an invalid or conflicting function argument is supplied. This error generally corresponds to construction time state errors.
  - `CircularDependencyError` - Raised by topological sorts when a circular dependency is detected
  - `CompileError` - Raised when an error occurs during SQL compilation
  - `ConcurrentModificationError`
  - `DBAPIError` - Raised when the execution of a database operation fails. If the error-raising operation occured in the execution of a SQL statement, that statement and its parameters will be available on the exception object in the `statement` and `params` attributes. The wrapped exception object is available in the `orig` attribute. Its type and properties are DB-API implementation specific.
  - `DataError` Wraps a DB-API `DataError`.
  - `DatabaseError` - Wraps a DB-API `DatabaseError`.
  - `DisconnectionError` - A disconnect is detected on a raw DB-API connection. be raised by a `PoolListener` so that the host pool forces a disconnect.
  - `FlushError`
  - `IdentifierError` - Raised when a schema name is beyond the max character limit
  - `IntegrityError` - Wraps a DB-API `IntegrityError`.
  - `InterfaceError` - Wraps a DB-API `InterfaceError`.
  - `InternalError` - Wraps a DB-API `InternalError`.
  - `InvalidRequestError` - SQLAlchemy was asked to do something it can't do. This error generally corresponds to runtime state errors.
  - `NoReferenceError` - Raised by `ForeignKey` to indicate a reference cannot be resolved.
  - `NoReferencedColumnError` - Raised by `ForeignKey` when the referred `Column` cannot be located.
  - `NoReferencedTableError` - Raised by `ForeignKey` when the referred `Table` cannot be located.
  - `NoSuchColumnError` - A nonexistent column is requested from a `RowProxy`.
  - `NoSuchTableError` - Table does not exist or is not visible to a connection.
  - `NotSupportedError` - Wraps a DB-API `NotSupportedError`.
  - `OperationalError` - Wraps a DB-API `OperationalError`.
  - `ProgrammingError` - Wraps a DB-API `ProgrammingError`.
  - SADeprecationWarning - Issued once per usage of a deprecated API.
  - SAPendingDeprecationWarning - Issued once per usage of a deprecated API.
  - SAWarning - Issued at runtime.
  - `SQLAlchemyError` - Generic error class.
  - `SQLError` - Raised when the execution of a database operation fails.
  - `TimeoutError` - Raised when a connection pool times out on getting a connection.
  - `UnboundExecutionError` - SQL was attempted without a database connection to execute it on.
  - `UnmappedColumnError`

- ```
  sqlalchemy.orm.exc
  ```

  :

  - `ConcurrentModificationError` - Rows have been modified outside of the unit of work.
  - `FlushError` - A invalid condition was detected during flush().
  - MultipleResultsFound - A single database result was required but more than one were found.
  - NoResultFound - A database result was required but none was found.
  - `ObjectDeletedError` - An refresh() operation failed to re-retrieve an object's row.
  - `UnmappedClassError` - An mapping operation was requested for an unknown class.
  - `UnmappedColumnError` - Mapping operation was requested on an unknown column.
  - `UnmappedError` - TODO
  - `UnmappedInstanceError` - An mapping operation was requested for an unknown instance.