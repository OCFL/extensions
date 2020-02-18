# OCFL Community Extension 0002: N-tuple Trees for OCFL Storage Hierarchies

  * Authors: Neil Jefferies
  * Minimum OCFL Version: 1.0
  * Obsoletes: n/a
  * Obsoleted by: n/a

## Overview

This extension provides a general mechanism for describing a set of algorithms for mapping numerical identifers onto a tree structure for constructing an OCFL Storage Hierarchy in a way that maintains performance. The aim is to balance the problem of having too many files or subdirectories in any one directory with having an overly deeply nested set of subdirectories. As both identifier formats and specific filesystem instances may vary there is not a single optimal approach.       

## Parameters

* name: identifier length
  * description: The number of meaningful characters in the base identifier 
  * type: integer
  * range: 0,4096
  * default:
* name: case mapping
  * description: Indicates how case in the source identifier should be handled when mapping to a path
  * type: enumerated
  * range: ToUpper,ToLower,Literal
  * default:
* name: tuple size
  * description: Indicates the size of the chunks (in characters) that the identifier is split into during mapping
  * type: integer
  * range: 1,32
  * default:
* name: number of tuples
  * description: Indicates how many chunks are used for path generation, 0 means use the entire identifer 
  * type: integer
  * range: 0,32
  * default:
* object root format
  * description: Indicates how the OCFL object root directory name should be generated from the identifier
  * type: enumerated
  * range: Full,Stripped,Tail
  * default:

## Detailed explanation

The approach described here is a generalisation of the [PairTree](https://tools.ietf.org/html/draft-kunze-pairtree-01) algorithm designed to be more flexible as file storage technolgies have developed. Conventional filesystems are generally better able to handle large numbers of files in a directory and object stores tend to favour much flatter storage hierarchies. In short, the approach is to derive a unique file path for an OCFL object from its unique identifier in a programmatic and repeatable manner. 

### identifier length

This extension assumes that object unique identifiers are all the same length and that all the characters of the identifer are reasonably well distributed (e.g. for a hexadecimal-based identifier, each character can be any value from 0-f). This may mean that it is prudent to adjust the format of the identifier before it can be safely used to generate a path. For example, a [UUID](https://tools.ietf.org/html/rfc4122) is typically written in the form "uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6" where the "uuid:" premable and the hyphens are clearly non-unique elements. The "stripped" version used for path generation would be "f81d4fae7dec11d0a76500a0c91e6bf6". The **identifier length** parameter indicates the length of the stripped version of the identifier, in this example, 32.   

### case mapping

Depending on how identifiers are generated they may not always be consistent in their usage of case. A UUID is actually a hexdecimal number, so both "f81d4fae7dec11d0a76500a0c91e6bf6" and "F81D4FAE7DEC11D0A76500A0C91E6BF6" would be valid renderings of the example given above. However, many storage systems are case sensitive so if we want to map identifiers to paths consistently we need to specify which case to map to. The **case mapping** parameter allows this to be specified but also allows for identifiers that are lexical and thus should not necessarily be case mapped. Note especially that upper/lower case mappings are often language/locale dependent for characters outside the basic \[A-z\]\[a-z\] range and thus quite likely to be non-portable.

### tuple size



... more in here ...
