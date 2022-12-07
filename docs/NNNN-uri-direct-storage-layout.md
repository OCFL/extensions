# OCFL Community Extension NNNN: URI Direct Storage Layout

  * **Extension Name:** NNNN-uri-direct-storage-layout
  * **Authors:** Alvin Sebastian, Peter Sefton
  * **Minimum OCFL Version:** 1.0
  * **OCFL Community Extensions Version:** 1.0
  * **Obsoletes:** n/a
  * **Obsoleted by:** n/a

## Overview

This storage root extension describes a transparent path-based OCFL storage layout. URI and path based identifiers are mapped directly to multi-level directory path that are direct children of the OCFL storage root directory.

This extension assumes that the OCFL object identifier is a URI or a path name which is used directly to create nested paths under the OCFL storage root. An extra directory called `__object__` is added to the path to safely ensure that OCFL object is not nested under anoother OCFL object.

The limitations of this layout are filesystem dependent, but are generally as follows:

* The size of each directory name cannot exceed the maximum allowed directory name size (eg. 255 bytes). The maximum combined length of path name cannot exceed the maximum allowed limit (eg. 4096 bytes).
* Object IDs cannot include characters that are illegal in directory names.
* Depending on the path structure, performance may degrade as the size of a repository increases if a lot of objects are put under a single directory.


## Parameters

* **Name:** replace
  * **Description:** An array of [string, string] that contains a regular expression pattern and a replacement string.
  * **Type:** array of [string, string]
  * **Default:** [] (An empty array)
* **Name:** suffix
  * **Description:** The suffix to be appended to the end of the path
  * **Type:** string
  * **Default:** "/__object__"

## Procedure

The following is an outline of the steps to map an OCFL object identifier to an OCFL object root path:
1. For each tuple `[pattern, replacement]` in `replace` paramater, replace the matching `pattern` from the OCFL object identifier with `replacement` string.
2. If the identifier is a URI:
  * If the URI scheme is `file:`, remove the scheme from the path
  * If the URI contains hostname:
      * Replace any occurrence of `,` character with `_`
      * Replace any occurrence of `;` character with `/`
  * Construct the object root path by joining the schema and hostname, if exists, with an underscore `_` and the rest of the pathname verbatim
3. If the identifier is not a valid URI, assume that it is a pathname. Remove any leading and trailing slash `/` from it.
4. Append the `suffix` to the final pathname.

## Examples
### Example 1

This example demonstrates what the OCFL storage hierarchy looks like when using the default configuration.

#### Mappings

NOTE: The [The Archive and Package (arcp) URI scheme
](https://www.research.manchester.ac.uk/portal/files/76956641/arcp.html) (ARCP) is used in these examples; it allows URI IDs to be minted locally by an archive which can be used in Linked Data systems as URIs.

| Object ID | Object Root Path |
| --- | --- |
| https://example.com/a | `https_example.com/a/__object__` |
| https://example.com/a/b.c | `https_example.com/a/b.c/__object__` |
| arcp://name,md/a/b/c | `arcp_name_md/a/b/c/__object__` |
| arcp://ni,sha-256;f4OxZX_x_FO5LcGBSKHWXfwtSx-j1ncoSt3SABJtkGk/ | `arcp_ni_sha-256/f4OxZX_x_FO5LcGBSKHWXfwtSx-j1ncoSt3SABJtkGk/__object__` |
| file:///temp/a/b | `temp/a/b/__object__` |
| file://temp/a/b | `temp/a/b/__object__` |
| doi:/10.3897/rio.8.e93937 | `doi/10.3897/rio.8.e93937/__object__` |
| //a/b/c | `a/b/c/__object__` |
| /a/b/c | `a/b/c/__object__` |
| a/b/c | `a/b/c/__object__` |

#### Storage Hierarchy

```
[storage_root]/
├── 0=ocfl_1.0
├── ocfl_layout.json
├── https_example.com
│   └── a
│       ├── __object__
│       │   ├── 0=ocfl_object_1.0
│       │   ├── inventory.json
│       │   ├── inventory.json.sha512
│       │   └── v1 [...]
│       └── b.c
│           └── __object__
│               ├── 0=ocfl_object_1.0
│               ├── inventory.json
│               ├── inventory.json.sha512
│               └── v1 [...]
├── arcp_name_md
│   └── a
│       └── b
│           └── c
│               └── __object__ [...]
[...]
```


### Example 2

This example demonstrates the effects configuring `replace` to do a static string replacement.
This feature can be used to remove the scheme and/or host name from the result.

#### Parameters

```json
{
    "extensionName": "NNNN-uri-direct-storage-layout",
    "replace": [["https://example\.com", "example"], ["(.+)doi\.org", ""]]
}
```

#### Mappings

| Object ID | Object Root Path |
| --- | --- |
| https://example.com/object-01 | `example/object-01/__object__` |
| https://doi.org/10.3897/rio.8.e93937 | `10.3897/rio.8.e93937/__object__` |


### Example 3

This example demonstrates the effect of using a custom `suffix` to change the default `__object__` name convention as the leaf directory that contains an OCFL Object. If set to an empty string, the user must ensure that all the supplied URIs have a structure that does not allow nested objects.

#### Parameters

```json
{
    "extensionName": "NNNN-uri-direct-storage-layout",
    "suffix": ".object"
}
```

#### Mappings

| Object ID | Object Root Path | Notes
| --- | --- | --- |
| /a/b | `a/b.object` | |
| /a/b/c | `a/b/c.object` | |
| /a/b/c2 | `a/b/c2.object` | |
