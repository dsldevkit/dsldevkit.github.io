---
title: Keywords
keywords: keywords, identifiers
sidebar: mydoc_sidebar
permalink: keywords.html
folder: grammar
---

## Keywords and reserved words

A [reserved word](https://en.wikipedia.org/wiki/Reserved_word) is a word that cannot be used as an identifier.

A [keyword in Xtext grammar](https://www.eclipse.org/Xtext/documentation/301_grammarlanguage.html#keywords) is a kind of terminal rule literal.

A keyword in a programming language is a token with a special meaning in the language.

## Using keywords as identifiers

There are generally two options to use keywords as identifiers in Antlr:
- use identifier rules by listing keywords allowed as identifiers along with the ID terminal rule
- use ID rule instead of keyword guarded by semantic predicate

First option is possible with default Xtext, whether second option requires an adjustment to the generator of Antlr grammar from Xtext grammar which DDK provides. So with DDK both options are possible to use.

### Identifier rules pattern

Define a rule like the following

```
Identifier : 
    'exception'
  | 'extends'
  | ID
;
```

Then use this rule everywhere instead of ID terminal rule

```
FunctionDeclaration :
  'function' name=Identifier
    (body=MethodBody)?
  ';'
;

```

### Keyword rules using semantic predicates

Antlr suggests the following pattern to use [keywords as identifiers](https://theantlrguy.atlassian.net/wiki/spaces/ANTLR3/pages/2687320/How+can+I+allow+keywords+as+identifiers).

However as Xtext hids semantic predicates from the user one cannot write in Xtext.
```
KeyFunction : {input.LT(1).getText().equals("function")}? ID ;
```

DDK adjusts the generators and scans comments for each Xtext rule, so you can write 

```
/**
 * @KeywordRule(mod)
 */
KeyMod returns ecore::EString :
  ID
;
```

Then whenever the KeyMod is used a semantic predicate (as suggested by Antlr) will be insertred. Semantic predicates will be propagated before actions inserted by Xtext generator.

