---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">

#### Lexic driven parsers

---

Compiler driven parsers are the standard. They get created, invoked with an input stream, which is then read one character at a time, checking for grammar compliance and generating code and other structures as a result.

Lexical driven parsers differ in the fact that the input stream is discontinuous, usually user driven. The parser gets created and then waits for an input to come as a method call. The parser will receive the input and check it for correctness, causing shifts and reduces with the given symbol. When a state is reached that requires a new symbol, the parser will stop, store its state, and return the method call.

As part of keeping state, a lexical driven parser will be able to inform what possible symbols are available in the next method call, thus allowing user interfaces to enable/disable symbols. The typical case that I have encountered in the past was a calculator with operators, numbers and parenthesis. The buttons get enabled only on the presence of valid symbol transitions on the current parser state. Consider the following calculator:

Given the calculator UI:
<pre><code>  [1] [2] [3]  [+]  
  [4] [5] [6]  [-]  
  [7] [8] [9]  [*]  
  [(] [0] [)]  [/]  
        [ = ]  </code></pre>
Consider the following two states in an expression grammar:
<pre><code>State 1:  
S -&gt;  E .  
E -&gt; E . + E  
E -&gt; E . - E  
E -&gt; E . * E  
E -&gt; E . / E  

State 2:  
E -&gt; (  E . )  
E -&gt; E . + E  
E -&gt; E . - E  
E -&gt; E . * E  
E -&gt; E . / E  </code></pre>
State 1 can be seen as allowing an operator or the end of the input while state 2 shows that a parenthesis was used and thus a closing parenthesis is valid, but not the end of the input. Lets show this in the calculator diagrams. Invalid tokens in a state will be shown as periods.

For state 1:
<pre><code> .   .   .   [+]  
 .   .   .   [-]  
 .   .   .   [*]  
 .   .   .   [/]  
      [ = ]  </code></pre>
And for state 2:
<pre><code> .   .   .   [+]  
 .   .   .   [-]  
 .   .   .   [*]  
 .   .  [)]  [/]  
       ...    </code></pre>
Other examples have included programming languages based on button interfaces like the ones used back in the old macintosh days (Silver Surfer, or 4D, by Guy Kawasaki)
</div>
</div>