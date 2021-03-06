# 51.62. pg\_type

The catalog `pg_type` stores information about data types. Base types and enum types \(scalar types\) are created with [CREATE TYPE](https://www.postgresql.org/docs/10/static/sql-createtype.html), and domains with [CREATE DOMAIN](https://www.postgresql.org/docs/10/static/sql-createdomain.html). A composite type is automatically created for each table in the database, to represent the row structure of the table. It is also possible to create composite types with `CREATE TYPE AS`.

**Table 51.62. `pg_type` Columns**

| Name | Type | References | Description |
| :--- | :--- | :--- | :--- |
| `oid` | `oid` |  | Row identifier \(hidden attribute; must be explicitly selected\) |
| `typname` | `name` |  | Data type name |
| `typnamespace` | `oid` | [`pg_namespace`](https://www.postgresql.org/docs/10/static/catalog-pg-namespace.html).oid | The OID of the namespace that contains this type |
| `typowner` | `oid` | [`pg_authid`](https://www.postgresql.org/docs/10/static/catalog-pg-authid.html).oid | Owner of the type |
| `typlen` | `int2` |  | For a fixed-size type, `typlen` is the number of bytes in the internal representation of the type. But for a variable-length type, `typlen`is negative. -1 indicates a “varlena” type \(one that has a length word\), -2 indicates a null-terminated C string. |
| `typbyval` | `bool` |  | `typbyval` determines whether internal routines pass a value of this type by value or by reference. `typbyval` had better be false if `typlen` is not 1, 2, or 4 \(or 8 on machines where Datum is 8 bytes\). Variable-length types are always passed by reference. Note that `typbyval` can be false even if the length would allow pass-by-value. |
| `typtype` | `char` |  | `typtype` is `b` for a base type, `c` for a composite type \(e.g., a table's row type\), `d` for a domain, `e` for an enum type, `p` for a pseudo-type, or `r` for a range type. See also `typrelid` and `typbasetype`. |
| `typcategory` | `char` |  | `typcategory` is an arbitrary classification of data types that is used by the parser to determine which implicit casts should be “preferred”. See [Table 51.63](https://www.postgresql.org/docs/10/static/catalog-pg-type.html#CATALOG-TYPCATEGORY-TABLE). |
| `typispreferred` | `bool` |  | True if the type is a preferred cast target within its `typcategory` |
| `typisdefined` | `bool` |  | True if the type is defined, false if this is a placeholder entry for a not-yet-defined type. When `typisdefined` is false, nothing except the type name, namespace, and OID can be relied on. |
| `typdelim` | `char` |  | Character that separates two values of this type when parsing array input. Note that the delimiter is associated with the array element data type, not the array data type. |
| `typrelid` | `oid` | [`pg_class`](https://www.postgresql.org/docs/10/static/catalog-pg-class.html).oid | If this is a composite type \(see `typtype`\), then this column points to the `pg_class` entry that defines the corresponding table. \(For a free-standing composite type, the `pg_class` entry doesn't really represent a table, but it is needed anyway for the type's`pg_attribute` entries to link to.\) Zero for non-composite types. |
| `typelem` | `oid` | [`pg_type`](https://www.postgresql.org/docs/10/static/catalog-pg-type.html).oid | If `typelem` is not 0 then it identifies another row in `pg_type`. The current type can then be subscripted like an array yielding values of type `typelem`. A “true” array type is variable length \(`typlen` = -1\), but some fixed-length \(`typlen` &gt; 0\) types also have nonzero `typelem`, for example `name` and `point`. If a fixed-length type has a `typelem` then its internal representation must be some number of values of the `typelem` data type with no other data. Variable-length array types have a header defined by the array subroutines. |
| `typarray` | `oid` | [`pg_type`](https://www.postgresql.org/docs/10/static/catalog-pg-type.html).oid | If `typarray` is not 0 then it identifies another row in `pg_type`, which is the “true” array type having this type as element |
| `typinput` | `regproc` | [`pg_proc`](https://www.postgresql.org/docs/10/static/catalog-pg-proc.html).oid | Input conversion function \(text format\) |
| `typoutput` | `regproc` | [`pg_proc`](https://www.postgresql.org/docs/10/static/catalog-pg-proc.html).oid | Output conversion function \(text format\) |
| `typreceive` | `regproc` | [`pg_proc`](https://www.postgresql.org/docs/10/static/catalog-pg-proc.html).oid | Input conversion function \(binary format\), or 0 if none |
| `typsend` | `regproc` | [`pg_proc`](https://www.postgresql.org/docs/10/static/catalog-pg-proc.html).oid | Output conversion function \(binary format\), or 0 if none |
| `typmodin` | `regproc` | [`pg_proc`](https://www.postgresql.org/docs/10/static/catalog-pg-proc.html).oid | Type modifier input function, or 0 if type does not support modifiers |
| `typmodout` | `regproc` | [`pg_proc`](https://www.postgresql.org/docs/10/static/catalog-pg-proc.html).oid | Type modifier output function, or 0 to use the standard format |
| `typanalyze` | `regproc` | [`pg_proc`](https://www.postgresql.org/docs/10/static/catalog-pg-proc.html).oid | Custom `ANALYZE` function, or 0 to use the standard function |
| `typalign` | `char` |  | `typalign` is the alignment required when storing a value of this type. It applies to storage on disk as well as most representations of the value inside PostgreSQL. When multiple values are stored consecutively, such as in the representation of a complete row on disk, padding is inserted before a datum of this type so that it begins on the specified boundary. The alignment reference is the beginning of the first datum in the sequence.Possible values are:`c` = `char` alignment, i.e., no alignment needed.`s` = `short` alignment \(2 bytes on most machines\).`i` = `int` alignment \(4 bytes on most machines\).`d` = `double` alignment \(8 bytes on many machines, but by no means all\).NoteFor types used in system tables, it is critical that the size and alignment defined in `pg_type`agree with the way that the compiler will lay out the column in a structure representing a table row. |
| `typstorage` | `char` |  | `typstorage` tells for varlena types \(those with `typlen` = -1\) if the type is prepared for toasting and what the default strategy for attributes of this type should be. Possible values are`p`: Value must always be stored plain.`e`: Value can be stored in a “secondary” relation \(if relation has one, see `pg_class.reltoastrelid`\).`m`: Value can be stored compressed inline.`x`: Value can be stored compressed inline or stored in “secondary” storage.Note that `m` columns can also be moved out to secondary storage, but only as a last resort \(`e` and `x` columns are moved first\). |
| `typnotnull` | `bool` |  | `typnotnull` represents a not-null constraint on a type. Used for domains only. |
| `typbasetype` | `oid` | [`pg_type`](https://www.postgresql.org/docs/10/static/catalog-pg-type.html).oid | If this is a domain \(see `typtype`\), then `typbasetype` identifies the type that this one is based on. Zero if this type is not a domain. |
| `typtypmod` | `int4` |  | Domains use `typtypmod` to record the `typmod` to be applied to their base type \(-1 if base type does not use a `typmod`\). -1 if this type is not a domain. |
| `typndims` | `int4` |  | `typndims` is the number of array dimensions for a domain over an array \(that is, `typbasetype` is an array type\). Zero for types other than domains over array types. |
| `typcollation` | `oid` | [`pg_collation`](https://www.postgresql.org/docs/10/static/catalog-pg-collation.html).oid | `typcollation` specifies the collation of the type. If the type does not support collations, this will be zero. A base type that supports collations will have `DEFAULT_COLLATION_OID` here. A domain over a collatable type can have some other collation OID, if one was specified for the domain. |
| `typdefaultbin` | `pg_node_tree` |  | If `typdefaultbin` is not null, it is the `nodeToString()` representation of a default expression for the type. This is only used for domains. |
| `typdefault` | `text` |  | `typdefault` is null if the type has no associated default value. If `typdefaultbin` is not null, `typdefault` must contain a human-readable version of the default expression represented by `typdefaultbin`. If `typdefaultbin` is null and `typdefault` is not, then `typdefault` is the external representation of the type's default value, which can be fed to the type's input converter to produce a constant. |
| `typacl` | `aclitem[]` |  | Access privileges; see [GRANT](https://www.postgresql.org/docs/10/static/sql-grant.html) and [REVOKE](https://www.postgresql.org/docs/10/static/sql-revoke.html) for details |

[Table 51.63](https://www.postgresql.org/docs/10/static/catalog-pg-type.html#CATALOG-TYPCATEGORY-TABLE) lists the system-defined values of `typcategory`. Any future additions to this list will also be upper-case ASCII letters. All other ASCII characters are reserved for user-defined categories.

**Table 51.63. `typcategory` Codes**

| Code | Category |
| :--- | :--- |
| `A` | Array types |
| `B` | Boolean types |
| `C` | Composite types |
| `D` | Date/time types |
| `E` | Enum types |
| `G` | Geometric types |
| `I` | Network address types |
| `N` | Numeric types |
| `P` | Pseudo-types |
| `R` | Range types |
| `S` | String types |
| `T` | Timespan types |
| `U` | User-defined types |
| `V` | Bit-string types |
| `X` | `unknown` type |

