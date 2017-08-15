## 源码版本 ##
源码版本1.2.0
需要了解闭包和原型链的知识

## 源码结构 ## 
### 整体结构 ### 
```
// 一个立即执行的表达式函数(function(){})()
(
function(global, factory){
  if (typeof define === 'function' && define.amd)
    define(function() { return factory(global) })
  else
    factory(global)
})(this, function(window){
	var Zepto = (function(){
	
	})()
	
	window.Zepto = Zepto
	window.$ === undefined && (window.$ = Zepto)
	
	;(function($){
	// 代码
	})(Zepto)
	
	;(function($){
	// 代码
	})(Zepto)
	
	;(function($){
	// 代码
	})(Zepto)
	
	;(function(){
	// 代码
	})()
	
	return Zepto
})
)

```

### 支持AMD规范 ###

```
  if (typeof define === 'function' && define.amd)
    define(function() { return factory(global) })
  else
    factory(global)
```

这段话的意思是检查define是否定义为函数，定义了就执行AMD命名规范，没有就普通调用


### 核心结构 ### 
这里只关注zepto实现的核心结构图，不关注具体的功能实现
Zepto的整体结构
```
var Zepto = (function(){

// 实际构造函数
function Z(dom, selector){
	var i, len = dom ? dom.length : 0

	for (i = 0; i < len; i++)
		this[i] = dom[i]
	
	this.length = len;
	this.selector = selector || ''
}

zepto.Z = function(dom, selector) {
	return new Z(dom, selector)
}

// 判断是否是Zepto对象
zepto.isZ = function(object) {
	return object instanceof zepto.Z
}

// 初始化参数，DOM
zepto.init = function(selector, context) {

	···
	
	return zepto.Z(dom, selector)
}

// 构造函数
$ = function(selector, context) {
	return zepto.init(selector, context)
}

// 原型设置
$.fn = {

	···
}

zepto.Z.prototype = Z.prototype = $.fn

$.zepto = zepto

return $

})()


window.Zepto = Zepto
window.$ === undefined && (window.$ = Zepto)

```

#### 变量初始化 #### 
```
  var undefined, key, $, classList, 
	
	// 获取cancat, filter, slice方法，并且优化作用域链
	emptyArray = [], 
	concat = emptyArray.concat, filter = emptyArray.filter, slice = emptyArray.slice,

    document = window.document,
    elementDisplay = {}, classCache = {},
    cssNumber = { 'column-count': 1, 'columns': 1, 'font-weight': 1, 'line-height': 1,'opacity': 1, 'z-index': 1, 'zoom': 1 },

	// 匹配HTML标签
    fragmentRE = /^\s*<(\w+|!)[^>]*>/,
	// 匹配单个HTML标签
    singleTagRE = /^<(\w+)\s*\/?>(?:<\/\1>|)$/,
	// 匹配自闭合标签
    tagExpanderRE = /<(?!area|br|col|embed|hr|img|input|link|meta|param)(([\w:]+)[^>]*)\/>/ig,
    // 匹配根节点
	rootNodeRE = /^(?:body|html)$/i,
    // 匹配A-Z
	capitalRE = /([A-Z])/g,

    // 需要提供get和set的方法名?
    methodAttributes = ['val', 'css', 'html', 'text', 'data', 'width', 'height', 'offset'],

	// 相邻节点的一些操作
    adjacencyOperators = [ 'after', 'prepend', 'before', 'append' ],

	// 创建table的节点，当需要给tr，tbody，thead，tfoot，td，th设置innerHTML的时候，需要用其父元素作为容器来装载
    table = document.createElement('table'),
    tableRow = document.createElement('tr'),
    containers = {
      'tr': document.createElement('tbody'),
      'tbody': table, 'thead': table, 'tfoot': table,
      'td': tableRow, 'th': tableRow,
      '*': document.createElement('div')
    },

	// 当DOM ready的时候，document会有以下三种状态的一种
    readyRE = /complete|loaded|interactive/,

    simpleSelectorRE = /^[\w-]*$/,
    class2type = {},
    toString = class2type.toString,
    zepto = {},
    camelize, uniq,
    tempParent = document.createElement('div'),
    propMap = {
      'tabindex': 'tabIndex',
      'readonly': 'readOnly',
      'for': 'htmlFor',
      'class': 'className',
      'maxlength': 'maxLength',
      'cellspacing': 'cellSpacing',
      'cellpadding': 'cellPadding',
      'rowspan': 'rowSpan',
      'colspan': 'colSpan',
      'usemap': 'useMap',
      'frameborder': 'frameBorder',
      'contenteditable': 'contentEditable'
    },
    isArray = Array.isArray || function(object){ return object instanceof Array }
```

#### 元素匹配选择器的实现 #### 
```
  /**
   * 元素是否匹配选择器
   * @param  {[type]} element  [description]
   * @param  {[type]} selector [description]
   * @return {[type]}          [description]
   */
  zepto.matches = function(element, selector) {
    // 没参数，非元素，退出
    if (!selector || !element || element.nodeType !== 1) return false

    // 如果浏览器支持MatchesSelector，就直接调用
    var matchesSelector = element.matches || element.webkitMatchesSelector ||
                          element.mozMatchesSelector || element.oMatchesSelector ||
                          element.matchesSelector
    if (matchesSelector) return matchesSelector.call(element, selector)

    // 如果浏览器不支持MatchesSelector
    var match, parent = element.parentNode, temp = !parent

    // 元素没有父元素，存入到临时的div tempParent
    if (temp) (parent = tempParent).appendChild(element)

    /*
      再通过父元素来搜索此表达式。
      找不到-1，找到有索引从0开始
      注意：~取反位运算符，作用是将值取负数再减1，例如-1变成0、0变成-1
     */ 
    match = ~zepto.qsa(parent, selector).indexOf(element)

    // 清理临时父节点
    temp && tempParent.removeChild(element)
    
    // 返回匹配
    return match
  }

```
知识点讲解：
elem.matchesSelector(selector)  判断元素elem 是否匹配 selector

例如：
```
$("#id").click(function(e){
	if(e.target.matches('a.btn')) {
		e.preventDefault();
		//TODO
	}
});
```
如果matchesSelector不支持，则找出elem的父元素（若没有则创造一个）利用zepto.qsa(parent, selector)来找出满足条件的元素

#### 通过选择器表达式来查找dom ####
```
/**
   * 通过选择器表达式查找DOM
   * 原理 判断选择器的类型（id/class/标签/表达式）
   * 使用对应方法getElementById getElementsByClassName getElementsByTagName querySelectorAll 查找
   * @param  element  [description]
   * @param  selector [description]
   * @return  {Array} [description]
   */
  zepto.qsa = function(element, selector){
    var found,
        maybeID = selector[0] == '#', // ID标识
        maybeClass = !maybeID && selector[0] == '.', // class标识

        nameOnly = maybeID || maybeClass ? selector.slice(1) : selector, // 是id或者class，就取'#/.'后的字符串，如'#test'取'test'
        
        isSimple = simpleSelectorRE.test(nameOnly) // 是否位单个选择器 没有空格

    return (element.getElementById && isSimple && maybeID) ? // Safari DocumentFragment doesn't have getElementById
      // 通过getElementById查找DOM，找到返回[dom]，找不到返回[]
      ( (found = element.getElementById(nameOnly)) ? [found] : [] ) :

      // 当element不为元素节点或document frgment时，返回空
      // 元素element 1  属性attr 2  文本text 3  注释comments 8  文档document 9  片段fragment 11
      (element.nodeType !== 1 && element.nodeType !== 9 && element.nodeType !== 11) ? [] :
      slice.call(
        // 如果是class，通过getElmentsByClassName查找DOM
        isSimple && !maybeID && element.getElementsByClassName ? // DocumentFragment doesn't have getElementsByClassName/TagName
          
          maybeClass ? element.getElementsByClassName(nameOnly) : // If it's simple, it could be a class
          // 如果是标签名，通过getElementsByTagName
          element.getElementsByTagName(selector) : // Or a tag
          
          // 最后调用querySelectorAll
          element.querySelectorAll(selector) // Or it's not simple, and we need to query all
      )
  }
```

整个逻辑过程：
首先判断selector是ID类型还是class类型，将标识先赋值准备好。

知识点讲解：
1. documentFragment节点不属于文档数，继承的parentNode属性总是null

特殊行为：当请求把一个documentFragment节点插入文档树时，插入的不是documentFragment自身，而是它的所有子孙节点。这使得该节点成了有用的占位符，暂时存放那些一次插入文档的节点。
有利于实现文档的剪切、复制和粘贴操作

2. element.querySelectorAll(selector)意思是 查找element的子元素并且document内css选择器符合selector的元素