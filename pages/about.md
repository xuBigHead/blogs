---
layout: page
title: 关于
description:
keywords: 许大头
comments: false
menu: 关于
permalink: /about/
---

## 联系

<ul>
    {% for website in site.data.social %}
    <li>{{website.sitename }}：<a href="{{ website.url }}" target="_blank">@{{ website.name }}</a></li>
    {% endfor %}
    {% if site.url contains 'mazhuang.org' %}
    <li>
        微信公众号：<br />
        <img style="height:192px;width:192px;border:1px solid lightgrey;" src="{{ site.url }}/assets/images/qrcode.jpg" alt="闷骚的程序员" />
    </li>
    {% endif %}
</ul>


## Skill Keywords

{% for skill in site.data.skills %}
### {{ skill.name }}
<div class="btn-inline">
{% for keyword in skill.keywords %}
<button class="btn btn-outline" type="button">{{ keyword }}</button>
{% endfor %}
</div>
{% endfor %}
