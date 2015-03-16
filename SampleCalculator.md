# Tutorial introduction #

A simple yet fun way to show how the semantic actions work is to create a complete simple formula calculator application.

# Preparations #

To complete this tutorial, you need the following:
  * VS.NET 2005 or newer, any edition which supports C# console applications
  * A compiled `bsn.GoldParser` library (see compiling section)
  * [GOLD Parser Builder](http://www.devincook.com/goldparser/builder/index.htm), 4.1 would be fine (but older versions work too)
  * An empty C# console application created using VS.NET.

# Baking the grammar #

The grammar is easy to do, since the GOLD Parser Builder can create some of the most common constructs for new grammars by using a wizard:
  * Open GOLD Parser Builder
  * Start the "New grammar wizard"
  * Choose "No - new line characters are just whitespace" on the first page
  * Choose "Do not create identifiers" on the next page
  * Choose "Do not create string declarations" on the next page
  * Finally, choose "Create basic mathematical expressions" on the last page
  * Complete the wizard

You're now in the grammar editor, where you can type in the info about the grammar. However, more important is to change the start symbol declaration to read to `<Expression>` and to add two terminal definitions for numbers, which you use to replace the `Identifier` used in the `<Value>` rule.

You should get something like this:
```
"Name"     = 'Calculator Sample Grammar'
"Author"   = 'Arsène von Wyss'
"Version"  = '0.1'
"About"    = 'Sample grammar for simple calculation expressions'

"Start Symbol" = <Expression>

Integer = {Digit}+
Float = {Digit}*'.'{Digit}+([Ee][+-]?{Digit}+)?

<Expression>  ::= <Expression> '+' <Mult Exp> 
               |  <Expression> '-' <Mult Exp> 
               |  <Mult Exp> 

<Mult Exp>    ::= <Mult Exp> '*' <Negate Exp> 
               |  <Mult Exp> '/' <Negate Exp> 
               |  <Negate Exp> 

<Negate Exp>  ::= '-' <Value> 
               |  <Value> 

<Value>       ::= Integer
               |  Float
               |  '(' <Expression> ')'
```

You can now save the grammar file (I saved mine as "Calculator.grm") into the folder of the VS project. Hit the "Next ->" button until you can save the compiled grammar file, and save this file ("Calculator.cgt") in the project folder as well.

If you like you can now open the test window and try out your grammar. For more info on the GOLD Parser Builder, visit Devin's page.

# Embedding the grammar #

Everything else now takes place in VS.NET. Open the project and make sure to reference the `bsn.GoldParser` library.

Choose to show the hidden files in the project (the "Show All Files" toolbar button in the Solution Explorer) and include both files we created in the previous step. Click on the compiled file ("Calculator.cgt") and change the Build Action in the Properties to "Embedded Resource".

We'll create the semantic classes in the next step. Right now, let's just define a class which will serve as root for all our semantic classes used in the parsing process. This base class, which must inherit from `SemanticToken`, is a requirement by the semantic type action engine. The engine takes the given class and looks for the grammar definition attributes in all descendant classes inside the same assembly. This allows you to host multiple different grammars in one assembly.

Let's call this base class `CalculatorToken` and define it like this:
```
public class CalculatorToken: SemanticToken {}
```

Also, let's fill out the `Program.cs` file with the main "user interface" code, a simple command line interface reading lines from the keyboard:
```
internal class Program {
	private static void Main(string[] args) {
		Console.WriteLine("*** CALCULATOR SAMPLE *** (input formula, empty line terminates)");
		CompiledGrammar grammar = CompiledGrammar.Load(typeof(CalculatorToken), "Calculator.cgt");
		SemanticTypeActions<CalculatorToken> actions = new SemanticTypeActions<CalculatorToken>(grammar);
		try {
			actions.Initialize(true);
		} catch (InvalidOperationException ex) {
			Console.Write(ex.Message);
			Console.ReadKey(true);
			return;
		}
		for (string formula = Console.ReadLine(); !string.IsNullOrEmpty(formula); formula = Console.ReadLine()) {
			SemanticProcessor<CalculatorToken> processor = new SemanticProcessor<CalculatorToken>(new StringReader(formula), actions);
			ParseMessage parseMessage = processor.ParseAll();
			if (parseMessage == ParseMessage.Accept) {
				Console.WriteLine(string.Format(NumberFormatInfo.InvariantInfo, "Result: {0}", ((Computable)processor.CurrentToken).GetValue()));
			} else {
				IToken token = processor.CurrentToken;
				Console.WriteLine(string.Format("{0} {1}", "^".PadLeft(token.Position.Column), parseMessage));
			}
		}
	}
}
```
The following should be noted here:
  * The grammar can directly be loaded from the embedded resource. This is very handy because it removes dependencies on external files.
  * The `SemanticTypeActions<>` generic class gathers the bindings of the grammar to the classes in the project. While it is not necessary to explicitly initialize it, doing so can help you during development because it does perform a complete check of all bindings to ensure error-free parsing if the initialization is successful. Since initialization is pretty expensive, you should only initialize it once and re-use it; it is even thread-safe.
  * The parsing itself is done by the `SemanticProcessor<>`, which can also give you information about anything which went wrong while parsing.

If you run the application now (to successfully compile, you need to first create the `Computable` file shown below), you should get the following output on the console:
```
*** CALCULATOR SAMPLE *** (input formula, empty line terminates)
The semantic engine found errors:
Semantic token is missing for terminal (EOF)
Semantic token is missing for terminal (Error)
Semantic token is missing for terminal (Whitespace)
Semantic token is missing for terminal '-'
Semantic token is missing for terminal '('
Semantic token is missing for terminal ')'
Semantic token is missing for terminal '*'
Semantic token is missing for terminal '/'
Semantic token is missing for terminal '+'
Semantic token is missing for terminal Float
Semantic token is missing for terminal Integer
Semantic token is missing for rule <Expression> ::= <Expression> '+' <Mult Exp>
Semantic token is missing for rule <Expression> ::= <Expression> '-' <Mult Exp>
Semantic token is missing for rule <Mult Exp> ::= <Mult Exp> '*' <Negate Exp>
Semantic token is missing for rule <Mult Exp> ::= <Mult Exp> '/' <Negate Exp>
Semantic token is missing for rule <Negate Exp> ::= '-' <Value>
Semantic token is missing for rule <Value> ::= '(' <Expression> ')'
```
Basically, the engine tells you what is missing or not matching in your code in order to successfully map the grammar to semantic classes. So let's start and add the semantic classes and the required declarations.

# Creating and binding the semantic classes #

Note that the engine needs to be able to create an instance for all defined terminals and rules, so that you also have to assign the special terminals such as `(EOF)` or the ones for comments (if comments are being used) here. So first we assign the "dummy" terminals to some token. Using the root class seems to be a good fit here:
```
[Terminal("(EOF)")]
[Terminal("(Error)")]
[Terminal("(Whitespace)")]
[Terminal("(")]
[Terminal(")")]
public class CalculatorToken: SemanticToken {}
```

Good. What we need now for the calculator is to have some classes which can be used to compute and get a value, and some which know how to perform mathematical operations. Le's start with the latter and create an abstract base class for the operators:
```
public abstract class Operator: CalculatorToken {
	public abstract double Calculate(double left, double right);
}
```

We don't need metadata here, but we need some on the four math operator classes deriving from it:
```
[Terminal("+")]
public class PlusOperator: Operator {
	public override double Calculate(double left, double right) {
		return left+right;
	}
}

[Terminal("-")]
public class MinusOperator: Operator {
	public override double Calculate(double left, double right) {
		return left-right;
	}
}

[Terminal("*")]
public class MultiplyOperator: Operator {
	public override double Calculate(double left, double right) {
		return left*right;
	}
}

[Terminal("/")]
public class DivideOperator: Operator {
	public override double Calculate(double left, double right) {
		return left/right;
	}
}
```
So whenever there is an `Operator` instance, we can have it compute the result of its operation. Whet we now need is another base class which represents everything which can compute a value:
```
public abstract class Computable: CalculatorToken {
	public abstract double GetValue();
}
```
Let's start by implementing the numbers, which are parsed as `Integer` or `Float`. Note that whenever a constructor accepting exactly one string is defined, it will be used during object creation and receive the matched text of the terminal. We use this to get the number to be parsed into a numeric representation:
```
[Terminal("Integer")]
[Terminal("Float")]
public class Number: Computable {
	private readonly double value;

	public Number(string value) {
		this.value = Double.Parse(value, NumberFormatInfo.InvariantInfo);
	}

	public override double GetValue() {
		return value;
	}
}
```
By now, all the terminals should have been properly defined and mapped to classes. The next thing will be the class which applies operations to two values. Since rules can have an arbitrary number of "handles", the rule attributes need to be defined on the matching constructor:
```
public class Operation: Computable {
	private readonly Computable left;
	private readonly Operator op;
	private readonly Computable right;

	[Rule(@"<Expression> ::= <Expression> '+' <Mult Exp>")]
	[Rule(@"<Expression> ::= <Expression> '-' <Mult Exp>")]
	[Rule(@"<Mult Exp> ::= <Mult Exp> '*' <Negate Exp>")]
	[Rule(@"<Mult Exp> ::= <Mult Exp> '/' <Negate Exp>")]
	public Operation(Computable left, Operator op, Computable right) {
		this.left = left;
		this.op = op;
		this.right = right;
	}

	public override double GetValue() {
		return op.Calculate(left.GetValue(), right.GetValue());
	}
}
```
There is no limit to the combinations used; you can have multiple constructors where each has different rules assigned to it, and you can even have the same class be both a terminal as well as a "reduction" token. Sometimes it is, however, necessary to apply a custom mapping between the rule handles and the constructor arguments. This is even the case in our simple application, but it's very easy to do, as we can see with the last class we're going to write:
```
public class Negate: Computable {
	private readonly Computable computable;

	[Rule("<Negate Exp>  ::= ~'-' <Value>")]
	public Negate(Computable computable) {
		this.computable = computable;
	}

	public override double GetValue() {
		return -computable.GetValue();
	}
}
```
So let's go ahead and try our application. Oops! There is still something missing:
```
*** CALCULATOR SAMPLE *** (input formula, empty line terminates)
The semantic engine found errors:
Semantic token is missing for rule <Value> ::= '(' <Expression> ')'
```
Since it is clear that we actually do not want to perform any computation here, but rather just pass through the `<Expression>`, we have a situation where we need a trim operation. the trim is nothing but a pass-through for exactly one handle token in the rule. Those can be defined as assembly attributes, like so (make sure to put it outside of a namespace declaration):
```
[assembly: RuleTrim("<Value> ::= '(' <Expression> ')'", "<Expression>", SemanticTokenType = typeof(CalculatorToken))]
```
Okay, that's it. You've got yourself a simple formula evaluator! Every bit of code required is on this page - the declarative binding makes the code very clean and clutter-free.

And this concludes the tutorial. Suggestions if you want to extend this sample:
  * Introduce other operators such as the modulo or the power operator
  * Introduce functions
  * Introduce variables and assignment statements