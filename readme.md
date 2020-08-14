#### Vue.js 源码剖析-响应式原理、虚拟 DOM、模板编译和组件化

##### 一、简答题

* 1、请简述 Vue 首次渲染的过程。
--->解析：     
  - 在首次渲染之前先进行Vue初始化实例成员、静态成员
  - 完成初始化后执行Vue的构造函数new Vue()
  - new Vue()中调用this._init()方法，是整个vue的入口
  - 在_init()中调用entry-runtime-compiler.js中的vm.$mount()方法，这个方法的作用是帮我们把模板编译成render函数，首先会判断是否传入了render函数，如果没有传入的话会获取template选项，如果template也没有的话，就会把el中的内容作为模板，然后使用compileToFunctions()把模板编译成render渲染函数，保存在options.render中。
  - 接着调用index.js中的vm.$mount()方法，这个方法中会重新获取el(因为在运行时版本是不会进这个入口)
  - 接着调用mountComponent(this,el)首先判断是否有render选项，如果没有但是传入了模板并且当前是开发环境的话会发送警告，运行时不支持编译器，然后触发beforeMount生命周期钩子函数，定义了uptateComponent方法。创建Watcher对象，在创建watcher完成后会调用一次get方法，在get中会调用updateComponent()函数，在此updateComponent()方法中调用了vm._update()和vm._render()方法，vm._render()中调用实例化时Vue传入的render()或者编译template生成的render()生成VNode，vm._update()中调用了vm.__patch__()方法将虚拟DOM转换成真实DOM并且挂载到页面，他会把生成的真实DOM设置到vm.$el中，触发mounted生命周期钩子函数，最终返回vue实例。

* 2、请简述 Vue 响应式原理。     
--->解析：      
  - data的属性被转换成getter和setter,并且记录相应的依赖。当它被改动时，会通知相应的依赖。
  - 所有的组件实例都有相应的watcher实例，而watcher实例会依赖相应的setter。
  - 当数据变化的时候，会调用setter，而setter会通知watcher实例，watcher会更新相应的视图

* 3、请简述虚拟 DOM 中 Key 的作用和好处。    
--->解析：      
  - 设置key属性，可以跟踪每个节点的身份，从而重用和重新排序现有元素
  - 设key DOM操作会少很多
  - 在vue-cli创建的项目中，如果在使用v-for时不设置key会有警告   


* 4、请简述 Vue 中模板编译的过程。   
--->解析：       
  - 在compileToFunctions(template, ...) 先从缓存中加载编译好的render函数，如果缓存中没有调用compile(template, options)开始编译
  - 在compile(template, options)中，首先合并选项，再调用baseCompile(template,.trim(), finalOptions)编译模板。
  - 把模板和合并后的选项传递给baseCompile(template,.trim(), finalOptions)，内部完成了模板编译核心的三件事：1.调用parse()把模板转换成AST tree，然后调用optimize()对语法树进行优化，标记语法树中的静态根节点，静态根节点不需要每次都重绘，patch的过程中会跳过静态更节点，最后调用generate()将语法树生成js形式的字符串代码
  - compile执行完成后又会回到编译的入口函数compileToFunctions()，内部继续把字符串形式的代码转换成函数，通过调用createFunction()，当render和staticRenderFns初始化完毕，最终他们会被挂载到Vue实例的options对应的属性中。到此，模板编译的过程就结束。