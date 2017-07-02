---
layout: page
title: About
description: 生命不息，编程不止
keywords: Jianjian Du, 杜健健
comments: true
menu: 关于
permalink: /about/
---

南京航空航天大学硕士研究生

skill：java，python，c，js

热爱生活，热爱编程。

## 联系

{% for website in site.data.social %}
* {{ website.sitename }}：[@{{ website.name }}]({{ website.url }})
{% endfor %}

## Skill Keywords

{% for category in site.data.skills %}
### {{ category.name }}
<div class="btn-inline">
{% for keyword in category.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
