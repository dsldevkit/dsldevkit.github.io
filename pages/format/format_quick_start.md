---
title: Format Quick-start Guide
keywords: Format
sidebar: mydoc_sidebar
permalink: format_quick_start.html
folder: format
---

## Formatter definition

Create a `MyDsl.format` file in the core plugin of the language in the
`<dsl-name>.formatting` package - e.g. `org.xtext.example.mydsl.formatting`.

## Base class

The default base class for `AbstractXXXFormatter` is `AbstractDdkFormatter` -
if any other has to be specified then the extends clause has to be defined in the format
configuration file (.format), just after the grammar and optional extended formatter names,
together with this custom base formatter class qualified name - see below - 2nd line:

```
    formatter for org.xtext.example.mydsl.MyDsl with org.eclipse.xtext.common.Terminals
    extends com.avaloq.tools.foundation.xtext.core.formatting.AbstractCustomFormatter

    //constants, rules, etc. 2

```

## Formatter custom code

Ensure that `MyDslFormatter` extends `AbstractMyDslFormatter` generated out of Format DSL.

## Project configuration

Ensure that your language core plugin has a Xtext nature

## Example

`MyDsl` Xtext grammar on which a basic and a complex Format examples are based.

```xtext
grammar org.xtext.example.mydsl.MyDsl with org.eclipse.xtext.common.Terminals

generate myDsl "http://www.xtext.org/example/mydsl/MyDsl"

MyDsl :
  "begin"
  (elements+=Element)*
  "end";

Element :
  "element" name=ID
  (
    ("min" "=" min=SignedInt)
    | ("max" "=" max=SignedInt)
  );

SignedInt :
  ("+" | "-")? INT;
```

### Basic example

```
formatter for org.xtext.example.mydsl.MyDsl

MyDsl {
  "begin" : increment after, linewrap after;
  "end" : decrement before, linewrap before;
}
```

### Complex example

```
formatter for org.xtext.example.mydsl.MyDsl

const int FOO = 2;

MyDsl {
  "begin" : increment after, linewrap FOO after;
  "end" : decrement before, linewrap before;
}

Element {
  rule : linewrap before; // (1)
  group 1 => group 1 { // (2)
    "=" : space " " around; // (3)
    "min" : linewrap before, column 10 before;
  }
  group 1 => group 2 : linewrap before; // (4)
  "="(2,2) : space " " around; // (5)
  =max, =min : offset 20 before; // (6)
  SignedInt, "max" : no_linewrap between; // (7)
}

* {
  ["=", "."] : no_space before; // (8)
  ("begin" "end") : left.linewrap after, right.linewrap before; // (9)
}
```

An explanation to the Element declarations in the above example:

1. `rule`

   specifies the grammar rule Element, i.e. line-wrap before each Element

2. `group 1 => group 1 ...`

   specifies that the context is the min alternative

3. `"="`

   this is used in the nested group context and refers to the 1st equals sign i.e.
   the `"="` following keyword `"min"`

4. `group 1 => group 2 : ...`

   specifies the compound element containing the max assignment

5. `"="(2,2)`

   this references equals sign 2 of 2 in Element, i.e. the `"="` following keyword `"max"`

6. `=max, =min`

   formatting applies to min and max assignments

7. `SignedInt, "max"`

   suppress line wrapping bewteen rule SignedInt and keyword "max"

8. `["=", "."]`

   All `"="` and `"."` keywords in the Grammar. In contrast to declaration in 3. this
   does not refer to a specific keyword, but all keywords with values `"="` or `"."`.

9. `("begin" "end")`

   Find all keyword pairs begin and end in the Grammar and configure a linewrap after
   begin and a linewrap before end.
