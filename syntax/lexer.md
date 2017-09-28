---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">

#### Lexer
---

* <a href="#token-def">Token Definitions</a>
* <a href="#lexer-code">Additional Lexer Code</a>
* <a href="#code-symbols">Code Symbols</a>
* <a href="#lexer-modes">Lexer Modes</a>
* <a href="#lexer-disambiguation">Disambiguation</a>
* <a href="#lexer-by-hand">Writing Your Own Lexer</a>
* <a href="#lexer-getchar">Getting Characters</a>
* <a href="#lexer-regex">Regular Expressions</a>

##### <a name="token-def">Token Definitions</a>
The lexic of a language is its *vocalbulary*. Just like english where you have pronouns, verbs, adverbs, numbers, etc., the lexic of a language defines the most basic elements. The lexic of a language includes things like number, identifier, operator, etc. They do not necessarily follow a set of rules, but rather a set of patterns. The lexical analyzer of the language is also known as a **scanner**, and it produces/recognizes **tokens**, or **terminal symbols**. These can be either represented by regular expressions like:

```
%token INTEGER : "an integer" /[0-9]*/
```

or by simple text pattern code snippets

```
%token INTEGER : "an integer" = {
  if (Character.isDigit($c)) {
    %v.n = 0;
    while (Character.isDigit($c)) {
      $v.n = $vn * 10 + $c - '0';
      $+;
    }
  }
  return $t;
}
```

Tokens have an **identifier**. In the previous example, the identifier is INTEGER. This identifier is used in syntactical rules, or productions in the grammar. A token also has a **name**. A name can then be used to appear in error messages as opposed to the identifier. For instance, in the previous examples the name is *an integer*. With this name, the grammar can produce errors like *"expecting an integer"*.

A token can also have a type. A type is an element of a data structure called the %struct, %union, %class or %stack (They are all synonyms and you can choose the one you want. For C it is usually %union, while in Java it could be %class.) Independently of what is selected, we will call it the %struct. A %struct can be constructed as follows, for Java:

```
%class {
  public int n;
  public String s;

  public StackElement () {
    this.n = 0;
    this.s = "";
  }
}
```

The types in the stack element identified by the %class are then **&lt;s&gt;** and **&lt;n&gt;**. Thus, to assign a type to a token, you declare it like 

```
%token INTEGER <n> : "an integer" /[0-9]*/ {
  $v.n = Integer.parseInt($r);
  return $t;
}
```

The type can then be used in the grammar to simplify the code generation.

For pascal output, you can declare the structure as follows:

```
%struct
  n:LONGINT;
  s:STRING;
%
```

and

```
%token   <n> INTEGER:"an integer" = 
  IF   ($c >= '0') AND ($c <= '9')
  THEN BEGIN 
       $v.n := 0;
       WHILE ($c >= '0') AND ($c <= '9') DO
             BEGIN 
             $v.n := $v.n * 10 + ORD($c) - ORD('0');
             $+;
             END;
       $v.n := number;
       $x(INTEGER);
       END;
%
```

or in Javascript:

```
%token <n> INTEGER : "an integer" /[0-9]*/ = {
    $v.n = parseInt($r);
    return INTEGER;
}
```

##### <a name="lexer-code">Lexer Code</a>

Sometimes you may want to add additional code to be inserted in your output lexer code. You can do so by using the %lexer declaration as follows:

```
%lexer = {
  while ($c <= ' ' && $c > '\0') {
    $+;
  }
}
```

or in pascal

```
%lexer =
  WHILE ($c = ' ') OR ($c = CHR(9)) DO $+;
%
```

##### <a name="code-symbols">Code Symbols</a>

When writing code in your lexer, you can use the following symbols

Symbol | Type    | Description
-------|:--------|:------------
$c     | CHAR    | The current character in the input stream.
$v     | %struct | The output associated element in the parsing stack. Its elements are those of the structure.
$+     | VOID    | Instruction that means "get the next character".
$l     | INT     | The current lexer mode.
$m     |         | The current lexer function name, which changes with the mode.
$t     |         | The current token id
$r     | String  | The string recognized by the regular expression
$x     |         | Used to return from the function. return in java, javascript and C, and exit() in Pascal. use like $x($t) which returns the current token.

##### <a name="lexer-modes">Lexer Modes</a>

Sometimes writing a lexer, you discover that certain parts of the lexer scan in a different ways than others. For instance, imagine that your main scanner skips white space to locate elements, like in perl or other language. Perl, however, uses delimiters to mark regular expressions, like qx, or /, After recognizing the / character you have to scan character by character not skipping blanks since blanks are valid characters in a regular expression.

You can create lexer code that is separated from each other by using the **[MODE]** notation in tokens as in the next example:

```
%token aa:"an a symbol" = if ($c == 'a') {$v.s = "" + $c; $+; return aa;};
%token bb:"a b symbol" [B] = if ($c == 'b') {$v.s = "" + $c; $+; return bb;};
%token cc:"a c symbol" [C] {
  if ($c == 'c') {
    $v.s = "" + $c; 
    $+; 
    return cc;
  };
}

%lexer [B] = if ($c == 'a') {$v.s = "" + $c; $+; $l = DEFAULT_LEXER_MODE; return aa;};
%lexer [C] = if ($c == 'a') {$v.s = "" + $c; $+; $l = DEFAULT_LEXER_MODE; return aa;};

%lexer {
  if ($c == 'b') {
    $v.s = "" + $c; 
    $l = B_LEXER_MODE;
    $+;
    return bb;
  }
  
  if ($c == 'c') {
    $v.s = "" + $c; 
    $l = C_LEXER_MODE;
    $+;
    return cc;
  }
}

```

This would generate something like this:

```
  private int parserElementMode = DEFAULT_LEXER_MODE;
  ...
  private String recognized;

  int parserElement(boolean initialize) {
    if (initialize) {
      currentChar = getNextChar(true);
    }

    lexicalValue = new StackElement();

    switch (parserElementMode) {

      case B_LEXER_MODE:
        return parserElement_B();

      case C_LEXER_MODE:
        return parserElement_C();

      case DEFAULT_LEXER_MODE:
        return parserElement_default();

    }

    return 0; // UNKNOWN
  }

  int parserElement_B() {
    if (currentChar == 'b') {lexicalValue.s = "" + currentChar; currentChar = getNextChar(false); return bb;};
    if (currentChar == 'a') {lexicalValue.s = "" + currentChar; currentChar = getNextChar(false); parserElementMode = DEFAULT_LEXER_MODE; return aa;};

    return 0; // UNKNOWN
  }

  int parserElement_C() {

      if (currentChar == 'c') {
        lexicalValue.s = "" + currentChar; 
        currentChar = getNextChar(false); 
        return cc;
      };
    
    if (currentChar == 'a') {lexicalValue.s = "" + currentChar; currentChar = getNextChar(false); parserElementMode = DEFAULT_LEXER_MODE; return aa;};

    return 0; // UNKNOWN
  }

  int parserElement_default() {
    if (currentChar == 'a') {lexicalValue.s = "" + currentChar; currentChar = getNextChar(false); return aa;};

      if (currentChar == 'b') {
        lexicalValue.s = "" + currentChar; 
        parserElementMode = B_LEXER_MODE;
        currentChar = getNextChar(false);
        return bb;
      }
      
      if (currentChar == 'c') {
        lexicalValue.s = "" + currentChar; 
        parserElementMode = C_LEXER_MODE;
        currentChar = getNextChar(false);
        return cc;
      }
    

    return 0; // UNKNOWN
  }
```

##### <a name="lexer-disambiguation">Disambiguation</a>

Syntax will warn you when a Shift/Reduce conflicts exist in the grammar. Sometimes these are harmless, and thus can be ignored if you know how the parsing works by default. Some other times you can explicitly resolve these conflicts with token symbols for this purpose.

A Shift/Reduce conflict happens when the parser does not know for sure if, when encountering a token, a new state needs to be reached, or if a rule has been recognized. A Reduce/Reduce conflict happens when a grammar is not context free. Pretty much two or more rules are recognized. Syntaxt will raise its hands in the air and say: "I don't know what I should do!" and then it will report an error. Special token definitions are present to disambiguate Shift/Reduce conflicts. Reduce/Reduce conflicts require a redesign of the rules.

Let's see one example. This example shows a simple calculator in a fully SLR(1) compatible mode.

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
      ;

%%
```

An expression is defined in in terms of a term, and a term is defined via a factor. There are no conflicts.  However, if we would describe E in terms of E, that would cause problems, as follows:

```
%token num : "number"

%start S

%%
S     : E
      ;

E     : E '+' E
      | E '-' E
      | E '*' E
      | E '/' E
      | num
      ;

%%

```

Results in 

```
expr.sy(0) : Warning: Shift/Reduce conflict on state 8[+ Shift:4 Reduce:2].
expr.sy(0) : Warning: Shift/Reduce conflict on state 8[- Shift:5 Reduce:2].
expr.sy(0) : Warning: Shift/Reduce conflict on state 8[* Shift:6 Reduce:2].
expr.sy(0) : Warning: Shift/Reduce conflict on state 8[/ Shift:7 Reduce:2].
expr.sy(0) : Warning: Shift/Reduce conflict on state 9[+ Shift:4 Reduce:3].
expr.sy(0) : Warning: Shift/Reduce conflict on state 9[- Shift:5 Reduce:3].
expr.sy(0) : Warning: Shift/Reduce conflict on state 9[* Shift:6 Reduce:3].
expr.sy(0) : Warning: Shift/Reduce conflict on state 9[/ Shift:7 Reduce:3].
expr.sy(0) : Warning: Shift/Reduce conflict on state 10[+ Shift:4 Reduce:4].
expr.sy(0) : Warning: Shift/Reduce conflict on state 10[- Shift:5 Reduce:4].
expr.sy(0) : Warning: Shift/Reduce conflict on state 10[* Shift:6 Reduce:4].
expr.sy(0) : Warning: Shift/Reduce conflict on state 10[/ Shift:7 Reduce:4].
expr.sy(0) : Warning: Shift/Reduce conflict on state 11[+ Shift:4 Reduce:5].
expr.sy(0) : Warning: Shift/Reduce conflict on state 11[- Shift:5 Reduce:5].
expr.sy(0) : Warning: Shift/Reduce conflict on state 11[* Shift:6 Reduce:5].
expr.sy(0) : Warning: Shift/Reduce conflict on state 11[/ Shift:7 Reduce:5].

```

This means, for instance, that in state 8 you have  recognized that E + E was found, except that there is no clarity to what happens with the '+' sign. Should the parser proceed exploring E, or just add at that moment?

The default result is that the reduction, or recognition of E + E should be done. Syntax assumes that and proceeds creating the parser reporting a warning in the screen and report file. There are situations where that may not be the desired result. Sometimes you may want to assume the Shift.

For that reason, a way to declare tokens is present, that in addition can declare precedence.

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
      | num
      ;

%%

```

Notice the use of commas. They are not required as in yacc, but they are recommended for clarity. The first %left declares '+' and '-' as first precedence, with left associativity. The second %left declares '*', and '/' as second precedence with left associativity, and then %token declares num as third precedence with undefined associativity.

Declarator    | Associativity | Shift/Reduce behavior
--------------|:--------------|:---------------------------
%left, %<     | left          | Reduce is assumed
%right, %>    | right         | Shift is assumed
%token, %0    | none          | Assume reduce, produce a warning
%nonassoc, %2 | binary        | Considered an error

%left, %right declare what happens in situations like *a* + *b* + *c*. %left says that *a* + *b* is done first. %right says that *b* + *c* is executed first. The generated parser uses a stack to preserve a, b and c. For right precedence, in *a* + *b* + *c* + *d* + *e* + ... the stack stores a, b, c, d, e, etc. in a stack. At the end it starts popping items from the stack and adding them. %right precedence can thus cause stack overflows at compilation/interpret.

With symbols, the previous grammar would look like

```
%< '+' : "addition", '-' : "subtraction"
%< '*' : "multiplication", '/' : "division"
%0 num : "number"

%start S

%%
S     : E
      ;

E     : E '+' E
      | E '-' E
      | E '*' E
      | E '/' E
      | num
      ;

%%
```

Please refer to <a href="{{ site.baseurl }}/syntax/rules#rules-ambiguous">Ambiguous Rules</a> to see how to further refine precedence with the %prec symbol.

##### <a name="lexer-by-hand">Writing Your Own Lexer</a>

If you decide not to add code in the lexer via %token (or any of its derivatives), or via %lexer, syntax will not add a scanner code. In that case you need to generate a parserElement method as follows:

Java:
```
  int parserElement(boolean initialize) {
   ...
  }
```

Javascript:
```
    function parserElement(initialize) {
      ...
    }
```

Pascal:
```
function StxLexer:longint;
begin
    ...
    StxLexer := 0;
end;(* StxLexer *)
```

C:
```
unsigned long int StxLexer()
{
    ...
    return 0; /* UNKNOWN */
}/* End of StxLexer */
```

initialize is true on the first call in case you need to open streams or other initialization.

##### <a name="lexer-getchar">Getting Characters</a>

If you use the code generation utilities of syntax via %token (or its derivatives), or by using %lexer, then you need to provide a routine to obtain characters, one at a time, for the scanner as follows:

Java:
```
  private char getNextChar(boolean initialize) {
    if (initialize) {
      ...
    }
    if (...) {
      return ...;
    }
    return EOS;
  }

  private void ungetChar(char c) {
    ...
  }
```

Javascript:
```
    function getNextChar(initialize) {
        if (initialize) {
          ...
        }
        if (charNum < expression.length) {
            return ...;
        }
        return '\0';
    }

    function ungetChar(c) {
        ...
    }
```

Pascal:
```
FUNCTION StxNextChar: CHAR;
BEGIN
    IF   ...
    THEN BEGIN
         StxNextChar := ...;
         END
    ELSE StxNextChar := CHR(EOS);
END;

PROCEDURE StxUngetChar(c:char);
BEGIN
    ...
END;
```

C:
```
char StxNextChar()
{
    if (...) {
        ...
        return ...;
    } else {
      return EOS;
    }
}

void StxUngetChar(char c) {
    ...
}
```

**EOS** stands for end-of-string

##### <a name="lexer-regex">Regular Expressions</a>

The regular expressions allowed in Syntax are simple regular expressions. There are plans to enrich them in the future to be perl compliant. However, this is not yet the case. But they are quite powerful and enough for most cases.

Regular expressions are defined (in syntax language, of course) as:

```
%token <c>        PIPE             : "'|'"
%token <c>        STAR             : "'*'" 
%token <c>        PLUS             : "'+'" 
%token <c>        HUH              : "'?'" 
%token <c>        ANY              : "'.'"
%token <c>        CHAR             : "character"
%token <cc>       CHARACTER_CLASS  : "a valid character class"

...

%%

...

RegExp        : RegExp PIPE Concatenation
              | Concatenation
              ;
              
Concatenation : Concatenation UnaryRegex
              | UnaryRegex
              ;
         
UnaryRegex    : BasicElement STAR
              | BasicElement PLUS
              | BasicElement HUH
              | BasicElement
              ;
              
BasicElement  : '(' RegExp ')'
              | CHAR
              | '[' CHARACTER_CLASS ']'
              | ANY
              ;
```

as such, you can generate lexer code as follows:

```
%token eq      : "==" /==/ = return eq;
%token ne      : "!=" /!=/ = return ne;

%token _if     : "if" /if/ = return _if;
%token _while  : "while" /while/ = return _while;

%token <n> num : "number" /[0-9]*/ = {
  return num;
}
```

Please open the resulting html report file to see the actual regular expression used and their diagrams.

</div>
</div>