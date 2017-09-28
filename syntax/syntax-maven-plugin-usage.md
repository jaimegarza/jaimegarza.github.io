---
layout: page
---

{::options parse_block_html="true" /}
<div class="syntax">
{% include syntax-header.html %}
{% include syntax-navigation.html %}

<div class="syntax-matter">
#### Syntax maven plugin usage


To install syntax-maven-plugin in your project, you need to enter the following plugin in your pom file under plugins.

```
<plugin>
    <groupId>me.jaimegarza</groupId>
    <artifactId>syntax-maven-plugin</artifactId>
    <version>1.2.0</version>
    <executions> 
      <execution>
        <id>generate-parser</id>
        <phase>generate-sources</phase>
        <goals>
          <goal>generate</goal>
        </goals>
      </execution>
      <configuration>
      ...
      </configuration>
    </executions>
  </plugin>
```

Please refer to the <a href="{{ site.baseurl }}/syntax/syntax-maven-plugin-reference">Reference</a> for additional configuration details
</div>
</div>
