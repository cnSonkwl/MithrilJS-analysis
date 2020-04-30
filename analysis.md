## mithril 简单看源码
---

### 一个简单的测试代码

文件index.html

	<script src="mithril.js"></script>
	<script>
	m.render(document.body,m("p","test"))
	</script>


使用chrome浏览器调试，设置断点于m.render(document.body,m("p","test"))处。

> 我们先略过mithril.js初始化的部分，从用户已执行代码开始一步一步分析。 
  
#### 断点调试1  

	var m = function m() { return hyperscript.apply(this, arguments) }


> 本人开始比较迷惑，代码是m.render(arg,arg)，不是应该先执行render,怎么直接执行m()了，
后来多看几遍代码，自我苦笑，自己忘记了函数编程的基本理论，这里需要自我检讨。

便于理解，把测试代码拆分一下。

	var test=m("p","test")
	m.render(document.body,test)


回到断点调试1
这个m()回调一个hyperscript.appley()函数,arguments实参数组,即调用hyperscript.appley(this,["p","test"])

#### 断点调试2

	function hyperscript(seletor){
		//这里是判断seletor定义类型，必须是string或函数
		if (selector == null || typeof selector !== "string" && typeof selector !== "function" && typeof selector.view !== "function") {
			throw Error("The selector must be either a string or a component.");
		}
		//调用hyperscriptVnode
		var vnode = hyperscriptVnode.apply(1, arguments)
		if (typeof selector === "string") {
		vnode.children = Vnode.normalizeChildren(vnode.children)
		if (selector !== "[") return execSelector(selectorCache[selector] || compileSelector(selector), vnode)
		}
		vnode.tag = selector
		return vnode
	}

进入hyperscriptVnode  

	var hyperscriptVnode = function() {
		var attrs1 = arguments[this], start = this + 1, children1
		if (attrs1 == null) {
			attrs1 = {}
		} else if (typeof attrs1 !== "object" || attrs1.tag != null || Array.isArray(attrs1)) {
			attrs1 = {}
			start = this
		}
		if (arguments.length === start + 1) {
			children1 = arguments[start]
			if (!Array.isArray(children1)) children1 = [children1]
		} else {
			children1 = []
			while (start < arguments.length) children1.push(arguments[start++])
		}
		return Vnode("", attrs1.key, attrs1, children1)
	}
