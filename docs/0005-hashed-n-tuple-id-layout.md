# OCFL Community Extension 0005: Hashed Truncated N-tuple Trees with Encapsulating Directory for OCFL Storage Hierarchies

  * **Extension ID:** 0005-hashed-n-tuple-id-layout
  * **Authors:** Ben Cail
  * **Minimum OCFL Version:** 1.0
  * **Obsoletes:** n/a
  * **Obsoleted by:** n/a

## Overview

This storage root extension describes how to safely map OCFL object identifiers containing any characters to OCFL object root directories.

Using this extension, OCFL object identifiers are hashed and encoded as hex strings (all letters lower-case). These digests are then divided into _N_ n-tuple segments, which are used to create nested paths under the OCFL storage root. Finally, the OCFL object identifier is URL-encoded to create a directory name for the OCFL object root.

The n-tuple segments approach allows OCFL object identifiers to be evenly distributed across the storage hierarchy. The maximum number of files under any given directory is controlled by the number of characters in each n-tuple, and the tree depth is controlled by the number of n-tuple segments each digest is divided into. The encoded encapsulation directory name provides visibility into the object identifier from the file path, but it does require that the encoded object identifier not be longer than 255 characters (because of filesystem limitations on the length of a directory name).

## Parameters

### Summary

* **Name:** `digestAlgorithm`
  * **Description:** The digest algorithm to apply on the OCFL object identifier; MUST be an algorithm that is allowed in the OCFL fixity block
  * **Type:** string
  * **Range:** 1,4095
  * **Default:** sha256
* **Name**: `tupleSize`
  * **Description:** Indicates the size of the segments (in characters) that the digest is split into
  * **Type:** integer
  * **Range:** 0,32
  * **Default:** 3
* **Name:** `numberOfTuples`
  * **Description:** Indicates how many segments are used for path generation
  * **Type:** integer
  * **Range:** 0,32
  * **Default:** 3

### Details

#### digestAlgorithm

`digestAlgorithm` is defaulted to `sha256`,and it MUST either contain a digest algorithm that's [officially supported by the OCFL spec](https://ocfl.io/draft/spec/#digest-algorithms) or defined in a community extension. The specified algorithm is applied to OCFL object identifiers to produce hex encoded digest values that are then mapped to OCFL object root paths.

#### tupleSize

`tupleSize` determines the number of digest characters to include in each tuple. The tuples are used as directory names. The default value is `3`, which means that each directory in the OCFL storage hierarchy could contain up to 4096 files. Increasing this value increases the maximum number of files per directory.

If `tupleSize` is set to `0`, then no tuples are created and `numberOfTuples` MUST also equal `0`.

The product of `tupleSize` and `numberOfTuples` MUST be less than or equal to the number of characters in the hex encoded digest.

#### numberOfTuples

`numberOfTuples` determines how many tuples to create from the digest. The tuples are used as directory names, and each successive directory is nested within the previous. The default value is `3`, which means that every OCFL object root will be 4 directories removed from the OCFL storage root, 3 tuple directories plus 1 encapsulation directory. Increasing this value increases the depth of the OCFL storage hierarchy.

If `numberOfTuples` is set to `0`, then no tuples are created and `tupleSize` MUST also equal `0`.

The product of `numberOfTuples` and `tupleSize` MUST be less than or equal to the number of characters in the hex encoded digest.

## Procedure

The following is an outline of the steps to follow to map an OCFL object identifier to an OCFL object root path using this extension:

1. The OCFL object identifier is encoded as UTF-8 and hashed using the specified `digestAlgorithm`.
2. The digest is encoded as a lower-case hex string.
3. Starting at the beginning of the digest and working forwards, the digest is divided into `numberOfTuples` tuples each containing `tupleSize` characters.
4. The tuples are joined, in order, using the filesystem path separator.
5. The OCFL object identifier is URL-encoded to create the encapsulation directory name: all characters that aren't letters, digits, "\_" or "-" are changed to the %xx equivalent of their UTF-8 bytes. For example, "..hor/rib:lè-$id" becomes "%2e%2ehor%2frib%3al%c3%a8-%24id" as the encapsulation directory name.
6. The whole encapsulation directory name is made lower-case, and is joined on the end of the path.

## Examples

### Example 1

This example demonstrates what the OCFL storage hierarchy looks like when using this extension's default configuration.

#### Parameters

It is not necessary to specify any parameters to use the default configuration. However, if you were to do so, it would look like the following:

```json
{
    "digestAlgorithm": "sha256",
    "tupleSize": 3,
    "numberOfTuples": 3
}
```

#### Mappings

| Object ID | Digest | Object Root Path |
| --- | --- | --- |
| object-01 | 3c0ff4240c1e116dba14c7627f2319b58aa3d77606d0d90dfc6161608ac987d4 | `3c0/ff4/240/object-01` |
| ..hor/rib:le-$id | 487326d8c2a3c0b885e23da1469b4d6671fd4e76978924b4443e9e3c316cda6d | `487/326/d8c/%2e%2ehor%2frib%3ale-%24id` |

#### Storage Hierarchy

```
[storage_root]/
├── 0=ocfl_1.0
├── ocfl_layout.json
├── extensions/
│   └── 0005-hashed-n-tuple-id-layout/
│       └── 0005-hashed-n-tuple-id-layout.json
├── 3c0/
│   └── ff4/
│       └── 240/
│           └── object-01/
│               ├── 0=ocfl_object_1.0
│               ├── inventory.json
│               ├── inventory.json.sha512
│               └── v1 [...]
└── 487/
    └── 326/
        └── d8c/
            └── %2e%2ehor%2frib%3ale-%24id/
                ├── 0=ocfl_object_1.0
                ├── inventory.json
                ├── inventory.json.sha512
                └── v1 [...]
```

### Example 2

The example demonstrates the effects of modifying the default parameters to use a different `digestAlgorithm`, smaller `tupleSize`, and a larger `numberOfTuples`.

#### Parameters

```json
{
    "digestAlgorithm": "md5",
    "tupleSize": 2,
    "numberOfTuples": 15
}
```

#### Mappings

| Object ID | Digest | Object Root Path |
| --- | --- | --- |
| object-01 | ff75534492485eabb39f86356728884e | `ff/75/53/44/92/48/5e/ab/b3/9f/86/35/67/28/88/object-01` |
| ..hor/rib:le-$id | 08319766fb6c2935dd175b94267717e0 | `08/31/97/66/fb/6c/29/35/dd/17/5b/94/26/77/17/%2e%2ehor%2frib%3ale-%24id` |

#### Storage Hierarchy

```
[storage_root]/
├── 0=ocfl_1.0
├── ocfl_layout.json
├── extensions/
│   └── 0005-hashed-n-tuple-id-layout/
│       └── 0005-hashed-n-tuple-id-layout.json
├── 08/
│   └── 31/
│       └── 97/
│           └── 66/
│               └── fb/
│                   └── 6c/
│                       └── 29/
│                           └── 35/
│                               └── dd/
│                                   └── 17/
│                                       └── 5b/
│                                           └── 94/
│                                               └── 26/
│                                                   └── 77/
│                                                       └── 17/
│                                                           └── %2e%2ehor%2frib%3ale-%24id/
│                                                               ├── 0=ocfl_object_1.0
│                                                               ├── inventory.json
│                                                               ├── inventory.json.sha512
│                                                               └── v1 [...]
└── ff/
    └── 75/
        └── 53/
            └── 44/
                └── 92/
                    └── 48/
                        └── 5e/
                            └── ab/
                                └── b3/
                                    └── 9f/
                                        └── 86/
                                            └── 35/
                                                └── 67/
                                                    └── 28/
                                                        └── 88/
                                                            └── object-01/
                                                                ├── 0=ocfl_object_1.0
                                                                ├── inventory.json
                                                                ├── inventory.json.sha512
                                                                └── v1 [...]
```

### Example 3

This example demonstrates what happens when `tupleSize` and `numberOfTuples` are set to `0`. This is an edge case and not a recommended configuration.

#### Parameters

```json
{
    "digestAlgorithm": "sha256",
    "tupleSize": 0,
    "numberOfTuples": 0
}
```

#### Mappings

| Object ID | Digest | Object Root Path |
| --- | --- | --- |
| object-01 | 3c0ff4240c1e116dba14c7627f2319b58aa3d77606d0d90dfc6161608ac987d4 | `object-id` |
| ..hor/rib:le-$id | 487326d8c2a3c0b885e23da1469b4d6671fd4e76978924b4443e9e3c316cda6d | `%2e%2ehor%2frib%3ale-%24id` |

#### Storage Hierarchy

```
[storage_root]/
├── 0=ocfl_1.0
├── ocfl_layout.json
├── extensions/
│   └── 0005-hashed-n-tuple-id-layout/
│       └── 0005-hashed-n-tuple-id-layout.json
├── object-id/
│   ├── 0=ocfl_object_1.0
│   ├── inventory.json
│   ├── inventory.json.sha512
│   └── v1 [...]
└── %2e%2ehor%2frib%3ale-%24id/
    ├── 0=ocfl_object_1.0
    ├── inventory.json
    ├── inventory.json.sha512
    └── v1 [...]
```

