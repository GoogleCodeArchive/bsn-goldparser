# The XML use case #

Any grammar read with the default `Tokenizer` and `LalrProcessor` implementations can be read as XML by using the `TokenXmlReader` class. This can be processed with an XSLT style sheet to generate a different representation of data not available as XML, basically the combination of the parser and an XSLT stylesheet enable parsing almost anything to a nice XML representation.

Possible use cases:
  * Convert textual formulas to [MathML](http://www.w3.org/TR/MathML3/)
  * Convert drawing instructions to [SVG](http://www.w3.org/TR/SVG/)
  * Convert some Wiki markup or markdown to [(X)HTML](http://www.w3.org/TR/xhtml1/) or even [XSL-FO](http://www.w3.org/TR/xsl11/)
  * Implement [syntax highlighting](http://en.wikipedia.org/wiki/Syntax_highlighting) or  [pretty printing](http://en.wikipedia.org/wiki/Prettyprint)

# What it does #

It exposes the grammar tree previously parsed into a `Token` as XML tree, using elements with the same name as their symbol (as far as XML can handle those) in the grammar and using namespaces (`urn:nonterminal` for reductions and `urn:terminal` for text tokens).

Things you should know:
  * Only terminals have line information attached to them
  * All insignificant data, such as comments and whitespace, is not included in the XML output
  * Symbol names are converted to XML compatible names by removing all invalid characters from the name. If the resulting name is empty, an `_` (underscore) character is used as name. Note that XML names may therefore be ambiguous compared to the original names in the grammar, the symbols `<A B>` and `<AB>` for instance would get the same name (AB), as well as `'=='` and `'.'` etc. would (`_`). Therefore you may want to name all terminals (in the GOLD grammar) which you want to transform later on :
```
EQUAL = '=='
DOT = '.'
! etc.
```