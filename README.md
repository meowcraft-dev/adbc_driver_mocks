# adbc_driver_mocks
A mock driver for ADBC.

# Usage example (Python)

The query result can be specified by providing a CSV list of types in the query. For instance, if the query is `int8, string`, the resulting table will have two columns—a column for int8 and a column for string.

```python
import pyarrow

import adbc_driver_manager
import adbc_driver_mocks

with adbc_driver_mocks.connect() as db:
    with adbc_driver_manager.AdbcConnection(db) as conn:
        with adbc_driver_manager.AdbcStatement(conn) as stmt:
            stmt.set_sql_query("int8,string")
            stream, _ = stmt.execute_query()
            reader = pyarrow.RecordBatchReader._import_from_c(stream.address)
            print(reader.read_all())

```

Run the above code and observe the result:
```
pyarrow.Table
int8: int8 not null
string: string not null
----
int8: [[0]]
string: [["abcdefghij"]]
```

The number of rows can be specified by adding `<rows>:` at the beginning of the query. For example, `7:int32,bool` will generate a table with 7 rows of int32 and bool values.

To indicate a list type, signify the length and element type within angle brackets. For instance, to create a list of 5 boolean values: `list<5:bool>`.
A list without angle brackets will default to `list<1:int8>`.

Structs operate similarly to lists but without a specified length. To define a struct containing an int8 and a boolean value, use the format: `struct<int8,bool>`.

## Special use cases

If the query string is `passthrough`, the query will return anything that was passed to Bind or BindStream as the query result. This is useful for testing the Bind and BindStream functions.

# Supported DataTypes

Here is a list of supported DataTypes in the query

|Type String| Arrow Type|
|--------|---------|
|int8|arrow.PrimitiveTypes.Int8|
|int16|arrow.PrimitiveTypes.Int16|
|int32|arrow.PrimitiveTypes.Int32|
|int64|arrow.PrimitiveTypes.Int64|
|uint8|arrow.PrimitiveTypes.Uint8|
|uint16|arrow.PrimitiveTypes.Uint16|
|uint32|arrow.PrimitiveTypes.Uint32|
|uint64|arrow.PrimitiveTypes.Uint64|
|float32|arrow.PrimitiveTypes.Float32|
|float64|arrow.PrimitiveTypes.Float64|
|date32|arrow.PrimitiveTypes.Date32|
|date64|arrow.PrimitiveTypes.Date64|
|binary|arrow.BinaryTypes.Binary|
|string|arrow.BinaryTypes.String|
|daytimeinterval|arrow.FixedWidthTypes.DayTimeInterval|
|duration_s|arrow.FixedWidthTypes.Duration_s|
|duration_ms|arrow.FixedWidthTypes.Duration_ms|
|duration_us|arrow.FixedWidthTypes.Duration_us|
|duration_ns|arrow.FixedWidthTypes.Duration_ns|
|float16|arrow.FixedWidthTypes.Float16|
|monthInterval|arrow.FixedWidthTypes.MonthInterval|
|time32s|arrow.FixedWidthTypes.Time32s|
|time32ms|arrow.FixedWidthTypes.Time32ms|
|time64us|arrow.FixedWidthTypes.Time64us|
|time64ns|arrow.FixedWidthTypes.Time64ns|
|timestamp_s|arrow.FixedWidthTypes.Timestamp_s|
|timestamp_ms|arrow.FixedWidthTypes.Timestamp_ms|
|timestamp_us|arrow.FixedWidthTypes.Timestamp_us|
|timestamp_ns|arrow.FixedWidthTypes.Timestamp_ns|
