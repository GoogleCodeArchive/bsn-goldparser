# When the semantic mapping is not applicable #

The semantic action bindings are the common use case the the `bsn.GoldParser` library. However, if the grammar to be used is not known in advance (for instance loaded from a file at runtime), one may need to use the parser with the "normal" text token and reduction tokens similar to the ones supported by other engines.

# Parsing text #

The code should be quite self-explaining for anyone who has used GOLD engines in the past:
```
// loading the grammar; the grammar is re-usable (even in multithreaded applications)
CompiledGrammar grammar;
using (Stream stream = new FileStream("MyGrammarFile.cgt")) {
	grammar = CompiledGrammar.Load(stream);
}

// parsing some text
Token accepted;
using (TextReader reader = GetTextReader()) {
	Tokenizer tokenizer = new Tokenizer(reader, grammar);
	LalrProcessor processor = new LalrProcessor(tokenizer, true);
	processor.Trim = true; // trim reductions with only one nonterminal
	ParseMessage parseMessage = processor.ParseAll();
	if (parseMessage != ParseMessage.Accept) {
		// you could build a detailed error message here:
		// the position is in processor.CurrentToken.Position
		// and use processor.GetExpectedTokens() on syntax errors
		throw new InvalidOperationException("Parsing failed");
	}
	accepted = processor.CurrentToken;
}
```