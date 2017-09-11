## zepto.init ##
简明扼要：
使用$()时，会直接调用zepto.init生成zepto对象，那zepto.init如何根据不同类型的参数来生产指定对象呢？
```
  zepto.init = function(selector, context) {
    var dom
    // 未传参，undefined进行boolean转换，返回空Zepto对象
    if (!selector) return zepto.Z()

    // 如果选择器是字符串，即CSS表达式
    else if (typeof selector == 'string') {
	  // 去掉前后空格
      selector = selector.trim()
      // If it's a html fragment, create nodes from it
      // Note: In both Chrome 21 and Firefox 15, DOM error 12
      // is thrown if the fragment doesn't begin with <
	  // 如果是<开头，>结尾
      if (selector[0] == '<' && fragmentRE.test(selector))
	  // 调用片段生成DOM
        dom = zepto.fragment(selector, RegExp.$1, context), selector = null
      // If there's a context, create a collection on that context first, and select
      // nodes from there
	  // 如果传递了上下文，在上下文中查找元素
      else if (context !== undefined) return $(context).find(selector)
      // 通过CSS表达式查找元素
      else dom = zepto.qsa(document, selector)
    }

    // 如果selector是函数，则在DOM ready的时候执行它
    else if (isFunction(selector)) return $(document).ready(selector)

    // 如果selector是一个Zepto对象，返回它自己
    else if (zepto.isZ(selector)) return selector
    else {

      // normalize array if an array of nodes is given
	  // 如果selector是数组，过滤null、undefined
      if (isArray(selector)) dom = compact(selector)

      // Wrap DOM nodes.
	  // 如果selector是对象，转换为数组？注意DOM节点的typeof值也是object，所以在里面还要再进行一次判断
      else if (isObject(selector))
        dom = [selector], selector = null

      // If it's a html fragment, create nodes from it
	  // 如果selector是复杂的HTML代码，调用片段换成DOM节点
      else if (fragmentRE.test(selector))
        dom = zepto.fragment(selector.trim(), RegExp.$1, context), selector = null
      
	  // If there's a context, create a collection on that context first, and select
      // nodes from there
      // 如果存在上下文context，仍在上下文中查找selector
	  else if (context !== undefined) return $(context).find(selector)
      // And last but no least, if it's a CSS selector, use it to select nodes.
      // 如果没有给定上下文，在document中查找selector
	  else dom = zepto.qsa(document, selector)
    }
    // create a new Zepto collection from the nodes found
    // 将查询结果转换成Zepto对象
	return zepto.Z(dom, selector)
  }

```
总结：
1. 这里首先判断如果没有传入参数，则返回新建的zepto对象
2. 如果传入的是函数，即$(function(){})，则执行$(document).ready()，即在文档加载完后执行自定义函数
3. 如果selector本身是一个zepto对象，则直接返回它自己
4. 如果selector不是一个zepto对象，则根据不同类型的selector生产节点对象并赋值于dom
5. 如果selector是数组，则将数组里的无效值去掉后赋值给dom
6. 如果selector是对象
7. 如果selector是html片段
8. 如果selector是css选择字符串且context有值
9. 如果selector是css选择字符串且context无值
10. 调用zepto.Z，生产zepto对象

## 细节讲解 ## 
### zepto.Z的内部实现 ###
```
  // 实际构造函数
  function Z(dom, selector) {
    var i, len = dom ? dom.length : 0
    for (i = 0; i < len; i++) this[i] = dom[i]
    this.length = len
    this.selector = selector || ''
  }

  
  zepto.Z = function(dom, selector) {
    return new Z(dom, selector)
  }

``` 
这里Z里面传入的dom参数要求是数组，若dom为null或undefined，则this.length = 0; this.selector = selector，并返回this。（可以认为this继承了dom）

### zepto.fragment ###

作用：将HTML转化为DOM

```
  /**
   * 内部函数 HTML 转换成 DOM
   * 原理是 创建父元素，innerHTML转换
   * @param html       [html片段]
   * @param name       [容器标签名]
   * @param properties [附加的属性对象]
   * @return {*}      
   */
  zepto.fragment = function(html, name, properties) {
    var dom, nodes, container

    // A special case optimization for a single tag
    // 如果是单个元素，创建DOM
    // 注意：RegExp 是javascript中的一个内置对象。为正则表达式
    // RegExp.$1是RegExp的一个属性，指的是与正则表达式匹配的第一个 子匹配（以括号为标志）字符串，以此类推，RegExp.$2···RegExp.$99有99个匹配
    if (singleTagRE.test(html)) dom = $(document.createElement(RegExp.$1))

    if (!dom) {
      // 修正自闭合标签 如<div />，转换成<div></div>
      if (html.replace) html = html.replace(tagExpanderRE, "<$1></$2>")

      // 给name取元素名
      if (name === undefined) name = fragmentRE.test(html) && RegExp.$1
      
      // 设置容器名，如果不是tr, tbody, thead, tfoot, td, th，则容器名为div
      // 为什么设置容器，是严格按照HTML语法，虽然tr td th浏览器会自动添加tbody 
      if (!(name in containers)) name = '*'

      container = containers[name] // 创建容器
      container.innerHTML = '' + html // 生成DOM

      // 取容器的子节点，（子节点集会返回） $.each返回第一个参数
      dom = $.each(slice.call(container.childNodes), function(){
        container.removeChild(this) // 把创建的子节点逐个删除
      })
    }

    // 如果properties是对象，遍历它，将它设置成DOM的属性
    if (isPlainObject(properties)) {
      // 转换成Zepto Obj，方便调用Zepto的方法
      nodes = $(dom)

      // 遍历对象，设置属性
      $.each(properties, function(key, value) {
        // 优先获取属性修正对象，通过修正对象读写值
        // methodAttributes包含'val', 'css', 'html', 'text', 'data', 'width', 'height', 'offset'？？
        if (methodAttributes.indexOf(key) > -1) nodes[key](value)
        else nodes.attr(key, value)
      })
    }

    // 返回dom数组
    return dom
  }
```

涉及到的几个正则表达式：
1. singleTagRE 匹配单个元素
```
var singleTagRE = /^<(\w+)\s*\/?>(?:<\/\1>|)$/;
var name = (singleTagRE.test("<p></p>")&&RegExp.$1);　　//这里可以正常匹配<p>,<p/>,<p></p>
console.log(name);

结果：p
```
2. tagExpanderRE 修复自闭合标签
```
var tagExpanderRE = /<(?!area|br|col|embed|hr|img|input|link|meta|param)(([\w:]+)[^>]*)\/>/ig;
var html = "<div/>".replace(tagExpanderRE, "<$1></$2>");
console.log(html);
VM412:4 <div></div>
undefined
var tagExpanderRE = /<(?!area|br|col|embed|hr|img|input|link|meta|param)(([\w:]+)[^>]*)\/>/ig;
var html = "<input/>".replace(tagExpanderRE, "<$1></$2>");
console.log(html);
VM413:4 <input/>
```
3. fragmentRE 取出元素名 name 是为了判断是否为表格元素，为以后的 container 做准备
```
var fragmentRE = /^\s*<(\w+|!)[^>]*>/;
var name = fragmentRE.test("<div></div>") && RegExp.$1;
console.log(name);
VM443:4 div
undefined
var fragmentRE = /^\s*<(\w+|!)[^>]*>/;
var name = fragmentRE.test("<input/>") && RegExp.$1;
console.log(name);
VM444:4 input
undefined
var fragmentRE = /^\s*<(\w+|!)[^>]*>/;
var name = fragmentRE.test("<div><input/></div>") && RegExp.$1;
console.log(name);
VM445:4 div
undefined
var fragmentRE = /^\s*<(\w+|!)[^>]*>/;
var name = fragmentRE.test("<div><input/>") && RegExp.$1;
console.log(name);
VM446:4 div
undefined
var fragmentRE = /^\s*<(\w+|!)[^>]*>/;
var name = fragmentRE.test("<div></div><input/>") && RegExp.$1;
console.log(name);
VM447:4 div
```