The grammar currently only defines the character sets and terminals for the SELF language as specified here: http://docs.selflanguage.org/langref.html#lexical-elements

```
"Name"     = 'SELF'
"Author"   = 'Arsène von Wyss'
"Version"  = '1.0'
"About"    = 'A grammar for the SELF language, following the definition at http://docs.selflanguage.org/langref.html'

"Case Sensitive" = 'True'
"Character Mapping" = 'Unicode'
"Auto Whitespace" = 'False'

"Start Symbol" = <Program>

! -------------------------------------------------
! Character Sets
! -------------------------------------------------

{Small Letter}  = [abcdefghijklmnopqrstuvwxyz]
{Cap Letter}    = [ABCDEFGHIJKLMNOPQRSTUVWXYZ]
{ID Head}       = {Small Letter} + [_]
{ID Tail}       = {ID Head} + {Cap Letter} + {Digit}
{General Digit} = {Letter} + {Digit}
{Normal Char}   = {All Valid} - [''\]
{Op Char}       = [!@#$%^&*-+=~/?<>,;|]
{Comment Char}  = {All Valid} - ["]

! -------------------------------------------------
! Terminals
! -------------------------------------------------

Whitespace    = ({Space} | {HT} | {LF} | {CR} | {VT} | {#8} | {FF} | '"' {Comment Char}* '"')+
Identifier    = {ID Head} {ID Tail}*
SmallKeyword  = {ID Head} {ID Tail}* ':'
CapKeyword    = {Cap Letter} {ID Tail}* ':'
Argument      = ':' {ID Head} {ID Tail}*
Operator      = {Op Char}+
Integer       = '-'? {Digit}+
BaseInteger   = '-'? {Digit} {Digit}? ('r' | 'R') {General Digit}+
FixedPoint    = '-'? {Digit}+ '.' {Digit}+
Float         = '-'? {Digit}+ ('.' {Digit}+)? ('e' | 'E') ('+' | '-')? {Digit}+
String        = '' ({Normal Char} | '\' ('t' | 'b' | 'n' | 'f' | 'r' | 'v' | 'a' | '0' | '\' | '' | '"' | '?' | 'x' {General Digit} {General Digit} | 'd' {Digit} {Digit} {Digit} | 'o' {Digit} {Digit} {Digit}))* ''

! -------------------------------------------------
! Rules
! -------------------------------------------------

! The grammar starts below
<Program> ::= <Value> <Program>
            |

<Value> ::= Identifier
          | SmallKeyword
          | CapKeyword
          | Argument
          | Operator
          | Integer
          | BaseInteger
          | FixedPoint
          | Float
          | String
```