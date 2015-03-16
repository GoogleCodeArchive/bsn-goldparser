# Isn't saving the grammar table as EGT all that's needed? #

Unfortunately, this is usually not enough, because of the changes in the group handling introduced in the GOLD Builder 5. There are a few of the consequences:
  * The builder now automatically defines a newline whitespace terminal, which must be added to your semantic binding if you're using this functionality of the engine. Note that it is a special terminal, and as such the name is in braces: `(NewLine)`
  * The tokenizer no longer generates uses the `ParseMessage.CommentLineRead` value, even if a line comment was parsed. This is now always a `BlockRead`.
  * Generated tokens for comment blocks and line comments no longer use the `CommentStart` or `CommentLine` symbol kind. The start and end tokens are now being replaced by the container token (see the [GOLD documentation on the topic](http://goldparser.org/doc/grammars/index.htm)), which by default is just noise (this can, however, be configured by using the new [token attributes](http://goldparser.org/doc/grammars/index.htm)). So if your code relies on these symbol kinds in the processing chain, you'll have to replace this with another type of check.
  * The block start and end tokens are also special tokens. As such, they need to be written in braces when binding them to a terminal: `(--)`, `(/*)` or `(*/)` for the common line and block comment tokens.
  * On the other hand, the old tokens are no longer applicable, and you can remove them: `(Comment Line)`, `(Comment Start)` and `(Comment End)`

# New name, old thing #

Several enum values have received a new name, and the old names have been marked as obsolete. This will make sure that the developer using the engine will get warnings in the spots where "legacy" CGT features are being used.

The caveat is that if you're indeed using a CGT file, then those warnings will still appear since the .NET compiler (C# or VB.NET etc.) cannot know whether these obsolete warnings are applicable or not. In this case, just ignore them. Otherwise, make sure to make the necessary changes (which usually will be just using the new name, with the exception of the line and block comment detection mentionned above).

| **Obsolete member** | **Replaced by** | **Remarks** |
|:--------------------|:----------------|:------------|
| `CompiledGrammar.CaseSensitive` | N/A | Apparently, this information is no longer contained in the compiled grammar file (EGT) |
| `ParseMessage.CommentBlockRead` | `ParseMessage.BlockRead` | Same value, just a new name |
| `ParseMessage.CommentLineRead` | N/A | Line comments in EGT files are now just normal blocks |
| `ParseMessage.CommentError` | `ParseMessage.BlockError` | Same value, just a new name |
| `SymbolKind.CommentEnd` | `SymbolKind.BlockEnd` | Same value, just a new name |
| `SymbolKind.CommentLine` | N/A | This is no longer assigned to symbols in EGT files, line comments are now also just blocks |
| `SymbolKind.CommentStart` | `SymbolKind.BlockStart` | Same value, just a new name |
| `SymbolKind.WhiteSpace` | `SymbolKind.Noise` | Same value, just a new name |

# Non-consuming blocks and the "end of file" #

When an "open" block is defined (that is, it doesn't consume the end token, such as a newline in a line comment), then the engine will accept an EOF as termination of the block as well. This makes sure that files ending with a line comment that is not followed by a newline parse fine, and in general I expect this to be the expected behavior since the end token is not really meaningful for the block content anyways.