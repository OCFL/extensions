# OCFL Community Extensions

This repository is intended as a place for community extensions to the [OCFL Specification and Implementation Notes](https://ocfl.io/). Community extensions are intended as a way to share and collaborate outside of the specification process. They are intended to be citable and mostly static once published. Substantial revisions of content beyond simple fixes warrants publishing a new extension, perhaps marking the old one deprecated. Anybody may propose a community extension via a pull-request (PR) against this repository. The community will review and discuss it before it is merged in.

The current set of extensions can be read on [GitHub Pages](https://ocfl.github.io/extensions/).

See also [pending pull requests](https://github.com/OCFL/extensions/pulls) for extensions under discussion.

## Organization of this repository

Community extensions should be written as GitHub flavored markdown in the `docs` directory of this repository. They should be numbered sequentially using a 4-digit, zero-padded prefix; should use hyphens to separate words; and have the `.md` extension.

An example/template is available in this repository as [OCFL Community Extension](docs/0000-example-extension) and is rendered
via GitHub pages as https://ocfl.github.io/extensions/0000-example-extension

## Extension Parameters

For efficiency, it is likely that many extension definitions might actually cover a number of variants. Therefore, when an
extension is referenced, it MAY be accompanied by a number of parameters that specify the particular variant in use. This
provides both more effective documention of an OCFL structure and allows the implementation of generic extension code that
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
  * integer - may be signed or not as specified in the range parameter.
  * string - aligned with JSON strings, these should be UTF-8 encoded and avoid control characters.  
  * enumerated - one of an ordered set of labels which MUST conform to the same limitations as parameter names.  
  * boolean - may have the values false or true (lower case and unquoted as in JSON)
  * For integer parameters the range specifies minimum and maximum values, separated by a comma, which MUST be integers themselves.
  * For string parameters, the range specifies the maximum length of the string as an integer number of characters, not bytes. Again, based on a survey of parsers, try to keep strings shorter than 4095 characters.
  * For enumerated parameters, the range is a comma separated ordered list of labels that are valid for the parameter. Enumerated parameters are case sensitive and MUST always be quoted, so they are JSON strings. 
* Default: Default value for parameter, which MUST be consistent with the range limitations. If this is left blank then the parameter is mandatory 

## Referencing Parameters

Wherever a parameterised extension is referenced, include any parameters in an accompanying JSON file. If using an extensions directory, the JSON file MUST be named for the extension and included in the directory. For example, the example extension above would have an accompanying file *0000-example-extension.json* which might contain:

    "0000-example-extension.md": {  
        "firstExampleParameter": 12,  
        "secondExampleParameter": "Hello",  
        "thirdExampleParameter": "Green"  
    }
There MUST only be one parameter set defined for each extension.
