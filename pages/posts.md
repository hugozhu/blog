---
title: Blogs
description:
---

{{# posts_latest }}
<div class="post">
  <h3 class="title"><a href="{{url}}">{{title}}</a> <span class="date">{{ date }}</span></h3>

  {{{ summary }}}

  <div class="more">
    <a href="{{url}}" class="btn">read more..</a>
  </div>
</div>
{{/ posts_latest }}