---
title: 為什麼是 Coroutine？（ㄧ）- Multithreading 不好嗎？
description: ""
date: 2022-05-28T07:54:19.008Z
preview: ""
draft: false
tags:
  - 技術
  - Coroutine
categories:
  - 技術
keywords: ""
slug: 為什麼是-coroutine？（ㄧ）-multithreading-不好嗎？
---

<!--more-->

## 前言
	這陣子在使用 [UniVRM](https://github.com/vrm-c/UniVRM)<sup>[1]</sup> 時，發現在 Unity 中有許多開發情境會使用到 Coroutine 這個方法，好比說開啟系統視窗選取檔案或是載入資源時。在這些情境中，為了不讓這些背景工作影響使用者的體驗，比方說載入較大資源時，視窗停止在當前畫面，應用程式無法進行其他動作的情況，而選擇使用 Coroutine 的解決辦法。
	在這篇文章中，我會說明什麼是 Coroutine ，並且將它與 Thread 進行比較。接著我會更詳細的講解 Coroutine 的作法，並做一個簡單的結論。若您對如何實作有興趣的話，可以參考下一篇文章 [[為什麼是 Coroutine？（二）- 使用 C語言實作 Coroutine]]。
## 內文
	Coroutine 中文翻譯為共常式，顧名思義即是共同運作的次常式 (Subroutine)，我們可以想像他的行為模式是多個 Subroutines 一起共享資源，使得資源有機會在不同任務間輪流使用。對於這些 Subroutines ，我們首先需要一個 Subroutine 負責當作生產者，他也是眾多 Subroutine 中的其中一員，但身份比較特殊，主要是用來管理其他消費者的 subroutine。當某個 Subroutine 分配到資源開始工作後，會在某個時間點交出 (yield) 執行權回生產者。生產者會負責把消費者歸還的資源交給下一個使用的消費者，被管理的 subroutine 間是不會互相遞交資源的權限的。透過這樣的機制，讓次常式間的任務都能獲得資源而進行處理。
	舉個例子來說，Coroutine 的機制就像是炎炎夏日你和朋友們點了一碗冰回家，結果發現店家只付了一支湯匙。為求公平你就告訴大家：「不如我們一人挖一口輪流吃吧！」於是這碗冰的資源就可以在你和你朋友們各自吃冰消暑的任務中被輪流使用，冰仍然是一碗，但你和朋友都因此能吃到美味冰涼的冰品消暑。然而，這個輪流吃的方式看似皆大歡喜，卻因為你的朋友小美貪心，霸佔著冰品不讓其他人吃而惹怒眾人，看似公平的方式卻因為她的一己私心引起眾怒。
	沒錯，相信聰明的你看到第一段 Coroutine 的機制後，一定也想到了這個問題。這個機制看似完美，但是假如有一個人霸佔著資源不釋放呢？我們要記得，作為任務傳遞分派角色的生產者 ，它本身也是個需要資源的次常式，當今天霸佔著資源不放的 Subroutine 不把資源還回去給他，他是無法進行他的分派的任務的。
	這就是協同式多工 (Cooperative Multitasking <sup>[2]</sup> ) 會遇到的問題，所謂的協同式多工通常會出現在使用單核心處理多個任務時使用，好比說方才提到的例子，當我們只有一支湯匙時，就必須使用輪流的方式來讓每個人都吃到冰品而心滿意足。
	另一個常會被拿來與 coroutine 進行比較的作法是多執行緒 (Multi-threading)。透過多執行緒的方式，我們也能在單核上「同時」 <sup>註1</sup> 的進行多個任務，除此之外，兩者都有自己的 Context。不同於 Coroutine 的是，在多執行緒下任務的切換是由作業系統來決定的，這也導致了透過多執行緒來執行多任務的時候，資料是需要受到保護的。我們必須要對關鍵區段 (Critical section) 進行保護，其中包括 mutex 、signal 等作法來實現。
	關於兩者的比較可以參考下表：
		#### 

		| | Coroutine | Thread |
		|---|---|---|
		| 切換時機 | 由使用者決定 yield 的位置 | 由 OS 決定 context switch 的時刻 |
		| 資料保護 | 毋需 | 需要 |
		| 獨立的 context | 有 | 有 |
		| 其他 | 依附於 thread 下 | |
*注1: 之所以在同時這兩個字上面加上引號，是因為所謂同時，其實是我們不停的在任務間進行切換處理，而造成同時的假象*
## 作法
	由上面的說明，我們可以知道 Coroutine 的運作原理即是在多個 Subroutine 間作切換，讓每個人輪流分到資源進行運算。當今天工作做到一半卻要把資源交給別人的時候，我們就得把目前的工作紀錄下來，以便下一次再輪到這個工作時，我們能更快速的上手。
	在 [System V-like](https://zh.wikipedia.org/zh-tw/UNIX_System_V) <sup>[3]</sup> 的環境中，我們可以透過 *ucontext_t* 的型別來對使用者層級的 context 進行操作。這個型別的定義在 *<ucontext.h>* 的標頭檔中，並且提供了以下四種方法來對 *ucontext_t* 進行操作，分別是 *makecontext*, *setcontext*, *getcontext*, 與 *swapcontext* 。
	我們要可以透過這四個方法，來創建、指定、取得、切換於不同的 subroutine 之間。首先，我們需要先創建一個生產者，這位生產者除了要能具備註冊新的消費者的能力外，亦要有管理已註冊消費者的功能。接著，在生產者之後，我們逐一為消費者創建各自的 context，每當消費者工作被讓出 (yield) 時，就會記錄在各自的 context 之中，並切換回生產者進行工作的分配。直到所有消費者的任務都完成，生產者便可功成身退，或者留下待命。
	統整一下我們要完成 Coroutine 會需要實作的基本功能與物件：

	| 名稱 | 類別 | 描述 |
	|---|---|---|
	| producer | 物件 | 可註冊消費者，也可排程消費者 |
	| consumer | 物件 | 有自己的任務，運行時可能會被中斷 (yield) |
	| yield() | 方法 | 將當前的消費者中斷，並把資源歸還給生產者 |
	| create_producer() | 方法 | 創建 producer |
	| create_consumer() | 方法 | 創建 consumer |
	| register_consumer() | 方法 | 在 producer 中註冊一個新的 consumer |
	| schedule() | 方法 | producer 的排程 |
	完整的流程如下：
		1. create_producer()
			2. create_consumer()
			3. register_consumer()
			4. 開始運行 producer
			5. producer 釋出資源給 consumer
			6. consumer 運行
			7. consumer 呼叫 yield()
			8. consumer 把資源還給 producer
			9. 如果尚有 consumer 還沒完成，回到步驟 5。否則繼續
			10. producer 結束運行或待命

## 小結
	總結本篇文章提到的內容與技術，並且許可的話給出快速指南，幫助讀者快速複習文章內容。
在這篇文章中，我們講解了 Coroutine 的運行方式，並且對 Coroutine 與 Thread 進行了小小的比較。看完了兩者的差異後，我們又更詳細的看了若要實作 Coroutine 時所需的方法與流程。接下來，在下一篇文章 [[為什麼是 Coroutine？（二）- 使用 C語言實作 Coroutine]] 中，我們會以 C語言來實作本節提到的 Coroutine。希望能透過實作的方式更了解這個多工的機制。
## 參考資料
	[1]: **UniVRM - Github page** - https://github.com/vrm-c/UniVRM
	[2]: **Cooperative Multitasking** - https://en.wikipedia.org/wiki/Cooperative_multitasking
	[3]: **System V-like - Wikipedia** - https://zh.wikipedia.org/zh-tw/UNIX_System_V
	[4]: **C/C++ 如何實作 coroutine? (fiber)** - http://zevoid.blogspot.com/2017/11/cc-coroutine-fiber.html