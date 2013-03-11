---
title: Blogs
description: My blogs
---

{{# posts_latest }}
<div class="post">
  <h1 class="title"><a href="{{url}}">{{title}}</a> <span class="date">{{ date }}</span></h1>

  {{{ summary }}}

  <div class="more">
    <a href="{{url}}" class="btn btn-small">阅读全文</a>
  </div>
</div>
{{/ posts_latest }}

<div class="pagination">
  <ul>
      <li><a href="/posts/2/">更多文章...</a></li>
  </ul>
</div>