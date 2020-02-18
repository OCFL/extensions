# OCFL Community Extension 0002: N-tuple Trees for OCFL Storage Hierarchies

  * Authors: Neil Jefferies
  * Minimum OCFL Version: 1.0
  * Obsoletes: n/a
  * Obsoleted by: n/a

## Overview

This extension provides a general mechanism for describing a set of algorithms for mapping numerical identifers onto a tree structure for constructing an OCFL Storage Hierarchy in a way that maintains peformance. The aim is to balance the problem of having too many files or subdirectories in any one directory with having an overly deeply nested set of subdirectories. As both identifier formats and specific filesystem instances may vary there is not a single optimal approach.       

## Parameters

* IdentifierLength
  * description: The number of meaningful characters in the base identifier 
  * type: integer
  * range: 0,4096
  * default:
* CaseMapping
  * description: Indicates how case in the source identifier should be handled when mapping to a path
  * type: enumerated
  * range: ToUpper,ToLower,Literal
  * default:
* TupleSize
  * description: Indicates the size of the chunks (in characters) that the identifier is split into during mapping
  * type: integer
  * range: 1,32
  * default:
* NumberOfTuples
  * description: Indicates how many chunks are used for path generation
  * type: integer
  * range: 1,32
  * default:
* ObjectRootFormat
  * description: Indicates how the OCFL object root directory name should be generated from the identifier
  * type: enumerated
  * range: Full,Stripped,Tail
  * default:

## Detailed explanation

The algorithms described here is a generalisation of the PairTree 

... more in here ...
