---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">
#### Syntax maven plugin reference

---

The following parameters can be entered through the configuration.

**Configuration&nbsp;Argument** | **Required** | **Description**
:--------------------------     |:-------------|:----------- 
&lt;sourceFile&gt;              | Yes          | This is the specification of the Syntax file to be used. Add the extension.
&lt;outputFile&gt;              | Yes          | This is the specification of the file to be generated. Add the extension.
&lt;includeFile&gt;             | No           | When not provided and external include is desired, the include file name will be changed.
&lt;reportFile&gt;              | No           | The report file is an HTML file with the results of the analysis of the grammar, and its results, together with a summary.
&lt;skeletonFie&gt;             | No           | If specified, the given file should contain the parse code.
&lt;bundleFile&gt;              | No           | If provided, it is a properties file for java with the error codes
&lt;language&gt;                | No           | If omitted, java is assumed. Valid values are C, java, pascal.
&lt;algorithm&gt;               | No           | When omitted, lalr is assumed. Valid values are slr, lalr.
&lt;externalInclude&gt;         | No           | Decide if a file with extenal definitions is desired. Not valid for java.
&lt;verbose&gt;                 | No           | Produces additional display output.
&lt;debug&gt;                   | No           | Produces parser phase output.
&lt;emitLine&gt;                | No           | For C, do you want #line statements pointing to the .syntax file? 
&lt;packed&gt;                  | No           | Default to tabular. Valid values are packed, tabular. Use tabular for a clearer parser table. Not recommended for large grammars.
&lt;driver&gt;                  | No           | Default to parser. Valid values are parser, <a href="https://github.com/jaimegarza/syntax/wiki/Lexic-driven-parsers">scanner</a>
&lt;margin&gt;                  | No           | Used to change the emitted code's right margin. Should be greater than 80
&lt;indent&gt;                  | No           | Used to change the emitted code's indentation. Default is 2

</div>
</div>