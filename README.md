# OCFL Community Extensions

This repository contains community extensions to the [OCFL Specification and Implementation Notes](https://ocfl.io/). Extensions are a means of adding additional functionality and documenting standards outside of the main OCFL specification process. For example, storage layout extensions define how OCFL object IDs are mapped to OCFL object root directories within an OCFL storage root. This mapping is outside of the scope of the OCFL spec, but is valuable information to capture so that repositories are self-describing.

This is a community driven repository. Community members are encouraged to contribute by submitting new extensions and reviewing others' submissions. For more details, see the [review/merge policy](#review--merge-policy) below.

The current set of adopted extensions are listed [here](https://ocfl.github.io/extensions/).

Extensions the are open for review and discussion are listed [here](https://github.com/OCFL/extensions/pulls).

## Using Community Extensions

To use an extension you need an OCFL client that supports the desired extensions. OCFL clients do not need to support extensions to be compliant with the OCFL spec.

## Implementing Community Extensions

Reference the spec's description of [object extensions](https://ocfl.io/1.0/spec/#object-extensions) and [storage root extensions](https://ocfl.io/1.0/spec/#storage-root-extensions).

Each extension specification details how it should be implemented, but there are a few general rules that apply to every extension.

The OCFL storage root MAY contain a copy of the extension specification.

Extension parameters are serialized as a JSON object and written to a file that is named for its *Registered Name* with a `.json` extension. If the extension is a storage layout extension and referenced in `ocfl_layout.json`, then the parameter file MUST be written to the storage root. Otherwise, it MUST be written to the extension's extension directory.

For example, the extension [0000-example-extension](docs/0000-example-extension.md) could be parameterized as follows:

```json
{ 
  "firstExampleParameter": 12, 
  "secondExampleParameter": "Hello", 
  "thirdExampleParameter": "Green" 
}
```

Based how the extension is used, its parameter file is written to one of the following locations, relative the storage root:

* `0000-example-extension.json`, if it is a storage layout extension
* `extensions/0000-example-extension/0000-example-extension.json`, if it is a [storage root extension](https://ocfl.io/1.0/spec/#storage-root-extensions)
* `OBJECT_ROOT/extensions/0000-example-extension/0000-example-extension.json`, if it is an [object extension](https://ocfl.io/1.0/spec/#object-extensions)

## Specifying Community Extensions

### Layout

Community extensions should be written as GitHub flavored markdown files in the `docs` directory of this repository. The
filename of an extension is based on its *Registered Name* with a `.md` extension.

Extensions are numbered sequentially, and the *Registered Name* of an extension is prefixed with this 4-digit, zero-padded
decimal number. The *Registered Name* should be descriptive, use hyphens to separate words, and have a maximum of 250
characters in total. New extensions MUST use the next available prefix number that's available at the time of merging.

Extensions are intended to be mostly static once published. Substantial revisions of content beyond simple fixes warrants publishing a new extension, and marking the old extension obsolete by updating the *Obsoletes/Obsoleted by* sections in each extension respectively.

An example/template is available [here](docs/0000-example-extension.md) and is rendered
via GitHub pages as https://ocfl.github.io/extensions/0000-example-extension

### Headers

Extensions MUST contain a header section that defines the following fields:

* **Extension Name**: The extension's unique *Registered Name*
* **Authors**: The names of the individuals who authored the extension
* **Minimum OCFL Version**: The minimum OCFL version that the extension requires, eg. *1.0*
* **Obsoletes**: The *Registered Name* of the extension that this extension obsoletes, or *n/a*
* **Obsoleted by**: The *Registered Name* of the extension that obsoletes this extension, or *n/a*

### Parameters

Extensions MAY define parameters to enable configuration as needed. Extension parameters are serialized as JSON values, and therefore must conform to the [JSON specification](https://tools.ietf.org/html/rfc8259). Parameters MUST be defined in the following structure:

* **Name**: A short, descriptive name for the parameter. The name is used as the parameter's key within its JSON representation.
   * **Description**: A brief description of the function of the parameter. This should be expanded in the main description of the extension which MUST reference all the parameters.
   * **Type**: The JSON data type of the parameter value. One of `string`, `number`, `boolean`, `array`, or `object`. The structure of complex types MUST be further described.
   * **Constraints**: A description of any constraints to apply to parameter values. For example, "May not be empty."
   * **Default**: The default value of parameter. If no default is specified, then the parameter is mandatory.

## Review / Merge Policy

1. A pull-request is submitted per the guidelines described in the "[Organization of this repository](https://github.com/OCFL/extensions#organization-of-this-repository)" section of this document
1. Authors of (legitimate) pull-requests will be added by an owner of the OCFL GitHub organization to the [extension-authors](https://github.com/orgs/OCFL/teams/extension-authors) team
   - The purpose of being added to this team is to enable adding `labels` to their pull-request(s)
1. If a pull-request is submitted in order to facilitate discussion, the `draft` label should be applied by the author
1. If a pull-request is ready for review, it should have a title that is suitable for merge (i.e. not have a title indicating "draft"), and optionally have the `in-review` label applied by the author
1. A pull-request must be merged by an OCFL Editor if the following criteria are met:
   1. At least two OCFL Editors have "[Approved](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/approving-a-pull-request-with-required-reviews)" the pull-request
   1. At least one other community member has "[Approved](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/approving-a-pull-request-with-required-reviews)" the pull-request
   1. The approvers represent three distinct organizations
