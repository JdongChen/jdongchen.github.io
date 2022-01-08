---
title: "Expand"
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

Expand shortcode can help to decrease clutter on screen by hiding part of text. Expand content by clicking on it.

## Example
### Default

```tpl
{{</* expand */>}}
## Markdown content
Lorem markdownum insigne...
{{</* /expand */>}}
```

{{< expand >}}
## Markdown content
Lorem markdownum insigne...
{{< /expand >}}

### With Custom Label

```tpl
{{</* expand "Custom Label" "..." */>}}
## Markdown content
Lorem markdownum insigne...
{{</* /expand */>}}
```

{{< expand "Custom Label" "..." >}}
## Markdown content
Lorem markdownum insigne. Olympo signis Delphis! Retexi Nereius nova develat
stringit, frustra Saturnius uteroque inter! Oculis non ritibus Telethusa
protulit, sed sed aere valvis inhaesuro Pallas animam: qui _quid_, ignes.
Miseratus fonte Ditis conubia.
{{< /expand >}}
