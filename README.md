# OCFL Community Extensions

This repository contains community extensions to the [OCFL Specification and Implementation Notes](https://ocfl.io/). Extensions are a means of adding new functionality and documenting standards outside of the main OCFL specification process. For example, storage layout extensions define how OCFL object IDs are mapped to OCFL object root directories within an OCFL storage root. This mapping is outside of the scope of the OCFL specification, but is valuable information to capture so that repositories are self-describing and easily accessible using generic OCFL tooling.

This is a community driven repository. Community members are encouraged to contribute by submitting new extensions and reviewing others' submissions. For more details, see the [review/merge policy](#review--merge-policy) below.

See the current set of [adopted extensions](https://ocfl.github.io/extensions/) and [extensions open for review and discussion](https://github.com/OCFL/extensions/pulls).

## Using Community Extensions

To use OCFL extensions you first need an OCFL client that supports the desired extensions. OCFL clients are not required to support extensions to be compliant with the OCFL specification, and the extensions that any given client supports will vary. The idea behind this repository is to encourage the development and implementation of common extensions so that there can be interoperability between OCFL clients.

## Organization of this repository

Community extensions should be written as GitHub flavored markdown files in the `docs` directory of this repository. The
filename of an extension is based on its *Registered Name* with a `.md` extension. 

Extensions are numbered sequentially, and the *Registered Name* of an extension is prefixed with this 4-digit, zero-padded
decimal number. The *Registered Name* should be descriptive, use hyphens to separate words and have a maximum of 250 
characters in total. New, or substantially revised, extensions MUST use the next available number based on extensions current
at the time of merging. The *Registered Name* MUST be listed in the header of the extension file. 

An example/template is available in this repository as [OCFL Community Extension](docs/0000-example-extension.md) and is rendered
via GitHub pages as https://ocfl.github.io/extensions/0000-example-extension

## Extension Parameters

For efficiency, it is likely that many extension definitions might actually cover a number of variants. Therefore, when an
extension is referenced, it MAY be accompanied by a number of parameters that specify the particular variant in use. This
provides both more effective documentation of an OCFL structure and allows the implementation of generic extension code that
covers a wider variety of use cases. Parameters MUST be single valued. For each parameter the following properties should
be defined:    

* Name: A short name for the parameter. Since this has the potential to be used as part of programmatic access the name MUST
comply with the Javascript restrictions on identifiers (The first character must be a letter, an underscore, or a dollar sign. 
Subsequent characters may be letters, digits, underscores, or dollar signs.) and SHOULD be shorter than 127 characters. The
length limit is based on a survey of the defaults for various JSON parsers. 
* Description: A brief description of the function of the parameter. This should be expanded in the main description of the
extension which MUST reference all the parameters.
* Type: Data type for the parameter. In order to allow validation and limit the scope for implementation specific variations,
parameters are typed.
  * integer - may be signed or not as specified in the range property.
  * string - aligned with JSON strings, these should be UTF-8 encoded and avoid control characters.  
  * enumerated - one of an ordered set of labels which MUST conform to the same limitations as parameter names.  
  * boolean - MUST have the value *false* or *true* (lower case and unquoted as in JSON)
* Range: Further qualifies the valid values for a parameter depending on its type.
  * For integer parameters, the range specifies minimum and maximum values, separated by a comma, which MUST be integers themselves.
  * For string parameters, the range specifies the maximum length of the string as an integer number of characters, not bytes. Again, based on a survey of parsers, strings SHOULD be shorter than 4095 characters.
  * For enumerated parameters, the range is a comma separated ordered list of labels that are valid for the parameter. Enumerated parameters are case sensitive and MUST always be quoted, so they are JSON strings. 
* Default: Default value for parameter, which MUST be consistent with the range limitations. If this is left blank then the parameter is mandatory (if parameters are defined)  

## Referencing Parameters

Wherever a parameterised extension is referenced, any parameters MAY be included in an accompanying sidecar JSON file that
uses the registered name with the filename extension `.json`. If using an 'extensions' directory, the JSON file MUST be 
included in the root directory of the extension and not a subdirectory. Another place that an extension may be 
referenced is `ocfl_layout.json` where the sidecar file should be alongside it in the Storage Root. 

For example, the example extension above would have an accompanying file *0000-example-extension.json* which might contain:

    "0000-example-extension": {  
        "firstExampleParameter": 12,  
        "secondExampleParameter": "Hello",  
        "thirdExampleParameter": "Green"  
    }
    
An extension MAY be referenced without providing any parameters, even if they are defined for that extension. However, if parameters are defined they MUST be complete, with valid values specified for all parameters that do not have a default. There MUST only be one parameter set defined for each extension reference.

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
