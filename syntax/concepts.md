---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">

#### Concepts
---

* <a href="#concept-lexer">Lexic</a>
* <a href="#concept-rules">Syntax</a>
* <a href="#concept-context-free-language">Context Free Languages</a>
* <a href="#concept-top-bottom">Top-down, Bottom-up</a>
* <a href="#concept-context-free-grammar">Context Free Grammar</a>
* <a href="#concepts-algorithms">Algorithms</a>

A language is defined by its **lexic** and its **syntax**. 

##### <a name="concepts-lexer">Lexic</a>

The lexic is the vocabulary of the language. For instance, in english, the vocabulary includes pronouns, nouns, verbs, numbers, etc. The lexic of a language is defined by tokens and either a regular expression pattern, or a simple code snippet. 

* <a href="{{ site.baseurl }}/syntax/lexer">Lexer</a>

##### <a name="concept-rules">Syntax</a>

The grammar of a language is the set of rules that allow the construction of valid sentences. For instance in english, you can have a rule that says that a sentence can be a pronoun followed by a verb, then followed by a noun. That would make "I like pizza" valid, but if that is the only rule, then "I pizza like" would result in a syntax error.

* <a href="{{ site.baseurl }}/syntax/rules">Production Rules</a>

##### <a name="concept-context-free-language">Context Free Languages</a>

Writing computer languages, unlike spoken languages, requires a great deal of disambiguation. As such, computer languages are based on context free grammars where a parse tree can be constructed without ambiguities. Context free grammars are production-rule based. Depending on the algorithm utilized to construct the parser, a grammar can be considered context free, while other algorithms disallow that.

##### <a name="concept-top-bottom">Top Down Parsing vs. Bottom Up Parsing</a>

Systems like javacc and antlr produce top down parsers. What that means is that there is an algorithmic piece of code for each rule, usually in the form of loops and conditionals. A typical top down parser is a series of subroutines generated in a predeterminated set of patterns. Thus the "top-down" moniker.

Bottom up parsing is usually driven by a set of tables and a state engine, and a single predefined routine that changes states based on the token received. The main table is a set of rows that represent the states, and a set of columns representing the tokens and other non terminals. The cells in the table represent what is to be done on a given token. It can be: an error, a transition to a new state (**shift**), a recognition of a given rule (**reduce**) or an accept of the program code. The main parsing table is usually a sparse matrix, and thus a compressed format is usually used. yacc and Syntax produce bottom up parsers.

##### <a name="concept-context-free-grammar">Context Free Grammars</a>

A context free grammar is defined with (V, &Sigma;, R, S)

V is the vocabulary, or the set of non-terminals, like if-statement, or while-loop, etc. 
&Sigma; is the set of terminals, or tokens, of the language as extracted/produced by the scanner, or lexer.
R is a set of rule productions.
S is the starting v &epsilon; V

Production rules look like the following:

```
A : A B  = $$ = $A + $B;
  | B    = $$ = $B;
  ;

B : aa
  | bb
  | cc
  ;
```

The symbols on the left side are non terminals (&epsilon; V). The right side represents other symbols (terminal and non-terminal) representing the strucure. Code can be placed at the extreme right with equals signs and '{', which is code to be executed when a rule is recognized.

##### <a name="concepts-algorithms">Algorithms</a>

There are multiple algorithms used to produce a parsing table: LR(1), LALR(1), SLR(1), Honalee, and others. Some grammars are LR(1), but not LALR(1) or SLR(1). A grammar can be LALR(1), and by consequence it would be a LR(1) grammar as well. SLR(1) Grammars in turn are both LALR(1) and LR(1). yacc accepts LALR(1) grammars, while sytax supports LALR(1) and SLR(1). SLR(1) tables are typically smaller. Interestingly, yacc produces a set of packed tables that, at the end, behave like if they had been produced by SLR(1) algorithms. In the case of syntax you can usually start with SLR(1) and if you get reduce/reduce conflicts, you can try with LALR(1). Your tables will be smaller that way.

</div>
</div>