---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">

#### Error Management
---

* <a href="#errors-tokens">Error Tokens</a>
* <a href="#errors-csample">Errors: C Example</a>
* <a href="#errors-pascalsample">Errors: Pascal Example</a>

##### <a name="errors-tokens">Error Tokens</a>

Unlike top down parsers, it is actually quite easy to produce a recoverable parser. A recoverable parser is a parser that can report on one error, backtrack states and forward tokens until a valid transition can be made, and proceed from there either producing more errors or continuing to the end.

The first step in syntax for error recovery process is declaring %error tokens. Unlike yacc and bison (which predeclare an error token by default,) syntax requires you to declare your error tokens with %error. There is a reason for it: error messages. Let's work through an example

```
%token '{'     : "opening curly brace"
%token '}'     : "closing curly brace"
%token ';'     : "semicolon"
%token '('     : "opening parenthesis"
%token ')'     : "closing parenthesis"

%left '+'      : "addition"
    , '-'      : "subtraction"
%left '*'      : "multiplication"
    , '/'      : "division"
%left unary

%token eq      : "=="
%token ne      : "!="

%token _if     : "if"
%token _while  : "while"

%token <n> num : "number"

%name Block          : "a block"
%name StatementList  : "a statement list"
%name Statement      : "a statement"
%name IfStatement    : "an if statement"
%name WhileStatement : "a while loop"
%name Condition      : "a condition"
%name Expression     : "an expression"

%start Block

%%

Block         : '{' StatementList '}'
              ;

StatementList : StatementList Statement
              | 
              ;

Statement     : IfStatement
              | WhileStatement
              | Expression ';'
              ;

IfStatement   : _if Condition Block
              ;

WhileStatement: _while Condition Block
              ;

Condition     : '(' Expression eq Expression ')'
              | '(' Expression ne Expression ')'
              ;

Expression    : Expression '+' Expression
              | Expression '-' Expression
              | Expression '*' Expression
              | Expression '/' Expression
              | '-' Expression %prec unary
              | '(' Expression ')'
              | num
              ;

%%
```

We have declared our tokens, now let's add a lexer:

```
%lexer = {
  while ($c <= ' ' && $c > '\0') {
    $+;
  }
}

%token '{'     : "opening curly brace"
%token '}'     : "closing curly brace"
%token ';'     : "semicolon"
%token '('     : "opening parenthesis"
%token ')'     : "closing parenthesis"

%left '+'      : "addition"
    , '-'      : "subtraction"
%left '*'      : "multiplication"
    , '/'      : "division"
%left unary

%token eq      : "==" /==/ = return eq;
%token ne      : "!=" /!=/ = return ne;

%token _if     : "if" /if/ = return _if;
%token _while  : "while" /while/ = return _while;

%token <n> num : "number" /[0-9]*/ = {
  return num;
}

%lexer = {
  if ("{};()+-*/".indexOf($c) >= 0) {
    char c = $c;
    $+;
    return c;
  }
}

%name Block          : "a block"
%name StatementList  : "a statement list"
%name Statement      : "a statement"
%name IfStatement    : "an if statement"
%name WhileStatement : "a while loop"
%name Condition      : "a condition"
%name Expression     : "an expression"

%start Block

%%

Block         : '{' StatementList '}'
              ;

StatementList : StatementList Statement
              | 
              ;

Statement     : IfStatement
              | WhileStatement
              | Expression ';'
              ;

IfStatement   : _if Condition Block
              ;

WhileStatement: _while Condition Block
              ;

Condition     : '(' Expression eq Expression ')'
              | '(' Expression ne Expression ')'
              ;

Expression    : Expression '+' Expression
              | Expression '-' Expression
              | Expression '*' Expression
              | Expression '/' Expression
              | '-' Expression %prec unary
              | '(' Expression ')'
              | num
              ;

%%

```

As you can see, we have defined a language that supports ifs, whiles and expressions. For instance, the following code is accepted by the previous grammar:

```
{
  if (2==3) {
    (1 + 3) * 4 / 5 + 20;
    22 + -3;
  }
  while (2 == 3) {
    2*2;
  }
}
```

We can now add our java code around it, to a more complete program. Please note that we have added our error routine as defined in the following table

**Language**   | **Routine**
---------------|:------------------------
Java           | parserError
C              | StxError
Javascript     | parserError
Pascal         | StxError

```
%{
public class Statement {
%}

%lexer = {
  while ($c <= ' ' && $c > '\0') {
    $+;
  }
}

%token '{'     : "opening curly brace"
%token '}'     : "closing curly brace"
%token ';'     : "semicolon"
%token '('     : "opening parenthesis"
%token ')'     : "closing parenthesis"

%left '+'      : "addition"
    , '-'      : "subtraction"
%left '*'      : "multiplication"
    , '/'      : "division"
%left unary

%token eq      : "==" /==/ = return eq;
%token ne      : "!=" /!=/ = return ne;

%token _if     : "if" /if/ = return _if;
%token _while  : "while" /while/ = return _while;

%token <n> num : "number" /[0-9]*/ = {
  return num;
}

%lexer = {
  if ("{};()+-*/".indexOf($c) >= 0) {
    char c = $c;
    $+;
    return c;
  }
}

%name Block          : "a block"
%name StatementList  : "a statement list"
%name Statement      : "a statement"
%name IfStatement    : "an if statement"
%name WhileStatement : "a while loop"
%name Condition      : "a condition"
%name Expression     : "an expression"

%start Block

%%

Block         : '{' StatementList '}'
              ;

StatementList : StatementList Statement
              | 
              ;

Statement     : IfStatement
              | WhileStatement
              | Expression ';'
              ;

IfStatement   : _if Condition Block
              ;

WhileStatement: _while Condition Block
              ;

Condition     : '(' Expression eq Expression ')'
              | '(' Expression ne Expression ')'
              ;

Expression    : Expression '+' Expression
              | Expression '-' Expression
              | Expression '*' Expression
              | Expression '/' Expression
              | '-' Expression %prec unary
              | '(' Expression ')'
              | num
              ;

%%

  int charNum = 0;

  static String program = 
  "{\n" +
  "  if (2==3) {\n" +
  "    (1 + 3) * 4 / 5 + 20;\n" +
  "    22 + -3;\n" +
  "  }\n" +
  "  while (2 == 3) {\n" +
  "    2*2;\n" +
  "  }\n" +
  "}\n";

  private char getNextChar(boolean initialize) {
    if (initialize) {
      charNum = 0;
    }
    if (charNum < program.length()) {
      return program.charAt(charNum++);
    }
    return EOS;
  }

  private void ungetChar(char c) {
    charNum --;
    if (charNum < 0 || program.charAt(charNum) != c) {
      throw new RuntimeException("Error putting a character back");
    }
  }

  private int parserError(int state, int token, int top, String message) {
    System.out.println("error near " + getTokenName(token) + ": " + message);
    return ERROR_RE_ATTEMPT;
  }
 
  public static void main(String ... args) {
    Statement stmt = new Statement();
    if (args.length > 0 && args[0].equals("-v")) {
      stmt.setVerbose(true); 
    }
    stmt.parse();
  }
}
```

The return code of the error routine parserError is meaningful. Returning **ERROR_FAIL** will stop the parsing. You need to return **ERROR_RE_ATTEMPT**. This will kick in the first part of the recovery process.

You can now compile and execute your code as follows

```
java -jar syntax-<version>.jar -l java Statement.sy
javac Statement.java
java -classpath . Statement.
```

Now we are ready to add error recovery to our grammar. The next step is to declare one or more %error tokens to be used. These tokens will be later used in rules to show error structure.

```
%error statement-error : "Statement error ($m)"
%error condition-error : "Malformed condition ($m)"
```

These errors indicate somehow that they will be used to identify bad statements and bad conditions. They contain the error message. In addition, they contain a token $m, which stands for message. The message is a default generic error message that syntax produced based on "expected ..." text. This default generic message is per state, generated based on each state's possitive transitions to tokens and non terminals in a succint algorithm. And with such, you can either have an error message, plus the state's error message. Please also remember that you can use %group to group tokens to have the state's default error message be more succint. For instance, you can group operators:


```
...

%group op : "operator" '+', '-', '*', '/'

...
```

These %errors can now be used in production rules. For instance

```
Statement     : IfStatement
              | WhileStatement
              | Expression ';'
              | statement-error ';'
              ;

...

Condition     : '(' Expression eq Expression ')'
              | '(' Expression ne Expression ')'
              | '(' condition-error ')'
              ;
```

And now we can use a program with invalid syntax, as follows:

```
  static String program = 
  "{\n" +
  "  if (2==3) {\n" +
  "    (1 + 3) **4 / 5 + 20;\n" +
  "    22 + -3;\n" +
  "  }\n" +
  "  while (2 * 3) {\n" +
  "    2*2;\n" +
  "  }\n" +
  "}\n";

```

Notice the text around &#42;&#42;4 and in the while condition where we are missing either == or !=.

Syntax will now report errors as follows:

```
error near *: Statement error (Expecting an expression)
error near ): Malformed condition (addition, subtraction, multiplication, division, == or != expected)
```

These messages contain our error message with the state's error message in parenthesis.

What syntax does with statement-error is find that &#42;&#42; token and raise an error message. It then proceeds backtracking states until it finds a state in which statement-error is a transition. Now we are in a good state. Syntax then proceeds to introduce the token statement-error as a valid transition, after which a semicolon will be valid. It then discards tokens forward until it sees that semicolon. And now we have a "valid" Statement recognized albeit an error.

Please note that you could also add code to such error rule. These can be used to free and undo code generated, symbol table entries, etc.

##### <a name="errors-csample">Errors: C Sample</a>

This is the example in C

```
%{
#include <stdio.h>
#include <string.h>
#include "$$b.h"
%}

%lexer = {

  char temp_c;

  while ($c <= ' ' && $c > '\0') {
    $+;
  }
}

%token '{'     : "opening curly brace"
%token '}'     : "closing curly brace"
%token ';'     : "semicolon"
%token '('     : "opening parenthesis"
%token ')'     : "closing parenthesis"

%left '+'      : "addition"
    , '-'      : "subtraction"
%left '*'      : "multiplication"
    , '/'      : "division"
%left unary

%token eq      : "==" /==/ = return eq;
%token ne      : "!=" /!=/ = return ne;

%token _if     : "if" /if/ = return _if;
%token _while  : "while" /while/ = return _while;

%token <n> num : "number" /[0-9]*/ = {
  return num;
}

%lexer = {
  if (strchr("{};()+-*/", $c) != NULL) {
    temp_c = $c;
    $+;
    return temp_c;
  }
}

%name Block          : "a block"
%name StatementList  : "a statement list"
%name Statement      : "a statement"
%name IfStatement    : "an if statement"
%name WhileStatement : "a while loop"
%name Condition      : "a condition"
%name Expression     : "an expression"

%error statement-error : "Statement error ($m)"
%error condition-error : "Malformed condition ($m)"

%start Block

%%

Block         : '{' StatementList '}'
              ;

StatementList : StatementList Statement
              | 
              ;

Statement     : IfStatement
              | WhileStatement
              | Expression ';'
              | statement-error ';'
              ;

IfStatement   : _if Condition Block
              ;

WhileStatement: _while Condition Block
              ;

Condition     : '(' Expression eq Expression ')'
              | '(' Expression ne Expression ')'
              | '(' condition-error ')'
              ;

Expression    : Expression '+' Expression
              | Expression '-' Expression
              | Expression '*' Expression
              | Expression '/' Expression
              | '-' Expression %prec unary
              | '(' Expression ')'
              | num
              ;

%%

int charNum = 0;

char * program = 
  "{\n"
  "  if (2==3) {\n"
  "    (1 + 3) **4 / 5 + 20;\n"
  "    22 + -3;\n"
  "  }\n"
  "  while (2 * 3) {\n"
  "    2*2;\n"
  "  }\n"
  "}\n";

char StxNextChar()
{
    if (charNum < strlen(program)) {
      return program[charNum++];
    }
    return EOS;
}

void StxUngetChar(char c) {
    charNum --;
}

int StxError(int state, int token, int top, char * message)
{
    printf("error near %s: %s\n", StxGetTokenName(token), message);
#ifdef DEBUG
    StxPrintStack();
#endif
    return ERROR_RE_ATTEMPT;
}

 
int main(int argc, char * argv[]) {
    StxParse();
}
```

##### <a name="errors-pascalsample">Errors: Pascal Sample</a>

This is the example in Pascal

```
%{
{Program that gets compiled and executed by free pascal}
PROGRAM pascaltest;
USES sysutils;
{$I $$b.inc}

VAR
  temp_c : CHAR;
%}

%lexer =
  while ($c <= ' ') AND ($c > CHR(0))  DO $+;
%

%token '{'     : "opening curly brace"
%token '}'     : "closing curly brace"
%token ';'     : "semicolon"
%token '('     : "opening parenthesis"
%token ')'     : "closing parenthesis"

%left '+'      : "addition"
    , '-'      : "subtraction"
%left '*'      : "multiplication"
    , '/'      : "division"
%left unary

%token eq      : "==" /==/ = $x(eq);%
%token ne      : "!=" /!=/ = $x(ne);%

%token _if     : "if" /if/ = $x(_if);%
%token _while  : "while" /while/ = $x(_while);%

%token <n> num : "number" /[0-9]*/ =
  $x(num);
%

%lexer =
  IF   POS($c, '{};()+-*/') > 0
  THEN BEGIN
       temp_c := $c;
        $+;
        $x(ORD(temp_c));
  END;
%

%name Block          : "a block"
%name StatementList  : "a statement list"
%name Statement      : "a statement"
%name IfStatement    : "an if statement"
%name WhileStatement : "a while loop"
%name Condition      : "a condition"
%name Expression     : "an expression"

%error statement-error : "Statement error ($m)"
%error condition-error : "Malformed condition ($m)"

%start Block

%%

Block         : '{' StatementList '}'
              ;

StatementList : StatementList Statement
              | 
              ;

Statement     : IfStatement
              | WhileStatement
              | Expression ';'
              | statement-error ';'
              ;

IfStatement   : _if Condition Block
              ;

WhileStatement: _while Condition Block
              ;

Condition     : '(' Expression eq Expression ')'
              | '(' Expression ne Expression ')'
              | '(' condition-error ')'
              ;

Expression    : Expression '+' Expression
              | Expression '-' Expression
              | Expression '*' Expression
              | Expression '/' Expression
              | '-' Expression %prec unary
              | '(' Expression ')'
              | num
              ;

%%

VAR
    charNum : INTEGER = 1;

CONST
    program_code = 
  '{'#10 +
  '  if (2==3) {'#10 +
  '    (1 + 3) **4 / 5 + 20;'#10 +
  '    22 + -3;'#10 +
  '  }'#10 +
  '  while (2 * 3) {'#10 +
  '    2*2;'#10 +
  '  }'#10 +
  '}'#10;

FUNCTION StxNextChar: CHAR;
BEGIN
    IF   charNum <= LENGTH(program_code) 
    THEN BEGIN
         StxNextChar := program_code[charNum];
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
    writeln('error near ', StxGetTokenName(StxSym), ': ', aMessage);
{$IFDEF DEBUG}
    StxPrintStack();
{$ENDIF}
    StxError := ERROR_RE_ATTEMPT;
END;

FUNCTION StxToString(value:TSTACK):STRING;
BEGIN
  StxToString := IntToStr(value);
END;

BEGIN
    StxParse();
END.
```
</div>
</div>