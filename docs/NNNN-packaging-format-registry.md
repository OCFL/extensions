# OCFL Community Extension NNNN: Packaging Format Registry
==========================================================

* **Extension Name:** packaging-format-registry
* **Authors:** Linda Reijnhoudt, Jan van Mansum
* **Minimum OCFL Version:** 1.0
* **OCFL Community Extensions Version:** 1.0
* **Obsoletes:** n/a
* **Obsoleted by:** n/a

Overview
--------

In order for an OCFL repository to be self-contained, it may want to explicitly specify the packaging formats used to package the content of object versions. We
broadly define "packaging format" as a set of rules about the way the content files of an OCFL object version are laid out, organized, and described.

We describe a standard layout for a packaging format registry, similar to [0008-schema-registry](./0008-schema-registry.md), where this extension stores the
specifications for the packaging formats used throughout the OCFL Repository. This packaging format registry must be implemented by creating and maintaining the
following items:

* A `config.json` file containing the configuration parameters for the extension.
* A `packaging_formats` directory containing one subdirectory for each packaging format.
* A `packaging_format_inventory.json` file providing an index for the stored packaging format specifications.
* A sidecar file `packaging_format_inventory.json.sha512` (or other configured digest) containing the digest of the `packaging_format_inventory.json` file, in
  the same manner as the OCFL inventory files.

Parameters
----------

Configuration is done by setting values in the file `config.json` at the top level of the extension's directory. The keys expected are:

- Name: `extensionName`
    - Description: String identifying the extension.
    - Type: String.
    - Constraints: Must be `packaging-format-registry`.
    - Default: `packaging-format-registry`.

- Name: `packagingFormatDigestAlgorithm`
    - Description: Algorithm used for calculating safe filenames from packaging formats.
    - Type: String
    - Constraints: Must be a valid digest algorithm returning strings which are safe file names on the target file system.
    - Default: `md5`.

- Name: `digestAlgorithm`
    - Description: Digest algorithm used for calculating fixity of the packaging registry inventory stored in the sidecar file.
    - Type: String.
    - Constraints: Must be a valid digest algorithm.
    - Default: The same value used elsewhere in the OCFL for integrity checking.

The packaging_formats directory
------------------------------- 

The `packaging_formats` directory is located in the top directory of the `packaging-format-registry` extension. Each packaging format is described in a
subdirectory of this `packaging_formats` directory. The name of this subdirectory must be the hexadecimal serialization of the digest of the string:
`<name>/<version>`, where `<name>` and `<version>` are the name and version of the packaging format as defined in the `packaging_format_inventory.json` (see
below). The digest algorithm used to calculate this digest is defined in the `config.json` file under the key `packagingFormatDigestAlgorithm`.

The subdirectory may include documentation, examples, machine actionable code and other information about the packaging format. How to interpret these files is
outside the scope of this extension. The subdirectory may include other subdirectories, recursively.

The packaging_format_inventory.json file
----------------------------------------

A manifest of the registered packaging formats must be maintained in `packaging_format_inventory.json`. This is a JSON with as top-level key `manifest`, which
contains an object with one entry per registered packaging format.

### Root object properties

- Name: `manifest`
    - Description: Object containing manifest entries.
    - Type: Object.
    - Constraints: Must contain one entry per registered packaging format. The key of this entry is the digest of the string `<name>/<version>` (
      see _[The packaging_formats directory](#the-packaging_formats-directory)_ above). The value of this entry is an object with the properties described
      below.
    - Default: Not applicable.

### Manifest entry properties

- Name: `name`
    - Description: The human-readable name of the packaging format.
    - Type: String.
    - Constraints: Not applicable.
    - Default: Not applicable.
- Name: `version`
    - Description: Version of this packaging format.
    - Type: String.
    - Constraints: The `name` and `version` together must be unique for the packaging formats in this storage root.
    - Default: Not applicable.
- Name: `summary`
    - Description: Short description of the packaging format.
    - Type: String.
    - Constraints: Not applicable.
    - Default: Not applicable.

Implementation
--------------

### Validation

Clients that implement this extension must validate the following as part of the validation of the OCFL storage root:

* JSON files must be well-formed JSON and conform to the schemas specified in this document.
* Manifest entries must correspond one-to-one to the packaging formats in the `packaging_formats` directory.
* Name/version pairs must be unique across the manifest.
* Digest algorithms must be one of the algorithms supported by the OCFL specification.

### Registering a packaging format with the extension

For each referenced packaging format in the OCFL object versions:

1. The implementation must check if the packaging format is already registered in the `packaging_format_inventory.json` manifest.
2. If the packaging format is not registered, the implementation must:
    1. Derive the digest of the string `<name>/<version>` using the `packagingFormatDigestAlgorithm` defined in the `config.json`.
    2. Create a subdirectory in the `packaging_formats` directory with the derived digest as its name.
    3. Store the documentation and other information about the packaging format in this subdirectory.
    4. Update the `packaging_format_inventory.json` manifest with a new entry for the packaging format, using the derived digest as the key and an object with
       the properties `name`, `version`, and `summary` as the value.
    5. Update the sidecar file `packaging_format_inventory.json.sha512` (or other configured digest) with the new digest of the
       `packaging_format_inventory.json` file.

The values of `digestAlgorithm` and `packagingFormatDigestAlgorithm` should not be changed once the registry is initialized. If changing the digest is
unavoidable, all existing entries in the registry must be updated to the new algorithm(s).

### Referencing a packaging format in an OCFL object version

An implementation can determine the packaging format used for an OCFL object version by finding the name and version of the packaging format used. How this is
recorded is outside the scope of this extension, but one possible way is to use
the [NNNN-object-version-properties extension](./NNNN-object-version-properties.md) <!-- pending approval of this extension --> to record the packaging format
as a property of the
object version. Other possibilities include using a designated version level metadata field, or a specific file in the object version content directory.

Example
-------

The `packaging_format_inventory.json` file contains the manifest with the available OCFL Object Packaging Formats for this OCFL Repository.

**packaging_format_inventory.json**

```json
{
  "manifest": {
    "76f773808534f2969d7a405b99e78b11": {
      "name": "BagIt",
      "version": "v0.97",
      "summary": "a hierarchical file packaging format for storage and transfer of arbitrary digital content."
    },
    "05b408a38e341de9bb4316aa812115ee": {
      "name": "BagIt",
      "version": "v1.0",
      "summary": "https://datatracker.ietf.org/doc/html/rfc8493"
    }
  }
}
```

The storage root of the OCFL repository would then look something like this:

```text
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  └── extensions
      └── packaging-format-registry/
          ├── config.json
          ├── packaging_format_inventory.json
          ├── packaging_format_inventory.json.sha512
          └── packaging_formats
              ├── 05b408a38e341de9bb4316aa812115ee
              |   ├── rfc8493.txt
              |   └── ... files describing the packaging_format BagIt/v1.0 ...
              └── 76f773808534f2969d7a405b99e78b11                  
                  └── ... files describing the packaging_format BagIt/v0.97 ...  
  
```

