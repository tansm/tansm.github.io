# Silverlight 4处理StartWith的BUG

> 原文：https://www.cnblogs.com/tansm/archive/2011/05/20/2051726.html

环境：

Windows 2003 32位中文版

Silverlight 4.0 中文版

描述

一个54KB左右的字符串，做StartWith操作，传入的字符仅7个字符，Silverlight消耗了2分钟。

分析

无

改为传入第二个参数，指定使用Ordina ，飞快处理。
