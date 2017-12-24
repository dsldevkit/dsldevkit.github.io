---
title: Format Language
keywords: Format
sidebar: mydoc_sidebar
permalink: format.html
folder: format
---

## Introduction
The Format DSL is intended to be used when creating a format configuration.
It is basically a frontend to Xtext's declarative formatter framework.

For a general introduction on Xtext formatting, see
the [Xtext manual](https://eclipse.org/Xtext/documentation/) section on formatting.

## Declaring a formatting configuration

The language specific formatting is declared using the Format DSL. The format configuration
source file is transformed into Java classes by the appropriate generator.

The generated artefacts are `MyDslFormatter` and `AbstractMyDslFormatter`. The former is only
generated once, i.e. never overwritten, and the latter is always regenerated.

In `MyDslFormatter` one can declare parts of the format configuration which cannot be expressed
in Format DSL (using instructions provided by Xtext formatting framework).

1. Format runtime

   The declared formatting configuration is applied between semantic tokens, i.e. a formatter
   inserts/removes/modifies white space between tokens. The Format DSL runtime extends the
   limited locator vocabulary of Xtext and and has an improved method to combine multiple
   locators for a grammar element. The Format DSL runtime component `DdkFormattingConfigBasedStream`
   combines locators by category, so a no_space locator will not affect a linewrap locator.

2. DDK / Variable space locators

   Xtext offers only one space locator - space. It takes a fixed string as an argument, which
   is added to a grammar element. More elaborate formatting, such as padding a grammar element
   to a minimum length is not possible.

   In DDK additional locators, such as `column` or `right_padding` are available.

3. Combining locators

   With the Xtext formatter space locators always take precedence over linewrap locators,
   which makes it very difficult to define line wrapping for optional elements.

   Consider the Xtext rule Block: `"begin" (ID)+ "end"`. We want to declare formatting
   such that there is a linewrap before end and two spaces after each `ID` - with `FormattingConfigBasedStream`
   there is no way to cause end to be on a new line.
