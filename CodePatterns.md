# Introduction #

Some common grammar constructs can be implemented with simple patterns when using the semantic engine of the `bsn.GoldParser` library.

# The sequence pattern #

In many grammars, you have recursive declarations which are representing a sequence of items. Let's create two of the common ones:
```
<OptionList> ::= <Option> <OptionList>
               |

<ArgumentList> ::= <Argument> ',' <ArgumentList>
                 | <Argument>
```
These can be mapped to a class implementing `IEnumerable<>`, which in turn enables it for use with LINQ (and therefore also allows simple conversions to lists or arrays).
```
public class Sequence<T>: TokenBase, IEnumerable<T> where T: TokenBase {
	private readonly T item;
	private readonly Sequence<T> next;

	[Rule("<OptionList> ::=", typeof(Option)]
	public Sequence(): this(null, null) {}

	[Rule("<ArgumentList> ::= <Argument>", typeof(Argument))]
	public Sequence(T item): this(item, null) {}

	[Rule("<OptionList> ::= <Option> <OptionList>", typeof(Option))]
	[Rule("<ArgumentList> ::= <Argument> ~',' <ArgumentList>", typeof(Argument))]
	public Sequence(T item, Sequence<T> next) {
		this.item = item;
		this.next = next;
	}

	public IEnumerator<T> GetEnumerator() {
		for (Sequence<T> sequence = this; sequence != null; sequence = sequence.next) {
			if (sequence.item != null) {
				yield return sequence.item;
			}
		}
	}

	IEnumerator IEnumerable.GetEnumerator() {
		return GetEnumerator();
	}
}
```
It may be surprising that the `Sequence<>` class here is generic. However, the support for generic classes is basically giving us the solution for any sequence grammar pattern.

It is important to map both rules of the list to this class, so that the generated class is always a sequence, even if there's only one element in it. Failing to do so will likely give you error messages about type mismatches during the initialization of the semantic actions.

The sequence could be consumed like this:
```
public class Function: TokenBase {
	[Rule("<Function> ::= Id ~'(' <ArgumentList> ~')'")]
	public Function(Identifier id, Sequence<Argument> args) {
		// do whatever you need here
	}
}
```
For `<OptionList>` sequences, the semantic engine will instantiate a `Sequence<Option>`, and for `<ArgumentList>` we'll get a `Sequence<Argument>`. Just add more rules to the constructors as you need them.

# The optional pattern #

Sometimes it is impractical to define all possible variations of a rule with optional parts, so that we resort to using empty rules in the grammar, like these:
```
<OptionalAlias> ::= AS <Alias>
                  |

<OptionalArgumentList> ::= <ArgumentList>
                         |
```
This can be solved in a similar way as the sequence with a generic class:
```
public class Optional<T>: TokenBase where T: TokenBase {
	private readonly T value;

	[Rule("<OptionalAlias> ::=", typeof(Alias))]
	[Rule("<OptionalArgumentList> ::=", typeof(Sequence<Argument>))]
	public Optional(): this(null) {}

	[Rule("<OptionalAlias> ::= ~AS <Alias>", typeof(Alias))]
	[Rule("<OptionalArgumentList> ::= <ArgumentList>", typeof(Sequence<Argument>))]
	public Optional(T value) {
		this.value = value;
	}

	public T Value {
		get {
			return value;
		}
	}

	public bool HasValue {
		get {
			return value != null;
		}
	}
}
```
These could be consumed like this:
```
public class Function: TokenBase {
	private readonly IEnumerable<Argument> args;
	private readonly Alias alias;

	[Rule("<Function> ::= Id ~'(' <OptionalArgumentList> ~')' <OptionalAlias>")]
	public Function(Identifier id, Optional<Sequence<Argument>> args, Optional<Alias> alias) {
		this.args = args.Value ?? new Argument[];
		this.alias = alias.Value;
	}
}
```