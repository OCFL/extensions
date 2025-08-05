OCFL Community Extension NNNN: Object Version Properties
=========================================

* **Extension Name:** NNNN-object-version-properties
* **Authors:** Linda Reijnhoudt, Jan van Mansum
* **Minimum OCFL Version:** 1.0
* **OCFL Community Extensions Version:** 1.0
* **Obsoletes:** n/a
* **Obsoleted by:** n/a

Overview
--------

This extension provides a way to record properties of an OCFL object version beyond the standard properties defined by OCFL itself (`created`, `message`, etc.).
This may be useful for cases where additional information about the version is needed, which cannot conveniently be stored in the object itself in a standard
way. Another use-case may be a property that is not available at the time the version content is created.

This extension only describes how the properties should be recorded, it does not prescribe what properties should be recorded or their semantics. One way to
define the semantics of properties is to use the [Property Registry extension](../property-registry/property-registry.md).

The object_version_properties.json file
---------------------------------------

The object version properties are stored in a JSON file named `object_version_properties.json` in the extensions directory of the OCFL object. This file must
contain entries for all versions in this object. Each version entry has as key the version identifier (e.g. `v1`, `v2`, etc.) and as value a JSON object with
the properties for that version. If no properties are recorded for a version, the entry for that version must be an empty JSON object.

The properties recorded under a version key pertain to that version only, so when a new version is created, properties that do not change must be copied from
the previous version. Otherwise, this extension does not restrict the properties that can be recorded, nor does it define their semantics.

Implementation
--------------

Clients that implement this extension must validate that the `object_version_properties.json` file is well-formed JSON and that it contains an entry for each
version in the object as part of the validation of the OCFL object.

Clients may support storing the version properties when creating a new version of an OCFL object, by passing the properties map as an extra parameter to the
version creation method. Likewise, clients may support retrieving the properties for a specific version of an OCFL object as a properties map. Other optional
features may include the ability to search for object versions based on their properties.

Examples
--------

### 1. Simple property

If the repository wants to record a simple property, for instance the User-Agent that made the request to create this version, this could be done as follows.

```text
[storage_root]
  ├── 0=ocfl_1.0
  ├── ocfl_1.0.txt
  ├── ocfl_layout.json
  ├── object-version-properties.md
  ├── 0de
  |   └── 45c
  |       └── f24
  |           └── item1
  │               └── 0=ocfl_object_1.0
  │                   ├── inventory.json
  │                   ├── inventory.json.sha512
  |                   └── extensions
  │                        └── object-version-properties/
  │                            ├── object_version_properties.json
  │                            └── object_version_properties.json.sha512
  └────────────────── v1/
                      ├── inventory.json
                      ├── inventory.json.sha512
                      └── content/
                          └── ... files ...
```

The `object-version-properties.json` of the OCFL object could have the following entry:

```json
{
  "v1": {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
  }
}
```

### 2. Complex property

If the repository wants to record a complex object for a version, for instance that it has been deaccessioned, this could be done with a similar structure. The
`object-version-properties.json` of the OCFL object might have the following entry:

```json
{
  "v1": {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
  },
  "v2": {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)",
    "deaccessioned": {
      "datetime": "2025-10-15T13:19:00",
      "reason": "Deaccessioned because dataset was deleted in Easy"
    }
  }
}
```

Note, that this example also demonstrates that a property that remains unchanged between versions (in this case "User-Agent") must be copied from the previous
version.
