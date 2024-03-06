# OCFL Community Extension Initial

  * **Extension Name:** initial
  * **Authors:** N Jefferies
  * **Minimum OCFL Version:** 1.0
  * **OCFL Community Extensions Version:** 1.0
  * **Obsoletes:** n/a
  * **Obsoleted by:** n/a

*Note: This is a placeholder for the special extension name "initial".*

## Overview

This extension definition ensures that the special extension name _initial_ is valid registered extension name.

In practice, this extension MUST be replaced by a functional extension as detailed below. 

## Parameters

None

## Optional _initial_ Extension 

A _root extension directory_ MAY optionally contain an _initial_ extension that, if it exists, SHOULD be applied before all other extensions in the directory.
An _initial extension_ is identified by the extension directory name "initial".

An _initial extension_ could be used to address some undefined behaviors, define how extensions are applied, and/or answer questions such as:

   - Is an extension deactivated, only applying to earlier versions of the object?
   - Should extensions be applied in a specific order?
   - Does one extension depend on another?
