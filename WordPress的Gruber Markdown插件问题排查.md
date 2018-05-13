# WordPress的Gruber Markdown插件问题排查

*Mission Start!*

## 一、问题 
今天写完博客发现一个很奇怪的问题：进入标签相关页时整个页面的颜色都变成了粉红色（画风清奇，一股魔法少女厄加特的味道），居然把我内心世界展示了出来，得赶紧修复。    
   
## 二、解题步骤（解决方法请看--->>三）
### 1、找寻问题根源
浏览器查看元素样式，发现整个页面被一个叫tranquil-heart.min.css的css文件渲染了一遍，其渲染的class为'tag'，突然想到我进入的页面是标签相关页面，于是查看body的class值，发现class值也是'tag'，果然是class重名了。

### 2、找寻解决办法
既然是重名，那么只要把两者其中一个修改一下就可以解决了，因为考虑到修改插件内容会比修改WordPress内容简单，所以选择修改Gruber Markdown插件的类名。

### 3、WordPress的Markdown插件原理
既然要修改Gruber Markdown插件，就得了解一下其原理：Markdown插件其实是在编写完博客之后，进行发布时，将Markdown语法转化为HTML语法并对各个标签对的class赋以相对应的值，最终将其保存在数据库中。在查看文章时，Markdown插件取用选择的主题css文件对文章进行渲染。

## 三、解决方案
1、在Linux下使用find命令定位到css文件，路径大致为*wordpress/wp-content/plugins/wp-gruber-markdown/theme/tranquil-heart.min.css*（根据主题选择自己的css文件），打开文件，搜索'tag'，结果类似于：

```css
tag{color:#ed5565}
```
将'tag'改为不易重复的名称，保存退出。  
   
2、在*wordpress/wp-content/plugins/wp-gruber-markdown/*下找到*prettify.js*文件，这是将Markdown转化为HTML的js文件，打开，搜索'tag'，结果类似：

```js
...[["tag",/^^<\...  // 前后省略
```
以及

```js
...PR_TAG:"tag",...  // 前后省略
```
将这两处的'tag'都改成与第1步修改时相同的名称，保存退出。

3、最后一步，将相关博客重新发布即可（更新博客中class值），若无效果则在清除浏览器缓存后刷新页面。

*Mission Complete!*

