---
layout: post
title:  "移花接木暗黑版(映射ExcelserverRCID)"
categories: 日志
tags:  excel服务器 excelserver ExcelserverRCID 映射 暗黑科技
---

* content
{:toc}

## 引子
前阵子村里有筒子提出了映射ExcelserverRCID这个问题，村长因为忙于开发ESAP3.0迟迟未写，现在终于告一段落，赶紧自觉补上。

## 什么是ExcelserverRCID
ExcelserverRCID是ES表单的内置表单ID，除了ExcelserverRCID，还有:
- ExcelserverRTID 模板ID
- ExcelserverWIID 工作流ID
- ExcelserverRN 明细表行
- ExcelserverCN 明细表列
- 等等

## ExcelserverRCID有什么用
主要用于工作台显示，一份表单要在工作台显示，那么需要表单上有这个字段值，并且在ES_RepCase表中的rcId字段也存在这个值。

在ESAP3.0的很多sql模板中，都会有这样一段代码：\{\{template "repcase"\}\}，即引用repcase模板。

repcase模板(位于sql/esap/es.post)的定义如下：

{% raw %} 
```sql
{{define "repcase"}}
  {{if es}}
	insert es_repcase (rcid,rtid,fillDept,fillDeptName,fillUser,fillUserName,state,fillDate,lstFiller,lstFillerName,lstFillDate) 
	values(:rcid,:rtid,1,'esap',1,'esap',1,getdate(),1,'esap',getdate())
  {{end}}
{{end}}
```
{% endraw %} 

其实就是在检查是否是ES库模式，如果是就尝试插入记录到ES_RepCase表，实现工作台显示。

## 如何映射ExcelserverRCID
既然ExcelserverRCID如此重要，早期版本的ES是不能直接看到该字段值的，如果想在工作台显示出来，那么就需要一些非常规手段了。

* 首先，建立一个测试模板，模板中包含两个字段。
![](/img/rcid-1.png)

* 其次，随便填点啥，然后用sql查询一下，此时rcid已经生成了。
![](/img/rcid-2.png)

* 再次，开始施展“暗黑版”移花接木，手工修改ES_DataField表中的“字段2”，将RealName改成“excelserverrcid”，isIdentity改为1。
![](/img/rcid-3.png)

* 接下来就是见证奇迹的时刻，重新登陆打开工作台，撒花！
![](/img/rcid-4.png)


## 拓展
1. 同上方式，还可以把ExcelserverRTID，ExcelserverRN等系统字段都映射出来，在工作台、表单中显示。

2. 完成映射以后，可以把物理表中的字段2删掉，没有任何影响！

3. 如果要提数中使用ExcelserverRCID，其实可以直接手工输入，不映射也能使用哦！

* by @一零村长

* 2017-8-29

