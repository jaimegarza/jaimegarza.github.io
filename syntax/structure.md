---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">

#### File Structure
---

* <a href="#struct">Structure</a>
* <a href="#struct-decl">Meta Symbols</a>
* <a href="#struct-java">Java Example</a>
* <a href="#struct-c">C Example</a>
* <a href="#struct-pascal">Pascal Example</a>
* <a href="#struct-javascript">Javascript Example</a>

##### <a name="struct">Structure</a>

```
%{
  Program declarations
%}

Syntax declarations

%%

Rules

%%

Additional code

```

The %{ .. %} and %% markers are present in every syntax file. The delimit the three sections of the file. Extension is usually .sy. %{ ...%} markers are completelly optional, and can appear multiple times.

* The program declarations are fragments of code for the native language being used for the output. For example, java files may have the package, import and class declarations.
* The syntax declarations declare the terminals, non terminals, error definitions, lexer and other components of the grammar.
* The rules specify the grammar rules by declaring how non terminals are to be constructed.
* The additional code can be anything on the desired target language.

##### <a name="struct-decl">Declaration Meta Symbols</a>

When in the program declaration section, you can use the following meta symbols

Symbol | Description
-------|:-----------
$$b    | The output filename without extensions, no path.
$$n    | The output filename with extension, no path
$$f    | The output filename with extension and path
$$e    | The output filename extension
$$p    | The output file's path, no filename nor extension

##### <a name="struct-java">Java example</a>
```
%{
public class TestParser {
%}

%class {
  int number;
    
  public StackElement () {
    this.number = 0;
  }
    
  public String toString() {
    return "n=" + number;
  }
  
  public void setNumber(int number) {
    this.number = number;
  }
}

%lexer = {
  while ($c <= ' ' && $c > '\0') {
    $+;
  }
}

%left             TOK_AND      256:"AND" =  if ($c == '&') {$+; return TOK_AND;};
%left             TOK_OR       257:"OR" =   if ($c == '|') {$+; return TOK_OR;}; 
%right            TOK_NOT      258:"NOT" =  if ($c == '!') {$+; return TOK_NOT;};
%left             TOK_LE       259:"'<='",
                  TOK_LT       260:"'<'",
                  TOK_GE       261:"'>='",
                  TOK_GT       262:"'>'",
                  TOK_NE       263:"'<>'",
                  TOK_EQ       264:"'=='" = {
  if ($c == '=') {
    $+; 
    return TOK_EQ;
  }
  if ($c == '<') {
    $+;
    if ($c == '=') {
      $+;
      return TOK_LE;
    }
    if ($c == '>') {
      $+;
      return TOK_NE;
    }
    return TOK_LT;
  }
  if ($c == '>') {
    $+;
    if ($c == '=') {
      $+;
      return TOK_GE;
    }
    return TOK_GT;
  }  
}

%left             '+' : "plus" =  if ($c == '+') {$+; return '+';};
                , '-' : '"minus"'=  if ($c == '-') {$+; return '-';};
%left             '*' =  if ($c == '*') {$+; return '*';};
                , '/' =  if ($c == '/') {$+; return '/';};

%right            TOK_UMINUS:"unary minus"  

%token            '(' =  if ($c == '(') {$+; return '(';};
                , ')' =  if ($c == ')') {$+; return ')';};

%token   <number> TOK_NUMBER:"number" = {
  if ($c >= '0' && $c <= '9') {
    int number = 0;
    while ($c >= '0' && $c <= '9') {
      number = number * 10 + $c - '0';
      $+;
    }
    lexicalValue.number = number;
    return TOK_NUMBER;
  }
}

%type    <number> Expression

%name Expression : "expression";

%start Expression

%lexer = {
}

%group OPS : "operator" TOK_AND, TOK_OR, TOK_LT, TOK_LE, TOK_GT, TOK_GE, TOK_NE, TOK_EQ, '+', '-', '*', '/';

%%
Expression   :  Expression TOK_AND Expression = $$ = ($1 != 0) && ($3 != 0) ? 1 : 0;
             |  Expression TOK_OR Expression  = $$ = ($1 != 0) || ($3 != 0) ? 1 : 0;
             |  TOK_NOT Expression            = $$ = ($2 != 0) ? 0 : 1;
             |  Expression TOK_LE Expression  = $$ = $1 <= $3 ? 1 : 0;
             |  Expression TOK_LT Expression  = $$ = $1 < $3 ? 1 : 0;
             |  Expression TOK_GE Expression  = $$ = $1 >= $3 ? 1 : 0;
             |  Expression TOK_GT Expression  = $$ = $1 > $3 ? 1 : 0;
             |  Expression TOK_NE Expression  = $$ = $1 != $3 ? 1 : 0;
             |  Expression TOK_EQ Expression  = $$ = $1 == $3 ? 1 : 0;
             |  Expression '+' Expression     = $$ = $1 + $3;
             |  Expression '-' Expression     = $$ = $1 - $3;
             |  Expression '*' Expression     = $$ = $1 * $3;
             |  Expression '/' Expression     = $$ = $1 / $3;
             |  '-' Expression %prec TOK_UMINUS = $$ = -$2;
             |  '(' Expression ')'            = $$ = $2;
             |  TOK_NUMBER
             ;
%%

// END OF GRAMMAR

  int charNum = 0;
  String expression = "(1 + 3) *4 / 5 + -20";
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
    System.out.println("An error occurred in state " + state + " with token " + token + " on the position " + top);
    System.out.println(message);
    parserPrintStack();
    return ERROR_RE_ATTEMPT;
  }
  
  private String toString(StackElement value) {
    if(value == null) {
      return "";
    } else {
      return value.toString();
    }
  }
  
  public int getTotal() {
    StackElement result = getResult();
    if (result != null) {
      return result.number;
    } else {
      return 0;
    }
  }
  
}
```

##### <a name="struct-c">C example</a>
```
%{
#include <stdio.h>
#include "$$b.h"
%}

%union {
  int number;
}

%lexer = {
  while ($c <= ' ' && $c > '\0') {
    $+;
  }
}

%left             TOK_AND      256:"AND" =  if ($c == '&') {$+; return TOK_AND;};
%left             TOK_OR       257:"OR" =  if ($c == '|') {$+; return TOK_OR;}; 
%right            TOK_NOT      258:"NOT" =  if ($c == '!') {$+; return TOK_NOT;};
%left             TOK_LE       259:"'<='",
                  TOK_LT       260:"'<'",
                  TOK_GE       261:"'>='",
                  TOK_GT       262:"'>'",
                  TOK_NE       263:"'<>'",
                  TOK_EQ       264:"'=='" = {
  if ($c == '=') {
    $+; 
    return TOK_EQ;
  }
  if ($c == '<') {
    $+;
    if ($c == '=') {
      $+;
      return TOK_LE;
    }
    if ($c == '>') {
      $+;
      return TOK_NE;
    }
    return TOK_LT;
  }
  if ($c == '>') {
    $+;
    if ($c == '=') {
      $+;
      return TOK_GE;
    }
    return TOK_GT;
  }  
}

%left             '+' : "plus" =  if ($c == '+') {$+; return '+';};
                , '-' : '"minus"'=  if ($c == '-') {$+; return '-';};
%left             '*' =  if ($c == '*') {$+; return '*';};
                , '/' =  if ($c == '/') {$+; return '/';};

%right            TOK_UMINUS:"unary minus";

%token            '(' =  if ($c == '(') {$+; return '(';};
                , ')' =  if ($c == ')') {$+; return ')';};

%token   <number> TOK_NUMBER:"number" = {
  if ($c >= '0' && $c <= '9') {
    int number = 0;
    while ($c >= '0' && $c <= '9') {
      number = number * 10 + $c - '0';
      $+;
    }
    $v.number = number;
    return TOK_NUMBER;
  }
}

%type    <number> Expression

%start Expression

%lexer = {
}


%%
Expression   :  Expression TOK_AND Expression = $$ = ($1 != 0) && ($3 != 0) ? 1 : 0;
             |  Expression TOK_OR Expression  = $$ = ($1 != 0) || ($3 != 0) ? 1 : 0;
             |  TOK_NOT Expression            = $$ = ($2 != 0) ? 0 : 1;
             |  Expression TOK_LE Expression  = $$ = $1 <= $3 ? 1 : 0;
             |  Expression TOK_LT Expression  = $$ = $1 < $3 ? 1 : 0;
             |  Expression TOK_GE Expression  = $$ = $1 >= $3 ? 1 : 0;
             |  Expression TOK_GT Expression  = $$ = $1 > $3 ? 1 : 0;
             |  Expression TOK_NE Expression  = $$ = $1 != $3 ? 1 : 0;
             |  Expression TOK_EQ Expression  = $$ = $1 == $3 ? 1 : 0;
             |  Expression '+' Expression     = $$ = $1 + $3;
             |  Expression '-' Expression     = $$ = $1 - $3;
             |  Expression '*' Expression     = $$ = $1 * $3;
             |  Expression '/' Expression     = $$ = $1 / $3;
             |  '-' Expression %prec TOK_UMINUS = $$ = -$2;
             |  '(' Expression ')'            = $$ = $2;
             |  TOK_NUMBER
             ;
%%

// END OF GRAMMAR

int charNum = 0;
char * expression = "(1 + 3) *4 / 5 + -20";

char StxNextChar()
{
    if (charNum < strlen(expression)) {
      printf("Char %c\n", expression[charNum]);
      return expression[charNum++];
    }
    return EOS;
}

void StxUngetChar(char c) {
    charNum --;
}

int StxError(int state, int token, int top, char * message)
{
    printf("An error occurred in state %d with token %d on the position %d\n", state, token, top);
    printf("%s\n", message);
#ifdef DEBUG
    StxPrintStack();
#endif
    return ERROR_RE_ATTEMPT;
}

#ifdef DEBUG
char staticToStringValue[1024];

char * StxToString(TSTACK value)
{
    sprintf(staticToStringValue, "%d", value.number);
    return staticToStringValue;
}
#endif
  
int GetTotal() {
    return StxGetResult().number;
}

#ifdef SCANNER_MODE
typedef struct {
    int token;
    int value;
    int result;
    
} PARAMETER, *PPARAMETER;

PARAMETER parameters[] = {
    {'(', 0, 0},
    {TOK_NUMBER, 1, 1},
    {'+', 0, 0},
    {TOK_NUMBER, 3, 3},
    {')', 0, 0},
    {'*', 0, 0},
    {TOK_NUMBER, 4, 4},
    {'/', 0, 0},
    {TOK_NUMBER, 5, 5},
    {'+', 0, 0},
    {'-', 0, 0},
    {TOK_NUMBER, 20, 20},
    {0, 0, -17},
    {-1,0,0}
};

int arrayContains(int *array, int value, int count) {
    int i;
    for (i = 0; i < count; i++) {
        if (array[i] == value) {
            return 1;
        }
    }
    return 0;
}
  
int main(char *argv) 
{
    PPARAMETER p = parameters;
    int count;
    int t;
    TSTACK value;
    
    StxInit();
    
    while (p->token != -1) {
        int *tokens = StxValidTokens(&count);
        if (!arrayContains(tokens, p->token, count)) {
            printf("Token %d ain't there\n", p->token);
            exit (1);
        }
        value.number = p->value;
        if (!StxParse(p->token, value)) {
            printf("Error when parsing symbol %d\n", p->token);
            exit (1);
        }
        t = GetTotal();
        if (t != p->result) {
            printf("Result is not %d\n", p->result);
            exit (1);
        }
        p++;
    }
    t = GetTotal();
    printf("Total: %d\n", t);
    if (t != -17) {
        printf("total does not match\n");
        exit (1);
     }
    exit (0);
}
#else

int main(char *argv) 
{
    if (StxParse()) {
        printf("Total=%d\n", GetTotal());
    }
    exit (0);
}
#endif
```

##### <a name="struct-pascal">Pascal example</a>
```
%{
{Program that gets compiled and executed by free pascal}
PROGRAM pascaltest;
USES sysutils;
{$I $$b.inc}
{$DEFINE DEBUG}
VAR
    number : LONGINT;
%}

%union
  number:LONGINT;
%

%lexer =
  WHILE ($c = ' ') OR ($c = CHR(9)) DO $+;
  writeln('Char is "', $c, '"');
%

%left             TOK_AND      256:"AND" =  if $c = '&' THEN BEGIN $+; $x(TOK_AND); END;%
%left             TOK_OR       257:"OR"  =  if $c = '|' THEN BEGIN $+; $x(TOK_OR); END;% 
%right            TOK_NOT      258:"NOT" =  if $c = '!' THEN BEGIN $+; $x(TOK_NOT); END;%
%left             TOK_LE       259:"'<='",
                  TOK_LT       260:"'<'",
                  TOK_GE       261:"'>='",
                  TOK_GT       262:"'>'",
                  TOK_NE       263:"'<>'",
                  TOK_EQ       264:"'='" = 
  IF   $c = '='
  THEN BEGIN 
       $+; 
       $x (TOK_EQ);
       END
  ELSE 
  IF   $c = '<'
  THEN BEGIN 
       $+;
       IF   $c = '='
       THEN BEGIN 
            $+;
            $x(TOK_LE);
       END
       ELSE
       IF   $c = '>'
       THEN BEGIN 
            $+;
            $x(TOK_NE);
       END;
       $x(TOK_LT);
  END
  ELSE
  IF   $c = '>'
  THEN BEGIN 
       $+;
       IF   $c = '='
       THEN BEGIN 
            $+;
            $x(TOK_GE);
       END;
       $x(TOK_GT);
       END;
%

%left             '+' : "plus"   =  IF   $c = '+' THEN BEGIN $+; $x(ORD('+')); END;%
                , '-' : '"minus"'=  IF   $c = '-' THEN BEGIN $+; $x(ORD('-')); END;%
%left             '*' =  IF   $c = '*' THEN BEGIN $+; $x(ORD('*')); END;%
                , '/' =  IF   $c = '/' THEN BEGIN $+; $x(ORD('/')); END;%

%right            TOK_UMINUS:"unary minus"  

%token            '(' =  IF   $c = '(' THEN BEGIN $+; $x(ORD('(')); END;%
                , ')' =  IF   $c = ')' THEN BEGIN $+; $x(ORD(')')); END;%

%token   <number> TOK_NUMBER:"number" = 
  IF   ($c >= '0') AND ($c <= '9')
  THEN BEGIN 
       number := 0;
       WHILE ($c >= '0') AND ($c <= '9') DO
             BEGIN 
             number := number * 10 + ORD($c) - ORD('0');
             $+;
             END;
       $v.number := number;
       $x(TOK_NUMBER);
       END;
%

%type    <number> Expression

%start Expression

%lexer = 
%


%%
Expression   :  Expression TOK_AND Expression = IF ($1 <> 0) AND ($3 <> 0) THEN $$ := 1 ELSE $$ := 0;%
             |  Expression TOK_OR Expression  = IF ($1 <> 0) OR ($3 <> 0) THEN $$ := 1 ELSE $$ := 0;%
             |  TOK_NOT Expression            = IF ($2 <> 0) THEN $$ := 0 ELSE $$ := 1;%
             |  Expression TOK_LE Expression  = IF $1 <= $3 THEN $$ := 1 ELSE $$ := 0;%
             |  Expression TOK_LT Expression  = IF $1 < $3 THEN $$ := 1 ELSE $$ := 0;%
             |  Expression TOK_GE Expression  = IF $1 >= $3 THEN $$ := 1 ELSE $$ := 0;%
             |  Expression TOK_GT Expression  = IF $1 > $3 THEN $$ := 1 ELSE $$ := 0;%
             |  Expression TOK_NE Expression  = IF $1 <> $3 THEN $$ := 1 ELSE $$ := 0;%
             |  Expression TOK_EQ Expression  = IF $1 = $3 THEN $$ := 1 ELSE $$ := 0;%
             |  Expression '+' Expression     = $$ := $1 + $3;%
             |  Expression '-' Expression     = $$ := $1 - $3;%
             |  Expression '*' Expression     = $$ := $1 * $3;%
             |  Expression '/' Expression     = $$ := $1 DIV $3;%
             |  '-' Expression %prec TOK_UMINUS = $$ := -$2;%
             |  '(' Expression ')'            = $$ := $2;%
             |  TOK_NUMBER
             ;
%%

// END OF GRAMMAR

VAR
    charNum : INTEGER = 1;

CONST
    expression = '(1 + 3) *4 / 5 + -20';

FUNCTION StxNextChar: CHAR;
BEGIN
    IF   charNum <= LENGTH(expression) 
    THEN BEGIN
         writeln('Char ', expression[charNum]);
         StxNextChar := expression[charNum];
         charNum := charNum + 1;
         END
    ELSE StxNextChar := CHR(EOS);
END;

PROCEDURE StxUngetChar(c:char);
BEGIN
    charNum := charNum - 1;
END;

FUNCTION StxError(StxState:INTEGER; StxSym: INTEGER; pStxStack: INTEGER; aMessage:STRING):INTEGER;
BEGIN
    writeln('An error occurred in state ', StxState, ' with token ', StxSym, ' on the position ', pStxStack);
    writeln(aMessage);
{$IFDEF DEBUG}
    StxPrintStack();
{$ENDIF}
    StxError := ERROR_RE_ATTEMPT;
END;
  
FUNCTION StxToString(value:TSTACK):STRING;
BEGIN
  StxToString := IntToStr(value.number);
END;

FUNCTION GetTotal() :INTEGER; 
BEGIN
    EXIT(StxGetResult().number);
END;

{$IFDEF SCANNER_MODE}
TYPE
  PPARAMETER = ^PARAMETER;
  PARAMETER = RECORD
    token: LongInt;
    value: integer;
    result: integer;
  END;

VAR
  parameters: ARRAY[0..13] OF PARAMETER = (
    (token:ORD('(')  ; value:0; result:0),
    (token:TOK_NUMBER; value:1; result:1),
    (token:ORD('+')  ; value:0; result:0),
    (token:TOK_NUMBER; value:3; result:3),
    (token:ORD(')')  ; value:0; result:0),
    (token:ORD('*')  ; value:0; result:0),
    (token:TOK_NUMBER; value:4; result:4),
    (token:ORD('/')  ; value:0; result:0),
    (token:TOK_NUMBER; value:5; result:5),
    (token:ORD('+')  ; value:0; result:0),
    (token:ORD('-')  ; value:0; result:0),
    (token:TOK_NUMBER; value:20;result: 20),
    (token:0         ; value:0; result:-17),
    (token:-1        ; value:0; result:0));

FUNCTION arrayContains(tokenArray: StxTokenArray; value: INTEGER; count: INTEGER): BOOLEAN; 
VAR
    i: INTEGER;
BEGIN
    FOR i := 0 TO count-1 DO
        IF  tokenArray[i] = value THEN EXIT(TRUE);
    arrayContains := FALSE;
END;

VAR
    count : INTEGER;
    t:INTEGER;
    value:TSTACK;
    validTokens: StxTokenArray;
    i: INTEGER;
BEGIN
    
    StxInit;
    
    i := 0;
    while parameters[i].token <> -1 do
          BEGIN
          validTokens := StxValidTokens(count);
          IF   NOT arrayContains(validTokens, parameters[i].token, count)
          THEN BEGIN
               writeln('Token ', parameters[i].token, ' ain''t there');
               HALT (1);
               END;
          value.number := parameters[i].value;
          IF   StxParse(parameters[i].token, value) = INTERNAL_ERROR
          THEN BEGIN
               writeln('Error when parsing symbol ', parameters[i].token);
               HALT (2);
               END;
          t := GetTotal();
          IF   t <> parameters[i].result
          THEN BEGIN
               writeln('Result is not ', parameters[i].result);
               HALT (3);
               END;
          i := i + 1;
          END;
    t := GetTotal();
    writeln('Total=', t);
    IF   t <> -17 
    THEN BEGIN
         writeln('total does not match');
         HALT (4);
         END;
END.
{$ELSE}
BEGIN
    IF   StxParse() THEN writeln('Total=', GetTotal);
END.
{$ENDIF}
```

##### <a name="struct-javascript">Javascript example</a>
```
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

%lexer = {
    while ($c <= ' ' && $c > '\0') {
        $+;
    }
}

%left             TOK_AND      256:"AND"       = if ($c == '&') {$+; return TOK_AND.charCodeAt(0);};
%left             TOK_OR       257:"OR"        = if ($c == '|') {$+; return TOK_OR.charCodeAt(0);}; 
%right            TOK_NOT      258:"NOT"       = if ($c == '!') {$+; return TOK_NOT.charCodeAt(0);};
%left             TOK_LE       259:"'<='" /<=/ = return TOK_LE.charCodeAt(0);
     ,            TOK_LT       260:"'<'"  /</  = return TOK_LT.charCodeAt(0);
     ,            TOK_GE       261:"'>='" />=/ = return TOK_GE.charCodeAt(0);
     ,            TOK_GT       262:"'>'"  />/  = return TOK_GT.charCodeAt(0);
     ,            TOK_NE       263:"'<>'" /<>/ = return TOK_NE.charCodeAt(0);
     ,            TOK_EQ       264:"'='"  /=/  = return TOK_EQ.charCodeAt(0);

%left             '+' : "plus"                 = if ($c == '+') {$+; return '+'.charCodeAt(0);};
                , '-' : '"minus"'              = if ($c == '-') {$+; return '-'.charCodeAt(0);};
%left             '*'                          = if ($c == '*') {$+; return '*'.charCodeAt(0);};
                , '/'                          = if ($c == '/') {$+; return '/'.charCodeAt(0);};

%right            TOK_UMINUS:"unary minus"  

%token            '('                          = if ($c == '(') {$+; return '('.charCodeAt(0);};
                , ')'                          = if ($c == ')') {$+; return ')'.charCodeAt(0);};

%token <number> TOK_NUMBER : "number" /[0-9]*/ = {
    $v.number = parseInt($r);
    return TOK_NUMBER;
}

%type    <number> Expression

%name Expression : "expression";

%start Expression

%lexer = {
}

%group OPS : "operator" TOK_AND, TOK_OR, TOK_LT, TOK_LE, TOK_GT, TOK_GE, TOK_NE, TOK_EQ, '+', '-', '*', '/';

%%
Expression   :  Expression TOK_AND Expression = $$ = ($1 != 0) && ($3 != 0) ? 1 : 0;
             |  Expression TOK_OR Expression  = $$ = ($1 != 0) || ($3 != 0) ? 1 : 0;
             |  TOK_NOT Expression            = $$ = ($2 != 0) ? 0 : 1;
             |  Expression TOK_LE Expression  = $$ = $1 <= $3 ? 1 : 0;
             |  Expression TOK_LT Expression  = $$ = $1 < $3 ? 1 : 0;
             |  Expression TOK_GE Expression  = $$ = $1 >= $3 ? 1 : 0;
             |  Expression TOK_GT Expression  = $$ = $1 > $3 ? 1 : 0;
             |  Expression TOK_NE Expression  = $$ = $1 != $3 ? 1 : 0;
             |  Expression TOK_EQ Expression  = $$ = $1 == $3 ? 1 : 0;
             |  Expression '+' Expression     = $$ = $1 + $3;
             |  Expression '-' Expression     = $$ = $1 - $3;
             |  Expression '*' Expression     = $$ = $1 * $3;
             |  Expression '/' Expression     = $$ = $1 / $3;
             |  '-' Expression %prec TOK_UMINUS = $$ = -$2;
             |  '(' Expression ')'            = $$ = $2;
             |  TOK_NUMBER
             ;
%%

// END OF GRAMMAR

    var charNum = 0;
    var expression = "(1 + 3) *4 / 5 + 20";

    function getNextChar(initialize) {
        if (initialize) {
            charNum = 0;
        }
        if (charNum < expression.length) {
            return expression.charAt(charNum++);
        }
        return '\0';
    }

    function ungetChar(c) {
        charNum --;
        if (charNum < 0 || expression.charAt(charNum) != c) {
            throw new Error("Error putting a character back");
        }
    }

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
        scanner: parserElement,
        parse: parse,
        scanner: function (token, value) {
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
        dumpTokens: dumpTokens,
        init: init,
        getValidTokens: getValidTokens,
        getResult: getResult
    }
 
})();

var calculate = function () {
    calc.parse();
    return 'The result is ' + calc.getResult().number;
}

// Scanner based routines
var init = function () {
    calc.init();
}

var parse = function(token, value) {
    return calc.scanner(token, value);
}

var getValidTokens = function() {
    return calc.getValidTokens().sort(function (a, b) {
        return a - b;
    }).join(",");
}

var getResult = function() {
    return calc.getResult().number;
}
```

</div>
</div>