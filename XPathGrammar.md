
```
"Name"     = 'XML Path Language (XPath) Version 1.0'
"Author"   = 'Arsène von Wyss'
"Version"  = '1.0'
"About"    = 'An implementation of XPath as defined by the W3C: http://www.w3.org/TR/xpath/'

"Case Sensitive" = True
"Start Symbol" = <Expr>
"Character Mapping" = Unicode
"Virtual Terminals" = mul ! '*' must be converted to this virtual token when there is a preceding token and the preceding token is not one of @, ::, (, [, , or an Operator -- see http://www.w3.org/TR/xpath/#exprlex

! -------------------------------------------------
! Character Sets
! -------------------------------------------------

{Char} = {&9} + {&A} + {&D} + {&20..&D7FF} !+ {&E000..&FFFD} + {&10000..&10FFFF}
{S} = {&9} + {&A} + {&D} + {&20}
{NameStartChar} = [abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_] + {&C0..&D6} + {&D8..&F6} + {&F8..&2FF} + {&370..&37D} + {&37F..&1FFF} + {&200C..&200D} + {&2070..&218F} + {&2C00..&2FEF} + {&3001..&D7FF} + {&F900..&FDCF} !+ {&FDF0..&FFFD} + {&10000..&EFFFF}
{NameChar} = {NameStartChar} + [-.0123456789] + {&B7} + {&0300..&036F} + {&203F..&2040}
{NonAposChar} = {Char}-['']
{NonQuotChar} = {Char}-["]

! -------------------------------------------------
! Terminals
! -------------------------------------------------

Whitespace = {S}+
NCName = {NameStartChar}{NameChar}*
AposLiteral = '' {NonAposChar}+ ''
QuotLiteral = '"' {NonQuotChar}+ '"'
Number = ({Digit}* '.')? {Digit}+ ([eE] [+-]? {Digit}+)?
! The original definition would be as follows: Number = {Digit}+ ('.' {Digit}*)? | '.' {Digit}+
! This would allow "1." but not "0.5e-4" - therefore the more common format with exponential number support is used here

! -------------------------------------------------
! Rules
! -------------------------------------------------

<Expr> ::= <OrExpr>

<OrExpr> ::= <AndExpr>
           | <OrExpr> 'or' <AndExpr>

<AndExpr> ::= <EqualityExpr>
            | <AndExpr> 'and' <EqualityExpr>

<EqualityExpr> ::= <RelationalExpr>
                 | <EqualityExpr> '=' <RelationalExpr>
                 | <EqualityExpr> '!=' <RelationalExpr>

<RelationalExpr> ::= <AdditiveExpr>
                   | <RelationalExpr> '<' <AdditiveExpr>
                   | <RelationalExpr> '>' <AdditiveExpr>
                   | <RelationalExpr> '<=' <AdditiveExpr>
                   | <RelationalExpr> '>=' <AdditiveExpr>

<AdditiveExpr> ::= <MultiplicativeExpr>
                 | <AdditiveExpr> '+' <MultiplicativeExpr>
                 | <AdditiveExpr> '-' <MultiplicativeExpr>

<MultiplicativeExpr> ::= <UnaryExpr>
                       | <MultiplicativeExpr> mul <UnaryExpr>
                       | <MultiplicativeExpr> 'div' <UnaryExpr>
                       | <MultiplicativeExpr> 'mod' <UnaryExpr>

<UnaryExpr> ::= <UnionExpr>
              | '-' <UnaryExpr>

<UnionExpr> ::= <PathExpr>
              | <UnionExpr> '|' <PathExpr>

<PathExpr> ::= <RelativeLocationPath>
             | <AbsoluteLocationPath>
             | <FilterExpr>
             | <FilterExpr> '/' <RelativeLocationPath>
             | <FilterExpr> '//' <RelativeLocationPath>

<AbsoluteLocationPath> ::= '//' <RelativeLocationPath>
                         | '/' <RelativeLocationPath>
                         | '/'

<RelativeLocationPath> ::= <RelativeLocationPath> '//' <Step>
                         | <RelativeLocationPath> '/' <Step>
                         | <Step>

<Step> ::= <AxisName> '::' <NodeTest> <PredicateList>
         | '@' <NodeTest> <PredicateList>
         | <NodeTest> <PredicateList>
         | '.' 
         | '..'

<PredicateList> ::= <Predicate> <PredicateList>
                  |

<Predicate> ::= '[' <Expr> ']'   

<AxisName> ::= 'ancestor'  
             | 'ancestor-or-self'    
             | 'attribute'   
             | 'child'   
             | 'descendant'  
             | 'descendant-or-self'  
             | 'following'   
             | 'following-sibling'   
             | 'namespace'   
             | 'parent'  
             | 'preceding'   
             | 'preceding-sibling'   
             | 'self'

<NodeTest> ::= <QName>
             | NCName ':' '*'
             | '*'
             | 'node' '(' ')'
             | 'text' '(' ')'
             | 'comment' '(' ')'
             | 'processing-instruction' '(' ')'
             | 'processing-instruction' '(' <Literal> ')'  

<QName> ::= NCName ':' NCName
          | NCName

<FilterExpr> ::= <PrimaryExpr>
               | <FilterExpr> <Predicate>

<PrimaryExpr> ::= <VariableReference>
                | '(' <Expr> ')'  
                | <Literal>
                | Number    
                | <FunctionCall>

<FunctionCall> ::= <QName> '(' <ArgumentList> ')'
                 | <QName> '(' ')'

<ArgumentList> ::= <Expr> ',' <ArgumentList>
                 | <Expr>

<VariableReference> ::= '$' <QName>

<Literal> ::= AposLiteral
            | QuotLiteral
```