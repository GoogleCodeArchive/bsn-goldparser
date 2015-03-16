
```

! ----------------------------------------------------------------------------
! C# 
!
! The C# Programming Language was created by the Microsoft Corporation to be 
! used with the .NET platform. The goal of C# was to counter the flaws and 
! difficulties found in Java and C++. In addition, the language was designed
! to work well with the Common Language Runtime which runs the .NET platform.
!
! C# was primarily the work of computer scientist Anders Hejlsberg who has 
! worked on a wide range of programming languages. These include, but are 
! not limited to, Visual J++, Borland Delphi and Turbo Pascal. His previous
! works and influences from Java and C++ can be seen in C#.
! 
!
! GRAMMAR NOTES:
!
! The grammar was designed, in part, using the official language 
! specification that can be found on the Microsoft website.
!
! Unfortunately, C# has a very complex grammar. As a result, considerable 
! time was required to write a LALR(1) compliant version. In most cases,
! I preserved the section names used in the original specification, but many
! productions, especially those regarding local variable declarations and 
! expressions, required modifications.
! 
! Feel free to modify and port this grammar to other parsing systems, but 
! please leave this information. Have a great day and happy programming! 
!
! Note: If there are any flaws, please visit www.DevinCook.com/GOLDParser
!
! - Devin Cook
!
!
! Updates:
!     05/09/2006
!         Devin Cook
!         The first version of the grammar was released. 
!
!     04/04/2007
!         Devin Cook
!         Fixed a flaw in the Primary Exp. Thanks to Patrick Kristiansen for
!         the reporting the error.
!
!     09/21/2007
!         Devin Cook
!         Modified the grammar to include the new features in C# 2.0. These include
!         delegate expressions and partial classes.
!
!     11/07/2007
!         Devin Cook
!         Made some additional changes to make the grammar compliant to the C# spec.
!
!     08/18/2010
!         Arsène von Wyss
!         Fixed several issues:
!         * Charset is now unicode
!         * Grammar is now case-sensitive
!         * Identifiers with extended characters are now accepted
!         * Verbatim literal strings are recognized
!         * Compiler directives are now a know token (so that they could be
!           intercepted during tokenization)
!         * Member names are now identifiers separated with a '.'
!         * Changed modifier and access behavior to allow "protected internal" and any order
!         - Generics are not implemented in this grammar!
!
! ----------------------------------------------------------------------------

"Name"     = 'C#'
"Version"  = '2.0'
"Author"   = 'Anders Hejlsberg'           

"About"    = 'C# was created by Anders Hejlsberg for the Microsoft Corporation.'
           | 'The language was designed primarily to both fix flaws found in'
           | 'other languages and to integrate with the the .NET platform.'
           | 'This grammar was written by Devin Cook and fixes were made by'
           | 'Arsène von Wyss'

"Case Sensitive" = 'True'
"Character Mapping" = 'Unicode'

"Start Symbol" = <Compilation Unit>

! ----------------------------------------------------------------- Sets

{ID Head}        = {Letter} + {Letter Extended} + [_]
{ID Tail}        = {AlphaNumeric} + {Letter Extended} + [_]
{String Ch}      = {All Valid} - {Whitespace} - ["\]
{Lit String Ch}  = {All Valid} - ["]
{Char Ch}        = {All Valid} - {Whitespace} - [''\]
{Hex Digit}      = {Number} + [abcdef] + [ABCDEF]
{Directive Ch}   = {All Valid} - {CR} - {LF}

! ----------------------------------------------------------------- Terminals

Identifier     = [@]? {ID Head} {ID Tail}*        !The @ is an override char

DecLiteral     = {Digit}+            ( [UuLl] | [Uu][Ll] | [Ll][Uu] )?
HexLiteral     = '0'[xX]{Hex Digit}+ ( [UuLl] | [Uu][Ll] | [Ll][Uu] )?
RealLiteral    = {Digit}*'.'{Digit}+

StringLiteral  = '"'( {String Ch} | '\'{Printable} )* '"' | '@' ('"' {Lit String Ch}+ '"')+
CharLiteral    = '' ( {Char Ch} | '\'{Printable} )''

Directive = '#' {Directive Ch}*

! ----------------------------------------------------------------- Comments

Comment Line = '//'
Comment Start = '/*'
Comment End = '*/'

! ===========================================================================
! Shared by multiple sections
! ===========================================================================

<Block or Semi>
       ::= <Block>
        |  ';'

<Valid ID>
      ::= Identifier
       |  this
       |  base
       |  <Base Type>

<Qualified ID>
       ::= <Valid ID> <Member List>
     
<Member List>
       ::= <Member List> '.' Identifier
        |  !Zero or more 

<Semicolon Opt>
       ::= ';'
        |  !Nothing 

! ===========================================================================
! C.1.8 Literals 
! ===========================================================================

<Literal>
       ::= true 
        |  false
        |  DecLiteral
        |  HexLiteral
        |  RealLiteral
        |  CharLiteral
        |  StringLiteral
        |  null
     

! ===========================================================================
! C.2.2 Types 
! ===========================================================================

! All date types in C# are objects. A distinction is made between different
! subtypes of objects, though. Some language constructs are restricted to 
! one type or another.

<Type>
        ::= <Non Array Type> 
         |  <Non Array Type> '*'
         |  <Non Array Type> <Rank Specifiers> 
         |  <Non Array Type> <Rank Specifiers> '*'
    
<Pointer Opt> 
        ::= '*'
         |  !Nothing
   
<Non Array Type>
!        ::= <Other Type>
!         |  <Integral Type>
!         |  <Qualified ID>  

        ::=  <Qualified ID>  


! The following defines built-in datatypes only. This is necessary for local
! variable declarations.

<Base Type>
        ::= <Other Type>
         |  <Integral Type>

<Other Type>
        ::= float
         |  double
         |  decimal
         |  bool
         |  void  
         |  object
         |  string
 
! Integral types are valid in enumeration declarations.

<Integral Type>
        ::= sbyte 
         |  byte 
         |  short
         |  ushort
         |  int
         |  uint 
         |  long 
         |  ulong
         |  char
 

! Rank specifiers are used to define the dimensions of arrays. The notation is odd.

<Rank Specifiers Opt>
       ::= <Rank Specifiers Opt> <Rank Specifier>
        |  

<Rank Specifiers>
       ::= <Rank Specifiers> <Rank Specifier>
        |  <Rank Specifier>

<Rank Specifier>
       ::= '[' <Dim Separators> ']'

<Dim Separators> 
       ::= <Dim Separators> ',' 
        |  !Nothing


! ===========================================================================
! C.2.4 Expressions 
! ===========================================================================

<Expression Opt>
       ::= <Expression>
        |  !Nothing 

<Expression List>
       ::= <Expression>
        |  <Expression> ',' <Expression List> 


<Expression>
       ::= <Conditional Exp> '='   <Expression>
        |  <Conditional Exp> '+='  <Expression>
        |  <Conditional Exp> '-='  <Expression>
        |  <Conditional Exp> '*='  <Expression>
        |  <Conditional Exp> '/='  <Expression>
        |  <Conditional Exp> '^='  <Expression>
        |  <Conditional Exp> '&='  <Expression>
        |  <Conditional Exp> '|='  <Expression>
        |  <Conditional Exp> '%='  <Expression>
        |  <Conditional Exp> '<<=' <Expression>
        |  <Conditional Exp> '>>=' <Expression>
        |  <Conditional Exp>

<Conditional Exp>      
       ::= <Or Exp> '?' <Or Exp> ':' <Conditional Exp>
        |  <Or Exp>

<Or Exp>
       ::= <Or Exp> '||' <And Exp>
        |  <And Exp>

<And Exp>
       ::= <And Exp> '&&' <Logical Or Exp>
        |  <Logical Or Exp>

<Logical Or Exp>
       ::= <Logical Or Exp> '|' <Logical Xor Exp>
        |  <Logical Xor Exp>

<Logical Xor Exp>
       ::= <Logical Xor Exp> '^' <Logical And Exp>
        |  <Logical And Exp>

<Logical And Exp>
       ::= <Logical And Exp> '&' <Equality Exp>
        |  <Equality Exp>

<Equality Exp>  
       ::= <Equality Exp> '==' <Compare Exp>
        |  <Equality Exp> '!=' <Compare Exp>
        |  <Compare Exp>

<Compare Exp>
       ::= <Compare Exp> '<'  <Shift Exp>
        |  <Compare Exp> '>'  <Shift Exp>
        |  <Compare Exp> '<=' <Shift Exp>
        |  <Compare Exp> '>=' <Shift Exp>
        |  <Compare Exp> is <Type>
        |  <Compare Exp> as <Type>    
        |  <Shift Exp>

<Shift Exp>
       ::= <Shift Exp> '<<' <Add Exp>
        |  <Shift Exp> '>>' <Add Exp>
        |  <Add Exp>

<Add Exp>
       ::= <Add Exp> '+' <Mult Exp>
        |  <Add Exp> '-' <Mult Exp>
        |  <Mult Exp>

<Mult Exp>
       ::= <Mult Exp> '*' <Unary Exp>  
        |  <Mult Exp> '/' <Unary Exp>  
        |  <Mult Exp> '%' <Unary Exp>  
        |  <Unary Exp>  

<Unary Exp>  
       ::= '!'  <Unary Exp>
        |  '~'  <Unary Exp>
        |  '-'  <Unary Exp>
        |  '++' <Unary Exp>
        |  '--' <Unary Exp>
        |  '(' <Expression> ')' <Object Exp>     !Cast "expression" is required to avoid a conflict
        |  <Object Exp>

! Primary: x.y  f(x)  a[x]  x++  x--  new  typeof  checked  unchecked  ->

<Object Exp>
       ::= delegate  '(' <Formal Param List Opt>  ')' <Block>    !New in 2.0
        |  <Primary Array Creation Exp>
        |  <Method Exp> 
 
<Primary Array Creation Exp>
       ::= new <Non Array Type> '[' <Expression List> ']' <Rank Specifiers Opt> <Array Initializer Opt>
        |  new <Non Array Type> <Rank Specifiers> <Array Initializer>

<Method Exp>
       ::= <Method Exp> <Method>
        |  <Primary Exp>

<Primary Exp>
       ::= typeof    '(' <Type> ')'
        |  sizeof    '(' <Type> ')'
        |  checked   '(' <Expression> ')'
        |  unchecked '(' <Expression> ')'

        |  new <Non Array Type> '(' <Arg List Opt> ')'     !Non array creation
        |  <Primary>
        |  '(' <Expression> ')' 
       
<Primary>
       ::= <Valid ID>
        |  <Valid ID> '(' <Arg List Opt> ')'    !Current object method
        |  <Literal>        

! ===========================================================================
! Arguments
! ===========================================================================

<Arg List Opt>
       ::= <Arg List>
        |  !Nothing
       
<Arg List>
       ::= <Arg List> ',' <Argument>
        |  <Argument>

<Argument>
       ::= <Expression>
        |  ref <Expression>
        |  out <Expression>
            

! ===========================================================================
! C.2.5 Statements 
! ===========================================================================

<Stm List>
       ::= <Stm List> <Statement>
        |  <Statement>


! This repetative productions below resolve the hanging-else problem by 
! restricting the "if-then" statement to remove ambiguity. Two levels of 
! statements are declared with the second, "restricted", group only used in
! the "then" clause of a "if-then-else" statement. 
!
! The "restricted" group is completely identical the the first with one 
! exception: only the "if-then-else" variant of the if statement is allowed. 
! In other words, no "if" statements without "else" clauses can appear inside
! the "then" part of an "if-then-else" statement. Using this solution, the 
! "else" will bind to the last "If" statement, and still allows chaining.

<Statement>
       ::= Identifier ':'                     ! label               
        |  <Local Var Decl> ';'

        |  if       '(' <Expression> ')' <Statement>
        |  if       '(' <Expression> ')' <Then Stm> else <Statement>        
        |  for      '(' <For Init Opt> ';' <For Condition Opt> ';' <For Iterator Opt> ')' <Statement>
        |  foreach  '(' <Type> Identifier in <Expression> ')' <Statement>  
        |  while    '(' <Expression> ')' <Statement>
        |  lock     '(' <Expression> ')' <Statement>
        |  using    '(' <Resource>   ')' <Statement>     
        |  fixed    '('  <Type> <Fixed Ptr Decs> ')' <Statement>    
        |  delegate '(' <Formal Param List Opt>  ')' <Statement>  
        |  <Normal Stm>   


<Then Stm>   
       ::= if       '(' <Expression> ')' <Then Stm> else <Then Stm>        
        |  for      '(' <For Init Opt> ';' <For Condition Opt> ';' <For Iterator Opt> ')' <Then Stm>
        |  foreach  '(' <Type> Identifier in <Expression> ')' <Then Stm>  
        |  while    '(' <Expression> ')' <Then Stm>
        |  lock     '(' <Expression> ')' <Then Stm>
        |  using    '(' <Resource>   ')' <Then Stm>     
        |  fixed    '('  <Type> <Fixed Ptr Decs>   ')' <Then Stm>   
        |  delegate '(' <Formal Param List Opt> ')' <Then Stm>          
        |  <Normal Stm>   
          
          
          
<Normal Stm>                   
       ::= switch '(' <Expression> ')' '{' <Switch Sections Opt> '}'
        |  do <Normal Stm> while '(' <Expression> ')' ';'
        |  try <Block> <Catch Clauses> <Finally Clause Opt>
        |  checked <Block>
        |  unchecked <Block>
        |  unsafe <Block>            
        |  break ';'
        |  continue ';'
        |  goto Identifier ';'
        |  goto case <Expression> ';'
        |  goto default ';'
        |  return <Expression Opt> ';'
        |  throw <Expression Opt> ';'
        |  <Statement Exp> ';'        
        |  ';'
        |  <Block>    

<Block>
       ::= '{' <Stm List> '}'
        |  '{' '}' 
         
<Variable Decs>
        ::= <Variable Declarator>
         |  <Variable Decs> ',' <Variable Declarator>

<Variable Declarator>
        ::= Identifier
         |  Identifier '=' <Variable Initializer>

<Variable Initializer>
        ::= <Expression>
         |  <Array Initializer>
         |  stackalloc <Non Array Type> '[' <Non Array Type> ']'

<Constant Declarators>
        ::= <Constant Declarator>
         |  <Constant Declarators> ',' <Constant Declarator>

<Constant Declarator>
        ::= Identifier '=' <Expression>


! ===========================================================================
! Switch Clauses
! ===========================================================================

<Switch Sections Opt>
        ::= <Switch Sections Opt> <Switch Section>
         |  !Nothing

<Switch Section>
        ::= <Switch Labels> <Stm List>

<Switch Labels>
        ::= <Switch Label>
         |  <Switch Labels> <Switch Label>

<Switch Label>
        ::= case <Expression> ':'
         |  default ':'


! ===========================================================================
! For Clauses
! ===========================================================================

<For Init Opt>
        ::= <Local Var Decl>
         |  <Statement Exp List>
         |  !Nothing

<For Iterator Opt>
        ::= <Statement Exp List>
         |  !Nothing 

<For Condition Opt>
        ::= <Expression>
         |  !Nothing 

<Statement Exp List>
        ::= <Statement Exp List> ',' <Statement Exp>
         |  <Statement Exp>

! ===========================================================================
! Catch Clauses
! ===========================================================================

<Catch Clauses>
        ::= <Catch Clause> <Catch Clauses>
         |  !Nothing

<Catch Clause>
        ::= catch '(' <Qualified ID> Identifier ')' <Block>
         |  catch '(' <Qualified ID>            ')' <Block>
         |  catch <Block>

<Finally Clause Opt>
        ::= finally <Block>
         |  !Nothing

! ===========================================================================
! Using Clauses
! ===========================================================================

<Resource>
        ::= <Local Var Decl>
         |  <Statement Exp>

! ===========================================================================
! Fixed Clauses
! ===========================================================================

<Fixed Ptr Decs>
        ::= <Fixed Ptr Dec>
         |  <Fixed Ptr Decs> ',' <Fixed Ptr Dec>

<Fixed Ptr Dec>
        ::= Identifier '=' <Expression>

! ===========================================================================
! Statement Expressions & Local Variable Declaration
! ===========================================================================

! The complex productions below are able to avoid the shift-reduce error caused
! by declaring an array. The notation used by C# (and the rest of the C++
! family) prevents an array declaration to be distinguished from an array 
! assignment statement until a number of characters are read.
!
! a.b.c[2] = "Test"
! a.b.c[] = new String[3]
!
! The system CANNOT make a decision between the two until it is reading the 
! contents the [ ... ]. 
!
! As a result, the local variable declaration below contains the full notation
! for each of the C# methods at the same level as local variable declarations. 
! Since the system does not have to reduce UNTIL it is within the [ ... ], no
! shift-reduce error will occur. Nasty, huh?

<Local Var Decl>
       ::= <Qualified ID> <Rank Specifiers> <Pointer Opt> <Variable Decs>
        |  <Qualified ID>                   <Pointer Opt> <Variable Decs>    
   !     |  <Base Type>    <Rank Specifiers> <Pointer Opt> <Variable Decs>
   !     |  <Base Type>                      <Pointer Opt> <Variable Decs>     

<Statement Exp>
       ::= <Qualified ID> '(' <Arg List Opt> ')'
        |  <Qualified ID> '(' <Arg List Opt> ')'       <Methods Opt> <Assign Tail>          
        |  <Qualified ID> '[' <Expression List> ']'    <Methods Opt> <Assign Tail>
        |  <Qualified ID> '->' Identifier              <Methods Opt> <Assign Tail>    
        |  <Qualified ID> '++'                         <Methods Opt> <Assign Tail>    
        |  <Qualified ID> '--'                         <Methods Opt> <Assign Tail>    
        |  <Qualified ID>                                            <Assign Tail>

<Assign Tail>
       ::= '++'
        |  '--'       
        |  '='   <Expression>
        |  '+='  <Expression>
        |  '-='  <Expression>
        |  '*='  <Expression>
        |  '/='  <Expression>
        |  '^='  <Expression>
        |  '&='  <Expression>
        |  '|='  <Expression>
        |  '%='  <Expression>
        |  '<<=' <Expression>
        |  '>>=' <Expression>

<Methods Opt>
        ::= <Methods Opt> <Method>
         |  !Null 

<Method>
        ::= '.' Identifier
         |  '.' Identifier '(' <Arg List Opt> ')'    !Invocation
         |  '[' <Expression List> ']' 
         |  '->' Identifier
         |  '++'
         |  '--'     

! ===========================================================================
! C.2.6 Namespaces
! ===========================================================================

<Compilation Unit>
       ::=  <Using List> <Compilation Items>     

<Using List>
       ::= <Using List> <Using Directive>
        |  !Nothing 

<Using Directive>
       ::= using Identifier '=' <Qualified ID> ';'
        |  using <Qualified ID> ';'

<Compilation Items>
       ::= <Compilation Items> <Compilation Item>
        |  ! Zero or more

<Compilation Item>
       ::= <Namespace Dec>
        |  <Namespace Item>   !Default namespace

! ===========================================================================
! Namespace
! ===========================================================================

<Namespace Dec>
       ::= <Attrib Opt> namespace <Qualified ID> '{' <Using List> <Namespace Items> '}' <Semicolon Opt>

<Namespace Items>
       ::= <Namespace Items> <Namespace Item>
        |  ! Zero or more

<Namespace Item>
       ::= <Constant Dec>
        |  <Field Dec>
        |  <Method Dec>
        |  <Property Dec>
        |  <Type Decl>

<Type Decl>
       ::= <Class Decl>
        |  <Struct Decl>
        |  <Interface Decl>
        |  <Enum Decl>
        |  <Delegate Decl>
       
! =================================  Modifiers 

<Header>
       ::= <Attrib Opt> <Modifier List Opt>

<Access Opt>
       ::= <Access> <Access Opt>
        |

<Access>
       ::= private
        |  protected
        |  public
        |  internal      !Friend

<Modifier List Opt>
       ::= <Modifier> <Modifier List Opt>
        |  <Access> <Modifier List Opt>
        |  !Nothing 

<Modifier>
       ::= abstract
        |  extern
        |  new
        |  override
        |  partial 
        |  readonly
        |  sealed
        |  static
        |  unsafe
        |  virtual
        |  volatile

! ===========================================================================
! C.2.7 Classes
! ===========================================================================

<Class Decl>
       ::= <Header> class Identifier <Class Base Opt> '{' <Class Item Decs Opt> '}' <Semicolon Opt>

<Class Base Opt>
       ::= ':' <Class Base List>
        |  !Nothing 

<Class Base List>
       ::= <Class Base List> ',' <Non Array Type>
        |  <Non Array Type>


<Class Item Decs Opt>
       ::= <Class Item Decs Opt> <Class Item>
        |  !Nothing 

<Class Item>
       ::= <Constant Dec>
        |  <Field Dec>
        |  <Method Dec>
        |  <Property Dec>
        |  <Event Dec>
        |  <Indexer Dec>
        |  <Operator Dec>
        |  <Constructor Dec>
        |  <Destructor Dec>
        |  <Type Decl>

<Constant Dec>
       ::= <Header> const <Type> <Constant Declarators> ';'

<Field Dec>
       ::= <Header> <Type> <Variable Decs> ';'

<Method Dec>
       ::= <Header> <Type> <Qualified ID> '(' <Formal Param List Opt> ')' <Block or Semi>
        
<Formal Param List Opt>
       ::= <Formal Param List>
        |  !Nothing 

<Formal Param List>
       ::= <Formal Param>
        |  <Formal Param List> ',' <Formal Param>

<Formal Param>
       ::= <Attrib Opt>        <Type> Identifier  
        |  <Attrib Opt> ref    <Type> Identifier  
        |  <Attrib Opt> out    <Type> Identifier  
        |  <Attrib Opt> params <Type> Identifier       !Parameter array

<Property Dec>
       ::= <Header> <Type> <Qualified ID> '{' <Accessor Dec> '}'
      
<Accessor Dec>
       ::= <Access Opt> get <Block or Semi> 
        |  <Access Opt> get <Block or Semi> <Access Opt> set <Block or Semi> 
        |  <Access Opt> set <Block or Semi> 
        |  <Access Opt> set <Block or Semi> <Access Opt> get <Block or Semi> 

<Event Dec>
       ::= <Header> event <Type> <Variable Decs> ';'
        |  <Header> event <Type> <Qualified ID> '{' <Event Accessor Decs> '}'

<Event Accessor Decs>
       ::= add <Block or Semi> 
        |  add <Block or Semi> remove <Block or Semi> 
        |  remove <Block or Semi> 
        |  remove <Block or Semi> add <Block or Semi> 


!<Indexer Dec>
!       ::= <Header> <Type> <Indexer This> '[' <Formal Param List> ']' '{' <Accessor Dec>'}'
!      
!<Indexer This>
!       ::= this
!        |  <Qualified ID>      !Ending in 'this' - This is a post-parse semantic check

<Indexer Dec>
       ::= <Header> <Type>  <Qualified ID> '[' <Formal Param List> ']' '{' <Accessor Dec>'}'
      
        !Ending in 'this' - This is a post-parse semantic check

! ===========================================================================
! Operator Declarations
! ===========================================================================

<Operator Dec>
       ::= <Header> <Overload Operator Decl>   <Block or Semi>
        |  <Header> <Conversion Operator Decl> <Block or Semi>

<Overload Operator Decl>
       ::= <Type>   operator <Overload Op> '(' <Type> Identifier ')'
        |  <Type>   operator <Overload Op> '(' <Type> Identifier ',' <Type> Identifier ')'
        
<Conversion Operator Decl>
       ::= implicit operator <Type> '(' <Type> Identifier ')'
        |  explicit operator <Type> '(' <Type> Identifier ')'


<Overload Op>
       ::= '+' 
        |  '-'
        |  '!' 
        |  '~' 
        |  '++' 
        |  '--' 
        |  true
        |  false
        |  '*' 
        |  '/' 
        |  '%' 
        |  '&' 
        |  '|' 
        |  '^'
        |  '<<' 
        |  '>>' 
        |  '==' 
        |  '!=' 
        |  '>' 
        |  '<' 
        |  '>=' 
        |  '<='


! ===========================================================================
! Constructor / Destructor Declarations
! ===========================================================================

<Constructor Dec>
       ::= <Header> <Constructor Declarator> <Block or Semi>

<Constructor Declarator>
       ::= Identifier '(' <Formal Param List Opt> ')' <Constructor Init Opt>

<Constructor Init Opt>
       ::= <Constructor Init>
        |  !Nothing 

<Constructor Init>
       ::= ':' base '(' <Arg List Opt> ')'
        |  ':' this '(' <Arg List Opt> ')'


<Destructor Dec>
       ::= <Header> '~' Identifier '(' ')' <Block>


! ===========================================================================
! C.2.8 Structs 
! ===========================================================================

! Note: Structures have the SAME members as normal classes. As a result, the 
!       <Class Item Decs Opt> rule is used


<Struct Decl>
       ::= <Header> struct Identifier <Class Base Opt> '{' <Class Item Decs Opt>'}' <Semicolon Opt>


! ===========================================================================
! C.2.9 Arrays 
! ===========================================================================

<Array Initializer Opt> 
       ::= <Array Initializer>
        |  ! NOTHING

<Array Initializer>
       ::= '{' <Variable Initializer List Opt> '}'
        |  '{' <Variable Initializer List> ',' '}'

<Variable Initializer List Opt>
       ::= <Variable Initializer List>
        |  ! Nothing 

<Variable Initializer List>
       ::= <Variable Initializer>
        |  <Variable Initializer List> ',' <Variable Initializer>


! ===========================================================================
! C.2.10 Interfaces 
! ===========================================================================

<Interface Decl>
       ::= <Header> interface Identifier <Interface Base Opt> '{' <Interface Item Decs Opt> '}' <Semicolon Opt>

<Interface Base Opt>
       ::= ':' <Class Base List>
        |  !Nothing 


<Interface Item Decs Opt>
       ::= <Interface Item Decs Opt> <Interface Item Dec>
        |  !Nothing

<Interface Item Dec>
       ::= <Interface Method Dec>
        |  <Interface Property Dec>
        |  <Interface Event Dec>
        |  <Interface Indexer Dec>

<Interface Method Dec>
       ::= <Attrib Opt> <New Opt> <Type> Identifier '(' <Formal Param List Opt> ')' <Interface Empty Body>
        
<New Opt>
       ::= new
        |  !Nothing 

<Interface Property Dec>
       ::= <Attrib Opt> <New Opt> <Type> Identifier '{' <Interface Accessors> '}'

<Interface Indexer Dec>
       ::= <Attrib Opt> <New Opt> <Type> this '[' <Formal Param List> ']'  '{' <Interface Accessors> '}'

<Interface Accessors>
       ::= <Attrib Opt> <Access Opt> get <Interface Empty Body>
        |  <Attrib Opt> <Access Opt> set <Interface Empty Body>
        |  <Attrib Opt> <Access Opt> get <Interface Empty Body> <Attrib Opt> <Access Opt> set <Interface Empty Body>
        |  <Attrib Opt> <Access Opt> set <Interface Empty Body> <Attrib Opt> <Access Opt> get <Interface Empty Body>

<Interface Event Dec>
       ::= <Attrib Opt> <New Opt> event <Type> Identifier <Interface Empty Body>

<Interface Empty Body>
       ::= ';'
        |  '{' '}'

! ===========================================================================
! C.2.11 Enums
! ===========================================================================

<Enum Decl> ::= <Header> enum Identifier <Enum Base Opt> <Enum Body> <Semicolon Opt>

<Enum Base Opt>
       ::= ':' <Integral Type>
        |  !Nothing 

<Enum Body>
       ::= '{' <Enum Item Decs Opt> '}'
        |  '{' <Enum Item Decs> ',' '}'

<Enum Item Decs Opt>
       ::= <Enum Item Decs>
        |  !Nothing 

<Enum Item Decs>
       ::= <Enum Item Dec>
        |  <Enum Item Decs> ',' <Enum Item Dec>

<Enum Item Dec>
       ::= <Attrib Opt> Identifier
        |  <Attrib Opt> Identifier '=' <Expression>


! ===========================================================================
! C.2.12 Delegates
! ===========================================================================


<Delegate Decl> ::= <Header> delegate <Type> Identifier '(' <Formal Param List Opt> ')' ';'


! ===========================================================================
! C.2.13 Attributes 
! ===========================================================================

<Attrib Opt>
       ::= <Attrib Opt> <Attrib Section>
        |  !Nothing 
        

<Attrib Section>
       ::= '[' <Attrib Target Spec Opt> <Attrib List> ']'
        |  '[' <Attrib Target Spec Opt> <Attrib List> ',' ']'

<Attrib Target Spec Opt>
       ::= assembly ':'
        |  field    ':'
        |  event    ':'
        |  method   ':'
        |  module   ':'
        |  param    ':'
        |  property ':'
        |  return   ':'
        |  type     ':'
        |  ! Nothing

<Attrib List>
       ::= <Attribute>
        |  <Attrib List> ',' <Attribute>

<Attribute>
       ::= <Qualified ID> '(' <Expression List> ')'
        |  <Qualified ID> '(' ')'
        |  <Qualified ID>
```