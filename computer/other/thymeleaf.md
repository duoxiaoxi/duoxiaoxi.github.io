# Thymeleaf





## 标签总结

* 片段包含
  * `th:insert` `th:replace`
* 遍历
  * `th:each`
* 条件
  * `th:if` `th:unless` `th:switch` `th:case`
* 声明变量
  * `th:object` `th:with`
* 修改变量属性
  * `th:attr` `th:attrappend` `th:attrprepend`
* 修改HTML属性
  * `th:value` `th:href` `th:id` ......
* 修改HTML标签体
  * `th:text` `th:utext`
* 声明片段
  * `th:fragment`
* 移除片段
  * `th:remove`



## 表达式总结

* `${...}`: 通过OGNL表达式来获取各种数据
  * Root
    * `param:Map` 
    * `session:Map`
    * 
  * Context
    * `#ctx`
    * `#session`
    * `#request`
    * `#response`
    * `#vars`
    * `#servletContext`
    * `#locale`
* `*{...}`: 在











## 基本语法









### th:text

```xml
<ele th:text="${EXP}"></ele>
```

此标签可以将元素中的内容替换为属性的值.



### th:任意属性

```html
<div id="div1" th:id="div2">
```

`th:标签可用的任意属性`都可以替换标签原有的属性值.



### th:each

