---
title: Vim使用笔记
date: 2019-05-18 12:01:54
categories: 工具&技巧 
tags: vim
---

#### 搜索替换
- sudo 保存只读 ```:w !sudo tee %```
- 文本替换  
``` :s/foo/bar/g```  当前行
```:%s/foo/bar/g``` 全文
```:.,$/foo/bar/g``` 当前行到末尾 
```:.,+2s/foo/bar/g``` 当前行及接下来2行
```:2,10/foo/bar/gic``` i:大小写不敏感 I:敏感 c:需要确认 C:不需确认 g:所有出现都替换
#### 宏
- 录制重复宏， e.g. 注释多行。 normal模式下 输入```qa```, ```^```光标到行头，```i``` 进入insert 模式，输入“#”，```esc``` 进入normal，```j```光标到下一行，```q```结束录制，```@a```注释当前行，```10@a```注释10行。
#### tabs
- ```:tabnew``` 打开新tab
- ```:tabc``` 关闭tab
- ```:tabo```关闭其他tab
- ```tabs``` 查看所有tab
- ```tabn 1```跳到tab1
- ```tabm 0 ``` 标签页到第一位置， 次序从0开始
#### 切屏
- ```vsplit filename```纵切 没有filename默认当前文件
-  ```split filename ``` 横切 
- ```ctrl w h|i|k|l``` 切换屏幕
- ```ctrl w w``` 后一个窗口 ```ctrl w p``` 前一个窗口
