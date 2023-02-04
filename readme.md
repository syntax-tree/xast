# ![xast][logo]

E**x**tensible **A**bstract **S**yntax **T**ree format.

***

**xast** is a specification for representing [XML][] as an abstract
[syntax tree][syntax-tree].
It implements the **[unist][]** spec.

This document may not be released.
See [releases][] for released documents.
The latest released version is [`1.0.0`][latest].

## Contents

*   [Introduction](#introduction)
    *   [Where this specification fits](#where-this-specification-fits)
    *   [Scope](#scope)
*   [Types](#types)
*   [Nodes](#nodes)
    *   [`Parent`](#parent)
    *   [`Literal`](#literal)
    *   [`Root`](#root)
    *   [`Element`](#element)
    *   [`Text`](#text)
    *   [`Comment`](#comment)
    *   [`Doctype`](#doctype)
    *   [`Instruction`](#instruction)
    *   [`Cdata`](#cdata)
*   [Glossary](#glossary)
*   [List of utilities](#list-of-utilities)
*   [References](#references)
*   [Related](#related)
*   [Contribute](#contribute)
*   [Acknowledgments](#acknowledgments)
*   [License](#license)

## Introduction

This document defines a format for representing XML as an [abstract syntax
tree][syntax-tree].
This specification is written in a [Web IDL][webidl]-like grammar.
Development started in January 2020.

### Where this specification fits

xast extends [unist][], a format for syntax trees, to benefit from its
[ecosystem of utilities][utilities].

xast relates to [JavaScript][] in that it has an [ecosystem of
utilities][list-of-utilities] for working with compliant syntax trees in
JavaScript.
However, xast is not limited to JavaScript and can be used in other programming
languages.

xast relates to the [unified][] project in that xast syntax trees are used
throughout its ecosystem.

### Scope

xast represents XML syntax, not semantics: there are no namespaces or local
names; only qualified names.

xast supports a sensible subset of XML by omitting the ostensibly bad DTD.
XML processors are not guaranteed to process DTDs, making them unsafe.

xast represents expanded entities and therefore does not deal with entities or
character references.
It is suggested that utilities around xast, that parse or serialize, do *not*
support *[parameter-entity references][concept-parameter-entity]* or
*[entity references][concept-entity]* other than the
*[predefined entities][concept-predefined-entities]*
(`&lt;` for `<` U+003C LESS THAN;
`&gt;` for `>` U+003E GREATER THAN;
`&amp;` for `&` U+0026 AMPERSAND;
`&apos;` for `'` U+0027 APOSTROPHE;
`&quot;` for `"` U+0022 QUOTATION MARK).
This prevents *[billion laughs][billion-laughs]* attacks.

###### Declarations

*[Declarations][concept-declaration]* ([\[XML\]][xml]) other than
[doctype][dfn-doctype] have no representation in xast:

```xml
<!ELEMENT %name.para; %content.para;>
<!ATTLIST poem xml:space (default|preserve) 'preserve'>
<!ENTITY % ISOLat2 SYSTEM "http://www.xml.com/iso/isolat2-xml.entities">
<!ENTITY Pub-Status "This is a pre-release of the specification.">
<![%draft;[<!ELEMENT book (comments*, title, body, supplements?)>]]>
<![%final;[<!ELEMENT book (title, body, supplements?)>]]>
```

###### Internal subset

Internal document type declarations have no representation in xast:

```xml
<!DOCTYPE greeting [
  <!ELEMENT greeting (#PCDATA)>
]>
<greeting>Hello, world!</greeting>
```

## Types

If you are using TypeScript, you can use the unist types by installing them
with npm:

```sh
npm install @types/xast
```

## Nodes

### `Parent`

```idl
interface Parent <: UnistParent {
  children: [Cdata | Comment | Doctype | Element | Instruction | Text]
}
```

**Parent** (**[UnistParent][dfn-unist-parent]**) represents a node in xast
containing other nodes (said to be *[children][term-child]*).

Its content is limited to only other xast content.

### `Literal`

```idl
interface Literal <: UnistLiteral {
  value: string
}
```

**Literal** (**[UnistLiteral][dfn-unist-literal]**) represents a node in xast
containing a value.

### `Root`

```idl
interface Root <: Parent {
  type: 'root'
}
```

**Root** (**[Parent][dfn-parent]**) represents a document fragment or a whole
document.

**Root** should be used as the *[root][term-root]* of a *[tree][term-tree]* and
must not be used as a *[child][term-child]*.

XML specifies that documents should have exactly one **[element][dfn-element]**
child, therefore a root should have exactly one element child when representing
a whole document.

### `Element`

```idl
interface Element <: Parent {
  type: 'element'
  name: string
  attributes: Attributes?
  children: [Cdata | Comment | Element | Instruction | Text]
}
```

**Element** (**[Parent][dfn-parent]**) represents an
*[element][concept-element]* ([\[XML\]][xml]).

The `name` field must be present.
It represents the element’s *[name][concept-name]* ([\[XML\]][xml]),
specifically its *[qualified name][concept-qualified-name]*
([\[XML-NAMES\]][xml-names]).

The `children` field should be present.

The `attributes` field should be present.
It represents information associated with the element.
The value of the `attributes` field implements the
**[Attributes][dfn-attributes]** interface.

For example, the following XML:

```xml
<package xmlns="http://www.idpf.org/2007/opf" unique-identifier="id" />
```

Yields:

```js
{
  type: 'element',
  name: 'package',
  attributes: {
    xmlns: 'http://www.idpf.org/2007/opf',
    'unique-identifier': 'id'
  },
  children: []
}
```

#### `Attributes`

```idl
interface Attributes {}
```

**Attributes** represents information associated with an element.

Every field must be a **[AttributeName][dfn-attribute-name]** and every value an
**[AttributeValue][dfn-attribute-value]**.

#### `AttributeName`

```idl
typedef string AttributeName
```

Attribute names are keys on **[Attributes][dfn-attributes]** objects and must
reflect XML attribute names exactly.

#### `AttributeValue`

```idl
typedef string AttributeValue
```

Attribute values are values on **[Attributes][dfn-attributes]** objects and must
reflect XML attribute values exactly as a string.

> In [JSON][], the value `null` must be treated as if the attribute was not
> included.
> In [JavaScript][], both `null` and `undefined` must be similarly ignored.

### `Text`

```idl
interface Text <: Literal {
  type: 'text'
}
```

**Text** (**[Literal][dfn-literal]**) represents
*[character data][concept-char]* ([\[XML\]][xml]).

For example, the following XML:

```xml
<dc:language>en</dc:language>
```

Yields:

```js
{
  type: 'element',
  name: 'dc:language',
  attributes: {},
  children: [{type: 'text', value: 'en'}]
}
```

### `Comment`

```idl
interface Comment <: Literal {
  type: 'comment'
}
```

**Comment** (**[Literal][dfn-literal]**) represents a
*[comment][concept-comment]* ([\[XML\]][xml]).

For example, the following XML:

```xml
<!--Charlie-->
```

Yields:

```js
{type: 'comment', value: 'Charlie'}
```

### `Doctype`

```idl
interface Doctype <: Node {
  type: 'doctype'
  name: string
  public: string?
  system: string?
}
```

**Doctype** (**[Node][dfn-unist-node]**) represents a
*[doctype][concept-doctype]* ([\[XML\]][xml]).

A `name` field must be present.

A `public` field should be present.
If present, it must be set to a string, and represents the document’s public
identifier.

A `system` field should be present.
If present, it must be set to a string, and represents the document’s system
identifier.

For example, the following XML:

```xml
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.0 Transitional//EN" "http://www.w3.org/TR/REC-html40/loose.dtd">
```

Yields:

```js
{
  type: 'doctype',
  name: 'HTML',
  public: '-//W3C//DTD HTML 4.0 Transitional//EN',
  system: 'http://www.w3.org/TR/REC-html40/loose.dtd'
}
```

### `Instruction`

```idl
interface Instruction <: Literal {
  type: 'instruction'
  name: string
}
```

**Instruction** (**[Literal][dfn-literal]**) represents a
*[processing instruction][concept-instruction]* ([\[XML\]][xml]).

A `name` field must be present.

For example, the following XML:

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

Yields:

```js
{
  type: 'instruction',
  name: 'xml',
  value: 'version="1.0" encoding="UTF-8"'
}
```

### `Cdata`

```idl
interface Cdata <: Literal {
  type: 'cdata'
}
```

**Cdata** (**[Literal][dfn-literal]**) represents a
*[CDATA section][concept-cdata]* ([\[XML\]][xml]).

For example, the following XML:

```xml
<![CDATA[<greeting>Hello, world!</greeting>]]>
```

Yields:

```js
{
  type: 'cdata',
  value: '<greeting>Hello, world!</greeting>'
}
```

## Glossary

See the [unist glossary][glossary].

## List of utilities

See the [unist list of utilities][utilities] for more utilities.

*   [`xastscript`](https://github.com/syntax-tree/xastscript)
    — create trees
*   [`xast-util-feed`](https://github.com/syntax-tree/xast-util-feed)
    — build feeds (RSS, Atom)
*   [`xast-util-from-xml`](https://github.com/syntax-tree/xast-util-from-xml)
    — parse from XML
*   [`xast-util-sitemap`](https://github.com/syntax-tree/xast-util-sitemap)
    — build `sitemap.xml`
*   [`xast-util-to-string`](https://github.com/syntax-tree/xast-util-to-string)
    — get the text value
*   [`xast-util-to-xml`](https://github.com/syntax-tree/xast-util-to-xml)
    — serialize to XML
*   [`hast-util-to-xast`](https://github.com/syntax-tree/hast-util-to-xast)
    — transform to xast

## References

*   **JSON**
    [The JavaScript Object Notation (JSON) Data Interchange Format][json],
    T. Bray.
    IETF.
*   **JavaScript**:
    [ECMAScript Language Specification][javascript].
    Ecma International.
*   **unist**:
    [Universal Syntax Tree][unist].
    T. Wormer; et al.
*   **XML**:
    [Extensible Markup Language (XML) 1.0 (Fifth Edition)][xml]
    T. Bray; et al.
    W3C.
*   **XML-NAMES**:
    [Namespaces in XML 1.0 (Third Edition)][xml-names]
    T. Bray; et al.
    W3C.
*   **Web IDL**:
    [Web IDL][webidl],
    C. McCormack.
    W3C.

## Related

*   [hast](https://github.com/syntax-tree/hast)
    — Hypertext Abstract Syntax Tree format
*   [mdast](https://github.com/syntax-tree/mdast)
    — Markdown Abstract Syntax Tree format
*   [nlcst](https://github.com/syntax-tree/nlcst)
    — Natural Language Concrete Syntax Tree format

## Contribute

See [`contributing.md`][contributing] in [`syntax-tree/.github`][health] for
ways to get started.
See [`support.md`][support] for ways to get help.
Ideas for new utilities and tools can be posted in [`syntax-tree/ideas`][ideas].

A curated list of awesome syntax-tree, unist, hast, mdast, nlcst, and xast
resources can be found in [awesome syntax-tree][awesome].

This project has a [code of conduct][coc].
By interacting with this repository, organization, or community you agree to
abide by its terms.

## Acknowledgments

The initial release of this project was authored by **[@wooorm][author]**.

## License

[CC-BY-4.0][license] © [Titus Wormer][author]

<!-- Definitions -->

[health]: https://github.com/syntax-tree/.github

[contributing]: https://github.com/syntax-tree/.github/blob/HEAD/contributing.md

[support]: https://github.com/syntax-tree/.github/blob/HEAD/support.md

[coc]: https://github.com/syntax-tree/.github/blob/HEAD/code-of-conduct.md

[awesome]: https://github.com/syntax-tree/awesome-syntax-tree

[ideas]: https://github.com/syntax-tree/ideas

[license]: https://creativecommons.org/licenses/by/4.0/

[author]: https://wooorm.com

[logo]: https://raw.githubusercontent.com/syntax-tree/xast/c91a0c9/logo.svg?sanitize=true

[releases]: https://github.com/syntax-tree/xast/releases

[latest]: https://github.com/syntax-tree/xast/releases/tag/1.0.0

[dfn-unist-node]: https://github.com/syntax-tree/unist#node

[dfn-unist-parent]: https://github.com/syntax-tree/unist#parent

[dfn-unist-literal]: https://github.com/syntax-tree/unist#literal

[unist]: https://github.com/syntax-tree/unist

[syntax-tree]: https://github.com/syntax-tree/unist#syntax-tree

[javascript]: https://www.ecma-international.org/ecma-262/9.0/index.html

[xml]: https://www.w3.org/TR/xml/

[xml-names]: https://www.w3.org/TR/xml-names/

[json]: https://tools.ietf.org/html/rfc7159

[webidl]: https://heycam.github.io/webidl/

[billion-laughs]: https://en.wikipedia.org/wiki/Billion_laughs_attack

[glossary]: https://github.com/syntax-tree/unist#glossary

[utilities]: https://github.com/syntax-tree/unist#list-of-utilities

[unified]: https://github.com/unifiedjs/unified

[concept-parameter-entity]: https://www.w3.org/TR/xml/#dt-PERef

[concept-entity]: https://www.w3.org/TR/xml/#dt-entref

[concept-predefined-entities]: https://www.w3.org/TR/xml/#sec-predefined-ent

[concept-element]: https://www.w3.org/TR/xml/#NT-element

[concept-name]: https://www.w3.org/TR/xml/#NT-Name

[concept-char]: https://www.w3.org/TR/xml/#NT-Char

[concept-comment]: https://www.w3.org/TR/xml/#NT-Comment

[concept-doctype]: https://www.w3.org/TR/xml/#NT-doctypedecl

[concept-instruction]: https://www.w3.org/TR/xml/#NT-PI

[concept-declaration]: https://www.w3.org/TR/xml/#dt-markupdecl

[concept-cdata]: https://www.w3.org/TR/xml/#NT-CDSect

[concept-qualified-name]: https://www.w3.org/TR/xml-names/#NT-QName

[term-tree]: https://github.com/syntax-tree/unist#tree

[term-child]: https://github.com/syntax-tree/unist#child

[term-root]: https://github.com/syntax-tree/unist#root

[list-of-utilities]: #list-of-utilities

[dfn-parent]: #parent

[dfn-literal]: #literal

[dfn-element]: #element

[dfn-attributes]: #attributes

[dfn-attribute-name]: #attributename

[dfn-attribute-value]: #attributevalue

[dfn-doctype]: #doctype
