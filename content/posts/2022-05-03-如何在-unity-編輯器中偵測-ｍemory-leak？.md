---
title: 如何在 Unity 編輯器中開啟並偵測 Ｍemory Leak？
slug: how-to-detect-memory-leak-in-unity-project
description: null
author: null
date: 2022-05-03T06:51:18.454Z
lastmod: 2022-05-03T08:55:34.665Z
draft: false
tags:
  - Unity
  - 技術
  - Memory Leak
categories:
  - 技術
keywords: Memory Leak
---
<!--more-->

## 前言
Memory Leak 通常被翻譯作記憶體流失，是記憶體的管理不當所產生的現象。發生記憶體流失時，不一定會馬上造成程式本身運行的問題，然而，很有可能因為可用記憶體數量的減少，逐漸造成電腦效能降低，嚴重時可能會造成程式一些不可預期的錯誤，甚至是安全性的問題[1]。

> Memory: 嗯？有人在臭？

通常造成記憶體流失的一大原因就是忘記釋放已經分配的記憶體，好比說在動態時我們 heap 中像記憶體要 (new) 了一段記憶體位置，結果使用完了卻忘記釋放，就會有「佔著茅坑不拉屎」的情況。然而因為發生的時間不可預期（無從得知什麼時候馬桶會被炸爛），這使得除錯上變得相當困難。

在 Unity 上，如果出現 memory leak 的情況，console 會出現以下錯誤提示：

![](https://i.imgur.com/7UpwLg1.png)
```unity
A Native Collection has not been disposed, resulting in a memory leak. Enable Full StackTraces to get more details.
```

除此之外，就沒有其他提示了，這讓我們很難這行資訊進行偵錯。

[1]: https://owasp.org/www-community/vulnerabilities/Memory_leak

## 作法
根據 Unity console 的錯誤提示，我們必須要開啟 Full StackTraces 方能得到進一步的資訊。

1. 首先第一步，我們需要安裝 Jobs 這個套件.

    1. 點選選單欄中的 `Window` > `Package Manager`
    2. 選取 package 的來源為 `Unity Registry`
    3. 搜尋 Jobs 這個套件
    4. 點擊安裝
    5. 完成安裝後，在選單欄中會出現 `Jobs` 的選項，若無請回第一步確認是否安裝成功

2. 接著我們要開啟 leak 偵測的功能
    
    1. 選擇選單欄中的 `Jobs` > `Leak Detection`
    2. 在 `Leak Detection` 中將預設的 `On` 改為 `Full Stack Traces (Expensive)`

3. 重新測試程式

此時，若遇上 memory leak 的情況，在 console 我們就可以清楚的看到發生的位置，並進一步的到該處進行修改了！

![](https://i.imgur.com/UGRSrNt.png)

## 小結
Memory leak 的問題起因於記憶體的管理不當，進一步造成程式無可預期的錯誤。這種情況讓程式設計師很難針對其除錯，特別是如果不是自己撰寫的程式碼，有時候要找到動態配置記憶體的位置就要花上一點時間閱讀。這篇文章中我們提供了在 Unity 上快速找到 memory leak 的地方，希望將來再遇上相同問題時，能更快速的解決！