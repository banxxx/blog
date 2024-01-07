---
title: Stellar魔改(link标签)
cover: /assets/image/BE035.jpg
tags:
  - Stellar
categories:
  - 前端
abbrlink: db719c7a
poster:
  headline: Stellar魔改(link标签)
  color: white
date: 2024-01-03 11:25:15
---

这是Stellar官方的link标签
{% link https://blog.banxx.cn/db719c7a desc:false  %}

外链卡片标签的语法格式为:
```
{% link href [title] [icon:src] [desc:true/false] %}
```

参数含义:
```yml
href: 链接  
title: 可选，手动设置标题（为空时会自动抓取页面标题）  
icon: 可选，手动设置图标（为空时会自动抓取页面图标）  
desc: 可选，是否显示摘要描述，为true时将会显示页面描述

```

不带摘要的样式:
```
{% link https://blog.banxx.cn/db719c7a %}
```
带摘要的样式:
```
{% link https://blog.banxx.cn/db719c7a desc:true  %}
```

由于官方的居中显示，而我的想要它居左或者居右，显示的好看些，于是就自己动手改改，还加上了图片显示，代码也很简单，先看效果:

{% link https://blog.banxx.cn/db719c7a desc:false position:left cover:https://coscdn.banxx.cn/blog/be047.jpg %}
{% link https://blog.banxx.cn/db719c7a position:right %}

语法格式:
```
{% link https://blog.banxx.cn/db719c7a desc:false position:left cover:https://coscdn.banxx.cn/blog/be047.jpg %}
```

参数含义:
```yml
position: 可选(left/right)  
cover: 可选，显示图片地址
```

---
完整代码:{% sup 适用版本:1.19.0 color:red %}

{% note color:green 文件路径: themes\stellar\scripts\tags\lib\link.js %}

```js
'use strict'
module.exports = ctx => function(args) {
  const full_url_for = require('hexo-util').full_url_for.bind(ctx)
  args = ctx.args.map(args, ['icon', 'desc', 'position', 'cover'], ['url', 'title'])
  var autofill = []
  if (!args.title) {
    autofill.push('title')
  }
  if (!args.icon) {
    autofill.push('icon')
  }
  if (args.desc) {
    autofill.push('desc')
  }
  // position定位标签
  if (args.position) {
    autofill.push('position')
  }
  // cover图片标签
  if (args.cover) {
    autofill.push('cover')
  }
  var el = ''
  el += '<div class="tag-plugin link dis-select">'
  el += '<a class="link-card' + (args.desc ? ' rich' : ' plain') + '" title="' + (args.title || '') + '" href="' + args.url + '"'
  if (args.url.includes('://')) {
    el += ' target="_blank" rel="external nofollow noopener noreferrer"'
  }
  // 判断是否含有position标签,根据条件载入样式
  if(typeof args.position != null && args.position != undefined){
    el += 'style="margin-'+ (args.position === 'left' ? 'right' : 'left') + ': auto"'
  }
  el += ' cardlink'
  el += ' autofill="'
  el += autofill.join(',')
  el += '"'
  el += '>'
  function loadIcon() {
    var el = ''
    if (ctx.theme.config.plugins.lazyload && ctx.theme.config.plugins.lazyload.enable) {
      el += '<div class="lazy img" data-bg="' + (args.icon || ctx.theme.config.default.link) + '"></div>'
    } else {
      el += '<div class="lazy img" style="background-image:url(&quot;' + (args.icon || ctx.theme.config.default.link) + '&quot;)"></div>'
    }
    return el
  }
  function loadTitle() {
    return '<span class="title">' + (args.title || args.url) + '</span>'
  }
  function loadDesc() {
    return '<span class="cap desc fs12"></span>'
  }
  function loadLink() {
    return '<span class="cap link fs12">' + full_url_for(args.url) + '</span>'
  }
  // 将cover图片标签加入样式中
  function loadCover() {
    var el = '';
    if (args.cover) {
      el += '<div class="image" onerror="' + ctx.theme.config.default.link + '" style="width: 100%; float: left; margin-right: 10px;">' +
		'<img src="' + args.cover + '">' +
		'</div>';
    }
    return el
  }
  if (args.desc) {
    el += loadCover()
    // top
    el += '<div class="top">'
    el += loadIcon() + loadLink()
    el += '</div>'
    // bottom
    el += '<div class="bottom">'
    el += loadTitle() + loadDesc()
    el += '</div>'
  } else {
    // left
    el += '<div class="left">'
    el += loadTitle() + loadLink()
    el += '</div>'
    // right
    el += '<div class="right">'
    el += loadIcon()
    el += '</div>'
  }
  // end
  el += '</a></div>'
  
  return el
}
```