---
title: "如何將 VRoid Studio 的模型匯入 Unity 中並加入 Mixamo 的動畫？"
date: 2021-10-25T12:38:23+08:00
categories:
- "技術"
Tags: 
- "Vtuber"
- "Unity"
- "VRoid"
- "Mixamo"
keywords:
- "GanniPiece"
- "Vtuber"
- "VRoid"
- "Unity"
- "Mixamo"
draft: false
---

## 前言
最近和想要來了解 vtuber 的生態與製作，因此開始接觸 vtuber 相關的知識，而要了解一個領域最好的方式就是實作啦！今天我就會簡單的介紹如何透過 [VRoid Studio][1] 這套工具快速的製作一個 vtubuer 的模型，並且將這個工具匯出的 VRM 檔案放到 Unity 中，再進一步的在模型上套入動畫。

在這篇文章中，我們會依照以下的步驟逐一介紹

1. 模型製作
	- 透過 VRoid 製作模型，並且匯出成的檔案形式
	
2. 檔案轉換
	- 使用 [Blender][2] 將 vrm 轉成 fbx 格式
	
3. 匯入 [Unity][3]
	- 匯入 [Mixamo][7] 的動畫
	- 製作動畫
	- 匯入模型


## 模型製作
[VRoid Studio][1] 是一套相當方便的工具，能讓我們快速的建立角色模型，同時也是一個免費軟體！對於新手來說相當的友善。

首先，我們先去官方網站依照作業系統下載。以筆者的 MBP 來說，點擊 **MacOS** 進行下載即可。

![](https://i.imgur.com/S5PmmdI.png)

下載後將 VRoid Studio 安裝並開啟，即可看到以下畫面。

![](https://i.imgur.com/2npxcvX.jpg)

點擊左方的 `Create New`，創立新的專案檔。之後依照個人喜好進行角色的設定，[VRoid Studio][1] 中提供了多種不同面向的調整，比如說髮型、服飾、身長⋯⋯等等，對於剛入手的使用者相當的友善，就像在玩遊戲創建角色時的階段相似。

調整完細節後，選擇右上角輸出的圖案 ⬆️，選擇 `Export as VRM` 。點擊後會出現 `VRM Settings` 的視窗，裡面有兩個必填的欄位 `Title`, `Creator`，在 `Title` 欄位填寫模型的名字，並在 `Creator` 欄位填入自己的名字。填寫完後，對其他 optional 的選項進行設定，即可按下下方的 `Export` 輸出。

## 檔案轉換
將 VRM 轉換成 [Filmbox (FBX)][4] 的格式可以方便讓我們直接套用 Mixamo 上面的動畫。如果只是想將模型匯入 Unity 中，可以直接跳到[下一個章節](#匯入\ Unity)

以下我將使用 [Blender][2] 來將 VRM 轉換成 FBX。我們需要在 blender 裝上兩個不同的插件：[CATS Blender Plugin][5] 和 [VRM Add-on for Blender][6]，按下專案連結後會導向他們的 github 頁面，可以直接點擊 `⬇️ Code > Download ZIP` 下載。

### 插件安裝
接著，開啟安裝好的 Blender 應用程式，在選項列點擊 `Edit > Preferences > Add-ons`，（mac 使用者可以使用快捷鍵 `cmd` + `,`）開啟以下介面。

![](https://i.imgur.com/i2q0CST.png)

按下右上角的 ⬇️ Install ，分別選擇方才下載的兩個 `.zip` 檔案：cats-blender-plugin-master.zip 與 VRM_Addon_for_Blender-release.zip 進行安裝。一旦安裝完後，按下右上角的 🔄 refresh 即可在右下角的區塊找到我們安裝的插件：

- 3D View: Cats Blender Plugin
- Import-Export: VRM format

> 如果搜尋後沒有出現這兩個插件，確認一下上方的 `Community` 是否有被選取


### 轉換輸出
> *In: VRM / Out: FBX*

按下 `File > Import > VRM (.vrm)` 選擇我們在[前一個階段](#模型製作)製作的 VRM 模型，選取模型後即可在主畫面看到方才建立的人物形象。如果主畫面上有正方形的 cube 物件的話，可以到右方的 `Scene Collection` 中將其移除。到這個階段為止，你應該會在畫面上看到如下圖所示的狀態。

![](https://i.imgur.com/F1monk9.jpg)

此時的人物是沒有 texture 的模型，開啟右方的設定列並選取 **CATS** 的分頁，找到 `Fix Model` 的按鈕，先到按鈕旁的 🔧設定，移除其他選項然後留下 `Fix Materials` 的選項後，再按下 `Fix Model` 按鈕，就會看到有 texture 的模型了。

![](https://i.imgur.com/HublXMz.gif)

最後，我們點選左上角的 `File > Export > FBX (.fbx)` 將模型輸出，輸出後會選產生一對應的 FBX 檔案，如此一來就完成了這個階段的任務。

## 匯入 Unity
歷經了千辛萬苦，總算來到我們的最後一個階段啦！在這個階段中我們要將 FBX 格式的模型匯入 Unity 之中，並且將其套上 Mixamo 上面下載的動畫，來實現模型的骨骼移動。

### 安裝插件
欲快速的在 Unity 中讀入 VRM 的模型檔案，我們需要安裝 [UniVRM](https://github.com/vrm-c/UniVRM/releases) 這個插件。如果在前一步已將模型轉換成 FBX 檔案的話，亦可以跳過這一步往下進行。

> **UniVRM** 這個套件是一個用來匯入與匯出 VRM 的 `unity package` 

在該專案之 Releases 頁面中，下載 `.unitypackage` 結尾（注意：檔名中不含 **Samples** 之檔案）的套件至本地端。接著，開啟 Unity 專案，在左下角的 `Project`  頁面下，按右鍵選擇 `Import package` 匯入方才下載的 UniVRM (.unitypackage) 檔案加入套件。

![](https://i.imgur.com/4RZm3AV.gif)

匯入該套件後，我們應該要在上方的狀態列看到 `VRM0` 的按鈕選項，該選項底下有 `VRM0 > Import` 的選項，能讓我們直接匯入 VRM 的模型。

### 動畫模型
對於像我這種沒有美術天份和建模能力的人來說，[Mixamo][7] 這個網站實在是一大救贖。在 Mixamo 上面有相當多現成的動畫動作供我們下載使用，我們可以直接在線上套用在[之前做好的 FBX 檔案](#檔案轉換)，選擇一個你喜歡的動作開始這個章節的練習吧！

#### 線上套用
如果你有模型的 FBX 檔案，就可以將自己的模型在 Mixamo 線上套用動畫（前提是你必須先創建一個 Mixamo 的帳號）。在 `Animations > UPLOAD CHARATERS` 上傳自己的 FBX 模型人物，就可以套用不同的動畫動作了。

![](https://i.imgur.com/Vgr43BV.gif)

找到自己喜歡的動畫後，只要按下下載，把 FBX 格式的檔案下載下來匯入 Unity 之中即可。

#### Unity 內套用
若要在 Unity 中才套用動畫會相對麻煩一點，首先我們一樣可以在 Mixamo 中下載動畫模型，將該 FBX 加入 Unity 中的 Assets 。

點擊我們在[前一步](#安裝插件)匯入的人物模型，並在 `Inspector` 視窗中，按新增一個動畫物件，選擇 `Inspector > Add Component > Animation` 加入動畫。在該動畫物件中，設定 `Animation > Animation` ，選擇剛剛匯入 Unity 的 Mixamo 動畫。

![](https://i.imgur.com/MJGpky1.gif)

如此一來，便完成模型的匯入與動畫的套用啦！

![](https://i.imgur.com/l0XJMHT.gif)

## 小結
這次將簡單的 vtuber 模型製作與 Unity 端匯入的過程實際做了一次。匯入 Unity 後的操作就相對的方便許多，接下來就要把近期買的 motion capture 工具與本次製作的模型搭上線，來試著開一場直播啦！

別忘了之後要多多打賞唷！


[1]: https://vroid.com
[2]: https://www.blender.org
[3]: https://unity.com
[4]: https://en.wikipedia.org/wiki/FBX
[5]: https://github.com/absolute-quantum/cats-blender-plugin
[6]: https://github.com/saturday06/VRM_Addon_for_Blender
[7]: https://www.mixamo.com/#/