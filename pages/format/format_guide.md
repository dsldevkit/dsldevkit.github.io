---
title: Format Language Guide
keywords: Format
sidebar: mydoc_sidebar
permalink: format_guide.html
folder: format
---

Structure of a Format source:

1. header
2. optional constant declarations
3. optional Rules

## Header

In the header section a name is provided which should coincide with that of the Xtext grammar it
pertains to. Optionally the Format source may reuse another Format source, provided that it pertains
to an Xtext grammar in the same hierarchy. Moreover, using an extends clause a user can specify
which formatter (e.g. `AbstractDdkFormatter`) should be set as the super class of the
generated `AbstractMyDslFormatter`.

## Format inheritance

Format supports the reuse of existing format configurations. Extending another format configuration causes its rules to be included unless they are overridden. Rules in the extending format configuration overwrite any rules in the reused configuration if they both apply to the same Xtext GrammarElement. So imported rules always have a lower priority than declared ones.

Example:

```
formatter for org.xtext.example.mydsl.MyDsl with org.eclipse.xtext.common.Terminals
override ID {
}

formatter for org.eclipse.xtext.common.Terminals
ID {
  rule : linewrap around;
}
STRING {
  rule : space " " before;
}
```

In the above the MyDsl formatter overrides Terminals' line-wrap around IDs so that there is no line-wrapping around IDs, but the STRING rule will be in effect in MyDsl sources.
Notice that even though the hierarchy of format configuration is tree-like, the hierarchy of generated AbstractMyDslFormatters is flat. Rules from all parent format configurations (all ancestors) of the given language are simply huddled together in the single Java class - the AbstractMyDslFormatter.

## Constant declarations
Some locators, such as linewrap, take arguments. Instead of providing the value in the form of a literal one can also reference a declared constant.
Only String and int constants are supported.

Example:

```
formatter for org.xtext.example.mydsl.MyDsl

const int FOO = 2;

MyDsl {
  "begin" : linewrap FOO after;
}
```

## Rules
For each grammar/Xtext rule a formatting rule can be declared. Furthermore an optional wildcard rule
can be declared. Rules may override rules declared in other Format sources if Format source reuse
exists (i.e. header declared as formatter for `Xyz` with `BaseXyz`).

## Rule

A Rule consists of locators which are applied to one or more grammar elements in a fashion - a
formatting directive is expressed with such a coupling, i.e. for *element* do *what where*.


```
MyDsl {
  "begin" : increment after, linewrap FOO after;
}
```

In the example above the grammar element is the keyword `"begin"`.
Locators are `increment` and `linewrap`. Locators are to be applied *after* the element `"begin"`.

In case of Format reuse, a rule declared in extending source must be marked with the keyword
`override` if there is a rule in any reused source applying to the same Xtext `GrammarElements`.

Example:

```
override X {
  ...
}
```

## Grammar elements
Grammar elements that can be used to express formatting directives are

1. Keywords enclosed by `"`, e.g. `"begin"`

   also applicable to wildcard rules

2. Rule-calls

   prefixed by `@`, e.g. `@ID`

   not applicable to wildcard rules

3. Assignments

   prefixed by `=`, e.g. `=elements`

   not applicable to wildcard rules

4. Groups

   an Xtext compound element (e.g. alternatives, group or unordered group)

   not applicable to wildcard rules

5. Reference to other Rules

   A (Xtext-) grammar rule identified by its name, e.g. Element

   not applicable to wildcard rules

   can only be used in combination with one other grammar element and matcher range or between

6. rule

   the current context/rule

   not applicable to wildcard rules

7. Keyword pairs

   enclosed by `(` and `)`, e.g. `("[" "]")`

8. Keyword search result

   comma separated keywords enclosed by `[` and `]`, e.g. `[";", ","]`

9. Rule-call search result

   Rules enclosed by `[ and `], e.g. `[INT, ID]`

## Locators
There are three categories of locators - indentation, linewrap and space. The locators within a category distinguish themselves by the effect they have and by precedence.

In all the following syntax descriptions, the keyword ident means that the constant name should be used.

### Indentation Locators
- Increment

  Increases the current global indentation.

  Syntax: `"increment" [number | ident] matcher`

  Example: `increment 5 after`

- Decrement

  Decreases the current global indentation.

  Syntax: `"decrement" [number | ident] matcher`

  Example: `decrement after`

### Linewrap Locators

- Linewrap

  Inserts a linewrap.

  The locator supports the optional specification of the number of lines to wrap or the specification of the minimum, default and maximum number of lines to wrap.

  Syntax: `"linewrap [(number | ident) | ((number | ident) (number | ident) (number | ident))] matcher`

  Examples:

  `linewrap before`

  `linewrap FOO after`

  `linewrap 0 2 2 before`

- NoLinewrap - HIGH #precedence

  Suppresses any line wrap.

  Syntax: `"no_linewrap" matcher`

  Example: `no_linewrap after`

### Special Locators

- NoFormat

  This locator disables any formatting for a grammar element

  Syntax: `"no_format"`

  Example: `no_format`

### Space Locators

- Space

  This locator appends a grammar element with a fixed amount of space. The space is given in the form of a string.

  Syntax: `"space" [(string | ident)] matcher`

  Example: `space " " after`

- NoSpace - HIGH #precedence

  Suppresses any spacing for a grammar element.

  Syntax: `"no_space" matcher`

  Example: `no_space after`

- RightPadding

  There is no such locator in the Xtext formatter framework.

  Requires a grammar element occupy at least a specified amount of characters. If necessary appends with white spaces.

  Syntax: `"right_padding" [(number | ident)] ("before" | "after" | "around")`

  Example: `right_padding 30 after`

- Column

  There is no such locator in the Xtext formatter framework.

  Requires a grammar element to have at least a specified amount of characters in front. If necessary prepends with white space. If the correctly formatted grammar element spans over multiple lines (e.g. if-statement) then the column alignment is applied to all correctly formatted lines of this element - i.e. column alignment spans also over multiple lines. This is the significant difference in comparison against the old column locator syntax.

  Syntax: `"column" "fixed"? [(number | ident)] "relative"? "nobreak"? "before"`

### Advanced Column Locator

To understand the meaning of the following keywords: `fixed`, `relative`, `nobreak` see the examples below.
All three keywords can be combined in the same column alignment rule.

Notice that only the keyword `before` is allowed. This means that the column alignment is applied to the current element (it refers to the grammar element for which it is defined). Contradictory, the keyword `after` applies to the next grammar element. Due to technical reasons - presence of opening and closing markers indicating the grammar element - the keyword `after` is disallowed, as it requires searching for the end of the next element (in the parse tree), when the current element is being considered, which makes the code overcomplicated and much less efficient. In return however, a user cannot benefit from any extra formatting feature. In other words: instead of using the `column after` formatting simply define the `column before` formatting for the subsequent element from the grammar.

Example: default column alignment (column alignment applied to the *case-statement*)

```
//column 30 before;         
//                           | <-- caret position = 30
      column descn            case tab.is_obsolete
                                when '+' then
                                  'Substitute Information: ' || nvl(ltrim(rtrim(tab.subst, ']'), '['), 'N/A')
                                else
                                  tab.descn
                              end
```

Example: non-fixed fixed (column alignment applied to the *case-statement*)

```
//column 10 before;         
//       | <-- caret position = 10
      column descn case tab.is_obsolete
            when '+' then
              'Substitute Information: ' || nvl(ltrim(rtrim(tab.subst, ']'), '['), 'N/A')
            else
              tab.descn
          end
```

The fixed keyword ensures that when the current content of the line exceeds the value required by the column, the column is not shifted to the position relative to the content of the line (current content of the line + single whitespace), but the line break occurs.

Example: fixed column alignment

```
//column fixed 10 before;         
//       | <-- caret position = 10
      column descn
          case tab.is_obsolete
            when '+' then
              'Substitute Information: ' || nvl(ltrim(rtrim(tab.subst, ']'), '['), 'N/A')
            else
              tab.descn
          end
```

Example: non-relative column alignment applied to the case-statement

```
//column 30 before;         
//                           | <-- caret position = 30
      column descn            case tab.is_obsolete
                                when '+' then
                                  'Substitute Information: ' || nvl(ltrim(rtrim(tab.subst, ']'), '['), 'N/A')
                                else
                                  tab.descn
                              end
```

Keyword `relative` changes the column alignment to be relative to the line's indentation. With no indentation it is equivalent with column. It has exactly the same behavior as the obsolete offset: obsolete offset alignment.

```
//column 30 relative before;         
//   | <-- indentation size = 6                       
//                           | <-- caret position = 30
//                                 | <-- 30+6
      column descn                  case tab.is_obsolete
                                      when '+' then
                                        'Substitute Information: ' || nvl(ltrim(rtrim(tab.subst, ']'), '['), 'N/A')
                                      else
                                        tab.descn
                                    end
```

In the next example `nobreak` keyword in column alignment 40 applied to the `notrim` keyword and column alignment 30 applied to the case-statement.

```
//column 40 before;         
//                                     | <-- caret position = 40
      column descn            case tab.is_obsolete
                                when '+' then
                                  'Substitute Information: ' || nvl(ltrim(rtrim(tab.subst, ']'), '['), 'N/A')
                                else
                                  tab.descn
                              end      
                                        notrim
//                                     | <-- caret position = 40
```

Normally when the column aligned grammar element, that spans over multiple lines, ends a line break occurs.
The `nobreak` keyword simply turns this behavior off (when applied to the grammar element aligned to the
column occurring just after the multi-lined, column aligned element).

```
//column 40 nobreak before;         
//                                     | <-- caret position = 40
      column descn            case tab.is_obsolete
                                when '+' then
                                  'Substitute Information: ' || nvl(ltrim(rtrim(tab.subst, ']'), '['), 'N/A')
                                else
                                  tab.descn
                              end       notrim
//                                     | <-- caret position = 40
```


## Locators precedence

Locators with a high precedence do not combine/aggregate with others within the same category,
whereas locators with lower precedence combine/aggregate with each other. For example combining
space `""` with space `" "` is equivalent to the latter, but `no_space` combined with space `" "` does not produce any spacing.

Use high precedence locators sparingly as they can have unexpected consequences.
The careful reader might have spotted an error in the complex Format example above:
in the wildcard rule all `"="` keywords are configured to have no space before or after.
This will supersede some of the declarations for Element.

## Matcher
Locators are applied relative to one or more grammar elements.

- before

  The locator is applied before the matched grammar element.

- after

  The locator is applied after the grammar element has been matched.

- around  

  Synonymous to applying the locator before and after the grammar element

- between

  May only be used in declarations with exactly two grammar elements.
  This matches if `ele2` directly follows `ele1` in the document. There may be no other tokens in between `ele1` and `ele2`.

- range

  May only be used in declarations with exactly two grammar elements
  The rule is enabled when ele1 is matched, and disabled when `ele2` is matched.
  Thereby, the rule is active for the complete region which is surrounded by `ele1` and `ele2`.
