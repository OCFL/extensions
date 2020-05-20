# OCFL Community Extension 0002: N-tuple Trees for OCFL Storage Hierarchies

  * Authors: Neil Jefferies
  * Minimum OCFL Version: 1.0
  * Obsoletes: n/a
  * Obsoleted by: n/a

## Overview

This extension provides a general mechanism for describing a set of algorithms for mapping fixed length numerical identifers 
onto a tree structure for constructing an OCFL Storage Hierarchy in a way that maintains performance. The aim is to balance
the problem of having too many files or subdirectories in any one directory with having an overly deeply nested set of 
subdirectories. As both identifier formats and specific filesystem instances may vary there is not a single optimal approach.       

## Parameters

* name: identifierLength
  * description: The number of meaningful characters in the base identifier 
  * type: integer
  * range: 1,255
  * default:
* name: caseMapping
  * description: Indicates how case in the source identifier should be handled when mapping to a path
  * type: enumerated
  * range: "toUpper","toLower","literal"
  * default:
* name: invertMapping
  * description: Indicate if mapping should begin at the rightmost (least significant) character of the stripped identifier
  * type: boolean
  * default: flase
* name: tupleSize
  * description: Indicates the size of the chunks (in characters) that the identifier is split into during mapping
  * type: integer
  * range: 0,32
  * default: 2
* name: numberOfTuples
  * description: Indicates how many chunks are used for path generation
  * type: integer
  * range: 0,32
  * default:
* name: shortObjectRoot 
  * description: When true, indicates that the OCFL object root directory name should contain the remainder of the identifier not used in the n-tuples
  * type: boolean
  * default: false

## Detailed explanation

The approach described here is a generalization of the [PairTree](https://tools.ietf.org/html/draft-kunze-pairtree-01) 
algorithm applied to fixed length identifiers. It is designed to be more flexible to reflect developments in file storage
technologies. Conventional filesystems have become better able to handle large numbers of files in a directory and object
stores tend to favour much flatter storage hierarchies. In short, the approach is to derive a unique file path for an OCFL object from its unique identifier in a programmatic and repeatable manner that can also be derived relatively easily from filesystem inspection in the absence of documentation. 

### identifierLength

This extension assumes that object unique identifiers are all the same length and that all the characters of the identifer
are reasonably well distributed (e.g. for a hexadecimal-based identifier, each character can be any value from 0-f). This
may mean that it is prudent to adjust the format of the identifier before it can be safely used to generate a path. 
For example, a [UUID](https://tools.ietf.org/html/rfc4122) is typically written in the form 
"uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6" where the "uuid:" premable and the hyphens are clearly non-unique elements.
The "stripped" version used for path generation would be "f81d4fae7dec11d0a76500a0c91e6bf6". The **identifierLength** 
parameter indicates the length of the stripped version of the identifier, in this example, 32.

### caseMapping

Depending on how identifiers are generated they may not always be consistent in their usage of case. A UUID is actually
a hexdecimal number, so both "f81d4fae7dec11d0a76500a0c91e6bf6" and "F81D4FAE7DEC11D0A76500A0C91E6BF6" would be valid 
renderings of the example given above. However, many storage systems are case sensitive so if we want to map identifiers 
to paths consistently we need to specify which case to map to. The **caseMapping** parameter allows this to be specified 
but also allows for identifiers that are lexical and thus should not necessarily be case mapped. Note especially that
upper/lower case mappings are often language/locale dependent for characters outside the basic \[A-z\]\[a-z\] range and 
thus quite likely to be non-portable.

### invertMapping

Some identifers are generated sequentially. In this case the N-tuple tree approach to generating paths does not generate
a well distributed tree if we begin at the leftmost (most significant) end of the stripped identifier. Instead, we end up
with a small number of fully populated individual branches which can be inefficient for larger numbers of objects. In order
to avoid this the *invert mapping* option indicates that mapping should start from the rightmost (least significant) end of
the identifier which changes most rapidly and therefore distributes more effectively. In the example,
"f81d4fae7dec11d0a76500a0c91e6bf6" would be flipped to "6fb6e19c0a00567a0d11ced7eaf4d18f" before path conversion.

Note that the mapping, and its inverse, operates at a character level and not bit-wise. Many identifier schemes are designed
to be opaque and thus have pseudo-random characteristics so the mapping defaults to the more intuitive most-signficant to
least-signficant approach.          

### tupleSize

Indicates the size of the chunks that the identifier is split into during path generation. The optimal chunk size depends 
on a number of factors:
* The number of values that each character in the identifier can have. For example, UUID's are hexadecimal based so each character may be in the range \[0-9,a-f\] giving 16 different values whereas an alphanumeric identifier might have the range \[0-9,a-z,A-Z\] giving 62 values.
* The characteristics of the underlying storage and associated code libraries. Although not the case in the past, modern storage systems can generally handle tens of thousands of files in a directory without difficulty. It is more likely that the code libraries and tools used to access and parse these systems will encounter some performance limitations when handling large numbers of files. In particular, Linux command-line wildcard expansions are typically limited to just under 128K characters which equates to around 4000 directory names if they have 32 characters. 
* Human readability is also reduced for long lists of files, which may make recovery *in extremis* more difficult.

For a **tupleSize** of 3, our example UUID of "f81d4fae7dec11d0a76500a0c91e6bf6" would be split up into a path beginning "/f81/d4f/ae7/...".

A **tupleSize** of zero indicates that identifiers are not split unto tuples at all - giving a flat structure with folders 
named for the source identifier. This can be useful for object stores that do not support the directory paradigm.

### numberOfTuples

In practice, you may wish to limit the depth of the OCFL Storage Hierarchy tree to avoid overly deep nesting of directories. 
The **numberOfTuples** determines this depth. Splitting the entire identifier down into tuples may not be necessary since the
number of objects to be stored is often much less than all the possible values for the source identifier. 

For example, if we split the example UUID into size 3 tuples, each directory can contain 4096 subdirectories so, with the 
number of tuples also set to three the resulting tree would have 4096^3 (=68719476736) directories, each of which could 
contain one or more objects. All the objects with UUID's that begin f81d4fae7... would be stored in OCFL object roots in the
directory /f81/d4f/ae7/. However, if the UUID's are reasonably pseudo-randomly distributed, the likelihood of many object
identifiers sharing even the first 9 characters is quite low until a signficant number of objects have been created.

The nature of the N-tuple algorithm means that the following additional conditions on parameters MUST be observed:
* **numberOfTuples * tuplesSize <= identifierLength**
* If **tupleSize = 0** then **numberOfTuples = 0**

### shortObjectRoot

Once a Storage Hierarchy has been created, the name of each OCFL Object Root directory should also be determined from the identifier in a consistent manner. The default approach is to use the full (stripped) identifier but, if identifiers are long or there is the need to keep Storage Hierarchy paths short because of object complexity, there is the option to just use the portion of the identifier that remains after Storage Hierarchy path generation. To continue the UUID example the full OCFL Object Root path could be:    
* **shortObjectRoot = false** /f81/d4f/ae7/f81d4fae7dec11d0a76500a0c91e6bf6/
* **shortObjectRoot = true** /f81/d4f/ae7/dec11d0a76500a0c91e6bf6/

Note, if **numberOfTuples * tuplesSize = identifierLength** there is no remaining portion of the identifer and so
**shortObjectRoot** MUST be false.

## Examples

These examples are taken from the OCFL Implementation notes:

* *Flat*: Each object is contained in a directory with a name that is simply derived from the unique identifier of the object.
  * identiferLength = 12
  * caseMapping = "toLower"
  * invertMapping = false
  * tupleSize = 0
  * numberOfTuples = 0
  * shortObjectRoot = false

                [storage_root]
                    ├── 0=ocfl_1.0
                    ├── ocfl_1.0.html (optional copy of the OCFL specification)
                    ├── d45be626e024
                    |   ├── 0=ocfl_object_1.0
                    |   ├── inventory.json
                    |   ├── inventory.json.sha512
                    |   └── v1...
                    ├── d45be626e036
                    |   ├── 0=ocfl_object_1.0
                    |   ├── inventory.json
                    |   ├── inventory.json.sha512
                    |   └── v1...
                    ├── 3104edf0363a
                    |   ├── 0=ocfl_object_1.0
                    |   ├── inventory.json
                    |   ├── inventory.json.sha512
                    |   └── v1...
                    └── ...
                

* PairTree: [PairTree] is designed to overcome the limitations on the number of files in a directory that most file systems have. It creates hierarchy of directories by mapping identifier strings to directory paths two characters at a time. For numerical identifiers specified in hexadecimal this means that there are a maximum of 256 items in any directory which is well within the capacity of any modern filesystem. However, for long identifiers, pairtree creates a large number of directories which will be sparsely populated unless the number of objects is very large. Traversing all these directories during validation or rebuilding operations can be slow.
  * identiferLength = 12
  * caseMapping = "toLower"
  * invertMapping = false
  * tupleSize = 2
  * numberOfTuples = 6
  * shortObjectRoot = false

                [storage_root]
                    ├── 0=ocfl_1.0
                    ├── ocfl_1.0.html (optional copy of the OCFL specification)
                    ├── d4
                    |   └── 5b
                    |       └── e6
                    |           └── 26
                    |               └── e0
                    |                   ├── 24
                    |                   |   └──d45be626e024
                    |                   |       ├── 0=ocfl_object_1.0
                    |                   |       └── ...
                    |                   └── 36
                    |                       └──d45be626e036
                    |                           ├── 0=ocfl_object_1.0
                    |                           └── ...
                    ├── 31
                    |   └── 04
                    |       └── ed
                    |           └── f0
                    |               └── 36
                    |                   └── 3a
                    |                       └── 3104edf0363a
                    |                           ├── 0=ocfl_object_1.0
                    |                           └── ...
                    └── ...
                

* Truncated n-tuple Tree: This approach aims to achieve some of the scalability benefits of PairTree whilst limiting the depth of the resulting directory hierarchy. To achieve this, the source identifier can be split at a higher level of granularity, and only a limited number of the identifier digits are used to generate directory paths. For example, using triples and three levels with the example above yields:
  * identiferLength = 12
  * caseMapping = "toLower"
  * invertMapping = false
  * tupleSize = 3
  * numberOfTuples = 3
  * shortObjectRoot = false

                [storage_root]
                    ├── 0=ocfl_1.0
                    ├── ocfl_1.0.html (optional copy of the OCFL specification)
                    ├── d45
                    |   └── be6
                    |       └── 26e
                    |           ├──d45be626e024
                    |           |  ├── 0=ocfl_object_1.0
                    |           |  └── ...
                    |           └──d45be626e036
                    |              ├── 0=ocfl_object_1.0
                    |              └── ...
                    ├── 310
                    |   └── 4ed
                    |       └── f03
                    |           └── 3104edf0363a
                    |               ├── 0=ocfl_object_1.0
                    |               └── ...
                    └── ...
               
