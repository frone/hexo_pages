---
title: 如何在hexo中支持Mathjax
date: 2018-07-16 18:11:51
tags:
    - Hexo
    - Mathjax
categories: Hexo
---
### 第一步： 使用Kramed代替 Marked
hexo 默认的渲染引擎是 marked，但是 marked 不支持 mathjax。 kramed 是在 marked 的基础上进行修改。我们在工程目录下执行以下命令来安装 kramed.
```
npm uninstall hexo-renderer-marked --save
npm install hexo-renderer-kramed --save
```

然后，更改/node_modules/hexo-renderer-kramed/lib/renderer.js，更改
```
// Change inline math rule
function formatText(text) {
    // Fit kramed's rule: $$ + \1 + $$
    return text.replace(/`\$(.*?)\$`/g, '$$$$$1$$$$');
}
```
为
```
// Change inline math rule
function formatText(text) {
    return text;
}
```

### 第二步: 停止使用 hexo-math
首先，如果你已经安装 hexo-math, 请卸载它,然后安装 hexo-renderer-mathjax 包
```
npm uninstall hexo-math --save
npm install hexo-renderer-mathjax --save
```
### 第三步: 更改默认转义规则
因为 hexo 默认的转义规则会将一些字符进行转义，比如 _ 转为 <em>, 所以我们需要对默认的规则进行修改. 
```
vi node_modules/kramed/lib/rules/nline.js
```
```
// 替换11行
escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
//修改为
escape: /^\\([`*\[\]()# +\-.!_>])/,

//替换20行
em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
//修改为
em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
```
### 第四步: 开启mathjax

```
vi themes/next/_config.yml
```
开启 mathjax功能
```
mathjax:
  enable: true
  perpage: false
  cdn: //cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML
```
并在博客中使用mathjax
```
---
title: Testing Mathjax with Hexo
category: Uncategorized
date: 2017/05/03
mathjax: true
---
```
