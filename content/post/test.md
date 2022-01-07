---
title: "Test"
subtitle: ""
description: 
date: 2022-01-07T15:55:18+08:00
lastmod: 2022-01-07T15:55:18+08:00

comments: true
hidden: false
draft: true
math: true
image: 
tags: []
categories: []
---

<!--more-->

In this example we will be using [KaTeX](https://katex.org/)

- Create a partial under `/layouts/partials/math.html`
- Within this partial reference the [Auto-render Extension](https://katex.org/docs/autorender.html) or host these scripts locally.
- Include the partial in your templates like so:  

```bash
{{ if or .Params.math .Site.Params.math }}
{{ partial "math.html" . }}
{{ end }}
```

- To enable KaTex globally set the parameter `math` to `true` in a project's configuration
- To enable KaTex on a per page basis include the parameter `math: true` in content files

**Note:** Use the online reference of [Supported TeX Functions](https://katex.org/docs/supported.html)

### Examples
{{ partial "math.html" . }}

Block math:
$$\varphi = 1+\frac{1} {1+\frac{1} {1+\frac{1} {1+\cdots} } }$$
Block math:
$$
c = a+b
$$
 
Block math:    
$$
\begin{alignat}{2}
   10&x+ &3&y = 2 \\
   3&x+&13&y = 4
\end{alignat}
$$
