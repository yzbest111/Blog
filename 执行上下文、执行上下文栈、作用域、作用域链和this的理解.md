## 作用域
- 作用域即词法环境，在我们写代码的时候就已经确定了，可以将其定义为一套规则。这套规则用来管理JS引擎如何在当前作用域以及嵌套的子作用域中根据标识符名称进行变量查找。

## 作用域链
作用域链和作用域是完全不同的概念。
- 作用域链，由当前环境和和上层环境的一系列变量对象组成，它保证了当前执行环境对符合访问权限的变量和函数的有序访问。作用域链是在函数声明阶段确认的。或者，结合JS引擎理解的话，作用域链是在代码解析阶段确认的。

## 执行上下文（Execution Context）
执行上下文可以理解为代码的执行环境，JS的执行环境分为三种：全局环境、函数环境、eval环境。
执行上下文的生命周期有两个阶段：准备阶段和代码执行阶段。
在准备阶段会做三件事：（1）用arguments创建当前执行上下文的活动对象；（2）确定当前执行上下文的作用域链；（3）绑定当前执行上下文的this对象

## 执行上下文栈（EC Stack）

