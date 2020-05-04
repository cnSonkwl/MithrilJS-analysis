## 简单看MithrilJS源码
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


> 本人开始比较迷惑，代码是m.render(arg,arg)，不是应该先执行render,怎么直接执行m()了，后来多看几遍代码，自我苦笑，自己忘记了函数编程的基本理论，这里需要自我检讨。

便于理解，把测试代码拆分一下。

	var test=m("p","test")
	m.render(document.body,test)


回到断点调试1
这个m()回调一个hyperscript.apply()函数,arguments实参数组,即调用hyperscript.apply(this,["p","test"])

> apply(),一个知识点，与call()一样,用于改变函数this的指向，apply使用arguments实参数组传递数据，即m("p","test")的两个参数打包成数组传递到hyperscript里面

#### 断点调试2

	function hyperscript(seletor){
		//这里是判断seletor的类型，必须是string
		if (selector == null || typeof selector !== "string" && typeof selector !== "function" && typeof selector.view !== "function") {
			throw Error("The selector must be either a string or a component.");
		}
		//调用hyperscriptVnode
		var vnode = hyperscriptVnode.apply(1, arguments)
		//虚拟节点vnode

		if (typeof selector === "string") {
			//vnode序列化
			vnode.children = Vnode.normalizeChildren(vnode.children)
			if (selector !== "[") return execSelector(selectorCache[selector] || compileSelector(selector), vnode)
		}
		vnode.tag = selector
		return vnode
	}

先看函数hypersrcipt(seletor)，只有一个参数，那么selector=arguments[0]="p",调用hyperscriptVnode 

	var hyperscriptVnode = function() {
		//start=1
		var attrs1 = arguments[this], start = this + 1, children1
		//atrrs1= arguments[1]="test"
		if (attrs1 == null) {
			attrs1 = {}
		} else if (typeof attrs1 !== "object" || attrs1.tag != null || Array.isArray(attrs1)) {
			//是字符串
			attrs1 = {} //属性清空
			start = this //start=1
		}
		if (arguments.length === start + 1) {
			//进入子标签定义
			children1 = arguments[start]
			if (!Array.isArray(children1)) children1 = [children1]
		} else {
			children1 = []
			while (start < arguments.length) children1.push(arguments[start++])
		}
		//进入虚拟节点定义
		return Vnode("", attrs1.key, attrs1, children1)
	}


	function Vnode(tag, key, attrs0, children0, text, dom) {
		return {tag: tag, key: key, attrs: attrs0, children: children0, text: text, dom: dom, domSize: undefined, state: undefined, events: undefined, instance: undefined}
	}

Vnode其实就一个功能，返回一个json,用户节点的定义。返回到hypersrcipt函数，进行vnode.normalizeChildren(vnode.children)

	Vnode.normalizeChildren = function(input) {
		var children0 = []
		if (input.length) {
			var isKeyed = input[0] != null && input[0].key != null
			// Note: this is a *very* perf-sensitive check.
			// Fun fact: merging the loop like this is somehow faster than splitting
			// it, noticeably so.
			for (var i = 1; i < input.length; i++) {
				if ((input[i] != null && input[i].key != null) !== isKeyed) {
					throw new TypeError("Vnodes must either always have keys or never have keys!")
				}
			}
			for (var i = 0; i < input.length; i++) {
				children0[i] = Vnode.normalize(input[i])
			}
		}
		return children0
	}

Vnode.normalizeChildren检查了vnode子节点相关类型，进去Vnode.normalize，并返回子节点数组。

	Vnode.normalize = function(node) {
		if (Array.isArray(node)) return Vnode("[", undefined, undefined, Vnode.normalizeChildren(node), undefined, undefined)
		if (node == null || typeof node === "boolean") return null
		if (typeof node === "object") return node
		return Vnode("#", undefined, undefined, String(node), undefined, undefined)
	}

Vnode.normalize序列化节点，返回#，不是‘[’，hyperscript中执行函数execSelector(selectorCache[selector] || compileSelector(selector), vnode)

	function compileSelector(selector) {
		//selector="p"
		var match, tag = "div", classes = [], attrs = {}
		while (match = selectorParser.exec(selector)) {
			var type = match[1], value = match[2]
			if (type === "" && value !== "") tag = value
			else if (type === "#") attrs.id = value
			else if (type === ".") classes.push(value)
			else if (match[3][0] === "[") {
				var attrValue = match[6]
				if (attrValue) attrValue = attrValue.replace(/\\(["'])/g, "$1").replace(/\\\\/g, "\\")
				if (match[4] === "class") classes.push(attrValue)
				else attrs[match[4]] = attrValue === "" ? attrValue : attrValue || true
			}
		}
		if (classes.length > 0) attrs.className = classes.join(" ")
		return selectorCache[selector] = {tag: tag, attrs: attrs}
	}

match = selectorParser.exec(selector),匹配tagname

最终返回selectorCache[selector]并执行execSelector，返回vnode链表

> 到这里，m("p","test")就执行完成，最终返回一个vnode链表，可以推测，后面代码都是不断返回这个vnode链表，最后m.render()遍历vnode链表，返回HTML内容。

