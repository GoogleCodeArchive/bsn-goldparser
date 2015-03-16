# Introduction #

In order to better illustrate the BSN engine, I've created a basic, Read-Evaluate-Print-Loop interpreter for Devin Cook's Simple 2 grammar.  [(Grammar)](http://www.devincook.com/goldparser/grammars/files/example-simple.zip)

# What's Missing #

**The Optional Pattern**

The optional pattern, as explained on the [CodePatterns](http://code.google.com/p/bsn-goldparser/wiki/CodePatterns) page, is not required by the Simple 2 grammar. If I were to add support for Function calls, this pattern would be necessary to support the fact that some calls would require parameters and some wouldn't.

# What's not missing #

**Execution Context**

The execution context is the part of the code that ties the environment and 'memory space' to the code to be run.  In this case, it's very simple (`SimpleExecutionContext`) as it only contains a reference to the input and output streams and a block of global variables implemented as a String-Object dictionary.

**Boolean Expressions**

Boolean expressions (those which return a true/false value) are used to compare values as well as in the 'sentinal test' expression of looping constructs (while loops/if-then-else blocks.)

**Type Checking**

I have a rudimentary type-checker that tries to find the 'greatest common type' between two operands of a binary operation and converts them appropriately for the execution of the operation.  Interestingly enough, it does not convert variable values 'permanently,' but rather just for the evaluation of operation itself.  You can add a variable whose value is a string containing numeric characters to an integer, and the next time you read the value of the variable, it will still be a string.  I'm not sure what the 'industry standard' is with respect to this behavior, but that's how mine works.

**REPL-i-fier**

I made a class called the `REPLcator` which does not very much except wrap a context and accept a series of command line entries, pretty much forever until you CTRL+C to exit.

```
class REPLCator {
       

        public void Run(){

            // grab an execution context to be used for the entire 'session'
            using (var ctx = new SimpleExecutionContext()){

                Banner(ctx);

                // compile/run each statement one entry at a time
                CompiledGrammar grammar = CompiledGrammar.Load(typeof (SimpleToken), "Simple2.cgt");
                var actions = new SemanticTypeActions<SimpleToken>(grammar);

                for (string input = ReadABit(ctx); !string.IsNullOrEmpty(input); input = ReadABit(ctx)) {
                    var processor = new SemanticProcessor<SimpleToken>(new StringReader(input), actions);
                   
                    ParseMessage parseMessage = processor.ParseAll();
                    if (parseMessage == ParseMessage.Accept){
                        
                        ctx.OutputStream.WriteLine("Ok.\n");

                        var stmts = processor.CurrentToken as Sequence<Statement>;
                        if (stmts != null){
                            foreach (Statement stmt in stmts){
                                stmt.Execute(ctx);
                            }
                        }
                    }
                    else{

                        
                        IToken token = processor.CurrentToken;
                        ctx.OutputStream.WriteLine("At index: {0} [{1}]", token.Position.Index, parseMessage );
                        //Console.WriteLine(string.Format("{0} {1}", "^".PadLeft(token.Position.Index + 1), parseMessage));
                    }
                }


                

            }
        }

        // credits
        private void Banner(SimpleExecutionContext ctx) {
            ctx.OutputStream.WriteLine("*** SIMPLE2 REPL *** ");
            ctx.OutputStream.WriteLine(" by Dave Dolan on August 25, 2010.");
            ctx.OutputStream.WriteLine("\n -- Enter a blank line to 'go' or Ctrl+C to exit\n\n");
        }

        // reads until a blank line is entered
        private string ReadABit(SimpleExecutionContext ctx){

            ctx.OutputStream.Write("> ");

            string inputLine = null;

            StringBuilder sb = new StringBuilder();
            
            while (string.Empty != inputLine){
               inputLine = ctx.InputStream.ReadLine();
               if (!string.IsNullOrEmpty(inputLine)){
                   sb.AppendLine(inputLine);
               }
            }

            return sb.ToString();
        }
    }

```

# Sample Run #


```

*** SIMPLE2 REPL ***
 by Dave Dolan on August 25, 2010.

 -- Enter a blank line to 'go' or Ctrl+C to exit


> assign x = 12

Ok.

> display x

Ok.

12
> display "The number is: " & X

Ok.

The number is: 12
> assign x = x + 2

Ok.

> display x

Ok.

14
> while x > 1 do
    display x
    assign x = x - 1
  end

Ok.

14
13
12
11
10
9
8
7
6
5
4
3
2
>

```

Coming soon will be some more detailed explanation, but for now, here's the source for the interpreter (done in one file, like a play in one act.)

```


/* 
 * Program: Simple 2 REPL Interpreter 
 * Grammar: Gold Parser Example Grammar "Simple 2" by Devin Cook
 * Author: Dave Dolan (based on Arsene von Wyss's Calculator example and wiki entry tutorials)
 * Date: 2010-08-29
 * Author Interjection: I have no idea what the "&" operator is supposed to do based on the grammar, 
 * so I opted for string concatenation for fun and games. 
 * 
 * Please don't complain that it's all in one file. Get ReSharper and split it up if you want! 
 * 
 * Also worthy of notice is that because of the BSN engine approach, this originally took me only 90 minutes to hack up.
 * I spent a little time after that cleaning it up for the kids at home. Still not all that clean, but it's better.
 * 
 * See http://code.google.com/p/bsn-goldparser for more information.
 * 
 * This is a Gold Parser Builder based project. Check out: http://goldparser.com 
 * 
 * I just added a type checker. It's not very thorough as far as your average type checker goes, but it works for this grammar.
 */

using System;
using System.Collections;
using System.Collections.Generic;
using System.IO;
using bsn.GoldParser.Grammar;
using bsn.GoldParser.Semantic;
using Simple2REPL;

[assembly: RuleTrim("<Value> ::= '(' <Expression> ')'", "<Expression>", SemanticTokenType = typeof (SimpleToken))]

namespace Simple2REPL{

    [Terminal("(EOF)")]
    [Terminal("(Error)")]
    [Terminal("(Whitespace)")]
    [Terminal("(")]
    [Terminal(")")]
    [Terminal("=")]
    public class SimpleToken : SemanticToken{
    }

    [Terminal("assign")]
    [Terminal("display")]
    [Terminal("do")]
    [Terminal("else")]
    [Terminal("end")]
    [Terminal("if")]
    [Terminal("read")]
    [Terminal("then")]
    [Terminal("while")]
    public class KeywordToken : SimpleToken{
    }

    public class SimpleExecutionContext : IDisposable{
        // we don't want to dispose of the console streams
        private readonly bool DoDisposeStreams;

        private Dictionary<string, object> GlobalVariables = new Dictionary<string, object>();
        public TextReader InputStream;
        public TextWriter OutputStream;

        public SimpleExecutionContext(){
            InputStream = Console.In;
            OutputStream = Console.Out;
        }

        public SimpleExecutionContext(Stream input, Stream output){
            InputStream = new StreamReader(input);
            OutputStream = new StreamWriter(output);
            DoDisposeStreams = true;
        }

        #region IDisposable Members

        public void Dispose(){
            if (DoDisposeStreams){
                OutputStream.Dispose();
                InputStream.Dispose();
            }
        }

        #endregion

        public object this[string idx]{
            get{

                var uppperValue = idx.ToUpper();
                if (GlobalVariables.ContainsKey(uppperValue)){
                    return GlobalVariables[uppperValue];
                }
                throw new UndefinedVariableException(uppperValue);
            }

            set{
                var upperValue = idx.ToUpper();
                GlobalVariables[upperValue] = value;
            }
        }
    }

    public abstract class Statement : SimpleToken{
        public abstract void Execute(SimpleExecutionContext ctx);
    }

    public abstract class Expression : SimpleToken{
        public abstract object GetValue(SimpleExecutionContext ctx);
    }

    public abstract class BinaryOperator : SimpleToken{
        public abstract object Evaluate(object left, object right);
    }


    public class Negate : Expression{
        private readonly Expression computable;

        [Rule("<Negate Exp>  ::= ~'-' <Value>")]
        public Negate(Expression computable){
            this.computable = computable;
        }

        public override object GetValue(SimpleExecutionContext ctx){
            return -(Convert.ToDecimal(computable.GetValue(ctx)));
        }
    }

    [Terminal("+")]
    public class PlusOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return Convert.ToDecimal(left) + Convert.ToDecimal(right);
        }
    }

    [Terminal("-")]
    public class MinusOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return Convert.ToDecimal(left) - Convert.ToDecimal(right);
        }
    }

    [Terminal("*")]
    public class MultOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return Convert.ToDecimal(left)*Convert.ToDecimal(right);
        }
    }

    [Terminal("&")]
    public class AndOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return string.Concat(Convert.ToString(left), Convert.ToString(right));
        }
    }

    [Terminal("/")]
    public class DivideOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return Convert.ToDecimal(left)/Convert.ToDecimal(right);
        }
    }

    [Terminal("==")]
    public class EqualEqualOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return ((IComparable) left).CompareTo(right) == 0;
        }
    }

    [Terminal("<>")]
    public class NotEqualOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return ((IComparable) left).CompareTo(right) != 0;
        }
    }


    [Terminal("<")]
    public class LTOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return ((IComparable) left).CompareTo(right) < 0;
        }
    }

    [Terminal("<=")]
    public class LTEOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return ((IComparable) left).CompareTo(right) <= 0;
        }
    }

    [Terminal(">")]
    public class GTOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return ((IComparable) left).CompareTo(right) > 0;
        }
    }

    [Terminal(">=")]
    public class GTEOperator : BinaryOperator{
        public override object Evaluate(object left, object right){
            return ((IComparable) left).CompareTo(right) >= 0;
        }
    }


    public class BinaryOperation : Expression{
        private readonly Expression _left;
        private readonly BinaryOperator _op;
        private readonly Expression _right;


        [Rule(@"<Expression> ::= <Expression> '>' <Add Exp>")]
        [Rule(@"<Expression> ::= <Expression> '<' <Add Exp>")]
        [Rule(@"<Expression> ::= <Expression> '<=' <Add Exp>")]
        [Rule(@"<Expression> ::= <Expression> '>=' <Add Exp>")]
        [Rule(@"<Expression> ::= <Expression> '==' <Add Exp>")]
        [Rule(@"<Expression> ::= <Expression> '<>' <Add Exp>")]
        [Rule(@"<Add Exp> ::= <Add Exp> '+' <Mult Exp>")]
        [Rule(@"<Add Exp> ::= <Add Exp> '-' <Mult Exp>")]
        [Rule(@"<Add Exp> ::= <Add Exp> '&' <Mult Exp>")]
        [Rule(@"<Mult Exp> ::= <Mult Exp> '*' <Negate Exp>")]
        [Rule(@"<Mult Exp> ::= <Mult Exp> '/' <Negate Exp>")]
        public BinaryOperation(Expression left, BinaryOperator op, Expression right){
            _left = left;
            _op = op;
            _right = right;
        }
        
       
        public override object GetValue(SimpleExecutionContext ctx){

            object lStart = _left.GetValue(ctx);
            object rStart = _right.GetValue(ctx);

            object lFinal;
            object rFinal;

            // this attempts to determine the Greatest Common Type
            TypeChecker.GreatestCommonType gct = TypeChecker.GCT(lStart, rStart);

            // convert the values to the Greatest Common Type
            switch(gct){
                case TypeChecker.GreatestCommonType.StringType:
                    lFinal = Convert.ToString(lStart);
                    rFinal = Convert.ToString(rStart);
                    break;
                case TypeChecker.GreatestCommonType.BooleanType:
                    lFinal = Convert.ToBoolean(lStart);
                    rFinal = Convert.ToBoolean(rStart);
                    break;
                case TypeChecker.GreatestCommonType.NumericType:
                    lFinal = Convert.ToDecimal(lStart);
                    rFinal = Convert.ToDecimal(rStart);
                    break;
                default:
                    throw new ArgumentOutOfRangeException();
            }

            // execute on the converted values
            return _op.Evaluate(lFinal, rFinal);
        }
    }

    [Terminal("Id")]
    public class Identifier : Expression{
        internal readonly string _idName;

        public Identifier(string idName){
            _idName = idName;
        }

        public override object GetValue(SimpleExecutionContext ctx){
            
            return ctx[_idName];
        }
    }


    [Terminal("StringLiteral")]
    public class StringLiteral : Expression{
        private readonly string _value;

        public StringLiteral(string value){
            _value = value.Substring(1, value.Length - 2);
        }


        public override object GetValue(SimpleExecutionContext ctx){
            return _value;
        }
    }

    [Terminal("NumberLiteral")]
    public class NumberLiteral : Expression{
        private readonly decimal _value;

        public NumberLiteral(string value){
            _value = Convert.ToDecimal(value);
        }

        public override object GetValue(SimpleExecutionContext ctx){
            return _value;
        }
    }

    public class AssignStatement : Statement{
        private readonly Expression _expr;
        private readonly Identifier _receiver;

        [Rule(@"<Statement> ::= ~assign Id ~'=' <Expression>")]
        public AssignStatement(Identifier receiver, Expression expr){
            _receiver = receiver;
            _expr = expr;
        }

        public override void Execute(SimpleExecutionContext ctx){
            ctx[_receiver._idName] = _expr.GetValue(ctx);
        }
    }

    public class DisplayStatement : Statement{
        private readonly Expression _expr;
        private readonly Identifier _identToRead;


        [Rule(@"<Statement> ::= ~display <Expression>")]
        public DisplayStatement(Expression expr)
            : this(expr, null){
        }

        [Rule(@"<Statement> ::= ~display <Expression> ~read Id")]
        public DisplayStatement(Expression expr, Identifier identToRead){
            _expr = expr;
            _identToRead = identToRead;
        }


        public override void Execute(SimpleExecutionContext ctx){
            object outputToDisplay = _expr.GetValue(ctx);

            if (_identToRead == null){
                ctx.OutputStream.WriteLine(outputToDisplay.ToString());
            }
            else{
                ctx.OutputStream.Write("{0} \n>", outputToDisplay);
                ctx[_identToRead._idName] = ctx.InputStream.ReadLine();
            }
        }
    }

    public class WhileStatement : Statement{
        private readonly Expression _test;
        private readonly Sequence<Statement> _trueStatements;

        [Rule(@"<Statement> ::= ~while <Expression> ~do <Statements> ~end")]
        public WhileStatement(Expression test, Sequence<Statement> trueStatements){
            _test = test;
            _trueStatements = trueStatements;
        }

        public override void Execute(SimpleExecutionContext ctx){
            while (Convert.ToBoolean(_test.GetValue(ctx))){
                foreach (Statement stmt in _trueStatements){
                    stmt.Execute(ctx);
                }
            }
        }
    }

    public class IfStatement : Statement{
        private readonly Sequence<Statement> _falseStatements;
        private readonly Expression _test;
        private readonly Sequence<Statement> _trueStatements;

        [Rule(@"<Statement> ::= ~if <Expression> ~then <Statements> ~end")]
        public IfStatement(Expression _test, Sequence<Statement> trueStatements)
            : this(_test, trueStatements, null){
        }

        [Rule(@"<Statement> ::= ~if <Expression> ~then <Statements> ~else <Statements> ~end")]
        public IfStatement(Expression test, Sequence<Statement> trueStatements, Sequence<Statement> falseStatements){
            _test = test;
            _trueStatements = trueStatements;
            _falseStatements = falseStatements;
        }

        public override void Execute(SimpleExecutionContext ctx){
            if (Convert.ToBoolean(_test.GetValue(ctx))){
                foreach (Statement stmt in _trueStatements){
                    stmt.Execute(ctx);
                }
            }
            else{
                if (_falseStatements != null){
                    foreach (Statement stmt in _falseStatements){
                        stmt.Execute(ctx);
                    }
                }
            }
        }
    }


    public class Sequence<T> : SimpleToken, IEnumerable<T> where T : SimpleToken{
        private readonly T item;
        private readonly Sequence<T> next;


        public Sequence() : this(null, null){
        }

        [Rule("<Statements> ::= <Statement>", typeof (Statement))]
        public Sequence(T item) : this(item, null){
        }


        [Rule("<Statements> ::= <Statement> <Statements>", typeof (Statement))]
        public Sequence(T item, Sequence<T> next){
            this.item = item;
            this.next = next;
        }

        #region IEnumerable<T> Members

        public IEnumerator<T> GetEnumerator(){
            for (Sequence<T> sequence = this; sequence != null; sequence = sequence.next){
                if (sequence.item != null){
                    yield return sequence.item;
                }
            }
        }

        IEnumerator IEnumerable.GetEnumerator(){
            return GetEnumerator();
        }

        #endregion
    }

    static  class TypeChecker{
        public static bool IsBoolean(object obj){
            return (Type.GetTypeCode(obj.GetType()) == TypeCode.Boolean);
        }

        public static bool IsNumeric(object obj){
            switch(Type.GetTypeCode(obj.GetType())){
                case TypeCode.Empty:
                case TypeCode.Object:
                case TypeCode.DBNull:
                case TypeCode.Boolean:
                case TypeCode.Char:
                case TypeCode.SByte:
                case TypeCode.Byte:
                    return false;
                case TypeCode.Int16:
                case TypeCode.UInt16:
                case TypeCode.Int32:
                case TypeCode.UInt32:
                case TypeCode.Int64:
                case TypeCode.UInt64:
                case TypeCode.Single:
                case TypeCode.Double:
                case TypeCode.Decimal:
                    return true;
                case TypeCode.DateTime:
                case TypeCode.String:
                    return false;
                default:
                    throw new ArgumentOutOfRangeException();
            }
        }

        public enum GreatestCommonType{
            StringType,
            BooleanType,
            NumericType
        }

        public static GreatestCommonType GCT(params object[] toCheck){
            int boolCount = 0;
            int stringCount = 0;
            int numCount = 0;

            foreach (var obj in toCheck){
                if (IsBoolean(obj))
                    boolCount++;
                else if (IsNumeric(obj))
                    numCount++;
                else{
                    stringCount++;
                }
            }

            if (boolCount == toCheck.Length){
                return GreatestCommonType.BooleanType;
            }
            
            if (numCount == toCheck.Length){
                return GreatestCommonType.NumericType;
            }

            if(numCount > 0 && boolCount == 0){
                return GreatestCommonType.NumericType;
            }

            return GreatestCommonType.StringType;

        }

    }

    internal class Program{
        private static void Main(string[] args){

            

            CompiledGrammar grammar = CompiledGrammar.Load(typeof (SimpleToken), "Simple2.cgt");
            var actions = new SemanticTypeActions<SimpleToken>(grammar);
            try{
                actions.Initialize(true);
            }
            catch (InvalidOperationException ex){
                Console.Write(ex.Message);
                Console.ReadKey(true);
                return;
            }


            REPLCator runner = new REPLCator();
            runner.Run();

            
        }
    }
}

```