---
title: "Details"
subtitle: ""
description: 
date: 2022-01-08T19:51:06+08:00
lastmod: 2022-01-08T19:51:06+08:00

comments: true
hidden: true
draft: false

image: 
tags: []
categories: []
---
<!--more-->
# Details

Details shortcode is a helper for `details` html5 element. It is going to replace `expand` shortcode.

## Example
```tpl
{{</* details "Title" [open] */>}}
## Markdown content
Lorem markdownum insigne...
{{</* /details */>}}
```
```tpl
{{</* details title="Title" open=true */>}}
## Markdown content
Lorem markdownum insigne...
{{</* /details */>}}
```

{{< details "Title" >}}
## Markdown content
Lorem markdownum insigne...
{{< /details >}}
