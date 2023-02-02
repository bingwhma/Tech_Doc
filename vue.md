
# 项目初始化
npm init vue@latest  
cd 项目名  
npm install  
npm run dev  

# 模板语法
## 文本插值 
“Mustache”语法 (即双大括号)：   
<span>Message: {{ msg }}</span>  
## 原始 HTML
插入 HTML，你需要使用 v-html 指令  
<span v-html="rawHtml"></span>  
## Attribute 绑定 
响应式地绑定一个 attribute，应该使用 v-bind 指令： 
如果绑定的值是 null 或者 undefined，那么该 attribute 将会从渲染的元素上移除。  
`<div v-bind:id="dynamicId"></div>`   
简写  
`<div :id="dynamicId"></div>`   
绑定对象  
`<div v-bind="objectOfAttrs"></div>`

`<a :href="url"> ... </a>` href 就是一个参数，它告诉 v-bind 指令将表达式 url 的值绑定到元素的 href attribute 上。  
## 动态参数
`<a v-bind:[attributeName]="url"> ... </a>`
`<a :[attributeName]="url"> ... </a>`
attributeName 会作为一个 JavaScript 表达式被动态执行，计算得到的值会被用作最终的参数。举例来说，如果你的组件实例有一个数据属性 attributeName，其值为 "href"，那么这个绑定就等价于 v-bind:href。

下例中当 eventName 的值是 "focus" 时，v-on:[eventName] 就等价于 v-on:focus  
`<a v-on:[eventName]="doSomething"> ... </a>`  
`<a @[eventName]="doSomething">`  

## 修饰符 Modifiers 
### 事件修饰符
.stop 阻止事件继续传播  
.prevent 阻止标签默认行为  
.capture 使用事件捕获模式,即元素自身触发的事件先在此处处理，然后才交由内部元素进行处理  
.self 只当在 event.target 是当前元素自身时触发处理函数  
.once 事件将只会触发一次  
.passive 告诉浏览器你不想阻止事件的默认行为  
### v-model的修饰符
.lazy 默认情况下，v-model同步输入框的值和数据。可以通过这个修饰符，转变为在change事件再同步  
.number 自动将用户的输入值转化为数值类型  
.trim  自动过滤用户输入的首尾空格
### 键盘事件的修饰符
.enter  
.tab  
.delete  
.esc  
.space  
.up  
.down  
.left  
.right  

.ctrl  
.alt  
.shift  
.meta  

### element的修饰符
对于elementUI的input，我们需要在后面加上.native, 因为elementUI对input进行了封装，原生的事件不起作用。

# VUE router
npm install vue-router --save   导入   
## 功能
  支持历史模式 hash模式  
  嵌套路由  
  路由参数 Query Param 
  编程式路由  
  命名路由  

`<router-link to='/aaa'><router-link>`
`<router-view></router-view>`
