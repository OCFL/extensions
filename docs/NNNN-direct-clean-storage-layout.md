# OCFL Community Extension NNNN: Direct Clean Storage Layout

* **Extension Name:** NNNN-direct-clean-storage-layout
* **Authors:** Jürgen Enge, Basel
* **Minimum OCFL Version:** 1.0
* **OCFL Community Extensions Version:** 1.0
* **Obsoletes:** n/a
* **Obsoleted by:** n/a

## Overview

This object/storage root extension maps content file names or OCFL object identifiers by replacing "dangerous characters" from file and folder names.

The functionality derived from David A. Wheeler - "Fixing Unix/Linux/POSIX Filenames" (https://www.dwheeler.com/essays/fixing-unix-linux-filenames.html)

## Parameters

### Summary

* **Name:** `maxLen`
   * **Description:** determines the maximum number of result characters.
     The default value is `255`, which means that a result with more than 255 characters will raise an error.
    * **Type:** number
   * **Constraints:** An integer greater 0
   * **Default:** 255
  
## Procedure

The following is an outline to the steps for mapping an identifier/filepath:

1. Replace all non-UTF8 characters with "_"
2. Split the string at path separator "/"
3. For each part do the following
   1. Replace any character from this list with "_": 0x00-0x1f 0x7f * ? : [ ] " <> | ( ) { } & ' ! ; # @
   2. Remove leading spaces, "-" and "~" / remove trailing spaces
4. Join the parts with path separator "/"
5. Check length of result according `maxLen`

## Examples

#### Parameters

It is not necessary to specify any parameters to use the default configuration.
However, if you were to do so, it would look like the following:

```json
{
    "extensionName": "NNNN-direct-clean-storage-layout",
    "maxLen": 255
}
```

#### Mappings
| ID or Path                     | Result                 |
|--------------------------------|------------------------|
| `..hor_rib:lé-$id`             | `..hor_rib_lé-$id` |
| `info:fedora/object-01`        | `info_fedora/object-01` |
| `~ info:fedora/-obj#ec@t-"01 ` | `info_fedora/obj_ec_t-_01` |

### Implementation

#### GO 

```go
package cleanpath

import (
	"regexp"
	"strings"
)

var rule_1_5 = regexp.MustCompile("[\x00-\x1F\x7F\n\r\t*?:\\[\\]\"<>|(){}&'!\\;#@]")
var rule_2_4_6 = regexp.MustCompile("^[\\s\\-~]*(.*?)\\s*$")

func CleanPath(fname string) string {

	// Step 1
	fname = strings.ToValidUTF8(fname, "_")

	// Step 2
	names := strings.Split(fname, "/")
	result := []string{}

	// Step 3
	for _, n := range names {
		// Step 3 i
		n = rule_1_5.ReplaceAllString(n, "_")
		// Step 3 ii
		n = rule_2_4_6.ReplaceAllString(n, "$1")
		result = append(result, n)
	}

	// Step 4
	fname = strings.Join(result, "/")

	return fname
}
```
