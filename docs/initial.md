# OCFL Community Extension `initial`: Initial Extension

  * **Extension Name:** `initial`
  * **Authors:** OCFL Editors
  * **Minimum OCFL Version:** 1.0
  * **OCFL Community Extensions Version:** 1.0
  * **Obsoletes:** n/a
  * **Obsoleted by:** n/a

## Overview

This extension allows indication that the semantics of a particular extension takes precedence over all other extensions. It ensures that the special extension name `initial` is a registered extension name and thus that an extension directory `initial` is also valid in both objects and storage roots.

An extension directory MAY contain an `initial` extension identified by the extension directory name `initial`. If it exists, the `initial` extension specifies another extension that MUST be applied before all other extensions in the directory.

An `initial` extension can be used to address otherwise undefined behaviors, such as:

   - Should extensions be applied in a specific order?
   - Is an extension deactivated, only applying to earlier versions of the object?
   - Does one extension depend on another?

## Parameters

The extension configuration file indicates the functional extension to be apply first by specifying that extension's name in the `extension` parameter (not `initial`).

Example `config.json`:

```
{
    "extensionName": "initial",
    "extension": "NNNN-functional-extension-name"
}
```

## Revision History

--------------------------------
| 2024-09-18 | First published |
--------------------------------
