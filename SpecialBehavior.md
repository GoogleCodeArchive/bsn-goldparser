# Semantic actions trims #

The GOLD Parser Builder has an option to trim unnecessary reductions away. What this does is basically to "skip" any reduction which itself contains exactly one reduction (note that the the simple, non-semantic tokens in the bsn engine support this mode as well).

The semantic action engine, on the other hand, supports implicit and explicit trims as well as trim overloading.

## Explicit trims ##

As shown in the [calculator sample](SampleCalculator.md), there are situations where you have some insignificant tokens which are needed to define the syntax for the grammar, but which are of no importance in the application later on. In such situations you can specify an explicit trim, which makes the trimmed rule return the same type as the rule handle passed in.

The definition is an assembly-level attribute, which can be scoped to a specific token base class in order to enable multiple grammars in one assembly. The syntax is as follows:
```
[assembly: RuleTrim("<Value> ::= '(' <Expression> ')'", "<Expression>", SemanticTokenType = typeof(CalculatorToken))]
```
The second parameter can be either the (distinct) name of a token handle, or an integer which defines the handle index (0 based); in this case this would be index 1.

## Implicit trims ##

Each rule which accepts exactly one token (terminal or nonterminal is indifferent) is implicitly trimmed by the semantic engine. In the [calculator sample](SampleCalculator.md) those are:
```
<Expression> ::= <Mult Exp> 
<Mult Exp> ::= <Negate Exp> 
<Negate Exp> ::= <Value> 
<Value> ::= Integer
<Value> ::= Float
```

This behavior feels intuitively right, since these rules are often just the "if not this rule, try the next" kind of rules.

## Trim overriding ##

However, in some cases you don't want the engine to trim a rule because you need to map it to a different type. One good example is the [sequence pattern](CodePatterns.md), where you need to add a constructor overriding the implicit trim in order to create the sequence end node.

For instance, the following sample will not work as expected, and the semantic action engine does not even complain about a missing rule mapping due to the implicit trim:
```
Grammar:
<ListA> ::= A <ListA>
          | A

Code:
[Terminal("A")]
public class A: TokenBase {
}

public class SequenceA: Tokenbase, IEnumerable<A> {
	private readonly A item;
	private readonly SequenceA next;

	[Rule("<ListA> ::= A <ListA>")]
	public SequenceA(A item, SequenceA next) {
		this.item = item;
		this.next = next;
	}
/// IEnumerable implementation here
}
```

Since the `<ListA>` contains an implicit trim here, the semantic engine knows that `<ListA>` may return not only a `SequenceA`, but an `A` instance as well, so that it looks for the common base type which is `TokenBase` for the `<ListA>` token. By adding the following constructor in `SequenceA` you will get the expected `SequenceA` class for any `<ListA>` rule:
```
	[Rule("<ListA> ::= A")]
	public SequenceA(A item): this(item, null) {}
```

# Block comment handling #

The builder and most engines (if not all but this one) fail to handle the block comments as expected. This is due to the way the tokenizer is required to handle line comments, which is more or less to skip all tokens until it finds the (matching) block comment end token. Now if a token is found which is valid and "swallows" the block comment end (such as a quoted string containing the end sequence), the reference implementation will read the string token and therefore not notice the block comment end as expected.

The `bsn GoldParser` engine tokenizer will make sure that only DFA states which may lead to a block comment token are taken into account; otherwise, it just resets the DFA state to the initial state. By doing this it should never run into the situation where a token is allowed to "swallow" a block comment token, and therefore the block comment will be handled as usually expected (that is, ignoring everything inside no matter what).