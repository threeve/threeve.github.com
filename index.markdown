---
layout: default
title: Jason Foreman
---

Jason Foreman
=============

Recently
-----------
{% for post in site.posts limit:2 %}
{{ post.date | date: "%b %d, %Y" }} - [{{post.title}}]({{ post.url }})
{% endfor %}


Previously
----------
{% for post in site.posts offset:2 limit:10 %}
{{ post.date | date_to_string }} &raquo; [{{ post.title }}]({{ post.url }})
{% endfor %}


Elsewhere
---------

[twitter.com/threeve](http://twitter.com/threeve)

[github.com/threeve](http://github.com/threeve)

[flickr.com/photos/threeve](http://flickr.com/photos/threeve)

