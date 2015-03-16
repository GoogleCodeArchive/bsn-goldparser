
```
"Name"     = 'MOF'
"Author"   = 'Arsène von Wyss'
"Version"  = '1'
"About"    = 'CIM Managed Object Format'

"Start Symbol" = <mofSpecification>

"Case Sensitive" = 'False'
"Character Mapping" = 'None'
                    
Comment Start = '/*'
Comment End   = '*/'
Comment Line  = '//'
              
! -------------------------------------------------
! Character Sets
! -------------------------------------------------

{StringChar} = {All Valid} - ["\]
{CharChar} = {Printable} + {HT} - [''\]
{FirstIdentifierChar} = {Letter} + [_]
{NextIdentiferChar} = {FirstIdentifierChar} + {Number}
{NonZeroNumber} = {Number} - [0]
{OctalNumber} = [01234567]
{HexNumber} = [0123456789ABCDEF]

! -------------------------------------------------
! Terminals
! -------------------------------------------------

StringValue = ('"' ( {StringChar} | '\' {Printable} )* '"')+
              
PositiveDecimalValue = '+'? ({NonZeroNumber} {Number}* | '0')
                     
NegativeDecimalValue = '-' ({NonZeroNumber} {Number}* | '0')

OctalValue = [+-]? '0' {Number}+

BinaryValue = [+-]? [01]+ 'B'
            
HexValue = [+-]? '0x' {HexNumber}+

RealValue = [+-]? {Number}* '.' {Number}+ ('e' [+-]? {Number}+)?

CharValue = '' ( {CharChar} | '\' {Printable} ) ''

BooleanValue = 'TRUE' | 'FALSE'

NullValue = 'NULL'

Identifier = {FirstIdentifierChar} {NextIdentiferChar}*
           
AliasIdentifier = '$' {FirstIdentifierChar} {NextIdentiferChar}*
                
QualifiedIdentifier @= { Source = Virtual }

NamespaceHandle @= { Source = Virtual }

DT_BOOL = 'boolean'

DT_CHAR16 = 'char16'

DT_DATETIME = 'datetime'

DT_REAL32 = 'real32'

DT_REAL64 = 'real64'

DT_SINT16 = 'sint16'

DT_SINT32 = 'sint32'

DT_SINT64 = 'sint64'

DT_SINT8 = 'sint8'

DT_STR = 'string'

DT_UINT16 = 'uint16'

DT_UINT32 = 'uint32'

DT_UINT64 = 'uint64'

DT_UINT8 = 'uint8'

PRAGMA = '#pragma'

! -------------------------------------------------
! Rules
! -------------------------------------------------

<mofSpecification> ::= <mofProduction> <mofSpecification>
                    |
           
<mofProduction> ::= <compilerDirective>
                 |  <classDeclaration>
                 |  <assocDeclaration>
                 |  <indicDeclaration>
                 |  <qualifierDeclaration>
                 |  <instanceDeclaration>

<compilerDirective> ::= PRAGMA Identifier '(' StringValue ')'

<classDeclaration> ::= <qualifierList> CLASS Identifier AS AliasIdentifier ':' Identifier '{' <classFeatureList> '}' ';'
                    |  <qualifierList> CLASS Identifier ':' Identifier '{' <classFeatureList> '}' ';'
                    |  <qualifierList> CLASS Identifier AS AliasIdentifier '{' <classFeatureList> '}' ';'
                    |  <qualifierList> CLASS Identifier '{' <classFeatureList> '}' ';'

<assocDeclaration> ::= '[' ASSOCIATION <additionalQualifier> ']' CLASS Identifier AS AliasIdentifier ':' Identifier '{' <associationFeatureList> '}' ';'
                    |  '[' ASSOCIATION <additionalQualifier> ']' CLASS Identifier ':' Identifier '{' <associationFeatureList> '}' ';'
                    |  '[' ASSOCIATION <additionalQualifier> ']' CLASS Identifier AS AliasIdentifier '{' <associationFeatureList> '}' ';'
                    |  '[' ASSOCIATION <additionalQualifier> ']' CLASS Identifier '{' <associationFeatureList> '}' ';'
                    
<indicDeclaration> ::= '[' INDICATION <additionalQualifier> ']' CLASS Identifier AS AliasIdentifier ':' Identifier '{' <classFeatureList> '}' ';'
                    |  '[' INDICATION <additionalQualifier> ']' CLASS Identifier AS AliasIdentifier '{' <classFeatureList> '}' ';'
                    |  '[' INDICATION <additionalQualifier> ']' CLASS Identifier ':' Identifier '{' <classFeatureList> '}' ';'
                    |  '[' INDICATION <additionalQualifier> ']' CLASS Identifier '{' <classFeatureList> '}' ';'

<classFeature> ::= <propertyDeclaration>
                |  <methodDeclaration>
                
<classFeatureList> ::= <classFeature> <classFeatureList>
                    |

<associationFeature> ::= <classFeature>
                      |  <referenceDeclaration>

<associationFeatureList> ::= <associationFeature> <associationFeatureList>
                          |

<qualifierList> ::= '[' <qualifier> <additionalQualifier> ']'
                 |
                 
<qualifier> ::= Identifier <qualifierParameter> ':' <flavor>
             |  Identifier <qualifierParameter>
             
<additionalQualifier> ::= ',' <qualifier> <additionalQualifier>
                       |
                
<qualifierParameter> ::= '(' <constantValue> ')'
                      |  <arrayInitializer>
                      |

<flavor> ::= ENABLEOVERRIDE 
          |  DISABLEOVERRIDE 
          |  RESTRICTED 
          |  TOSUBCLASS 
          |  TRANSLATABLE
          
<identifierOptionalArray> ::= Identifier '[' PositiveDecimalValue ']'
                           |  Identifier '[' ']'
                           |  Identifier

<propertyDeclaration> ::= <qualifierList> <dataType> <identifierOptionalArray> <defaultValue> ';'

<referenceDeclaration> ::= <qualifierList> <objectRef> Identifier <defaultValue> ';'

<methodDeclaration> ::= <qualifierList> <dataType> Identifier '(' <parameterList> ')' ';'

<dataType> ::= DT_UINT8
            |  DT_SINT8
            |  DT_UINT16
            |  DT_SINT16
            |  DT_UINT32
            |  DT_SINT32
            |  DT_UINT64
            |  DT_SINT64
            |  DT_REAL32
            |  DT_REAL64
            |  DT_CHAR16
            |  DT_STR
            |  DT_BOOL
            |  DT_DATETIME
            
<dataTypeOptionalArray> ::= <dataType> '[' PositiveDecimalValue ']'
                         |  <dataType> '[' ']'
                         |  <dataType>

<objectRef> ::= Identifier REF

<parameterList> ::= <parameter> <additionalParameter>
                 |

<additionalParameter> ::= ',' <parameter> <additionalParameter>
               |

<parameter> ::= <qualifierList> <dataType> <identifierOptionalArray>
             |  <qualifierList> <objectRef> <identifierOptionalArray>

<initializer> ::= <constantValue>
               |  <arrayInitializer>
               |  <objectHandle>
               |  AliasIdentifier
                                       
<defaultValue> ::= '=' <initializer>
                |

<arrayInitializer> ::= '{' <constantValueList> '}'
                    
<constantValueList> ::= <constantValue> <additionalConstantValue>
                     |
                     
<additionalConstantValue> ::= ',' <constantValue> <additionalConstantValue>
                  |

<constantValue> ::= PositiveDecimalValue
                 |  NegativeDecimalValue
                 |  BinaryValue
                 |  OctalValue
                 |  HexValue
                 |  RealValue
                 |  CharValue
                 |  StringValue
                 |  BooleanValue
                 |  NullValue

<objectHandle> ::= '"' NamespaceHandle ':' <modelPath> '"'
                |  '"' <modelPath> '"'

<modelPath> ::= Identifier '.' <keyValuePairList>

<keyValuePairList> ::= <keyValuePair> ',' <keyValuePairList>
                    |  <keyValuePair>

<keyValuePair> ::= Identifier '=' <initializer>

<qualifierDeclaration> ::= QUALIFIER Identifier ':' <dataTypeOptionalArray> <defaultValue> ',' SCOPE '(' <metaElementList> ')' <defaultFlavor> ';'

<metaElementList> ::= <metaElement> ',' <metaElementList>
                   |  <metaElement>

<metaElement> ::= SCHEMA
               |  CLASS 
               |  ASSOCIATION 
               |  INDICATION
               |  QUALIFIER
               |  PROPERTY 
               |  REFERENCE 
               |  METHOD 
               |  PARAMETER 
               |  ANY

<defaultFlavor> ::= ',' FLAVOR '(' <flavorList> ')'
                 |
                   
<flavorList> ::= <flavor> ',' <flavorList>
              |  <flavor>

<instanceDeclaration> ::= <qualifierList> INSTANCE OF Identifier AS AliasIdentifier '{' <valueInitializerList> '}' ';'
                       |  <qualifierList> INSTANCE OF Identifier '{' <valueInitializerList> '}' ';'

<valueInitializer> ::= <qualifierList> Identifier '=' <initializer> ';'

<valueInitializerList> ::= <valueInitializer> <valueInitializerList>
                        |  <valueInitializer>
```