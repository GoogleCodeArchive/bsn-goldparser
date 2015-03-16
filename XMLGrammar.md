
```
"Name"     = 'eXtensible Markup Language'
"Author"   = 'Arsène von Wyss'
"Version"  = '1.0'
"About"    = 'Grammar which tries to comply as well as possible with  http://www.w3.org/TR/2008/REC-xml-20081126/'

! This grammar should be able to properly parse any XML file, with the following caveats:
! - Processing Instructions, including the XML declaration, are parsed as one terminal
! - Same for the DOCTYPE with all its content
! - Since the tokenizer doesn't know anything about the productions, the content and attribute value rule output needs to be post-processed, concatenating data as required and making whitespace significant or unsignificant depending on the context

"Case Sensitive" = True
"Start Symbol" = <Document>
"Character Mapping" = Unicode
"Auto Whitespace" = False

! -------------------------------------------------
! Character Sets
! -------------------------------------------------

{Char} = {&9} + {&A} + {&D} + {&20..&D7FF} !+ {&E000..&FFFD} + {&10000..&10FFFF}
{S} = {&9} + {&A} + {&D} + {&20}
{NameStartChar} = [abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ:_] + {&C0..&D6} + {&D8..&F6} + {&F8..&2FF} + {&370..&37D} + {&37F..&1FFF} + {&200C..&200D} + {&2070..&218F} + {&2C00..&2FEF} + {&3001..&D7FF} + {&F900..&FDCF} !+ {&FDF0..&FFFD} + {&10000..&EFFFF}
{NameChar} = {NameStartChar} + [-.0123456789] + {&B7} + {&0300..&036F} + {&203F..&2040}
{NonNameChar} = {Char} - [<>&"''] - {S} - {NameStartChar}
{CDataChar} = {Char} - [']'>]
{PIChar} = {Char} - [?>]
{CommentChar} = {Char} - [->]
{DoctypeChar1} = {Char} - ['['>]
{DoctypeChar2} = {Char} - [']']

! -------------------------------------------------
! Terminals
! -------------------------------------------------

S = {S}+
Name = {NameStartChar}{NameChar}*
NonName = {NonNameChar}+ ! | Name
EntityRef = '&'{NameStartChar}{NameChar}*';'
CharRef = '&#'{Digit}+';'
CData = '<![CDATA[' ({CDataChar} | '>' | ']>' | ']'+{CDataChar} )* ']'+ ']>'
PI = '<?' {NameStartChar}{NameChar}* ({S}+ ({PIChar} | '>' | '?'+{PIChar} )* )? '?'+ '>'
Comment = '<!--' ({CommentChar} | '>' | '->' | '-'+{CommentChar} )* '-'+ '->'
Doctype = '<!DOCTYPE' {DoctypeChar1}+ ('[' {DoctypeChar2}+ ']' {S}*)? '>'

! -------------------------------------------------
! Rules
! -------------------------------------------------

! The grammar starts below
<Document> ::= <Prolog> <Element> <Misc>

<Prolog> ::= PI <S> Doctype <Misc>
           | <S> Doctype <Misc>
           | <Misc>

<Misc> ::= Comment <Misc>
         | PI <Misc>
         | S <Misc>
         |

<Content> ::= <Content> Name
            | <Content> NonName
            | <Content> '='
            | <Content> EntityRef
            | <Content> CharRef
            | <Content> Comment
            | <Content> CData
            | <Content> PI
            | <Content> S
            | <Content> <Element>
            |

<Element> ::= <ElementStart> <Content> <ElementEnd>
            | <ElementEmpty>

<ElementEmpty> ::= '<' Name <AttributeList> '/>'

<ElementStart> ::= '<' Name <AttributeList> '>'

<ElementEnd> ::= '</' Name '>'

<AttributeList> ::= S <Attribute> <AttributeList>
                  | <S>

<Attribute> ::= Name <S> '=' <S> <QuotedValue>

<QuotedValue> ::= '"' <QuotValue> '"'
                | '' <AposValue> ''

<AposValue> ::= <AposValue> Name
              | <AposValue> NonName
              | <AposValue> '='
              | <AposValue> '"'
              | <AposValue> '>'
              | <AposValue> EntityRef
              | <AposValue> CharRef
              | <AposValue> S
              |

<QuotValue> ::= <QuotValue> Name
              | <QuotValue> NonName
              | <QuotValue> '='
              | <QuotValue> ''
              | <QuotValue> '>'
              | <QuotValue> EntityRef
              | <QuotValue> CharRef
              | <QuotValue> S
              |

<S> ::= S
      |
```