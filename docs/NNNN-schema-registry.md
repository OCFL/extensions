# OCFL Community Extension - Schema Registry

Extension Name: NNNN-schema-registry
Authors: P. Cornwell, D. Granville
Minimum OCFL Version: 1.0
OCFL Community Extensions Version: 1.0
Extension Version: 0.1
Obsoletes: n/a
Obsoleted by: n/a

## Overview
An OCFL object will typically contain metadata serialised in JSON or XML files. These will normally conform to one or more specified schemata and reference the external JSONSchema/XSD/DTD.

In order for an OCFL root to represent a self-consistent repository, the specific versions of each schema referenced by content must be available. Maintaining a local copy of the schema is therefore prudent, especially where longer-term preservation is a goal and external online references cannot be relied upon.

This extension stores a copy of each schema referenced by the content of OCFL objects, thus avoiding the need to store a copy of the schema redundantly in each OCFL object. It also provides a convenient reference point for archive management software and future users inspecting the contents of the OCFL object.

We describe a standardised layout for a schema repository, where the extension stores a copy of all schemata referenced throughout the OCFL root in an OCFL object-like structure:

* The 'schemata' directory contains a reference copy of each schema in use.
* The 'schema_inventory.json' file similar to an OCFL Object's inventory.json - it provides an index and checksums of the stored schemata. This file is also externally checksummed with a sidecar file, in the same manner as OCFL inventory files.

Example file content and structures are shown in the example section.

## Parameters

Configuration is done by setting appropriate values in config.json at the top level of the extension's directory. The keys expected are:

* Name: extensionName
  * Description: String identifying the extension.
  * Type: string
  * Constraints: Must be "NNNN-schema-registry"
  * Default: NNNN-schema-registry

* Name: schemaDigestAlgorithm
  * Description: Algorithm used for calculating safe filenames from schema identifiers. "md5" is the default.
  * Type: string
  * Constraints: Must be a valid digest algorithm returning strings which are safe file names for the target file system.
  * Default: md5

Additionally a schema_registry.json file should be present. This contains the keys -

* Name: digestAlgorithm
  * Description: Digest algorithm used for calculating fixity of the manifest and stored schema files.
  * Type: string
  * Constraints: Must be a valid digest algorithm
  * Default: sha512

* Name: manifest
  * Description: An OCFL-like manifest of the stored schema files.
  * Type: object
  * Constraints: Must contain an entry for each schema in the 'schemata' folder.
  * Default: n/a

* Name: schemaMap
  * Description: Provides a mapping of the original schema identifiers to the digest used to identify the schema.
  * Type: object
  * Constraints: Must contain an entry for each schema in the 'schemata' folder.
  * Default: n/a

Example files are shown below.


### Implementation

In an OCFL root where this extension is configured, tools reading the OCFL objects are not required to be aware of it. However, accessing local copies of schemata is also likely to decrease latency and may provide a small performance benefit. Thus OCFL reading applications relying on schemata access (eg for validation) may wish to consider reading these from the schema registry.

It is important that a system implementing this extension ensures all OCFL objects written are checked for schema references. Where not already registered, the schema is retrieved and registered as described below.

Two possible implementation cases are considered:

* Where an OCFL is being generated as a snapshot for export or backup purposes, it is reasonable to build or check the completeness of the schema registry as each OCFL object is created. Thus the exported root will contain all necessary schemata.

* Where an OCFL provides the persistence layer for a live repository, it is recommended that a background process discovers (or is informed) of new OCFL object/versions. This process can asynchronously process the object contents, retrieve the required schemata and perform necessary registrations with the schema registry.

Since schemata are typically identified by the URL of the specific version referenced, a hashed representation of the identifier is used to create a safe filename for storage. MD5 is likely to be sufficient although other algorithms may be configured. The extension's inventory stores a map of the hashed identifiers, thus allowing easy access to the appropriate schema from unmodified OCFL objects.

### Registering a schema with the extension

* New OCFL objects/versions are inspected for JSON, XML files. 
* Within these files, references to external schema ($schema / XML-DTD etc) are extracted
* The local filename is derived by hashing the schema's identifier using the schemaDigestAlgorithm.
* schemata_inventory.json is checked for this key. If not already present in the schema repository:
  * A copy of the remote schema is retrieved, and stored in the schemata/ directory with this name.
  * The schemata_inventory.json is updated:
    * a ‘manifest’ object entry is created for fixity.
    * an entry in the ‘schemaMap’ object is created to provide a mapping of the hashed identifiers to their original value.
  * The schema_inventory.json’s sidecar file must also be updated.
* If the schema's checksum is already registered, the identifiers must be compared to exclude the possibilty of hash-collision (malicious or otherwise).


## Example

Given OCFL objects:

item1
: with descriptive metadata in content/item1.xml 

```
<!DOCTYPE rdf:RDF SYSTEM "http://dublincore.org/specifications/dublin-core/dcmes-xml/2001-04-11/dcmes-xml-dtd.dtd">
...
```
item2
: with descriptive metadata in content/item2.json

```
{
  "$schema" : "http://schemata.hasdai.org/historic-persons/historic-person-entry-v1.0.0.json"
...
```

The extension is configured via config.json as shown:

```
{
  "extensionName" : "NNNN-schema-registry",
  "schemaDigestAlgorithm": "md5"
}
```

The identifiers' md5 digests are calculated -

The schemata are then registered in the extension's schema_inventory.json as shown below. 


```
{
  "digestAlgorithm": "sha512",
  "manifest": {
    "91da2b...9c8": [
      "schemata/40cdd53d9a263e5466b8954d82d23daa"
    ],
    "31f53b...7ff": [
      "schemata/95d751340dcdc784fd759dbc7ddb9633"
    ]
  }
  "schemaMap": {
    "40cdd53d9a263e5466b8954d82d23daa": "http://dublincore.org/specifications/dublin-core/dcmes-xml/2001-04-11/dcmes-xml-dtd.dtd",
    "95d751340dcdc784fd759dbc7ddb9633": "http://schemata.hasdai.org/historic-persons/historic-person-entry-v1.0.0.json"
  }
}
```

Copies of the schema files are retrieved and placed in the extension folder's schemata directory as shown below. 

```
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  ├── extensions
  │   └── NNNN-schema-registry
  │       └── config.json
  |       └── schema_inventory.json
  |       └── schema_inventory.json.sha512
  │       └── schemata
  │           └── 40cdd53d9a263e5466b8954d82d23daa
  │           └── 95d751340dcdc784fd759dbc7ddb9633
  ├── 0de
  |   └── 45c
  |       └── f24
  |           └── item1
  │               └── 0=ocfl_object_1.0
  │                   └── inventory.json
  │                   └── inventory.json.sha512
  │                   └── v1
  │                       └── inventory.json
  │                       └── inventory.json.sha512
  │                       └── content
  │                           └── item1.xml
  ...
```

