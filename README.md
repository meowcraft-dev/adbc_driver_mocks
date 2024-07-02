# adbc\_driver\_mocks
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


## Special use cases

If the query string is `passthrough`, the query will return anything that was passed to Bind or BindStream as the query result. This is useful for testing the Bind and BindStream functions.

# Supported DataTypes

Here is a list of supported DataTypes in the query. For some types it is possible to use the aliases instead of full type string, multiple aliases are seperated by comma.

|Type String| Alias |
|--------|---------|
|[null](#null)|n|
|[bool](#bool)|boolean,b|
|[int8](#signed-and-unsigned-integers)|i8,c|
|[uint8](#signed-and-unsigned-integers)|u8,C|
|[int16](#signed-and-unsigned-integers)|i16,s|
|[uint16](#signed-and-unsigned-integers)|u16,S|
|[int32](#signed-and-unsigned-integers)|i32,i|
|[uint32](#signed-and-unsigned-integers)|u32,I|
|[int64](#signed-and-unsigned-integers)|i64,l|
|[uint64](#signed-and-unsigned-integers)|u64,L|
|[float16](#signed-and-unsigned-integers)|f16,e|
|[float32](#signed-and-unsigned-integers)|f32,f|
|[float64](#signed-and-unsigned-integers)|f64,g|
|[binary](#binary)|z|
|[string](#string)|str|
|[date32](#date32)|d32,tdD|
|[date64](#date64)|d64,tdm|
|[time32s](#time32s)|t32s,tts|
|[time32ms](#time32ms)|t32ms,ttm|
|[time64us](#time64us)|t64us,ttu|
|[time64ns](#time64ns)|t64ns,ttn|
|[timestamp_s](#timestamp_s)||
|[timestamp_ms](#timestamp_ms)||
|[timestamp_us](#timestamp_us)||
|[timestamp_ns](#timestamp_ns)||
|[duration_s](#duration_s)||
|[duration_ms](#duration_ms)||
|[duration_us](#duration_us)||
|[duration_ns](#duration_ns)||
|[interval_month](#interval_month)||
|[interval_daytime](#interval_daytime)||
|[interval_monthdaynano](#interval_monthdaynano)||
|[list](#list)||
|[struct](#struct)||
|run\_end\_encoded||
|dictionary\_encoded\_array||
|dense_union||
|sparse_union||

# DateType details

## null

Null types will always return rows numbers of nulls.

## bool

Bool types will alternate between `true` and `false`.

> Example:
> 
> `3: bool` -> `True`, `False`, `True`

## Signed and Unsigned Integers

The integer types include: `int8`, `uint8`, `int16`, `uint16`, `int32`, `uint32`, `int64`, `uint64`.

### Signed Integers

- **First two values**: Minimum and maximum values when the integer is at the top level of the query (i.e. not inside a list, struct, union...), otherwise `0, 1`.
- **Subsequent values**: `-2, 3, -4, 5...`, overflows if `rows` exceeded maximum value.

### Unsigned Integers
- **First two values**: `0` and maximum value when the integer is at the top level of the query (i.e. not inside a list, struct, union...), otherwise `0, 1`.
- **Subsequent values**: `2, 3, 4, 5...`, overflows if `rows` exceeded maximum value.

> **Examples**:
> 
> `4: int16` -> `-32768`, `32767`, `-2`, `3`
> 
> `4: uint8` -> `0`, `255`, `2`, `3`
> 
> `list<3: int8>` -> `[0, 1, -2]`

## Floating points

The floating point types include: `float16`, `float32`, `float64`.

### Common Values

The first 9 values for all floating point types are:

1. Smallest negative finite value
2. Largest finite value
3. Infinity
4. Negative Infinity
5. NaN
6. Positive zero
7. Negative zero
8. Smallest positive non-zero value (i.e. precision limit)
9. Largest negative non-zero value (negative of the above)

## binary

Binary types are a byte array of `rows` amount of `rows-1`.

The value inside the byte array overflows if exceeded `255`.

> Example:
> 
> `5: binary` -> `[0x0]`,`[0x1,0x1]`,`[0x2,0x2,0x2]`,`[0x3,0x3,0x3,0x3]`,`[0x4,0x4,0x4,0x4,0x4]`

## string

mocked value is a string of `rows` with `rows-1` number of zeros to the left.

> Example:
> 
> `3: string` -> `0`, `01`, `002`

## Date and Time types

### date32

days since the UNIX epoch in int32.

> Example:
> 
> `3: date32` -> `1970-01-01`, `1970-01-02`, `1970-01-03`

### date64

milliseconds since the UNIX epoch in int64.

> Example:
> 
> `3: date64` -> `1970-01-01 00:00:00`, `1970-01-02 00:00:00.001`, `1970-01-03 00:00:00.002`

### time32s

either seconds since midnight.

> Example:
> 
> `3: time32s` -> `00:00:00`, `00:00:01`, `00:00:02`

### time32ms

milliseconds since midnight.

> Example:
> 
> `3: time32ms` -> `00:00:00`, `00:00:00.001`, `00:00:00.002`

### time64us

microseconds since midnight.

> Example:
> 
> `3: time64us` -> `00:00:00`, `00:00:00.000001`, `00:00:00.000002`

### time64ns

nanoseconds since midnight.

> Example:
> 
> `3: time64ns` -> `00:00:00`, `00:00:00.000000001`, `00:00:00.000000002`

### timestamp_s

seconds since the UNIX epoch in int64. in this mock driver the timezone is always UTC.

> Example:
> 
> `3: timestamp_s` -> `1970-01-01 00:00:00`, `1970-01-01 00:00:01`, `1970-01-01 00:00:02`

### timestamp_ms

miliseconds since the UNIX epoch in int64. in this mock driver the timezone is always UTC.

> Example:
> 
> `3: timestamp_ms` -> `1970-01-01 00:00:00`, `1970-01-01 00:00:00.001`, `1970-01-01 00:00:00.002`

### timestamp_us

microseconds since the UNIX epoch in int64. in this mock driver the timezone is always UTC.

> Example:
> 
> `3: timestamp_ms` -> `1970-01-01 00:00:00`, `1970-01-01 00:00:00.000001`, `1970-01-01 00:00:00.000002`

### timestamp_ns

nanoseconds since the UNIX epoch in int64. in this mock driver the timezone is always UTC.

> Example:
> 
> `3: timestamp_ms` -> `1970-01-01 00:00:00`, `1970-01-01 00:00:00.000000001`, `1970-01-01 00:00:00.000000002`

### duration_s

measure of elapsed time in seconds in int64.

> Example:
> 
> `3: duration_s` -> `0s`, `1s`, `2s`

### duration_ms

measure of elapsed time in miliseconds in int64.

> Example:
> 
> `3: duration_ms` -> `0ms`, `1ms`, `2ms`

### duration_us

measure of elapsed time in microseconds in int64.

> Example:
> 
> `3: duration_us` -> `0us`, `1us`, `2us`

### duration_ns

measure of elapsed time in nanoseconds in int64.

> Example:
> 
> `3: duration_ns` -> `0ns`, `1ns`, `2ns`

### interval_month

number of months.

> Example:
> 
> `3: interval_month` -> `months:0`, `months:1`, `months:2`

### interval_daytime

number of days and miliseconds(fraction of day).

> Example:
> 
> `3: interval_daytime` -> `days:0,ms:0`, `days:1,ms:1`, `days:2,ms:2`

### interval_monthdaynano

number of months, days and nanoseconds.

> Example:
> 
> `3: interval_monthdaynano` -> `months:0,days:0,ns:0`, `months:1,days:1,ns:1`, `months:2,days:2,ns:2`

## list

To indicate a list type, signify the length and element type within angle brackets. 

A list without angle brackets will default to list<1:int8>.

> Example:
> 
> `2: list<2: uint8>` -> `[0,1]`, `[2,3]`

## struct

Structs operate similarly to lists but without a specified length. For instance, to define a struct containing an int8 and a boolean value, use the query: `struct<int8,bool>`.

> Example:
> 
> `2: struct<int8,bool>` -> `{0,True}`, `{1,False}`