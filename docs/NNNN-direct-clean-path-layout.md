# OCFL Community Extension NNNN: Direct Clean Path Layout

* **Extension Name:** NNNN-direct-clean-path-layout
* **Authors:** Jürgen Enge, Basel
* **Minimum OCFL Version:** 1.0
* **OCFL Community Extensions Version:** 1.0
* **Obsoletes:** n/a
* **Obsoleted by:** n/a

## Overview

This extension can be used as root storage layout extension to map OCFL object
identifiers to path names as well as a mapping for content path names.
This is done by replacing or removing "dangerous characters" from names.

The functionality has been derived from David A. Wheeler - "Fixing Unix/Linux/POSIX Filenames" (https://www.dwheeler.com/essays/fixing-unix-linux-filenames.html)

#### Usage Scenario

If you want to make sure, that the internal layout of the OCFL content 
substructure or the object id is as close as possible to the original structure, but you want to 
make sure, that there are no possibly dangerous file- or foldernames, which 
may oppose with durability.

One Example could be complete filesystems, which come from estates, and are not
under control of the archivists.

### Caveat

#### Projection
This extension does not provide injective (one-to-one) projections of names. 
Therefor check first your source to make sure, that there's no chance of 
different names to be mapped on one target. Software which generates the OCFL 
structure should raise an error in this case.

#### Object Identifiers
Besides the caveat below, when using as storage layout root extension, you must 
be sure, that no object identifier is  contained in another identifier as a 
prefix path.  
I.e. https://hdl.handle.net/XXXXX/test and https://hdl.handle.net/XXXXX/test/blah 
would result to into storage root folders like  
https/hdl.handle.net/XXXXX/test    
https/hdl.handle.net/XXXXX/test/blah  
This is an invalid OCFL storage root hierarchy.

#### Length
Several filesystems (e.g. Ext2/3/4) have byte restrictions on filename length.
When using UTF16 (i.e. NTFS) or UTF32 characters in the filesystem, the byte length is double or quad of the character length.
Using UTF8 characters in filesystems, byte length can only be determined by checking each string for maximum.  
**Hint:** UTF8 has always less or equal bytes than UTF32 which means, that 
assuming UTF32 instead of UTF8 for length calculation is safe, but would give you
only 63 characters on 255 byte restrictions. 

## Parameters

### Summary

* **Name:** `maxFilenameLen`
   * **Description:** determines the maximum number of characters within parts of the full pathname separated by `/` (files or folders).   
     A result with more characters will raise an error.  
    * **Type:** number
   * **Constraints:** An integer greater 0
   * **Default:** 127
* **Name:** `maxPathnameLen`
    * **Description:** determines the maximum number of result characters.
      A result with more characters will raise an error.  
    * **Type:** number
    * **Constraints:** An integer greater 0
    * **Default:** 32000
* **Name:** `replacementString`
  * **Description: String which is used to replace non-whitespace characters which need replacement
  * **Type:** string
  * **Default:** "_"
* **Name:** `whitespaceReplacementString`
    * **Description:** String which is used to replace whitespaces (https://en.wikipedia.org/wiki/Template:Whitespace_(Unicode))  
      **Hint:** if you want to remove (inner) whitespaces, just use the empty string
    * **Type:** string
    * **Default:** " " (U+0020)
  
## Procedure

The following is an outline to the steps for mapping an identifier/filepath:

1. Replace all non-UTF8 characters with `replacementCharacter`
2. Split the string at path separator "/"
3. For each part do the following
   1. Replace any whitespace character from this list with `whitespaceReplacementCharacter`: U+0009, U+000a-U+000d, U+0020 U+0085 U+00a0 U+1680 U+2000-U+20a0 U+2028 U+2029 U+202f U+205f U+3000   
   2. Replace any character from this list with `replacementCharacter`: 0x00-0x1f 0x7f * ? : [ ] " <> | ( ) { } & ' ! ; # @ 
   3. Remove leading spaces, "-" and "~" / remove trailing spaces
   4. Replace any period ("."), if part contains only periods 
   4. Remove part completely, if its len is 0
   5. Check length of part according to `maxFilenameLen`
4. Join the parts with path separator "/"
5. Check length of result according `maxPathnameLen`

## Examples

#### Parameters

It is not necessary to specify any parameters to use the default configuration.
However, if you were to do so, it would look like the following:

```json
{
    "extensionName": "NNNN-direct-clean-path-layout",
    "maxFilenameLen": 127,
    "maxFilenameLen": 32000,
    "replacementString": "_",
    "whitespaceReplacementString": " "
}
```

#### Mappings

With Parameters above.

| ID or Path                                | Result                                  |
|-------------------------------------------|-----------------------------------------|
| `..hor_rib:lé-$id`                        | `..hor_rib_lé-$id`                      |
| `info:fedora/object-01`                   | `info_fedora/object-01`                 |
| `~ info:fedora/-obj#ec@t-"01 `            | `info_fedora/obj_ec_t-_01`              |
| `/test/ ~/.../blah`                       | `test/___/blah`                         |
 | `https://hdl.handle.net/XXXXX/test/bl ah` | `https_/hdl.handle.net/XXXXX/test/bl ah` |
### Implementation

#### GO 

```go
package cleanpath

import (
	"emperror.dev/errors"
	"regexp"
	"strings"
	[...]
)

var flatDirectCleanRuleWhitespace = regexp.MustCompile("[\u0009\u000a-\u000d\u0020\u0085\u00a0\u1680\u2000-\u20a0\u2028\u2029\u202f\u205f\u3000]")
var flatDirectCleanRule_1_5 = regexp.MustCompile("[\u0000-\u001F\u007F\n\r\t*?:\\[\\]\"<>|(){}&'!\\;#@]")
var flatDirectCleanRule_2_4_6 = regexp.MustCompile("^[\\-~\u0009\u000a-\u000d\u0020\u0085\u00a0\u1680\u2000-\u20a0\u2028\u2029\u202f\u205f\u3000]*(.*?)[\u0009\u000a-\u000d\u0020\u0085\u00a0\u1680\u2000-\u20a0\u2028\u2029\u202f\u205f\u3000]*$")
var flatDirectCleanRulePeriods = regexp.MustCompile("^\\.+$")

var flatDirectCleanErrFilenameTooLong = errors.New("filename too long")
var flatDirectCleanErrPathnameTooLong = errors.New("pathname too long")

[...]

func (sl *DirectClean) ExecutePath(fname string) (string, error) {

	fname = strings.ToValidUTF8(fname, sl.ReplacementString)

	names := strings.Split(fname, "/")
	result := []string{}

	for _, n := range names {
		n = flatDirectCleanRuleWhitespace.ReplaceAllString(n, sl.WhitespaceReplacementString)
		n = flatDirectCleanRule_1_5.ReplaceAllString(n, sl.ReplacementString)
		n = flatDirectCleanRule_2_4_6.ReplaceAllString(n, "$1")
		if flatDirectCleanRulePeriods.MatchString(n) {
			strings.Replace(n, ".", sl.ReplacementString, -1)
		}
		n = flatDirectCleanRulePeriods.ReplaceAllString(n, sl.ReplacementString)

		lenN := len(n)
		if lenN > sl.MaxFilenameLen {
			return "", errors.Wrapf(flatDirectCleanErrFilenameTooLong, "filename: %s", n)
		}

		if lenN > 0 {
			result = append(result, n)
		}
	}

	fname = strings.Join(result, "/")

	if len(fname) > sl.MaxPathnameLen {
		return "", errors.Wrapf(flatDirectCleanErrPathnameTooLong, "pathname: %s", fname)
	}

	return fname, nil
}

[...]
```
