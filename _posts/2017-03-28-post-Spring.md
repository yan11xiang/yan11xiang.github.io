---
title: "Spring事务注意点"
excerpt_separator: "<!--more-->"
categories:
  - Post Formats
tags:
  - Spring
  - Java
---

如果有For Update,必须使用事务，否则会线程被锁住，最终抛异常。

<!--more-->

This post has a manual excerpt `<!--more-->` set after the second paragraph. The following YAML Front Matter has also be applied:

```yaml
excerpt_separator: "<!--more-->"
```