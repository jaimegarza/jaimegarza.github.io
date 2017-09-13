---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">
#### Syntax maven plugin

---

Subtopics:
  * <a href="{{ site.baseurl }}/syntax/syntax-maven-plugin-usage">Usage</a>
  * <a href="{{ site.baseurl }}/syntax/syntax-maven-plugin-reference">Reference</a>

---

The Maven Plugin is used to generate artifacts from syntax files through maven. It is a maven plugin whose execution is usually tied to the generate-sources life cycle phase.

**syntax-maven-plugin** supports two goals: help and generate. The help goal allows for a detailed information of the goals, and a detailed information of the configuration arguments when -Ddetail=true.

Please read the <a href="{{ site.baseurl }}/syntax/syntax-maven-plugin-usage">Usage</a> section for a description of the fragment of code to install the plugin in your pom.

To get help, please run
```
mvn syntax:help -Ddetail=true
```

In order to execute syntax as configured, please run
```
mvn syntax:generate
```

</div>
</div>