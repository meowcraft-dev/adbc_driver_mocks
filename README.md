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
|uint8|arrow.PrimitiveTypes.Uint8|
|int16|arrow.PrimitiveTypes.Int16|
|uint16|arrow.PrimitiveTypes.Uint16|
|int32|arrow.PrimitiveTypes.Int32|
|uint32|arrow.PrimitiveTypes.Uint32|
|int64|arrow.PrimitiveTypes.Int64|
|uint64|arrow.PrimitiveTypes.Uint64|
|float16|arrow.FixedWidthTypes.Float16|
|float32|arrow.PrimitiveTypes.Float32|
|float64|arrow.PrimitiveTypes.Float64|
|binary|arrow.BinaryTypes.Binary|
|string|arrow.BinaryTypes.String|
|date32|arrow.PrimitiveTypes.Date32|
|date64|arrow.PrimitiveTypes.Date64|
|time32s|arrow.FixedWidthTypes.Time32s|
|time32ms|arrow.FixedWidthTypes.Time32ms|
|time64us|arrow.FixedWidthTypes.Time64us|
|time64ns|arrow.FixedWidthTypes.Time64ns|
|timestamp_s|arrow.FixedWidthTypes.Timestamp_s|
|timestamp_ms|arrow.FixedWidthTypes.Timestamp_ms|
|timestamp_us|arrow.FixedWidthTypes.Timestamp_us|
|timestamp_ns|arrow.FixedWidthTypes.Timestamp_ns|
|duration_s|arrow.FixedWidthTypes.Duration_s|
|duration_ms|arrow.FixedWidthTypes.Duration_ms|
|duration_us|arrow.FixedWidthTypes.Duration_us|
|duration_ns|arrow.FixedWidthTypes.Duration_ns|
|interval_month|arrow.FixedWidthTypes.MonthInterval|
|interval_daytime|arrow.FixedWidthTypes.DayTimeInterval|
|interval_monthdaynano|arrow.FixedWidthTypes.MonthDayNanoInterval|
|sample_list|arrow.ListOf(arrow.PrimitiveTypes.Int32)|
|sample_list_with_struct|See below|
|sample_nested_list|arrow.ListOf(arrow.ListOf(arrow.PrimitiveTypes.Int32))|
|sample_list_view|arrow.ListViewOf(arrow.PrimitiveTypes.Int32)|
|sample_large_list_view|arrow.LargeListViewOf(arrow.PrimitiveTypes.Int32)|
|sample_fixed_size_list|arrow.FixedSizeListOf(3, arrow.PrimitiveTypes.Int32)|
|sample_nested_fixed_size_list|arrow.FixedSizeListOf(3,arrow.FixedSizeListOf(3, arrow.PrimitiveTypes.Int32))|
|sample_run_end_encoded_array|arrow.RunEndEncodedOf(arrow.PrimitiveTypes.Int32, arrow.PrimitiveTypes.Float32)|
|sample_dictionary_encoded_array|See below|
|sample_dense_union|See below|
|sample_sparse_union|See below|
|null|arrow.Null|

## Sample DataTypes

### sample_list_with_struct
> structs of two timestamps and an int32 nested inside lists

Schema:
```go
arrow.ListOf(
    arrow.StructOf(
        arrow.Field{Name: "start_time", Type: arrow.FixedWidthTypes.Timestamp_s},
        arrow.Field{Name: "end_time", Type: arrow.FixedWidthTypes.Timestamp_s},
        arrow.Field{Name: "data_points", Type: arrow.PrimitiveTypes.Int32},
    )
)
```

Example:

When `row == 3`

```
sample_list_with_struct: [[    -- is_valid: all not null
    -- child 0 type: timestamp[s, tz=UTC]
[1970-01-01 00:00:00]
    -- child 1 type: timestamp[s, tz=UTC]
[1970-01-01 00:01:40]
    -- child 2 type: int32
[0],    -- is_valid: all not null
    -- child 0 type: timestamp[s, tz=UTC]
[1970-01-01 00:00:01]
    -- child 1 type: timestamp[s, tz=UTC]
[1970-01-01 00:01:41]
    -- child 2 type: int32
[1],    -- is_valid: all not null
    -- child 0 type: timestamp[s, tz=UTC]
[1970-01-01 00:00:02]
    -- child 1 type: timestamp[s, tz=UTC]
[1970-01-01 00:01:42]
    -- child 2 type: int32
[2]]]
```

### sample_dictionary_encoded_array
> [dictionary encoded](https://arrow.apache.org/docs/format/Columnar.html#dictionary-encoded-layout) array of strings

Schema:
```go
arrow.DictionaryType{
    IndexType: arrow.PrimitiveTypes.Int32,
    ValueType: arrow.BinaryTypes.String,
},
```

Example:

When `row == 3`

```
sample_dictionary_encoded_array: [  -- dictionary:
["hello","goodbye"]  -- indices:
[0,1,0]]
```

### sample_dense_union
> [dense union](https://arrow.apache.org/docs/format/Columnar.html#dense-union) of int32 and string

Schema:
```go
arrow.DenseUnionOf(
    []arrow.Field{
        {Name: "a", Type: arrow.PrimitiveTypes.Int32},
        {Name: "b", Type: arrow.BinaryTypes.String},
    },
    []arrow.UnionTypeCode{0, 1},
)
```

Example:

When `row == 3`

```
sample_dense_union: [  -- is_valid: all not null  -- type_ids: [0,1,0]  -- value_offsets: [0,0,1]
  -- child 0 type: int32
[0,1,2]
  -- child 1 type: string
["str%d0","str%d1","str%d2"]]
```

### sample_sparse_union
> similar to dense union, a [sparse union](https://arrow.apache.org/docs/format/Columnar.html#sparse-union) of int32 and string

Schema:
```go
arrow.SparseUnionOf(
    []arrow.Field{
        {Name: "a", Type: arrow.PrimitiveTypes.Int32},
        {Name: "b", Type: arrow.BinaryTypes.String},
    },
    []arrow.UnionTypeCode{0, 1},
)
```

Example:

When `row == 3`

```
sample_sparse_union: [  -- is_valid: all not null  -- type_ids: [0,1,0]
  -- child 0 type: int32
[0,null,1]
  -- child 1 type: string
[null,"str%d0",null]]
```
