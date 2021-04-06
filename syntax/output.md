---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">

<style type="text/css">
  .left {
    text-align:left;
  }

  .right {
    text-align:left;
  }
  
  .heading {
    font-family: "arial", sans-serif;
    font-weight: bold;
    font-size: 16pt;
    color: #0050B2;
    margin-top:14px;
    padding-top:4px;
    padding-bottom: 4px;
  }
  
  .subheading, .title {
    font-family: "arial", sans-serif;
    font-weight: bold;
    font-size: 11pt;
    background-color:#deb887;
    margin-top:14px;
    padding-top:4px;
    padding-bottom: 4px;
    padding-left: 4px;
    border-top-left-radius: 5px;
    border-top-right-radius: 5px;
  }
  
  .close {
    background-color:#DEE8FC;
    font-family: "arial", sans-serif;
    font-weight: bold;
    font-size: 6pt;
    margin-bottom:5px;
    border-bottom-left-radius: 5px;
    border-bottom-right-radius: 5px;
  }
  
  .subtitle {
    font-family: "arial", sans-serif;
    font-weight: normal;
    font-size: 10pt;
  	background-color: #ffc;
  	padding-left: 4px;
  }
  
  .subtitle span {
    font-style: italic;
  }
  
  table {
    border-collapse: collapse;
    border: 1px solid #ddd;
    width: 100%;
  }
  
  th, .thead {
    font-family: "arial", sans-serif;
    font-weight: bold;
    font-size: 10pt;
    background-color:#9FB4D9;
    border: 1px solid #ddd;
  }
  
  td {
    font-family: "arial", sans-serif;
    font-weight: normal;
    font-size:10pt;
    background-color:#DEE8FC;
    vertical-align: top;
    border: 1px solid #ddd;
  }
  
  hr {
    border: 0;
    border-bottom: 1px;
    background: #333;
    height: 1px;
  }
  
  .dot {
    font-family: "arial", sans-serif;
    font-weight: bold;
    color: #EA1881;
  }
  
  .error-message {
    color: red;
  }
  
  .error-type {
    color: #999;
  }

  .same-action {
    color: #eee;
  }

  .g-starting-node {
    fill:rgb(255,0,0);
    stroke-width=0;
  }
  
  .g-node {
    fill:rgb(0,0,0);
    stroke-width=0;
  }
  
  .g-node-text {
    font-size:8pt;
    stroke:rgb(0,0,255);
    stroke-width:1;
    font-weight:normal;
  }
  
  .g-accept {
    fill:none;
    stroke-width:2;
    stroke:rgb(255,255,255);
  }
  
  .g-self-loop {
    fill:none;
    stroke-width:1;
    stroke:rgb(0,0,0);
  }
  
  .g-transition {
    fill:none;
    stroke:rgb(0,0,0);
    stroke-width:1;
  }
  
  .g-transition-text {
    font-size:8pt;
  }
</style>

#### SYNTAX
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

#### OUTPUT

<div class="heading">javascript-test.sy</div>

<div class="subheading">Terminal Symbols</div>
  <table class="symbols">
    <tr><th class="right">ID</th><th class="left">Name</th><th class="left">Full Name</th><th class="right">Value</th><th class="left">Err</th><th class="right">Refs</th><th class="right">Prec</th><th class="left">Assc</th><th class="left">Type</th></tr>
    <tr><td class="right">0</td><td class="left">$</td><td class="left">$</td><td class="right">0</td><td class="left">No </td><td class="right">0</td><td class="right">0</td><td class="left">N/A</td><td class="left"></td></tr>
    <tr><td class="right">1</td><td class="left">TOK_AND</td><td class="left">AND</td><td class="right">256</td><td class="left">No </td><td class="right">1</td><td class="right">1</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">2</td><td class="left">TOK_OR</td><td class="left">OR</td><td class="right">257</td><td class="left">No </td><td class="right">1</td><td class="right">2</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">3</td><td class="left">TOK_NOT</td><td class="left">NOT</td><td class="right">258</td><td class="left">No </td><td class="right">0</td><td class="right">3</td><td class="left">RIG</td><td class="left"></td></tr>
    <tr><td class="right">4</td><td class="left">TOK_LE</td><td class="left">'<='</td><td class="right">259</td><td class="left">No </td><td class="right">1</td><td class="right">4</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">5</td><td class="left">TOK_LT</td><td class="left">'<'</td><td class="right">260</td><td class="left">No </td><td class="right">1</td><td class="right">4</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">6</td><td class="left">TOK_GE</td><td class="left">'>='</td><td class="right">261</td><td class="left">No </td><td class="right">1</td><td class="right">4</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">7</td><td class="left">TOK_GT</td><td class="left">'>'</td><td class="right">262</td><td class="left">No </td><td class="right">1</td><td class="right">4</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">8</td><td class="left">TOK_NE</td><td class="left">'<>'</td><td class="right">263</td><td class="left">No </td><td class="right">1</td><td class="right">4</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">9</td><td class="left">TOK_EQ</td><td class="left">'='</td><td class="right">264</td><td class="left">No </td><td class="right">1</td><td class="right">4</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">10</td><td class="left">+</td><td class="left">plus</td><td class="right">43</td><td class="left">No </td><td class="right">1</td><td class="right">5</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">11</td><td class="left">-</td><td class="left">"minus"</td><td class="right">45</td><td class="left">No </td><td class="right">2</td><td class="right">5</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">12</td><td class="left">*</td><td class="left">*</td><td class="right">42</td><td class="left">No </td><td class="right">1</td><td class="right">6</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">13</td><td class="left">/</td><td class="left">/</td><td class="right">47</td><td class="left">No </td><td class="right">1</td><td class="right">6</td><td class="left">LEF</td><td class="left"></td></tr>
    <tr><td class="right">14</td><td class="left">TOK_UMINUS</td><td class="left">unary minus</td><td class="right">32768</td><td class="left">No </td><td class="right">0</td><td class="right">7</td><td class="left">RIG</td><td class="left"></td></tr>
    <tr><td class="right">15</td><td class="left">(</td><td class="left">(</td><td class="right">40</td><td class="left">No </td><td class="right">0</td><td class="right">0</td><td class="left">N/A</td><td class="left"></td></tr>
    <tr><td class="right">16</td><td class="left">)</td><td class="left">)</td><td class="right">41</td><td class="left">No </td><td class="right">0</td><td class="right">0</td><td class="left">N/A</td><td class="left"></td></tr>
    <tr><td class="right">17</td><td class="left">TOK_NUMBER</td><td class="left">number</td><td class="right">32769</td><td class="left">No </td><td class="right">0</td><td class="right">0</td><td class="left">N/A</td><td class="left">number</td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="subheading">Non Terminal Symbols</div>
  <table class="symbols">
    <tr><th class="right">ID</th><th class="left">Name</th><th class="left">FullName</th><th class="right">Refs</th><th class="left">Type</th></tr>
    <tr><td class="right">18</td><td class="left">Expression</td><td class="left">expression</td><td class="right">27</td><td class="left">number</td></tr>
    <tr><td class="right">19</td><td class="left">$start</td><td class="left">$start</td><td class="right">0</td><td class="left"></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="subheading">Types</div>
  <table class="symbols">
    <tr><th class="left">Name</th><th class="left">Used By</th></tr>
    <tr><td class="left">number</td><td class="left"><ul><li>TOK_NUMBER[id=17]</li>
<li>Expression[id=18]</li>
</ul></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="subheading">Error Groups</div>
  <table class="symbols">
    <tr><th class="left">Name</th><th class="left">Display Name</th><th class="left">Symbols</th></tr>
    <tr><td class="left">OPS</td><td class="left">operator</td><td class="left"><ul><li>TOK_AND[id=1]</li><li>TOK_OR[id=2]</li><li>TOK_LT[id=5]</li><li>TOK_LE[id=4]</li><li>TOK_GT[id=7]</li><li>TOK_GE[id=6]</li><li>TOK_NE[id=8]</li><li>TOK_EQ[id=9]</li><li>+[id=10]</li><li>-[id=11]</li><li>*[id=12]</li><li>/[id=13]</li></ul></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="subheading">Lexer Modes</div>
  <table class="lexermodes">
    <tr><th class="left">Name</th><th class="left">Routine</th></tr>
    <tr><td class="left">default</td><td class="left">parserElement_default()</td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="subheading">Regular Expressions</div>
  <table class="lexermodes">
    <tr><th class="left">Expression</th><th class="left">Graph</th></tr>
    <tr><td class="left">/<=/<br/>{*0 '<'->1}<br/>
{1 '='->2}<br/>
{(2)}<br/>
</td><td class="left"><svg width="225" height="225">
  <path class="g-transition" d="M156.38 63.49 L107.62 96.513215"/>
  <polygon class="g-transition-head" points="115.36,87.08 107.62,96.51 119.25,92.83"/>
  <text class="g-transition-text" x="132.00" y="90.00" alignment-baseline="middle" text-anchor="start"><</text>
  <circle class="g-starting-node" cx="163.00" cy="59.00" r="8.000000"/>
  <text class="g-node-text" x="163.00" y="73.00" alignment-baseline="middle" text-anchor="middle">0</text>
  <path class="g-transition" d="M94.09 105.04 L42.91 134.962427"/>
  <polygon class="g-transition-head" points="51.25,126.06 42.91,134.96 54.76,132.06"/>
  <text class="g-transition-text" x="68.50" y="130.00" alignment-baseline="middle" text-anchor="start">=</text>
  <circle class="g-node" cx="101.00" cy="101.00" r="8.000000"/>
  <text class="g-node-text" x="101.00" y="115.00" alignment-baseline="middle" text-anchor="middle">1</text>
  <circle class="g-node" cx="36.00" cy="139.00" r="8.000000"/>
  <circle class="g-accept" cx="36.00" cy="139.00" r="6.000000"/>
  <text class="g-node-text" x="36.00" y="153.00" alignment-baseline="middle" text-anchor="middle">2</text>
</svg>
</td></tr>
    <tr><td class="left">/</<br/>{*3 '<'->4}<br/>
{(4)}<br/>
</td><td class="left"><svg width="225" height="225">
  <path class="g-transition" d="M103.57 32.99 L96.43 166.011506"/>
  <polygon class="g-transition-head" points="93.59,154.15 96.43,166.01 100.52,154.52"/>
  <text class="g-transition-text" x="100.00" y="109.50" alignment-baseline="middle" text-anchor="start"><</text>
  <circle class="g-starting-node" cx="104.00" cy="25.00" r="8.000000"/>
  <text class="g-node-text" x="104.00" y="39.00" alignment-baseline="middle" text-anchor="middle">3</text>
  <circle class="g-node" cx="96.00" cy="174.00" r="8.000000"/>
  <circle class="g-accept" cx="96.00" cy="174.00" r="6.000000"/>
  <text class="g-node-text" x="96.00" y="188.00" alignment-baseline="middle" text-anchor="middle">4</text>
</svg>
</td></tr>
    <tr><td class="left">/>=/<br/>{*5 '>'->6}<br/>
{6 '='->7}<br/>
{(7)}<br/>
</td><td class="left"><svg width="225" height="225">
  <path class="g-transition" d="M38.38 73.10 L92.62 95.900323"/>
  <polygon class="g-transition-head" points="80.50,94.57 92.62,95.90 83.19,88.17"/>
  <text class="g-transition-text" x="65.50" y="94.50" alignment-baseline="middle" text-anchor="end">></text>
  <circle class="g-starting-node" cx="31.00" cy="70.00" r="8.000000"/>
  <text class="g-node-text" x="31.00" y="84.00" alignment-baseline="middle" text-anchor="middle">5</text>
  <path class="g-transition" d="M107.30 102.28 L161.70 126.721481"/>
  <polygon class="g-transition-head" points="149.61,125.10 161.70,126.72 152.46,118.76"/>
  <text class="g-transition-text" x="134.50" y="124.50" alignment-baseline="middle" text-anchor="end">=</text>
  <circle class="g-node" cx="100.00" cy="99.00" r="8.000000"/>
  <text class="g-node-text" x="100.00" y="113.00" alignment-baseline="middle" text-anchor="middle">6</text>
  <circle class="g-node" cx="169.00" cy="130.00" r="8.000000"/>
  <circle class="g-accept" cx="169.00" cy="130.00" r="6.000000"/>
  <text class="g-node-text" x="169.00" y="144.00" alignment-baseline="middle" text-anchor="middle">7</text>
</svg>
</td></tr>
    <tr><td class="left">/>/<br/>{*8 '>'->9}<br/>
{(9)}<br/>
</td><td class="left"><svg width="225" height="225">
  <path class="g-transition" d="M134.77 156.21 L64.23 42.792986"/>
  <polygon class="g-transition-head" points="73.35,50.89 64.23,42.79 67.45,54.56"/>
  <text class="g-transition-text" x="99.50" y="109.50" alignment-baseline="middle" text-anchor="end">></text>
  <circle class="g-starting-node" cx="139.00" cy="163.00" r="8.000000"/>
  <text class="g-node-text" x="139.00" y="177.00" alignment-baseline="middle" text-anchor="middle">8</text>
  <circle class="g-node" cx="60.00" cy="36.00" r="8.000000"/>
  <circle class="g-accept" cx="60.00" cy="36.00" r="6.000000"/>
  <text class="g-node-text" x="60.00" y="50.00" alignment-baseline="middle" text-anchor="middle">9</text>
</svg>
</td></tr>
    <tr><td class="left">/<>/<br/>{*10 '<'->11}<br/>
{11 '>'->12}<br/>
{(12)}<br/>
</td><td class="left"><svg width="225" height="225">
  <path class="g-transition" d="M38.38 73.10 L92.62 95.900323"/>
  <polygon class="g-transition-head" points="80.50,94.57 92.62,95.90 83.19,88.17"/>
  <text class="g-transition-text" x="65.50" y="94.50" alignment-baseline="middle" text-anchor="end"><</text>
  <circle class="g-starting-node" cx="31.00" cy="70.00" r="8.000000"/>
  <text class="g-node-text" x="31.00" y="84.00" alignment-baseline="middle" text-anchor="middle">10</text>
  <path class="g-transition" d="M107.30 102.28 L161.70 126.721481"/>
  <polygon class="g-transition-head" points="149.61,125.10 161.70,126.72 152.46,118.76"/>
  <text class="g-transition-text" x="134.50" y="124.50" alignment-baseline="middle" text-anchor="end">></text>
  <circle class="g-node" cx="100.00" cy="99.00" r="8.000000"/>
  <text class="g-node-text" x="100.00" y="113.00" alignment-baseline="middle" text-anchor="middle">11</text>
  <circle class="g-node" cx="169.00" cy="130.00" r="8.000000"/>
  <circle class="g-accept" cx="169.00" cy="130.00" r="6.000000"/>
  <text class="g-node-text" x="169.00" y="144.00" alignment-baseline="middle" text-anchor="middle">12</text>
</svg>
</td></tr>
    <tr><td class="left">/=/<br/>{*13 '='->14}<br/>
{(14)}<br/>
</td><td class="left"><svg width="225" height="225">
  <path class="g-transition" d="M134.77 156.21 L64.23 42.792986"/>
  <polygon class="g-transition-head" points="73.35,50.89 64.23,42.79 67.45,54.56"/>
  <text class="g-transition-text" x="99.50" y="109.50" alignment-baseline="middle" text-anchor="end">=</text>
  <circle class="g-starting-node" cx="139.00" cy="163.00" r="8.000000"/>
  <text class="g-node-text" x="139.00" y="177.00" alignment-baseline="middle" text-anchor="middle">13</text>
  <circle class="g-node" cx="60.00" cy="36.00" r="8.000000"/>
  <circle class="g-accept" cx="60.00" cy="36.00" r="6.000000"/>
  <text class="g-node-text" x="60.00" y="50.00" alignment-baseline="middle" text-anchor="middle">14</text>
</svg>
</td></tr>
    <tr><td class="left">/[0-9]*/<br/>{*15 [0-9]->16}<br/>
{(16) [0-9]->16}<br/>
</td><td class="left"><svg width="225" height="225">
  <path class="g-transition" d="M134.77 156.21 L64.23 42.792986"/>
  <polygon class="g-transition-head" points="73.35,50.89 64.23,42.79 67.45,54.56"/>
  <text class="g-transition-text" x="99.50" y="109.50" alignment-baseline="middle" text-anchor="end">[0-9]</text>
  <circle class="g-starting-node" cx="139.00" cy="163.00" r="8.000000"/>
  <text class="g-node-text" x="139.00" y="177.00" alignment-baseline="middle" text-anchor="middle">15</text>
  <circle class="g-self-loop" cx="53.66" cy="25.81" r="12.000000"/>
  <polygon class="g-self-loop-head" points="42.28,36.73 52.19,37.72 46.20,29.76"/>
  <text class="g-transition-text" x="44.68" y="11.38" alignment-baseline="middle" text-anchor="end">[0-9]</text>
  <circle class="g-node" cx="60.00" cy="36.00" r="8.000000"/>
  <circle class="g-accept" cx="60.00" cy="36.00" r="6.000000"/>
  <text class="g-node-text" x="60.00" y="50.00" alignment-baseline="middle" text-anchor="middle">16</text>
</svg>
</td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="subheading">Grammar</div>
  <table class="rules">
    <tr><th class="right">Prec</th><th class="right">Rule</th><th class="left">Grammar</th></tr>
    <tr><td class="right">0</td><td class="right">0</td><td class="left">$start &rArr; Expression </td></tr>
    <tr><td class="right">1</td><td class="right">1</td><td class="left">Expression &rArr; Expression TOK_AND Expression </td></tr>
    <tr><td class="right">2</td><td class="right">2</td><td class="left">Expression &rArr; Expression TOK_OR Expression </td></tr>
    <tr><td class="right">3</td><td class="right">3</td><td class="left">Expression &rArr; TOK_NOT Expression </td></tr>
    <tr><td class="right">4</td><td class="right">4</td><td class="left">Expression &rArr; Expression TOK_LE Expression </td></tr>
    <tr><td class="right">4</td><td class="right">5</td><td class="left">Expression &rArr; Expression TOK_LT Expression </td></tr>
    <tr><td class="right">4</td><td class="right">6</td><td class="left">Expression &rArr; Expression TOK_GE Expression </td></tr>
    <tr><td class="right">4</td><td class="right">7</td><td class="left">Expression &rArr; Expression TOK_GT Expression </td></tr>
    <tr><td class="right">4</td><td class="right">8</td><td class="left">Expression &rArr; Expression TOK_NE Expression </td></tr>
    <tr><td class="right">4</td><td class="right">9</td><td class="left">Expression &rArr; Expression TOK_EQ Expression </td></tr>
    <tr><td class="right">5</td><td class="right">10</td><td class="left">Expression &rArr; Expression + Expression </td></tr>
    <tr><td class="right">5</td><td class="right">11</td><td class="left">Expression &rArr; Expression - Expression </td></tr>
    <tr><td class="right">6</td><td class="right">12</td><td class="left">Expression &rArr; Expression * Expression </td></tr>
    <tr><td class="right">6</td><td class="right">13</td><td class="left">Expression &rArr; Expression / Expression </td></tr>
    <tr><td class="right">7</td><td class="right">14</td><td class="left">Expression &rArr; - Expression </td></tr>
    <tr><td class="right">0</td><td class="right">15</td><td class="left">Expression &rArr; ( Expression ) </td></tr>
    <tr><td class="right">0</td><td class="right">16</td><td class="left">Expression &rArr; TOK_NUMBER </td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="subheading">First of Non Terminals</div>
  <table class="firstfollow">
    <tr><th class="left">Non Terminal</th><th class="left">Set</th></tr>
    <tr><td class="left">Expression</td><td class="left"><ul><li>TOK_NOT(3)</li>
<li>-(11)</li>
<li>((15)</li>
<li>TOK_NUMBER(17)</li>
</ul></td></tr>
    <tr><td class="left">$start</td><td class="left"><ul><li>TOK_NOT(3)</li>
<li>-(11)</li>
<li>((15)</li>
<li>TOK_NUMBER(17)</li>
</ul></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="subheading">Follow of Non Terminals</div>
  <table class="firstfollow">
    <tr><th class="left">Non Terminal</th><th class="left">Set</th></tr>
Follow of Expression
    <tr><td class="left">Expression</td><td class="left"><ul><li>$(0)</li>
<li>TOK_AND(1)</li>
<li>TOK_OR(2)</li>
<li>TOK_LE(4)</li>
<li>TOK_LT(5)</li>
<li>TOK_GE(6)</li>
<li>TOK_GT(7)</li>
<li>TOK_NE(8)</li>
<li>TOK_EQ(9)</li>
<li>+(10)</li>
<li>-(11)</li>
<li>*(12)</li>
<li>/(13)</li>
<li>)(16)</li>
</ul></td></tr>
Follow of $start
    <tr><td class="left">$start</td><td class="left"><ul><li>$(0)</li>
</ul></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State #  0</div>
<div class="subtitle"><span>Root</span></div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">0</td><td class="left">$start&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 1 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 1</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State #  1</div>
<div class="subtitle"> Goto from state 0 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">0</td><td class="left">$start&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">ACCEPT BY 0</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Accept</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Shift to state 6</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Shift to state 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Shift to state 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Shift to state 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Shift to state 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Shift to state 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Shift to state 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Shift to state 13</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State #  2</div>
<div class="subtitle"> Goto from state 0 with symbol TOK_NOT</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;TOK_NOT&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 18 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 18</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State #  3</div>
<div class="subtitle"> Goto from state 0 with symbol -</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;-&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 19 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 19</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State #  4</div>
<div class="subtitle"> Goto from state 0 with symbol (</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;(&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 20 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 20</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State #  5</div>
<div class="subtitle"> Goto from state 0 with symbol TOK_NUMBER</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;TOK_NUMBER&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with TOK_AND</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with TOK_OR</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with TOK_LE</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with TOK_LT</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with TOK_GE</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with TOK_GT</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with TOK_NE</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with TOK_EQ</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with +</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with -</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with *</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with /</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 16 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With + Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With - Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With * Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With / Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 16</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State #  6</div>
<div class="subtitle"> Goto from state 1 with symbol TOK_AND</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_AND&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 21 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 21</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State #  7</div>
<div class="subtitle"> Goto from state 1 with symbol TOK_OR</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_OR&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 22 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 22</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State #  8</div>
<div class="subtitle"> Goto from state 1 with symbol TOK_LE</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_LE&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 23 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 23</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State #  9</div>
<div class="subtitle"> Goto from state 1 with symbol TOK_LT</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_LT&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 24 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 24</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 10</div>
<div class="subtitle"> Goto from state 1 with symbol TOK_GE</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_GE&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 25 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 25</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 11</div>
<div class="subtitle"> Goto from state 1 with symbol TOK_GT</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_GT&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 26 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 26</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 12</div>
<div class="subtitle"> Goto from state 1 with symbol TOK_NE</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_NE&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 27 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 27</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 13</div>
<div class="subtitle"> Goto from state 1 with symbol TOK_EQ</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_EQ&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 28 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 28</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 14</div>
<div class="subtitle"> Goto from state 1 with symbol +</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;+&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 29 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 29</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 15</div>
<div class="subtitle"> Goto from state 1 with symbol -</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;-&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 30 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 30</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 16</div>
<div class="subtitle"> Goto from state 1 with symbol *</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;*&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 31 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 31</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 17</div>
<div class="subtitle"> Goto from state 1 with symbol /</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;/&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NOT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;(&nbsp;Expression&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">16</td><td class="left">Expression&nbsp;&rArr;&nbsp;<span class="dot">.</span>&nbsp;TOK_NUMBER&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NOT TO STATE 2</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 3</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ( TO STATE 4</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NUMBER TO STATE 5</td></tr>
    <tr><td class="action" colspan="2">GO TO STATE 32 with symbol Expression</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions <span class="same-action">(same as state 0)</span></td></tr>
    <tr><td class="action" colspan="2">With TOK_NOT Shift to state 2</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 3</td></tr>
    <tr><td class="action" colspan="2">With ( Shift to state 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NUMBER Shift to state 5</td></tr>
    <tr><td class="action" colspan="2">With Expression Goto 32</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">Expecting expression</span><span class="error-type"> (non terminal)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 18</div>
<div class="subtitle"> Goto from state 2 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">3</td><td class="left">Expression&nbsp;&rArr;&nbsp;TOK_NOT&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with +</td></tr>
Conflict with + resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with -</td></tr>
Conflict with - resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 3 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 3</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 3</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 3</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Shift to state 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Shift to state 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Shift to state 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Shift to state 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Shift to state 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Shift to state 13</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 3</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 19</div>
<div class="subtitle"> Goto from state 3 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">14</td><td class="left">Expression&nbsp;&rArr;&nbsp;-&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with +</td></tr>
Conflict with + resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with -</td></tr>
Conflict with - resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with *</td></tr>
Conflict with * resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with /</td></tr>
Conflict with / resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 14 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With + Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With - Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With * Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With / Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 14</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 20</div>
<div class="subtitle"> Goto from state 4 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;(&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;)&nbsp;</td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON ) TO STATE 33</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Shift to state 6</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Shift to state 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Shift to state 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Shift to state 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Shift to state 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Shift to state 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Shift to state 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Shift to state 13</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Shift to state 33</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator or ) expected</span><span class="error-type"> (mostly tokens)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 21</div>
<div class="subtitle"> Goto from state 6 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_AND&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with +</td></tr>
Conflict with + resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with -</td></tr>
Conflict with - resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 1 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 1</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 1</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Shift to state 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Shift to state 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Shift to state 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Shift to state 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Shift to state 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Shift to state 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Shift to state 13</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 1</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 22</div>
<div class="subtitle"> Goto from state 7 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_OR&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with +</td></tr>
Conflict with + resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with -</td></tr>
Conflict with - resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 2 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 2</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 2</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 2</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Shift to state 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Shift to state 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Shift to state 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Shift to state 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Shift to state 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Shift to state 13</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 2</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 23</div>
<div class="subtitle"> Goto from state 8 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_LE&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with +</td></tr>
Conflict with + resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with -</td></tr>
Conflict with - resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 4 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 4</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 4</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 4</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 24</div>
<div class="subtitle"> Goto from state 9 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_LT&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with +</td></tr>
Conflict with + resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with -</td></tr>
Conflict with - resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 5 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 5</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 5</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 5</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 5</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 5</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 5</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 5</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 5</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 5</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 5</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 25</div>
<div class="subtitle"> Goto from state 10 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_GE&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with +</td></tr>
Conflict with + resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with -</td></tr>
Conflict with - resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 6 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 6</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 6</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 6</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 6</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 6</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 6</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 6</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 6</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 6</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 6</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 26</div>
<div class="subtitle"> Goto from state 11 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_GT&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with +</td></tr>
Conflict with + resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with -</td></tr>
Conflict with - resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 7 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 7</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 7</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 7</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 27</div>
<div class="subtitle"> Goto from state 12 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_NE&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with +</td></tr>
Conflict with + resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with -</td></tr>
Conflict with - resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 8 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 8</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 8</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 8</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 28</div>
<div class="subtitle"> Goto from state 13 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;TOK_EQ&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with +</td></tr>
Conflict with + resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with -</td></tr>
Conflict with - resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 9 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 9</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 9</td></tr>
    <tr><td class="action" colspan="2">With + Shift to state 14</td></tr>
    <tr><td class="action" colspan="2">With - Shift to state 15</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 9</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 29</div>
<div class="subtitle"> Goto from state 14 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;+&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with +</td></tr>
Conflict with + resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with -</td></tr>
Conflict with - resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 10 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With + Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With - Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 10</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 30</div>
<div class="subtitle"> Goto from state 15 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;-&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with +</td></tr>
Conflict with + resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with -</td></tr>
Conflict with - resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with *</td></tr>
Conflict with * resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with /</td></tr>
Conflict with / resolved by Shift
    <tr><td class="action" colspan="2">REDUCE BY RULE 11 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With + Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With - Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">With * Shift to state 16</td></tr>
    <tr><td class="action" colspan="2">With / Shift to state 17</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 11</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 31</div>
<div class="subtitle"> Goto from state 16 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;*&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with +</td></tr>
Conflict with + resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with -</td></tr>
Conflict with - resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with *</td></tr>
Conflict with * resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with /</td></tr>
Conflict with / resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 12 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With + Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With - Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With * Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With / Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 12</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 32</div>
<div class="subtitle"> Goto from state 17 with symbol Expression</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;/&nbsp;Expression&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="right">1</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_AND&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">2</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_OR&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">4</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">5</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_LT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">6</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">7</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_GT&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">8</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_NE&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">9</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;TOK_EQ&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">10</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;+&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">11</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;-&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">12</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;*&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="right">13</td><td class="left">Expression&nbsp;&rArr;&nbsp;Expression&nbsp;<span class="dot">.</span>&nbsp;/&nbsp;Expression&nbsp;</td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_AND TO STATE 6</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_OR TO STATE 7</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LE TO STATE 8</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_LT TO STATE 9</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GE TO STATE 10</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_GT TO STATE 11</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_NE TO STATE 12</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON TOK_EQ TO STATE 13</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON + TO STATE 14</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON - TO STATE 15</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON * TO STATE 16</td></tr>
    <tr><td class="action" colspan="2">SHIFT ON / TO STATE 17</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with TOK_AND</td></tr>
Conflict with TOK_AND resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with TOK_OR</td></tr>
Conflict with TOK_OR resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with TOK_LE</td></tr>
Conflict with TOK_LE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with TOK_LT</td></tr>
Conflict with TOK_LT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with TOK_GE</td></tr>
Conflict with TOK_GE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with TOK_GT</td></tr>
Conflict with TOK_GT resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with TOK_NE</td></tr>
Conflict with TOK_NE resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with TOK_EQ</td></tr>
Conflict with TOK_EQ resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with +</td></tr>
Conflict with + resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with -</td></tr>
Conflict with - resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with *</td></tr>
Conflict with * resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with /</td></tr>
Conflict with / resolved by Reduce
    <tr><td class="action" colspan="2">REDUCE BY RULE 13 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With + Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With - Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With * Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With / Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 13</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="title">State # 33</div>
<div class="subtitle"> Goto from state 20 with symbol )</div>
  <table class="statehead">
    <tr><th class="right">Rule</th><th class="left">Production</th></tr>
    <tr><td class="right">15</td><td class="left">Expression&nbsp;&rArr;&nbsp;(&nbsp;Expression&nbsp;)&nbsp;<span class="dot">.</span></td></tr>
    <tr><td class="line" colspan="2"><hr/></td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with $</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with TOK_AND</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with TOK_OR</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with TOK_LE</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with TOK_LT</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with TOK_GE</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with TOK_GT</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with TOK_NE</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with TOK_EQ</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with +</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with -</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with *</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with /</td></tr>
    <tr><td class="action" colspan="2">REDUCE BY RULE 15 with )</td></tr>
    <tr><td class="thead" colspan="2">Packed Actions</td></tr>
    <tr><td class="action" colspan="2">With $ Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With TOK_AND Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With TOK_OR Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With TOK_LE Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With TOK_LT Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With TOK_GE Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With TOK_GT Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With TOK_NE Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With TOK_EQ Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With + Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With - Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With * Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With / Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">With ) Reduce by rule 15</td></tr>
    <tr><td class="action" colspan="2">Default: Error</td></tr>
    <tr><td class="thead" colspan="2">Errors</td></tr>
    <tr><td class="error" colspan="2"><span class="error-message">operator expected</span><span class="error-type"> (token group)</span></td></tr>
  </table>
<div class="close">&nbsp;</div>

<div class="subheading">Summary</div>
  <table class="summary">
    <tr><th class="left">Property</th><th class="left">Value</th></tr>
    <tr><td class="left">Source</td><td class="left">/Users/jcgarza/dev/prototype/syntax/target/test-classes/javascript-test.sy</td></tr>
    <tr><td class="left">Output</td><td class="left">/Users/jcgarza/dev/prototype/syntax/test-output/working-files/JsTestParser.js</td></tr>
    <tr><td class="left">Include/Interface</td><td class="left">null</td></tr>
    <tr><td class="left">Bundle</td><td class="left">null</td></tr>
    <tr><td class="left">Skeleton</td><td class="left">null</td></tr>
    <tr><td class="left">Algorithm</td><td class="left">SLR</td></tr>
    <tr><td class="left">Language</td><td class="left">javascript</td></tr>
    <tr><td class="left">Packed?</td><td class="left">true</td></tr>
    <tr><td class="left">Tokens</td><td class="left">18</td></tr>
    <tr><td class="left">Non Terminals</td><td class="left">2</td></tr>
    <tr><td class="left">Lexer Modes</td><td class="left">1</td></tr>
    <tr><td class="left">Types</td><td class="left">1</td></tr>
    <tr><td class="left">Error Groups</td><td class="left">1</td></tr>
    <tr><td class="left">Regular Expressions</td><td class="left">7</td></tr>
    <tr><td class="left">Rules</td><td class="left">17</td></tr>
    <tr><td class="left">Errors</td><td class="left">3</td></tr>
    <tr><td class="left">Actions</td><td class="left">254</td></tr>
    <tr><td class="left">Gotos</td><td class="left">16</td></tr>
    <tr><td class="left">Recoveries</td><td class="left">0</td></tr>
    <tr><td class="left">States</td><td class="left">34</td></tr>
  </table>
<div class="close">&nbsp;</div>

</div>
</div>