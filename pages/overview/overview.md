---
title: DSL Developer Kit Introduction
keywords: DDK concepts
sidebar: mydoc_sidebar
permalink: overview.html
folder: overview
---

## DSL Implementations with DSLs

A Domain Specific Language starts with the domain model. For a language implemented with [Xtext] the model is typically defined either using [Ecore] directly or with [Xcore] DSL. For simple DSLs one can also let Xtext derive the Ecore model from the grammar.

The concrete syntax for the DSL is then defined with [Xtext Grammar Language]. It is then used to parse text into models and serialize models back into text.

Xtext and Xcore are the DSLs that define two main aspects of a DSL implementation. In DSL Developer Kit we introduced four further DSLs that cover further aspects of a DSL implementation. Each language comes with the required runtime libraries extending capabilities of Xtext runtime.

{% include image.html file="overview/ddk_intro_0.png" alt="4 DSLs" caption="Figure 1. The four languages in DSL Developer Kit are called Check, Format, Export and Scope." %}

### Check DSL

Check DSL is based on [Xbase] and allows defining model constraints using Xbase expressions. It generates an EMF model validator with all necessary registration hooks and preferences.

Below is an example of a simple check for ```ProcedureDeclaration``` EClass which has an attribute ```name``` of type EString. The check is configurable with ```maxLength``` parameter which has a default value ```10```. If the name of the procedure declaration exceeds ```maxLength``` the check issues a diagnostic message with error severity. Live execution time means that within IDE this check runs as you type.

{% include image.html file="overview/ddk_intro_1.png" alt="Check DSL example" caption="Figure 2. Example of a name length constraint in a Check DSL." %}

### Format DSL

Pretty printing of a DSL is important for model serializer and hence one of the core features of any DSL implementation. It is also used by IDE to perform auto format. In Format DSL one can define formatting rules for grammar rules in [Xtext Grammar Language]. It is also an [Xbase]-based language and Xbase expressions are used to compute conditions and dynamic values for conditional formatting baed on abstract syntax tree.

Below is an example of a formatting rule for ```NamedArgument``` grammar rule. The rule is conditional on the number of parameters. A call with just one parameter will be kept on a single line. When there are two or more parameters, each will be on a new line. Also the values will be aligned in a column and the column position is computed based on the longest parameter name.

Expression and the end of the rule is a Boolean guard. The rule is active only if it evaluated to true in the given context.

{% include image.html file="overview/ddk_intro_2.png" alt="Format DSL example" caption="Figure 3. Conditional formatting of a procedure call with named parameters in a PL/SQL-linke language." %}

### Export DSL

[Symbol Tables] in Xtext are implemented as an [Xtext Index]. What gets stored in an index for a specific DSL is defined by a resource description strategy. Export DSL defines for what EClass-es object descriptions are computed, their structure as well as strategies for computing object fingerprints and URI fragments.

Below is an example for a simple scripting language. Import section lists EPackages used in the export file. Interface section defines fingerprint computation for EObjects. The following two export statements declare how EObjectDescriptions for EClasses ```ScriptPackage``` and ```MethodDeclaration``` are constructed. Along these declarations one can also influence how fragment provider computes a segment for this object in URI fragments.

{% include image.html file="overview/ddk_intro_3.png" alt="Export DSL" caption="Figure 4. Example in Export DSL defining object descriptors for script package and a method of a scripting language." %}

With ```object-fingerprint``` entry we instruct resource description strategy to include object fingerprints into object descriptions. This enables fine-grained dependency analysis.

When the user makes a small change to a declaration in a source 1 - even if this change shouldn't invalidate any other sources (if this declaration isn't referenced anywhere) - the Xtext builder will revalidate (rebuild) all other sources which have a dependency on any declaration in the changed source 1. That is the standard behaviour of the Xtext builder.

More often than not many of these other sources actually don't need rebuilding, as they don't depend on the declaration which was originally changed in source 1. So the Xtext builder ends up spending a lot of time revalidating a still valid model.

DSL Developer Kit solves this by offering a fine-grained dependency analysis based on object fingerprints. The Xtext builder then only rebuild sources which actually depend on the changed declarations.

The following diagram illustrates the principal idea of fine-grained dependencies. Instead of having a single resource fingerprint accounting for all the exported objects of source 1 (as to the left), every exported object has its own object fingerprint. As a result the builder doesn't need to rebuild source 2 as it only depends on declaration C of 1, while it was declaration B which was changed.

{% include image.html file="overview/ddk_intro_4.png" alt="Fine-grained dependency analysis" caption="Figure 5. Illustration of fine-grained dependency analysis vs. resource level invalidation." %}

Fine-grained invalidation requires stable URI fragments. Export DSL allows defining
"semi-positional“ syntax with selectors for not unique names. For example ```0/1/3(0=='foo').0``` and ```0/1/3(0=='foo').1``` for objects ```foo```. If the name is unique fragment gets additional ```!```  and omit last part: e.g. ```0/1/3(0=='foo'!)```. Fine-grained dependency comes with a price: every visible declaration must have an exported object.

In the URI fragment example the first ```0``` is the index of the top-level object within the EMF resource. The following ```1``` is the feature ID of the single valued containment feature rather than a position index. The leading ```/``` indicates that what follows is the ID of a containment feature. In the same fashion the ```3``` is the ID of the multi-valued containment feature. What follows in brackets is the selector (similar to XPath selectors): The leading ```0``` in the selector is the feature ID of the selector feature ```Declaration#name```. After the equality operator follows the object's value for the selector feature. The last digit (the selector index) is the index of the object within the filtered (by the selector) containment feature ```ScriptPackage#declarations```. These fragments are thus very similar to the short fragments used internally by Xtext; except for the new selector, of course.

### Scope DSL

Scope language helps to describe the logic for exactly how a proxy for a cross-reference resolves to the correct target. Scope computation is a prerequisite for linking: the linker simply asks for all the elements in scope, then it chooses the first element (in order of precedence as defined by the scope chain) with a matching name.

Following examples illustrate some of the constructs of scope language. Naming section defines default naming functions for EClasses. These naming functions are applied by linker if no other naming function is specified explicitly in a scoping rule.

Scoping rule ```ObjType#type``` is used to resolve ```type``` feature of ```ObjType``` EClass.

{% include image.html file="overview/ddk_intro_5.png" alt="Scope file example part 1" caption="Figure 6. Declaring naming functions and a simple global scope lookup scoping rule." %}

Rule ```find(Row, key = "code_obj_type.*")``` means a global name lookup of an EClass ```Row``` with qualified names starting with ```code_obj_type.```. Context ```*``` means that the result of the scope is invariant on a resource level. Framework will make sure to provider proper caching on this level.

Next example illustrates chaining of scope rules and delegation to a named scope rule.

{% include image.html file="overview/ddk_intro_6.png" alt="Scope file example part 2" caption="Figure 7. Example of a scope declaration with one context and a chain of several scopes." %}

## Unit tests with Xtend

We find [Xtend] very useful for writing unit tests for DSLs. Multi-line strings are nice for copy-pasting code snippets directly within JUnit test method bodies. In DSL Developer Kit we've implemented a number of abstract classes helping to write linking, validation, content assist and other tests.

The following is an example of a linking test. In this test we link from label translation to a label definition. Label definition is in a different source from label translation. Label definition is added as a required source for the test. We tag the EObject with label definition by calling ```mark``` helper method from within the example template. Test implementation will simply take the EObject at the offset of the tag. Note that you don't need to initialise the tag constant - active annotation of Xtend does it. Then in the label implementation right before the cross reference we want to test we place a reference to the tag. Test will then verify that a cross-reference to the right from the ```ref``` call actually resolve against the expected object.

{% include image.html file="overview/ddk_intro_7.png" alt="Example of linking test" caption="Figure 8. Example of a linking test with Xtend." %}

[Xtext]: https://www.eclipse.org/Xtext/
[Xtext Grammar Language]: https://www.eclipse.org/Xtext/documentation/301_grammarlanguage.html
[Ecore]: https://www.eclipse.org/modeling/emf/
[Xcore]: https://wiki.eclipse.org/Xcore
[Xbase]: https://wiki.eclipse.org/Xbase
[Symbol Tables]: https://en.wikipedia.org/wiki/Symbol_table
[Xtext Index]: https://www.eclipse.org/Xtext/documentation/303_runtime_concepts.html#global-scopes
[Xtend]: https://www.eclipse.org/xtend/
