---
layout: page
title:  信息技术改变生活
tagline: xiaobo wu's shared space
---
{% include JB/setup %}




#### 博主信息：

* Name : Xiaobo Wu(吴小波) 
A student from CAREERI/CAS, P.R. China,
He is currently working on RS/GIS applications.

* Email: 
**<wavelet2008@163.com>** OR
**<wuxb@lzb.ac.cn>**


#### 博文列表

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>





