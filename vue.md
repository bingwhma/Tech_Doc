
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

## 修饰符 Modifiers 

