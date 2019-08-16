# 前端

### ModelAndView

方便一次返回这两个对象的holder，Model和View仍是分离的概念

最简单的ModelAndView是持有View的名称的返回，View名称被view resolver解析。

#### addObject()

必须指定名字，没有名字时将调用spring自己定义的Conventions 类的.getVariableName()方法来为这个model生成一个名字。

### FTL(FreeMarker Template Language)

#### 模板引擎

一种基于模板和要改变的数据， 并用来生成输出文本（[HTML](https://baike.baidu.com/item/HTML)网页、[电子邮件](https://baike.baidu.com/item/电子邮件)、[配置文件](https://baike.baidu.com/item/配置文件)、[源代码](https://baike.baidu.com/item/源代码/3969)等）的通用工具。

#### 工作流程

模板文件存放在[Web服务器](https://baike.baidu.com/item/Web服务器)上，当有人来访问这个页面，FreeMarker就会介入执行，然后动态转换模板，用最新的数据内容替换模板中${...}的部分，之后将结果发送到访问者的[Web浏览器](https://baike.baidu.com/item/Web浏览器)中。

#### 基本语法

1.  插值：${...}

2. FTL标签（即指令）：以#开始

   * if：判断

     `<#if condition>` and `</#if>`

     `<#if condition>`  `<#else> `    `</#if>`

   * list：列出

     <#list *sequence* as *loopVariable*>*repeatThis*</#list>

   * include：包含，将另一个文件插入当前模板

     <#include "文件相对目录">

3.  注释：<#--  comment -->
4. bulit-ins：内置函数(变量)，用"?"符号访问而不是"."

