---
layout: page
title: About
description: 记录生活的点点滴滴
keywords: Binz, BinBin
comments: true
menu: 关于
permalink: /about/
---

我是BinZhe，知行合一。

保持好奇，继续好奇，探索好奇。

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
