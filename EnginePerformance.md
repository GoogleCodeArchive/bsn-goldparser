# Tokenizer #

The tokenizer doesn't use much special functionality, except for a flexible character buffer which was designed to make the read-ahead required in some cases work correctly. Also, the tokenizer implements a [special block comment handling](SpecialBehavior.md).

# LALR processor #

The different LALR actions were implemented as different classes based on an abstract base class, so that invoking the correct actions is a matter of a virtual call, which is usually pretty fast in the .NET framework.

# Semantic actions engine #

Both the `SemanticTokenizer` (which is a private class inside the `SemanticActions<>`) as well as the `SemanticProcessor<>` are the stateful classes using the information in the stateless `SemanticActions<>`, which is basically a token factory repository. Therefore one `SemanticActions<>` instance may be used by any number of threads simultaneously.

The concrete implementation of this class, which gets its information from the attributes, is the `SemanticTypeActions<>`. Each terminal token and each rule have their own factories. Those factories contain a delegate to a dynamically created method (generated as IL in a `DynamicMethod`) which invokes the correct constructor with the correct arguments.

Therefore, after the first warm-up where the CLR JITs the dynamic methods, the code runs very fast with no reflection involved at all during parse time. In fact, compared to creating a "normal" token or reduction, there is not much more than an additional virtual call involved with some simple parameter mapping.