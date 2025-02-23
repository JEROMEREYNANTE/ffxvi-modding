---
icon: material/microsoft-excel
---

# :material-microsoft-excel: Nex (NXD) Conversion

Nex files are pretty much the database of the game. Originally in Excel form, SQEX converts them into a binary format named `.nxd` (sister of FF14's `.exd`).

## Requirements

* :material-github: [FF16Tools](https://github.com/Nenkai/FF16Tools) - **It is recommended to always use the latest version as column names will be renamed.**
* :simple-sqlite: [SQLiteStudio](https://sqlitestudio.pl/)

### Converting to SQLite

Using `FF16Tools.CLI`, run the following command:

```
FF16Tools.CLI nxd-to-sqlite -i <path to directory with nxd>
```

This will convert all the `.nxd`'s in a directory to a SQLite database you can open with **SQLiteStudio**.

---

### Converting back to Nex

```
FF16Tools.CLI sqlite-to-nxd -i <path to sqlite file>
```

!!! tip

    You can provide the `-t` argument to only convert certain tables. for instance, `-t equipitem command` will only save `equipitem.nxd` and `command.nxd`.

!!! note

    * Refer to the [table layouts here](FF16Tools.Files/Nex/Layouts) for the column value types. Note: this has been mapped mostly manually. **Please contribute!** Many columns are still unknown.
    * Always check the [Changelog](NEX_CHANGELOG.md) for updated table column names.
    * Nex can contain nested data, therefore arrays and other structs are converted to json strings.
    * Nex can contain row sets that don't actually contain any rows. This information is lost between SQLite conversion, but should *hopefully* not matter.
    * You may need to edit `root.nxl` to reflect the number of rows (if you've added/removed any).