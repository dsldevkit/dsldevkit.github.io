---
title: Export Language
keywords: export
sidebar: mydoc_sidebar
permalink: export.html
folder: export
---

The export language defines which objects are exported from a certain resource and thus made available to link against from other resources. Which objects are imported by a resource, on the other hand, is defined indirectly by means of scoping rules in the scope language. By matching up exported against imported objects the Xtext builder is able to compute the dependencies and thus determine which resources have to be rebuilt if one particular resource (or set of resources) has changed. To determine if a resource has changed the builder again uses the exported objects: It compares the exported objects as they were before the change with what they are after the change.

There are two main reasons to export an object description for an object

1. To be able to compare the previous and new states of object description to trigger invalidation

2. In addition to (1) we also want to discover the object using find () construct in scope language or using an index query from Java code

We differentiate these two kinds exports so we can tune performance accordingly.

## Background

As an introduction it is recommended that the reader first reads the relevant sections in the Xtext reference documentation on Linking and Scoping.

The set of all IResourceDescription objects is what we refer to as the Xtext index and it is managed by the Xtext builder. By exporting selected objects of a resource they are added as IEObjectDescription objects to the Xtext index and can then be linked against (using an appropriate scope provider) without first having to load (i.e. parse) the resource. This is the most important function of the Xtext index: Provide fast and convenient access to selected information of resources without requiring the client to load them or knowing where they are located.

As already pointed out the Xtext builder manages the Xtext index. By building resources it creates the IResourceDescription objects (as defined by the export file for the respective language) and then adds them to the index. But it also rebuilds other resources depending on a built resource. Naturally it should only rebuild depending resources if there was an actual change to the built resource. To be more precise: Only if the change represents a semantic change relevant to depending resources. This is again where the exported objects come in. By structurally comparing the set of exported objects before and after the change the builder is able to determine if dependent resources need rebuilding. It is therefore very important that the exported objects capture all the required information to perform this comparison.

So basically, a resource should export any object that can be accessed form another resource. Along with it all relevant semantic information about it.

Sometimes it may make sense to export more information than necessary for linking or invalidation. Further use-cases include discovery of contributions (i.e. partial classes in C#) or can be used beyond linking i.e. to keep track of contributions to generated objects as long as no dedicated mechanism available.

At the same time we have to make sure we don't export too many exported objects. It may for instance not make sense to export the parameters of a Java procedure as separate objects as the parameters can never be referenced on their own; i.e. without also having an explicit reference to the procedure. Note that procedure is also never referenced without a reference to the package, but is still exported to suport fine grain invaldiation.

## Structure

The general structure of an export file is:

```
import "..."

extension ...

interface {

  ...

}


export ... {

  ...

}
```

### File Naming

The name of an export file must correspond to that of the Xtext grammar it pertains to. In fact it must also reside in the same Java package. So if we have an Xtext grammar AvqScript.xtext we would create a file AvqScript.export in the same folder to define what objects are exported.

### Importing Meta Models

In the meta model imports we can import Ecore models and thus make the contained types available to subsequent declarations and expressions in the export section. We do this using the import keyword as in the following example:

```
import "http://www.avaloq.com/tools/dsl/mydsl"
```

We've imported the MyDsl Ecore model using its namespace URI.

Note that we did not use the Ecore model's workspace URI as we normally do in the Xtext grammars. In the Xtext grammar this is used as a trick as the Ecore models will usually not have been registered.

From then on, types defined in the imported meta models can be referenced by type name (e.g. class or method). For cases when referenced type names are not unique (i.e. multiple imported meta models define types with the same name) or when we want to make it clear, we can associate an alias with an imported meta model:

import "http://www.avaloq.com/tools/dsl/mydsl" as mydsl

Now we can reference types from that model by prefixing the reference with the alias and a "::" as in e.g. mydsl::Entity. Note that as long as the type name is unique we can still reference it without the alias.

### Importing Xtend extensions

The export language is based on Xtend, so any expressions will be Xtend expressions. Just like in regular Xtend or Xpand files we may want to reference other Xtend files (files with extension ext) to be able to use further extensions in our Xtend expressions. We import Xtend files using the extension keyword:

```
extension com::avaloq::tools::dsl::mydsl::MyDslScoping
```

Due to limitations in the Xtend to Java expression compiler we can only use static JAVA extensions defined in the imported Xtend files.

### Export Section

The export section is the main part of an export file and defines exactly which objects of the Xtext resource should be exported along with details regarding names, user data, fingerprints, and URI fragments. In Xtext terms the export section defines for which objects the builder creates IEObjectDescriptions in the IResourceDescription of a resource.

The export section is split in two parts. The first part is optional and is called the interface section and is used to compute the fingerprints for the exported objects. In the second part we define the actual rules which control what objects get exported.

## Interfaces

An interface defines exactly what pieces of information are semantically relevant to clients referencing a particular object. I.e. if any of these pieces of information change the dependent object must be revalidated. So for example if the visibility of a Java procedure changes from public to protected any sources calling that procedure must be revalidated, as they may no longer be allowed to call that procedure. On the other hand, if we change the procedure's implementation (its body) we don't need to revalidate any sources calling that procedure, as we haven't changed its interface.

Typically it makes sense to define the interface for every exported object. But since the interface for an exported object may also include pieces of information of other objects (e.g. the interface for a Java method must also include information about its type and its parameters, which are separate objects), we will often also have to define the interface for additional objects.

Once we've established which pieces of information constitute the interface of our exported objects we specify this as interface specifications in the export file.

We only need to define interfaces for sources which contain objects which can be referenced by other sources.

### Interface specifications

An interface specification for a given object is written as a list of expressions which navigate the object's Ecore model. So if we consider an example of an procedure in some scripting language (whose EClass name is MethodDeclaration) our specification may look like this:

```
interface {

  MethodDeclaration = name, @+modifiers, @parameterList;

  ModifierCompound  = visibility, extendability, eval(obsolete != null);

  ParameterList     = ...;

}
```

What the cryptic operators @, +, and eval mean will be explained in a moment. But we see that we've included information about the procedure's name, modifiers, and parameters by referencing the corresponding structural features of the EClass MethodDeclaration. Likewise a ModifierCompound's interface specification includes the value of its visibility, extendability, and obsolete features. Details about the specification for ParameterList was omitted to keep the example brief.

What follows after the equality sign in an interface specification is a list of what we refer to as interface items. An interface item can be one of three things:

*Value expression*: Used to include an object's value (or values in the case of multi-valued features) of a specific structural feature (both attributes and references are allowed) in the object's interface specification.The syntax is simply to write the feature's name (e.g. visibility as in the example).

*Reference expression*: Same as value expression except that this is for references only and that instead includes all interface items of the referenced object(s). I.e. there should be interface specification for the type(s) of the referenced object(s). The syntax is @ followed by the reference's name (e.g. @parameterList as in the example).

*Generic expression*: Used to include the evaluation value of an arbitrary expressions in an object's interface. The syntax is eval(expression), where expression is any Xtend expression with the implicit variable this bound to the object for which the interface is being computed (e.g. eval(obsolete != null) as in the example).

For all three kinds of expressions, a + sign can be added to indicate that if the expression evaluates to a many-valued result, the order of the elements in the list is irrelevant to the interface. So if a + is present the interface remains unchanged if only the order of the referenced elements changes. In the @+modifiers reference expression in the example this means that we can change the order of a procedure's modifiers without causing that to change the procedure's interface.

### Inheritance of interface specification

The syntax allows us to specify the interface specifications for abstract types or other common super types. In the example extract below we've declared name as an interface item for Declaration. This will be included in the interface for all Declaration's direct and indirect sub types (MethodDeclaration and FunctionDeclaration). FunctionDeclaration is a sub type of MethodDeclaration. Thus its complete interface specification will effectively be name, @+modifiers, @parameterList, @returnType.

```
interface {

  Declaration         = name;

  MethodDeclaration   = @+modifiers, @parameterList;

  FunctionDeclaration = @returnType;

}
```

### Guarded interface specifications

Interface definitions for a particular type can also have a guard:

```
interface {

  Declaration [isPublic()] = name;

}
```

What this means is that the given interface items will only be included in the object's interface if the guard evaluates to true.

### Fingerprints

Now we have seen how the interface can be specified for an exported object. It's the object's interface which decides when an object is to be regarded as changed. As we in the Xtext index don't have any use for the interface as a whole (nor is there any dedicated structures for storing it), we compute and store a hash of all interface items. We refer to this as a fingerprint and we store it as a user data field in the index for the exported object it pertains to.

The fingerprint is computed as an MD5 hash of all interface items. Also the fingerprint will include a hash for every object's URI fragment referenced by the interface items. Using an MD5 hash reduces the memory requirements to at most a string of 32 characters. MD5 is an 128-bit one-way hash; the collision probability is so low that we may ignore it for all practical purposes. The probability that two different inputs produce the same MD5 hash value is, with 128 bits, about 2-64 (~10-18). The probability that given a set of n inputs there is a collision between any two of the inputs is higher (that's the "birthday paradoxon"), but still comfortably low: even for n=8.2E11, the collision probability is still of the order of 10 -15. If that should be considered too high, a combined MD5/SHA-1 hash could be used (the two functions are independent), which gives a 288 bit hash value.

## Export declarations

We have now seen how we can define the interface for an exported object. What's missing is how to specify which objects should be exported and what details to attach to them.

In the following simple example we specify that we want to export all objects of type ScriptPackage and MethodDeclaration of a source.

```
export lookup ScriptPackage as name {

}


export MethodDeclaration as name {

}
```

So we start with the keyword export followed by the name of the type of the objects to export. Then we specify by adding a lookup keyword whether the export is meant for lookup in the index with an index query using the name of the object. If no lookup keyword is specified, then the object is just used for fingerprints and/or user data export. What follows after the keyword as is the mandatory name under which the object should be exported. This is an arbitrary Xtend expression (where the implicit variable this is bound to the exported object) but typically it will simply be the name of an attribute (e.g. name).

### Object names

As we have seen every exported object has a name. By default the name of every exported object is qualified. So if we for example have an package *util* in our scripting language with a function *bool_to_char* then the name of the function inside the index will be *util.bool_to_char*. The reason is that we would like to have names which are more or less unique.

So by default the name of every exported object is prefixed by that of its nearest exported container object. If there is no nearest exported container object the name is not prefixed. Sometimes we may want to suppress the automatic prefixing. We can do this by using the qualified keyword as in the following example:

```
export lookup ClassType as name {

}

export ClassTypeConstructor as qualified this.getName() {

}
```

A constructor will be exported under the same name as the class. This makes sense as that's how we reference a constructor from the code.

### Conditional exporting

Using a guard we can specify a condition under which an object is to be exported. In the following example we specify that a MethodDeclaration is only to be exported if it's a top-level declaration and it isn't private. I.e. we don't want to export nested or private procedures, which are not accessible to the outside world.

export MethodDeclaration as name [this.isTopLevel() && !this.isPrivate()] {

}

### User data

When exporting an object we can attach arbitrary user data entries to it. As we quite often want to export one of the object's attributes as a user data entry there is a dedicated syntax for that using the field keyword. In the following example we want to export the field id and compute user_id for every object:

export lookup Entity as this.getQualifiedName() {

  field id;

  data user_id  = this.getUserId();

}

As a result the exported object will now have two user data entries with name id and user_id where the first corresponds to value for the object's EAttribute and the second is compute by calling getUserId operation.

User data entries are skipped if the value is null.

Computations made to export objects should not depend on external references.

It is a bad practice to require references to external objects to be resolved to compute the name or the user data entries of an exported object. The reason is that the referenced resource(s) may not exist in the index at the time the value is computed as the builder may not build the sources in the required order. The builder is able to detect such problems and will retry to build the corresponding sources. But not only will that fail in case of cyclic references, but it is also bad practice as it may slow down the builder considerably.

Prefer: if this is really needed, typically we talk about "include" and not "refer", so use direct load of sources over file system. This would be an example of header files in C++. File names shall be known without having to involve index.
