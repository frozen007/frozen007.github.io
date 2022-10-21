---
layout: post
title:  "jekyll本地调试技巧-识别开发环境与线上环境"
date:   2022-08-09 12:25:01 +0800
blurb:  "jekyll本地调试技巧"
og_image:
syntax_lang: true
categories: tech
tags: 
  - jekyll
---

### Jekyll本地调试问题

在用Jekyll写blog时，经常需要在本地调试，查看渲染效果。

由于页面内嵌了百度统计和google统计的代码，在本地访问页面时也会在统计后台上留下记录：

![baidu-tongji](/assets/img/202208/baidu-tongji.png)

为了避免统计`localhost`这类访问请求，在本地环境时不需要加载内嵌的统计代码。


### 如何区分不同环境

根据Jekyll的官方文档(Jekyll版本V4.2.2)，可以有两种方式：
- 判断site.url
- 设置JEKYLL_ENV(推荐)

#### 判断site.url

site.url[^1]取值为`_config.yml`中配置的url属性，通常为线上环境的域名

```yaml
# the base hostname & protocol for the site
url: "https://frozen007.github.io"
```

但在本地执行`bundle exec jekyll serve`时，site.url会设置为默认值：http://localhost:4000。

所以可以直接在内嵌统计代码的地方加入下面的判断：
```javascript
  {% raw %}
  {% unless site.url contains "localhost" %}
  <!-- Baidu -->
  <script>
    var _hmt = _hmt || [];
    (function() {
      var hm = document.createElement("script");
      hm.src = "https://hm.baidu.com/hm.js";
      var s = document.getElementsByTagName("script")[0]; 
      s.parentNode.insertBefore(hm, s);
    })();
  </script>
  {% endunless %}
  {% endraw %}
```

这样渲染出的页面中就不会有baidu统计的代码。

#### 设置JEKYLL_ENV

在执行jeklly build 或 serve时，可以直接指定Jekyll环境变量：JEKYLL_ENV[^2]

指定为production环境
```shell
bundle exec jekyll build JEKYLL_ENV=production
```

如果不指定JEKYLL_ENV，默认值为development

上面的内嵌代码也可以修改为下面的：
```javascript
  {% raw %}
  {% if jekyll.environment == "production" %}
  <!-- Baidu -->
  <script>
    var _hmt = _hmt || [];
    (function() {
      var hm = document.createElement("script");
      hm.src = "https://hm.baidu.com/hm.js";
      var s = document.getElementsByTagName("script")[0]; 
      s.parentNode.insertBefore(hm, s);
    })();
  </script>
  {% endif %}
  {% endraw %}
```

### 参考文档

[^1]: [site-variables](https://jekyllrb.com/docs/variables/#site-variables)
[^2]: [environments](https://jekyllrb.com/docs/configuration/environments)

