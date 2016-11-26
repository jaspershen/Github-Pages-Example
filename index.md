---
layout: default
---

<body>
  <div class="index-wrapper">
    <div class="aside">
      <div class="info-card">
        <h1>Jasper Shen's blog</h1>
        <h1>申祖涛的博客</h1>
        <h2>请点击下面的链接关注我的动态</h2>
        <a href="http://weibo.com/2717397881/profile?topnav=1&wvr=6&is_all=1" target="_blank"><img src="http://www.weibo.com/favicon.ico" alt="微博" width="25"/></a>
        <a href="https://github.com/jaspershen" target="_blank"><img src="/github.ico" alt="Github" width="22"/></a>
        <a href="https://www.zhihu.com/people/shen-zu-tao-73/activities" target="_blank"><img src="/zhihu.ico" alt="知乎" width="22"/></a>
        <a href="http://www.metabolomics-china.org/" target="_blank"><img src="/zhulab.ico" alt="zhulab" width="22"/></a>
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
