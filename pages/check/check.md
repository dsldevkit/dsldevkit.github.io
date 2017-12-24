---
title: Check Language
keywords: Check
sidebar: mydoc_sidebar
permalink: check.html
folder: check
---

The Check DSL is used to specify model constraints.

## Concepts

Editing a check catalog requires that main concepts are understood. A high-level overview of them is explained in the following.

### Catalog

A Catalog is the main concept concerned with checks. Exactly one catalog is contained in
one check file. The catalog name should be identical to the file name excluding file extension.

### Category

Categories are contained by catalogs and are used for grouping of checks.
Any number of categories can be defined and any number of checks can be contained.

### Check

A Check defines a validation rule for the target model.
It is associated with a mode of execution (live, on save, or on demand),
a severity (error or warning), a message (which will appear in the user interface when violated)
and is configured for a number of contexts.

### Context

The Context is a technical concept which defines the entry point for the check it is defined for.
Typically, the context will correspond to the model element that the check intends to validate.
Conditions which should determine when a check is violated are defined by a constraint.

### Constraint

Constraints embed the logic deciding whether a given model element violate a check definition or not.
They are implemented using the [Xbase expression language](https://wiki.eclipse.org/Xbase) and are in the end responsible for creating markers.

## Defining Checks

A check definition consists of

- Meta-data (check’s severity, diagnostic message, descriptive properties, ...)
- A constraint

Whereas the meta-data is self-descriptive, the check constraint needs further attention.

Example of a constraint validating maximal name length:

``` java
for VariableDeclaration decl {
    if (decl.name.length > 30) {
         issue
    }
}
```

Constraints are always bound to a specific model element (e.g. VariableDeclaration), the keyword `for` is used for that binding.
The constraint itself is then defined using Xbase, a powerful expression language including linking to model
elements of the given language.

Furthermore Xbase allows JvmTypes-based Java methods to be inlined.
Please refer to the [Xbase language guide](https://wiki.eclipse.org/Xbase) for more information.

The Check language contributes two special expressions to those available from Xbase.

### Issue Expressions

In order to actually report a validation problem to the end user when a condition has been violated
(i.e. create an issue marker in the user interface), an explicit statement is required in the check implementation.
This is what the issue statement is used for. Once defined, it will trigger creation of an issue marker when called.

Note that the issue expression acts as a “return” statement. When invoked, the issue expression terminates the control flow
leaving the check being executed.
