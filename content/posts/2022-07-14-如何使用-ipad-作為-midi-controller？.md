---
title: 如何設定使 iPad 成為無線 MIDI Controller？
description: ""
date: 2022-07-14T02:57:32.537Z
preview: null
draft: false
tags:
  - MIDI Controller
  - iPad
  - DAW
categories:
  - 技術
keywords: ""
slug: 如何設定使-ipad-成為無線-midi-controller？
---

<!-- abstract -->
在本文中，我們將紀錄如何將 iPad 作為 MIDI 鍵盤，並在不同平台的電腦上接收訊號。
<!--more-->

## 前言
許多人剛入門進行編曲時，很容易因為 MIDI 鍵盤價格過於而遲遲不敢下手，但是額外的 MIDI 控制器卻可以有效的幫助編曲的流程進行。這時候我們就想，要是人手一台的行動裝置 (手機、平板⋯⋯）可以直接當成 MIDI 控制器那該有多好！

在這篇文章中，我們就來看看要如何把手邊的行動裝置改造成無線 MIDI 控制器。首先會提供一些 iPad 上的 MIDI 控制器的應用程式，接著說明如何對電腦端進行設定，最後我們會進行一個小實測來確認功能正常！

## iPad 端設定
### 應用程式下載
在 iPad 上面，我們需要下載可以發送 MIDI 訊號的應用程式來傳送訊息。底下是幾個 iPad 上面的 MIDI 控制器應用程式，大家可以視自己的需求來進行下載。

| 產品名稱 | 系統平台 | 收費 | 敘述 |
| --- | --- | --- | --- |
| [WiFiMIDI](https://apps.apple.com/tw/app/wifimidi/id456593414) | iOS/iPadOS | 免費 | WiFiMIDI 是一個具備鍵盤、鼓墊與控制訊號發送的 MIDI 控制器。使用者可以在相同網路的情況下，將 MIDI 訊號傳送給 mac 上匹配的應用程式，比如說 GarageBand, Reason 等數位工作站上。|
| [Lemur](https://apps.apple.com/us/app/lemur/id481290621) | iOS/iPadOS | $24.99 USD | Lemur 是 iOS 上的 MIDI/OSC 控制器 |
| [TouchOSC](https://apps.apple.com/us/app/touchosc/id288120394) | iOS/iPadOS | $4.99 USD | TouchOSC 讓您可以在 iPad 的觸控介面上新增旋鈕或是額外控制。可兼容 iOS 5.1.1 以上的版本。 |

接下來，我會以 [WiFiMIDI](https://apps.apple.com/tw/app/wifimidi/id456593414) 這個免費的應用程式來介紹如何進行電腦端的設定連線。

## 電腦端設定

> 開始設定之前請先確認您的平板與電腦在「相同的網路連線」之下 !!!

### macOS

| 步驟說明 | 圖示 |
| --- | --- |
| 1. 開啟「音訊 MIDI 設定應用程式」(Audio MIDI Setup) | ![](https://i.imgur.com/ZT9MhGt.png) |
| 2. 點擊選項列中「視窗」>「顯示 MIDI 錄音室」(Show MIDI Studio) 開啟 MIDI 錄音室設定 | ![](https://i.imgur.com/U70kezI.png) |
| 3. 在 MIDI 錄音室設定畫面中，選擇右上角「🌐」圖示按鈕開啟「MIDI網路設定視窗」 | ![](https://i.imgur.com/QqB8EbQ.png) |
| 4. 在左上角點擊「+」圖示按鈕新增一個工作階段，並將其勾選 | ![](https://i.imgur.com/sIUXfKX.png) |
| 5. 點擊相對應的裝置 (iPad) 後，按下「連線」按鈕 | ![](https://i.imgur.com/HuwctbK.png) |
| 6. 前往數位工作站設定 MIDI 輸入裝置 | ![](https://i.imgur.com/jDLnkmR.png) |


### Windows

| 步驟說明 | 圖示 |
| --- | --- |
| 1. 下載並安裝 [rtpMIDI](https://www.tobias-erichsen.de/software/rtpmidi.html) | ![](https://i.imgur.com/XfyUtZx.png) |
| 2. 開啟「rtpMIDI: using Apple Bonjour」 | ![](https://i.imgur.com/vA6OoEz.png) |
| 3. 點擊左上角「+」字樣按鈕新增一個工作階段，並將其勾選 | ![](https://i.imgur.com/cQkE6DK.png) |
| 4. 點擊左下角「+」字樣按鈕新增一個目的地，並設定目的地資訊。您可以在行動裝置的 WiFi 設定下取得 IP 位置資訊，並將 Port 設定為 5004 | ![](https://i.imgur.com/dQqpDUO.png) |
| 5. 按下連線 | ![](https://i.imgur.com/jvmVSlm.png) |
| 6. 前往數位工作站設定 MIDI 輸入裝置 | 同 macOS 設定之 步驟 6 |


## 實測
### 測試環境
* 裝置：MacBook Pro 2020 (13-inch, 16GB) / iPad Air 4
* 作業系統：macOS Monterey v12.4 / iPadOS 15.5
* 數位工作站：Reaper v6.57

### 示意圖
| | iPad | MacBook (Reaper) |
| --- | --- | --- |
| **圖示** | ![](https://i.imgur.com/YmgdEgf.png) | ![](https://i.imgur.com/b3ED3UM.png) |
| **說明** | 透過 WiFiMIDI 傳送 MIDI 訊號至數位工作站 (Reaper) | 數位工作站確實收取 MIDI 訊息 (黃色訊號條) |

## 小結
在本文中，我們說明了如何透過以 rtpMIDI 的協議，將 iPad 搖身一變成為無線的 MIDI 控制器。並且，我們將這個方法於兩個不同作業系統 (Windows/macOS) 上進行測試，確認此方法實際可行。

儘管本文中僅使用 iPad 來進行說明，但基於 rtpMIDI 的協議，我們同樣可以用相同的方法，使得 Android 的裝置達到同樣的效果，若有興趣的讀者也可以自行嘗試。

（將第二章節提到 iPad 的應用程式改為 Android 應用程式即可）