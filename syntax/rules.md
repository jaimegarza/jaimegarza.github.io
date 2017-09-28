---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">

#### Production Rules
---

* <a href="#rules-example">Learn by Example: Hello Calc!</a>
* <a href="#rules-code">Code Generation</a>
* <a href="#rules-intermediate">Intermediate Code</a>
* <a href="#rules-empty">Empty Rules</a>
* <a href="#rules-prec">Rules Precedence</a>

##### <a name="rules-example">Learn by Example: Hello Calc!</a>


At the heart of a grammar definition is the rules. Rules indicate structure as in

1. Order in which tokens can appear in a grammar, which define a non-terminal.
2. Different alternatives for a non-terminal, with the use of a pipe character.

Together with the rules themselves you can introduce code as part of the recognition of non terminals. For instance, consider the following grammar:

```
%token num : "number"

%start S

%%
S     : E
      ;

E     : E '+' T
      | E '-' T
      | T
      ;

T     : T '*' F
      | T '/' F
      | F
      ;

F     : num
      | '(' E ')'
      ;

%%
```

In it you can see the non-terminals S, E, T, F, which stand for **starting**, **expression**, **term** and **factor**. You can also see the tokens '+', '-', '*', '/', '(', ')'. By default, single character tokens do not need to be declared in C and Java. However, we may want to give them a name and some scanning code. Our operators, numbers and parenthesis need to be returned by the scanner, so lets add a scanner:

```
%class {
  public int n = 0;
}

%lexer = {
  while (Character.isWhitespace($c) && $c > '\0') {
    $+;
  }
}

%token '+' : "plus"     = if ($c == '+') {$+; return '+';};
     , '-' : "minus"    = if ($c == '-') {$+; return '-';};
     , '*' : "multiply" = if ($c == '*') {$+; return '*';};
     , '/' : "divide"   = if ($c == '/') {$+; return '/';};
     , '(' : "opening parenthesis"
                        = if ($c == '(') {$+; return '(';};
     , ')' : "closing parenthesis"
                        = if ($c == ')') {$+; return ')';};

%token <n> num : "number" /[0-9]*/ = {
  $v.n = Integer.parseInt($r);
  return num;
}

%start S

%%
S     : E
      ;

E     : E '+' T
      | E '-' T
      | T
      ;

T     : T '*' F
      | T '/' F
      | F
      ;

F     : num
      | '(' E ')'
      ;

%%
```

Our scanner skips blanks first, then it looks for operators and parenthesis. Numbers are recognized AND once recognized, added to the $v (value) as n. As it can be seen, some tokens are identified by their character while some others, like num, have a slightly more complex form. We have used regular expressions for it. In addition, we established a way to keep track of the given number. We used the type &lt;n&gt; for num, as used in the %class body.

Since we are building a calculator, we could say that E, T and F are also numeric. As such, we could declare them as having the same type &lt;n&gt;. Also, we can name them. This is done in the declaration section.

```
...

%type <n> E
%type <n> T
%type <n> F

%name E : "expression"
%name T : "term"
%name F : "factor"

...

```

We could also %group some tokens for more succint error messages, such as "operator expected":

```
...

%group op : "operator" '+', '-', '*', '/'

...
```

Now, it is simple enough to add some code to our calculator:

```
%class {
  public int n = 0;
}

%lexer = {
  while (Character.isWhitespace($c) && $c > '\0') {
    $+;
  }
}

%token '+' : "plus"     = if ($c == '+') {$+; return '+';};
     , '-' : "minus"    = if ($c == '-') {$+; return '-';};
     , '*' : "multiply" = if ($c == '*') {$+; return '*';};
     , '/' : "divide"   = if ($c == '/') {$+; return '/';};
     , '(' : "opening parenthesis"
                        = if ($c == '(') {$+; return '(';};
     , ')' : "closing parenthesis"
                        = if ($c == ')') {$+; return ')';};

%token <n> num : "number" /[0-9]*/ = {
  $v.n = Integer.parseInt($r);
  return num;
}

%type <n> E
%type <n> T
%type <n> F

%name E : "expression"
%name T : "term"
%name F : "factor"

%group op : "operator" '+', '-', '*', '/'

%start S

%%
S     : E        = System.out.println("The result is " + $1);
      ;

E     : E '+' T  = $$ = $E + $T;
      | E '-' T  = $$ = $E - $T;
      | T
      ;

T     : T '*' F  = $$ = $T * $F;
      | T '/' F  = $$ = $T / $F;
      | F
      ;

F     : num
      | '(' E ')' = $$ = $2;
      ;

%%
```

Several things need to be noted:

1. Code generated is delimited by an equal sign and terminated by a semicolon. For Pascal, the code is terminated with % instead. This is the simple code generation technique. However, if more complex code is needed, it can start with '{' and terminated '}' in C, Java and Javascript. Syntax will count and level opening and closing curly braces to identify the end of it.
2. We use the symbolic names of the non terminals as $E, $T, $F. yacc used positional numbers, as $1, $2, $3. We used such nomenclature in the expression in parenthesis. We could have used the more explicit $E.
3. The non terminal in the rule is denoted by $$.
4. There is always a default code rule ```= $$ = $1;``` when there is no code generated. You will not see this in the generated code, but it is implicit. This is the case for ```E : T, F : num, etc.```
5. Sometimes there are two symbols repeated in the same rule. For instance, in the ambiguous grammar where E : E '+' E, we need to use positional parameters.
6. Syntax will know that $E has a type &lt;n&gt;, so when the code is generated it will be generated with access to *element*.n automatically. We could have not declared the type of E, T or F, and as such, the code would have to read $E.n, $T.n, $F.n.
7. It is up to you to add additional getters, toString() or whatever is needed to your %class declaration. Remember that %class has synonyms %union, %struct.

Now, lets turn this into a java program by adding %{ %}, and code after the second %%.

```
%{
public class Calc {
%}

%class {
  public int n = 0;
}

%lexer = {
  while (Character.isWhitespace($c) && $c > '\0') {
    $+;
  }
}

%token '+' : "plus"     = if ($c == '+') {$+; return '+';};
     , '-' : "minus"    = if ($c == '-') {$+; return '-';};
     , '*' : "multiply" = if ($c == '*') {$+; return '*';};
     , '/' : "divide"   = if ($c == '/') {$+; return '/';};
     , '(' : "opening parenthesis"
                        = if ($c == '(') {$+; return '(';};
     , ')' : "closing parenthesis"
                        = if ($c == ')') {$+; return ')';};

%token <n> num : "number" /[0-9]*/ = {
  $v.n = Integer.parseInt($r);
  return num;
}

%type <n> E
%type <n> T
%type <n> F

%name E : "expression"
%name T : "term"
%name F : factor

%start S

%%
S     : E        = System.out.println("The result is " + $1);
      ;

E     : E '+' T  = $$ = $E + $T;
      | E '-' T  = $$ = $E - $T;
      | T
      ;

T     : T '*' F  = $$ = $T * $F;
      | T '/' F  = $$ = $T / $F;
      | F
      ;

F     : num
      | '(' E ')' = $$ = $2;
      ;

%%

  int charNum = 0;
  static String expression = "(1 + 3) *4 / 5 + 20";

  private char getNextChar(boolean initialize) {
    if (initialize) {
      charNum = 0;
    }
    if (charNum < expression.length()) {
      return expression.charAt(charNum++);
    }
    return EOS;
  }

  private void ungetChar(char c) {
    charNum --;
    if (charNum < 0 || expression.charAt(charNum) != c) {
      throw new RuntimeException("Error putting a character back");
    }
  }

  private int parserError(int state, int token, int top, String message) {
    System.out.println("An error occurred in position " + charNum + " with token " + getTokenName(token));
    System.out.println(message);

    return ERROR_RE_ATTEMPT;
  }
 
  public static void main(String ... args) {
    if (args.length > 0) {
      expression = args[0];
    }
    Calc calc = new Calc();
    calc.parse();
  }
}
```

Save this file as Calc.sy, and compile/execute with

```
java -jar syntax-<version>.jar --language java Calc.sy
javac Calc.java
java -classpath . Calc <optional-parameters>
```

To note:

1. the methods getNextChar(), ungetChar(), and parserError() were added to the class. They are implicitly called from calc.parse() as the parsing progresses. Our scanner process works on a hardcoded string by default.
2. The main method may receive an expression. Try multiple expressions, like "2+3**4" so that you can see the errors.
4. Please see the file called Calc.html for a full report of what went on. Also, feel free to peruse Calc.java. It should be legible.

As a note, these are the error messages that can be produced:

```
  // Error Messages
  private String errorTable[] = {
     /* 0 */ "opening parenthesis or number expected",
     /* 1 */ "No more elements expected",
     /* 2 */ "$ expected",
     /* 3 */ "operator expected",
     /* 4 */ "Expecting term",
     /* 5 */ "Expecting factor",
     /* 6 */ "plus, minus or closing parenthesis expected"
  };
```

And some of the code generated:

```
  private char currentChar;

  private String recognized;

  int parserElement(boolean initialize) {
    if (initialize) {
      currentChar = getNextChar(true);
    }

    lexicalValue = new StackElement();


      while (currentChar <= ' ' && currentChar > '\0') {
        currentChar = getNextChar(false);
      }

    if (currentChar == '+') {currentChar = getNextChar(false); return '+';};
    if (currentChar == '-') {currentChar = getNextChar(false); return '-';};
    if (currentChar == '*') {currentChar = getNextChar(false); return '*';};
    if (currentChar == '/') {currentChar = getNextChar(false); return '/';};
    if (currentChar == '(') {currentChar = getNextChar(false); return '(';};
    if (currentChar == ')') {currentChar = getNextChar(false); return ')';};
    if (matchesRegex(0)) {
        lexicalValue.n = Integer.parseInt(recognized);
        return num;

    }


    return 0; // UNKNOWN
  }

...

  boolean generateCode(int rule) {
    switch(rule){

      // 1. S ->  E
      case 1:
        System.out.println("The result is " + stack[stackTop].n);
        break;
      // 2. E ->  E + T
      case 2:
        stack[stackTop-2].n = stack[stackTop-2].n + stack[stackTop].n;
        break;
      // 3. E ->  E - T
      case 3:
        stack[stackTop-2].n = stack[stackTop-2].n - stack[stackTop].n;
        break;
      // 5. T ->  T * F
      case 5:
        stack[stackTop-2].n = stack[stackTop-2].n * stack[stackTop].n;
        break;
      // 6. T ->  T / F
      case 6:
        stack[stackTop-2].n = stack[stackTop-2].n / stack[stackTop].n;
        break;
      // 9. F ->  ( E )
      case 9:
        stack[stackTop-2].n = stack[stackTop-1].n;
        break;
    }
    return true; // OK
  }
```

And, from Calc.html

Regex  | Graph
-------|----------
[0-9]* | <img src="{{ site.baseurl }}/syntax/regex.png" />

##### <a name="rules-code">Code Generation</a>

To generate code in C, Java and Javascript, you can use the equals-sign, semicolon pattern, as in

```
E : E '+' T   = $$ = $E + $T;
```

In Pascal, the pattern ends with %, and it looks as follows:

```
E : E '+' T   = $$ := $E + $T;%
```

For more complex expressions in C, Java or Javascript, you can use code as follows:

```
E : E '+' T   { 
              $$ = $E + $T; 
              println("Found sum");
              }
```

In the generated code, syntax will try to remove any extra whitespace and will indent to the left.

##### <a name="rules-intermediate">Intermediate Code</a>

You can introduce code in the middle of a production, as in the following case:

```
E : E '+' { // Some code } T = $$ = $E + $T;
```

The code // Some code will be generated. In order to achieve this, syntax will generate a new rule, implicitly, as if this was done like this:

```
$code-fragment-1 : { // Some Code }
                 ;

E : E '+' $code-fragment-1 T = $$ = $E + $T ;
  ;
```

As you can see, the production code is implicitly converted to a new empty production with the code fragment in it. But there is a caveat: the code fragment occupies now a production position, so if you were doing this with positional parameters, you would have to do:

```
E : E '+' { // Some code } T = $$ = $1 + $4;
```

##### <a name="rules-empty">Empty Rules</a>

Rules can be left empty. It is usually OK. However, a great deal of issues with conflicts arises when two empty rules are nested. They usually create non-context sensitive grammars.  Consider the following (Both B and C are empty):

```
%start A

%%

A : B C D
  | D
  ;

B : B '1'
  |
  ;

C : C '2'
  |
  ;

D : '3'
  ;

%%
```

In this case, it says that A can be followed by B, which in turn can be empty, and then followed by C, which in turn can be empty, and then D. Or it can be simply D.  So imagine that you feed the parser the string "3". It can be reached both by the first or second rules. Which way would it go? It can be BOTH B C D, or just D.

In this case, syntax will mark a shift/reduce conflict. As specified in <a href="{{ site.baseurl }}/syntax/lexer#lexer-disambiguation">Disambiguation</a>, a Shift/Reduce assumes reduce. Interestingly enough the conflict is in B => <empty>. It assumes that B came if a 3 arrives.

##### <a name="rules-prec">Rules Precedence</a>

The section <a href="{{ site.baseurl }}/syntax/lexer#lexer-disambiguation">Disambiguation</a> gives you hints on how to resolve Shift/Reduce conflicts by using the %left and %right symbols. However, it is not always possible to identify these with the symbols. As such, you can add precedence to the rules themselves. You do that with the %prec declaration on a rule.

The way in which deambiguation works is by using tokens to define precedence. But at the end it is the rules themselves that have a final precedence. Each rule has a precedence. It is equal to the precedence of the **last** token in the production of the non terminal. 

We will extend the example on Disambiguation. The usual precedence of operators is +,- have the lowest precedence, with *,/ next, followed by unary negative or possitive numbers. For instance:

```
%left '+' : "addition", '-' : "subtraction"
%left '*' : "multiplication", '/' : "division"
%token num : "number"

%start S

%%
S     : E
      ;

E     : E '+' E
      | E '-' E
      | E '*' E
      | E '/' E
      | '-' E
      | num
      ;

%%
```

As we can see, there is now one additional rule with a negative unary operator. It needs to have a higher precedence. In order to do that we will introduce a token that will never appear in the scanner. Then we assign the rule to have the precedence of that token:

```
%left '+' : "addition", '-' : "subtraction"
%left '*' : "multiplication", '/' : "division"
%left TOK_UNARY

%token num : "number"

%start S

%%
S     : E
      ;

E     : E '+' E
      | E '-' E
      | E '*' E
      | E '/' E
      | '-' E %prec TOK_UNARY
      | num
      ;

%%
```

You will get

Precedence | Rule
----------:|:-------------
         0 | S : E
         1 | E : E '+' E
         1 | E : E '-' E
         2 | E : E '*' E
         2 | E : E '/' E
         3 | E : '-' E
         0 | E : num


</div>
</div>