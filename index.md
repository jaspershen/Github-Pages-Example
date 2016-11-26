---
layout: default
---

<body>
  <div class="index-wrapper">
    <div class="aside">
      <div class="info-card">
        <h1>Jasper Shen</h1>
        <a href="http://weibo.com/2717397881/profile?topnav=1&wvr=6&is_all=1" target="_blank"><img src="http://www.weibo.com/favicon.ico" alt="" width="25"/></a>
        <a href="https://github.com/jaspershen" target="_blank"><img src="/github.ico" alt="" width="22"/></a>
        <a href="https://www.zhihu.com/people/shen-zu-tao-73/activities" target="_blank"><img src="/zhihu.ico" alt="" width="22"/></a>
      </div>
      <div id="particles-js"></div>
    </div>

    <div class="index-content">
      <ul class="artical-list">
        {% for post in site.posts %}
        <li>
          <a href="{{ post.url }}" class="title">{{ post.title }}</a>
          <div class="title-desc">{{ post.description }}</div>
        </li>
        {% endfor %}
      </ul>
    </div>
  </div>
</body>
