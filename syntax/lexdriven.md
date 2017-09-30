---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">

#### Lexer Driven Parsing
---

* <a href="#lexdriven-concepts">Concepts</a>
* <a href="#lexdriven-javascriptexample">Javascript Example</a>

##### <a name="lexdriven-concepts">Concepts</a>

Compiler driven parsers are the standard. They get created, invoked with an input stream, which is then read one character at a time, 
checking for grammar compliance and generating code and other structures as a result.

**Lexical driven parsers** differ in the fact that the input stream is discontinuous, usually user driven. The parser gets created and then
waits for an input to come as a method call, keeping state of where it has been. The parser will receive the input and check it for correctness, causing shifts and reduces with the given symbol.  When a state is reached that requires a new symbol, the parser will stop, store its state, and return from the method call.

As part of keeping state, a lexical driven parser will be able to inform what possible symbols are possible in the next method call, thus allowing user interfaces to enable/disable symbols.  The typical case that I have encountered in the past was a calculator with operators, numbers and parenthesis.  The buttons get enabled only on the presence of valid symbol transitions on the current parser state.  Consider the following calculator:

```
[1] [2] [3]  [+]
[4] [5] [6]  [-]
[7] [8] [9]  [*]
[(] [0] [)]  [/]
      [ = ]
```

And the following grammar

```

%start S

%%

S : E
  ;

E : E '+' E
  | E '-' E
  | E '*' E
  | E '/' E
  | '(' E ')'
  | num
  ;

%%

```

We can display a state where an expression has been seen as follows. 

```
State 1:
S ->  E .
E -> E . + E
E -> E . - E
E -> E . * E
E -> E . / E

State 2:
E -> (  E . )
E -> E . + E
E -> E . - E
E -> E . * E
E -> E . / E
```

The dot in the definitions denotes what would the parser had seen. State 1 signifies that the parser saw an expression (E), and that after that you can have the end of the expression or an operator ('+', '-', '*', '/'). State 1 can then be seen as allowing an operator or the end of the input. State 2 on the other hand has been reached only when a parenthesis was used and thus a closing parenthesis is valid, but not the end of the input.  Let's show this in the calculator diagrams.  Invalid tokens in a state will  be shown
as periods.

For state 1:
```
 .   .   .   [+]
 .   .   .   [-]
 .   .   .   [*]
 .   .   .   [/]
      [ = ]
```

And for state 2:
```
 .   .   .   [+]
 .   .   .   [-]
 .   .   .   [*]
 .   .  [)]  [/]
       ...  
```

As it can be seen, we can call the parser and command it to stop when a new symbol is needed (in this case user input.) Then we can check the valid tokens and enable the user interface elements as required.

##### <a name="lexdriven-javascriptexample">Javascript Example</a>

{% highlight javascript%}
%{
if (!console) { // i.e. nashorn
    var console = { log: print };
}

var calc = (function() {
%}

%class {
    this.number = 0;
    
    this.toString = function() {
        return "number=" + this.number;
    }
}

%left             '+' : "plus"
                , '-' : "minus"
%left             '*' : "multiply"
                , '/' : "divide"

%right            TOK_UMINUS:"unary minus"  

%token            '('
                , ')'

%token <number> TOK_NUMBER : "number"

%type  <number> E

%name E : "expression";

%start S

%group OPS : "operator" '+', '-', '*', '/';

%%
S : E
  ;

E : E '+' E     = $$ = $1 + $3;
  | E '-' E     = $$ = $1 - $3;
  | E '*' E     = $$ = $1 * $3;
  | E '/' E     = $$ = $1 / $3;
  | '-' E %prec TOK_UMINUS = $$ = -$2;
  | '(' E ')'   = $$ = $2;
  | TOK_NUMBER
  ;

%%

// END OF GRAMMAR

    function parserError(state, token, top, message) {
        console.log("An error occurred in position " + charNum + " with token " + getTokenName(token));
        console.log(message);
        return ERROR_RE_ATTEMPT;
    }

    return {
        setVerbose: setVerbose,
        getTokenName: getTokenName,
        getTokenFullName: getTokenFullName,
        getTokenIndex: getTokenIndex,
        parse: function (token, value) {
            var lexicalValue = new StackElement();
            lexicalValue.number = value;
            var numericToken;
        
            if (typeof token == 'number') {
                numericToken = token;
            }
            else {
                numericToken = token.charCodeAt(0);
            }
            return parse(numericToken, lexicalValue);
        },
        init: init,
        getValidTokens: getValidTokens,
        getResult: getResult
    }
 
})();

var getValidTokens = function() {
    var tokens = calc.getValidTokens();
    var alpha = [];
    tokens.forEach(function(token) {
        token = calc.getTokenName(token);
        alpha.push(token);
    })
    return alpha.join(",");
}

var getResult = function() {
    return calc.getResult().number;
}

var parseOne = function(token, value) {
    var rc = calc.parse(token, value);
    console.log("parsed " + token + " with value " + value + " with status=>" + rc);
    var tokens = getValidTokens();
    console.log(" >", tokens);
}

calc.init();
parseOne('(', 0);
parseOne(32769, 1); // number 1
parseOne('+', 0);
parseOne(32769, 3); // number 3
parseOne(')', 0);
parseOne('*', 0);
parseOne(32769, 4); // number 4
parseOne('/', 0);
parseOne(32769, 5); // number 5
parseOne('+', 0);
parseOne(32769, 20); // number 20
parseOne(0, 0);
console.log("Result=", getResult());
{% endhighlight %}

As can be seen, the syntax file looks identical to a standard parser grammar, minus the lexical analyzer. No lexer code, nor regular expressions are needed anymore.

This example uses some routines that may need an explanation. 

* **init()** has to be called on the parser. It can be called multiple times but everytime it would reset the parser to its initial state.
* **getValidTokens()** obtains the possible tokens in the parser given its current state. It then converts the tokens to their token names by **calc.getTokenName()**, returning a single string separated by commas, ready for display.
* **parseOne()** is just a utility routine that calls the parser with the current token and a value. It just logs the token, value and status returned by the scanner, plus the valid tokens.
* a **parse function** interface routine is created that for our example converts characters to their token number nd creates an object of the StackElement type:

```
        parse: function (token, value) {
            var lexicalValue = new StackElement();
            lexicalValue.number = value;
            var numericToken;
        
            if (typeof token == 'number') {
                numericToken = token;
            }
            else {
                numericToken = token.charCodeAt(0);
            }
            return parse(numericToken, lexicalValue);
        },

```

You can now run it as follows:

```
java -jar syntax-<version>.jar --language js --driver scanner Calc.sy
jjs Calc.js
```

Notice the --driver option given

The output of the execution gives:

```
parsed ( with value 0 with status=>2
 > -,(,TOK_NUMBER
parsed 32769 with value 1 with status=>2
 > +,-,*,/,)
parsed + with value 0 with status=>2
 > -,(,TOK_NUMBER
parsed 32769 with value 3 with status=>2
 > +,-,*,/,)
parsed ) with value 0 with status=>2
 > EOS,+,-,*,/
parsed * with value 0 with status=>2
 > -,(,TOK_NUMBER
parsed 32769 with value 4 with status=>2
 > EOS,+,-,*,/
parsed / with value 0 with status=>2
 > -,(,TOK_NUMBER
parsed 32769 with value 5 with status=>2
 > EOS,+,-,*,/
parsed + with value 0 with status=>2
 > -,(,TOK_NUMBER
parsed 32769 with value 20 with status=>2
 > EOS,+,-,*,/
parsed 0 with value 0 with status=>1
 > EOS,+,-,*,/
Result= 23.2
```

</div>
</div>