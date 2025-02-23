---
icon: material/file-excel-box
---

# :material-file-excel-box: Nex - `.nxd`

Originally in Excel form, Nex (Next ExcelDB) are database table files. SQEX converts them into a binary format (`.nxd`) - sister of FF14's `.exd` but with less metadata and little endian.

Unlike `exd`, there are **no column metadata** whatsoever. No names, no cell fields. All had to be manually mapped.

* [.exd Documentation](https://xiv.dev/game-data/file-formats/excel#excel-data-.exd)
* [010 Editor Template](https://github.com/Nenkai/010GameTemplates/blob/main/Square%20Enix/Final%20Fantasy%2016/FF16_nxd_NXDF.bt)
* [Reference Implementation (FF16Tools, C#)](https://github.com/Nenkai/FF16Tools)
* [Table Layouts](https://github.com/Nenkai/FF16Tools/tree/master/FF16Tools.Files/Nex/Layouts)

*Additionally*, row data itself can contain nested structures or arrays.

## Header

```c
struct
{
    uint32_t magic <format=hex>; // 'NXDF'
    uint32_t version; // 1
    enum // uint8_t
    {
        NEX_ROWTYPE_ROWS = 1, // single-keyed rows
        NEX_ROWTYPE_ROWSETS = 2, // rows keyed by id, array index/key2
        NEX_ROWTYPE_DOUBLEKEYED = 3, // rows keyed by id, secondary id, array index/key3
    } type;
    
    enum // uint8_t
    {
        NEX_ROWS_UNLOCALIZED = 1,
        NEX_ROWS_LOCALIZED = 2,
        NEX_ROWSETS_UNLOCALIZED = 3,
        NEX_ROWSETS_LOCALIZED = 4,
        NEX_DOUBLEKEYED_ROWS_UNLOCALIZED = 5,
        NEX_DOUBLEKEYED_ROWS_LOCALIZED = 6
    } category;
    
    // Determines whether to use the baseRowId for searching.
    // if baseRowId is 1, searching for row id 0 will return the first row.
    bool usesBaseRowId; 
    uint8_t _reserved1_;
    uint32_t baseRowId;
    uint32_t _reserved2_[0x4];
} NexDataFileHeader; // size: 0x20
```

Following the main header, another header follows depending on the type of table.

### Type 1 (Rows)

Simple rows with only one key.

```c
struct
{
    int32_t rowInfosOffset; // Offset to a list of row infos for each row
    uint32_t numRows;
} NexRowTableHeader;
```

```c
struct
{
    uint32_t rowID;
    int32_t rowDataOffset; // Relative to start of row info.
} NexRowTableRowInfo;
```

---

### Type 2 (RowSet)

Row sets, with one key.

#### NexRowSetHeader
```c
struct
{
    uint32_t thisHeaderSize; // Also used as an offset relative to this for NexRowSetInfo[].
    uint32_t arrayCount; // Number of row arrays in this table.
    uint32_t reserved;
    int32_t rowsInfoOffset; // Points to another list of the data for each row (not row set).
    uint32_t totalRowCount; // Total number of rows in this table.
} NexRowSetHeader;
```

##### NexRowSetInfo
```c
struct
{
    uint32_t rowId; // main row id/set id.
    int32_t rowInfosOffset; // Offset to Type2RowSetRowInfo[], relative to this structure.
    uint32_t arrayLength; // Number of rows for this set/array.
} NexRowSetInfo;
```

##### NexRowSetRowInfo
```c
struct
{
    uint32_t rowId;
    uint32_t arrayIndex; // array index, or key2.
    int32_t rowDataOffset; // Relative to this structure.
} NexRowSetRowInfo;
```

##### NexRowInfo
```c
struct
{
    uint32_t rowId;
    uint32_t arrayIndex; // array index, or key2.
    int32_t rowDataOffset; // Relative to this structure. May be negative (reverse offset).
} NexRowInfo;
```

### Type 3 (Double-Keyed Sets)

Rows sets, with two keys.

#### NexDoubleKeyedHeader
```c
struct
{
    uint32_t thisHeaderSize;  // Also used as an offset relative to this for NexDoubleKeyedSetInfo[].
    uint32_t count; // Number of double-keyed sets in this table.
    int32_t rowInfoOffset;
    int32_t rowsInfoOffset; // Points to another list of the data for each row.
    uint32_t totalRowCount; // Total number of rows in this table.
} NexTriKeyedHeader;
```

##### NexDoubleKeyedSetInfo
```c
struct
{
    uint32_t rowId; // Main row id.
    uint32_t subRowSetInfoOffset; // Offset to NexDoubleKeyedSubSetInfo[], relative to this structure.
    uint32_t numSubKeys; // Number of sub row keys.
    uint32_t unkOffset; // Unknown, never used.
    uint32_t unkAlways0; // Unknown, possible count for above, never used.
} NexDoubleKeyedSetInfo;
```

##### NexDoubleKeyedSubSetInfo
```c
struct
{
    uint32_t subRowId; // or key2.
    uint32_t unkOffset; // Unknown, never used.
    uint32_t unkAlways0; // Unknown, possible count for above, never used.
    int32_t rowInfosOffset; // Offset to NexDoubleKeyedRowInfo[], relative.
    uint32_t numRows; // Number of rows for this subset.
} NexDoubleKeyedSubSetInfo;
```

##### NexDoubleKeyedRowInfo
```c
struct
{
    uint32_t rowId;
    uint32_t subRowId; // or key2.
    uint32_t arrayIndex; // array index, or key3.
    uint32_t unkAlways0; // Unknown, never used.
    int32_t rowDataOffset; // Relative to this structure. May be negative (reverse offset).
} NexRowInfo;
```