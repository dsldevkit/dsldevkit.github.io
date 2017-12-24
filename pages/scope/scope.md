---
title: Scope Language
keywords: Scope
sidebar: mydoc_sidebar
permalink: scope.html
---

## Introduction

Scoping and linking are topics that must be considered in most problem domains. Irrespective of whether the models are stored textually (using a DSL) or in some other (binary) format. They basically cover the questions:

- Which objects (of a given type) are visible in a certain context? I.e. What objects are in scope?

- Which one of these objects in scope does a specific cross reference point to? I.e. Which object should a cross reference be linked to?

Scoping defines not only inter-resource linking, but more generally which objects are considered visible for any particular reference in a resource. Scoping may define intra-resource visibility (links that point to other objects from the same resource) as well as inter-resource visibility (links that point to objects in other resources). Different instances of one and the same reference may link to objects in different resources.

The DSL presented on this page can be used to concisely specify the linking semantics for a given Xtext grammar. For more information regarding questions when, why, and how to apply the scoping language and its constructs please refer to the [Cross-Referencing User Guide](scope_guide.html).

## Scoping Concepts

For a general introduction on Xtext scoping, refer to the Xtext reference manual sections on scoping and linking.

### Scope Providers

Xtext has the concept of scope providers, which are called to determine which objects are visible for any particular cross-reference in a model. The scope provider thus calculates the scope. Once the scope provider has calculated the scope for the cross reference, the linking engine then asks that scope whether it contains an object accessible under the name used for the cross reference. If so, the cross reference is linked to that object; otherwise, the link is unresolved and an error diagnostic ("Could not resolve reference ...") is generated.

### Naming

Scopes therefore contain a mapping of names to objects. They can be thought of as symbol tables. The name under which an object is made available in a scope is not necessarily the same as the name under which the object was declared (if it has such a "declared name" at all). Moreover the same object may also be available under several different names in a scope, depending on the semantics of the source language (i.e., of the language containing the cross reference).

### Nesting and chaining

As shown in the Xtext reference manual, scopes may also be nested (aka chained) and the scope provider may -- depending on context -- return a scope chain. If a particular name cannot be found in the innermost scope, the next scope in the scope chain is asked for that name, and so on, until an object is found, or the scope chain is exhausted. This chaining is completely transparent to the client (e.g. the linker). An object in an inner scope will typically shadow objects in outer scopes (chained scopes) with the same name.

### Local and global scopes

Standard Xtext distinguishes between local and global scopes and scope providers; a concept that matches well the semantics of most programming languages where a cross reference can refer to an object declared in the same resource (i.e. provided by the local scope provider) or an object declared in another resource (provided by the global scope provider). Clients (e.g. the linker) would interact with the local scope provider which would return a scope chain where the scope computed by the global scope would be chained as an outer scope to its own calculated scope. And because of the transparency of scope chaining (see previous bullet) the client would automatically find local elements first, when available. The global scope provider would typically use the Xtext index to compute the outer global scope.

The Scope DSL (and its runtime library) does not have this distinction, because it can be too restrictive. Some of the languages implemented DSL Developer Kit can have scoping semantics where an identifier must be linked against an object from the global scope (available in the index) even if an object with the same name is available in the local scope. The Scope DSL therefore does not impose any restrictions on the order of scope chains with local elements and global elements. Having an inner scope that queries the global index, followed by an outer scope that actually is a local scope is perfectly fine, although rather unusual.

### Context object

The scope language provides a declarative approach to define the scope to use for a particular cross reference. This declarative approach is based on the concept of a context object which is also part of the Xtext IScopeProvider API: Whenever a client (e.g. the linking service or the content assist service) request the scope for a particular cross reference they must provide an original context object. Typically this should be the source object for the cross reference in question. Based on this context object the scope provider is able to determine what elements should be added to the scope it returns. E.g. when referencing a variable in a Java expression then the scope should only contain the variables and parameters declared within the same function or in an outer function or class. To apply these rules correctly the scope provider must be provided with the expression owning the cross reference as the original context object.

In the scope definition it is however often cumbersome to define the scope's content based on the original context object since the elements to add to the scope are not directly referenced. Quite often when referencing local objects (see previous section) these local objects will however be directly referenced by some parent object of the original context object. As in the example of referencing variables from a Java expression it is easier to compute the scope based on the function containing the expression rather than the expression itself. This is because the procedure directly references its formal parameters and its declared local variables.

As we will see later the scope language's declarative approach lets us do just that. For the scope of any given cross reference we can specify which object in the original context object's containment hierarchy we want to use as the context object. The approach is the same as described for Xtext's declarative scoping.

### Declarative scoping

Declarative scoping distinguishes between scopes for specific references, and general scopes for any reference referring to a particular type. Reference-specific scopes take precedence over type-specific scopes and should be used as the "entry point" for a particular scope. Type-specific scopes are used only if no reference-specific scope could be found for a particular reference.

A particular reference is globally identified by its containing type (the type in which it is declared) and its feature name. When looking for a scope, the runtime engine always starts with the current object containing the reference as the current *context object*. If it doesn't find a reference-specific scope for the reference and the type of the context object (the *context type*), it looks for a reference-specific scope matching any of the context type's super types. If none is found, it recursively repeats this process by iterating through the context object's containment hierarchy (i.e. by using the context object's eContainer as the next context object). If still no matching declared scope is found, the whole process is repeated again, but this time for type-specific scopes, starting with the type of the reference and continuing with its super types. Finally, if still no scope is found, an empty scope is returned. This may sound rather complicated, but after seeing some examples and trying it out you will probably find that it matches your mental model nicely.

You can define both reference-specific scopes and type-specific scopes. Any scope can be defined for a particular context type, and it may define a whole scope chain. For each element in that chain, you may also define under which names objects should be published. Additionally, you can define default namings for types of objects (to be used if no naming is given for a particular scope in a scope chain). If no naming is specified at all for a type or a scope, the default naming just uses the value of a string attribute named "name", if present in an object.

### Scope delegation

As we've learned a scope is typically easiest to define if the context object either owns or directly references the elements to be included in the scope. But sometimes this object is not in the containment hierarchy of the original context object and the declarative approach described above falls short. For this purpose the scoping DSL allows to declare delegate scopes which simply put are scopes calculated for another arbitrary context object (or set of context objects).

## Language guide

The structure of a scoping definition for a given DSL is the following:

```
<header>

<optional default naming>

<optional scope definitions>
```

For instance:

```
scoping com.avaloq.tools.dsl.mydsl.MyDsl

import "http://www.avaloq.com/tools/dsl/mydsl"


extension com::avaloq::tools::dsl::mydsl::MyDslScoping


case insensitive naming {

  Entity = name;

}

scope EntityReference#entity {

  context * = find(mydsl::Entity) as export;

}
```

### Header

In the header section the scope declarations are provided with a name which should coincide with that of the Xtext grammar it pertains to. Further it declares includes, imports of meta models, and Xtend extension files used in the Xtend expressions of names and scope rules.

### Includes

A scope file can include the definitions contained by other scope files. This can be used to factor out for instance common naming rules to a separate file, or to re-use the scoping defined for a base grammar in the case of scoping for a derived grammar. This is done by using the with keyword as we know it from Xtext.

```
scoping com.avaloq.tools.dsl.mydsl.MyDsl with com.avaloq.tools.dsl.common.MetaModel
```

Included scope files must be visible on the classpath.

The semantics of including other scope models will be explained in the advanced section.

### Imports

As an Xtext grammar is allowed to reference arbitrary other meta models as well as generate its own additional meta models, it must be possible to define the scopes for types from any of these meta models. With the import statement we can import all the meta models required to define the scoping.

There are no implicit imports: you have to import all meta models that are referred to in the scope file, including the one generated by the Xtext grammar for which you define scoping.

In the following example two meta models are imported. The import order is irrelevant.

```
import "http://www.avaloq.com/tools/dsl/mydsl"
```

Optionally, as with the second import, we can also specify an alias for the imported model. By default, the alias is the name of the imported EPackage.

Aliases are sometimes necessary to disambiguate a type reference, as two imported EPackages may both define a type with the same name.

### Extensions

The DSL supports defining scopes using Xtend expressions. To support modularization it is possible to specify Xtend extension files whose extensions are then available to the scope expressions. Example:

```
extension com::avaloq::tools::dsl::mydsl::MyDslScoping        // native helper operations
```

In order to be found, the extension must be on the classpath.

### Default namings

The default naming section, which can appear only once in a given scope file, is used to define both:

- A default case sensitivity for scopes (case sensitive is the default).

- Default naming definitions that apply to all scope rules. These are only used if no other, more specific naming rules are defined by the scope rules.

For instance:

```
case insensitive naming {

  mydsl::Entity                   = name;

  typeModel::INamedElement        = this.getName();

}
```

For each type, one or multiple naming expressions can be defined.

There are three types of naming expressions:

1. Normal Xtend expressions, with this referring to the scoped element which will be of the type specified. Such expressions may use operations defined in Xtend extensions. The expression can also be a constant string.

2. Factory expressions. These have the format factory functionCall(), where functionCall is a function declared in an Xtend extension file as a Java method. The function call is actually again an Xtend expression, with this referring to an object of the type for which the naming is defined. The function may also have additional parameters. The called function is supposed to return a NameFunction.
A NameFunction is the runtime representation of a naming expression. It has two methods, both named apply, one which is passed in an EObject (i.e., a model object), and a second one which takes an IEObjectDecsription as a parameter (i.e., an index entry). The default implementation for the index-entry just gets the indexed object or proxy and then invokes the apply method on that EObject. In some cases, factory methods may be useful to avoid loading resources too early by overriding the apply for the index entry to work directly with the information available in the index.

3. Additionally it is also possible to specify the keyword export as a naming expression. This then corresponds to the name the object was exported as in the index. (warning) Note that the export naming expression currently is valid for IEObjectDescriptions (not EObjects!) only. It thus applies to elements obtained from the global index and it is the preferred naming expression to use as index lookups can then be optimized.

As can be seen in the example it is perfectly legal to define names for abstract types. At runtime the most specific names applying to the scoped element will be used. I.e. the declarations are polymorphic.

### Scope definitions

Aside from the header declarations and the default naming section the top level elements of the language are scope definitions. As already briefly noted a scope definition states which elements are part of the scope in a given context. The context lets us individually define the actual elements of the scope for the various types or the actual cross reference of the context object and the objects comprising its containment hierarchy. We may also define the elements for a resource level catch all context.

A scope definition references the type corresponding to the type of the scope's elements and it further contains a set of rules stating what the scope's actual elements are in various contexts.
