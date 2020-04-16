---
title: 前端基础『Html+CSS+JavaScript』
tags: [前端,html,css]
---

1. 使\<div>自适应屏幕宽度：width:100%

```html
    <div style="background-color:#FFD700;height:200px;width:100%;float:left;">
```

2. **JSON.parse(text[,reviver])，String转JsonObject**
   - **text:**必需， 一个有效的 JSON 字符串。
   - **reviver:** 可选，一个转换结果的函数， 将为对象的每个成员调用此函数。

3. **JSON.stringify(value[, replacer[, space]])，将JavaScript对象转String**

   - value:

     必需， 要转换的 JavaScript 值（通常为对象或数组）。

   - replacer:

     可选。用于转换结果的函数或数组。

     如果 replacer 为函数，则 JSON.stringify 将调用该函数，并传入每个成员的键和值。使用返回值而不是原始值。如果此函数返回 undefined，则排除成员。根对象的键是一个空字符串：""。

     如果 replacer 是一个数组，则仅转换该数组中具有键值的成员。成员的转换顺序与键在数组中的顺序一样。当 value 参数也为数组时，将忽略 replacer 数组。

   - space:

     可选，文本添加缩进、空格和换行符，如果 space 是一个数字，则返回值文本在每个级别缩进指定数目的空格，如果 space 大于 10，则文本缩进 10 个空格。space 也可以使用非数字，如：\t。

4. 