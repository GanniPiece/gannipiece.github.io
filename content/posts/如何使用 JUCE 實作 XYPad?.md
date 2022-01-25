---
title: "如何使用 JUCE 實作 XYPad?"
date: 2022-01-25T14:13:30+08:00
categories:
- "技術"
tags:
- "技術"
- "JUCE"
- "XYPad"
keywords:
- "JUCE 中文教程"
- "Audio Plugin"
- "XY Pad"
draft: false
---
<!--more-->

## 前言
XY Pad 是 Audio Plugin 中很常見的 UI 設計方式，可以同時對兩個不同面向的因子進行操作。舉例來說，我們可以將 x 軸設定為對 panner 的方位調整，將 y 軸設定為對 volume 的音量大小調整，如此一來，在 XY Pad 上進行移動就可以如同聲源在聽者前後左右移動一般，是一個相當方便的可視化工具。

一個基本的 XY Pad 包含兩個部分 —— Pad 與 Thumb 。Pad 如同畫布一般，讓 Thumb 得以在上面自由移動。底下是來自 [Cabbage Audio Forum](https://forum.cabbageaudio.com/t/chnset-and-xypad/321) 的範例，白色的部分即是我們的 Pad ，而綠色的球則是 Thumb。Thumb 所在的座標位置對應到 X, Y 軸因子的值（可以是聲音、方位、效果器參數⋯⋯等），透過對 Thumb 的移動來設定這些因子。

![](https://forum.cabbageaudio.com/uploads/default/original/1X/59977b8777171297cbcd29c8d37f40d92498573a.gif)

本文假設讀者已對 [JUCE](https://juce.com) 這個 Framework 有一初步的了解，對於專案的建立不再細細贅述。若對該架構不太熟悉，可以自行參考 [JUCE: Tutorial: Projucer Part 1: Getting started with the Projucer](https://docs.juce.com/master/tutorial_new_projucer_project.html) 的教學建立初步的專案。

## 方法
### 專案建立
> 此為簡略的專案建立介紹，更詳細的介紹請參考 [[如何建立一個 JUCE 的專案?]]

首先開啟 Projucer 並建立一個 Plug-In (`Projucer` > `Plug-In` > `Basic`)，我們可以直接使用預設的設定，或是看個人需要引進不同的 module ，比如說常用的 `juce_dsp` 等等。

按下 `Create Project...` 後，確認 `File Explorer` > `Source` 底下是否有四個檔案，分別是 `PluginProcessor` 與 `PluginEditor` 的 header (.h) 檔與 source code file (.cpp)。若不是這四個檔案，代表最初選取的非 Plug-In 形式的專案，可以回到前一步建立 Projucer 專案重新確認與建立。

![Imgur](https://i.imgur.com/DqqcRRe.png)

接著，依照系統與個人的需求，在左側的 `Exporters` 底下新增需要的專案檔，以截圖為例，筆者選擇了 Xcode 與 Linux Makefile 兩種專案形式，作為之後編譯的方法。

如此一來，就完成了專案的建立！使用熟悉的 IDE 開啟專案，就可以開始來實作 XY Pad 啦！

### ProcessorEditor
在這篇文章中，我們僅會在介面實作 Audio Plugin 常見的 XY Pad ，並不會碰到與 audio 相關的操作，因此只需要聚焦在 `ProcessorEditor` 這個與 GUI 有關的檔案上面。開啟 `ProcessorEditor.h`，我們可以看到這是一個繼承 `juce::AudioProcessorEditor` 的類別，該類別繼承 `juce::Component` 類別，有兩個virtual member functions 可以覆寫，分別是 `paint()` 與 `resized()`。

![Imgur](https://i.imgur.com/9q9EHFV.png)

#### juce::Component::paint()
> virtual void Component::paint([Graphics](https://docs.juce.com/master/classGraphics.html) & _g_)

在 JUCE 當中，所有繼承 Component 類別的物件都可以覆寫 `paint()` 這個成員函式，來自行定義如何繪製該物件。

`paint()` 會在一個物件需要被重新繪製、呼叫 `repaint()`或是圖形介面視窗需要重新繪製時被呼叫。正常的情況下，子類別的 `paint()` 的優先序會大於父類別，也就是說任何我們在父類別對子類別繪製的操作都會被子類別的繪製覆蓋掉，如果我們不希望這件事情發生的話，可以透過實作 `paintOverChildren()` 這個成員函式來避免。

在官方文件中提到，假如我們想要重新繪製某個 component，我們可以呼叫 `repaint()` 這個方法，一旦呼叫 `repaint()` 後，這個元件就會被標記為 "dirty"， `paint()` 就會在之後的某個時間點自動被 message thread 呼叫，完成後剛剛的 dirty bit 就會被更新。

由此可知，在 JUCE 底下 UI 的更新是非同步的，事實上文檔中也有提到，在現今大多數的 UI 框架中，我們不太會同步的去更新介面的改動。

#### juce::Component::resized()
> virtual void Component::resized()

當 component 的大小改變的時候，就會一齊呼叫這個函式。我們可以透過實作這個函式來決定子元件在父元件的相對位置，不同於 `repaint()` 的是，當我們呼叫 `setBounds()` 或是 `setSize()` 的時候，這個方法是被同步呼叫的，因此當我們對元件做出大小改變的時候，`resize()` 就會反覆的被呼叫，然而重新 `paint()` 介面可能是好幾次的 `repaint()` 進行更改後集結而成的。

瞭解這兩個方法後，我們接下來就可以來撰寫 XY Pad 上面的兩個元件 Pad 以及 Thumb，並透過這兩個方法將元件繪製到 UI 上面！

### 零件製造
#### Pad
建立一個繼承 `juce::Component` 的類別，然後我們要對 `paint()` 與 `resized()` 兩個成員函式進行副寫。

```c++
class Pad: public juce::Component
{
public:
	Pad();
	~Pad();
	
	void paint(juce::Graphics& g);
	void resized();
};
```

呼叫 `setColour` 將 Pad 的顏色設定成白色，然後呼叫 `fillRoundedRectangle` 將 Pad 設定成圓角的矩形。

```c++
void Pad::paint(juce::Graphics& g)
{
	auto cornerSize = 10.0f;
	g.setColour (juce::Colours::white);
	g.fillRoundedRectangle (getLocalBounds().toFloat(), cornerSize);
}
```


#### Thumb
建立一個繼承 `juce::Component` 的類別，同樣對 `paint()`  進行覆寫，以此來設定一個圓形的外觀。除此之外，我們也要偵測按下 `mouseDown` 以及拖曳 `mouseDrag` 的事件，進一步移動 Thumb 的位置。一旦開始拖曳 Thumb，便執行對應的 callback function `moveCallback` 。

> 關於 callback function，可以參考先前的文章 [[[如何使用std::function撰寫callback function]]](https://gannipiece.github.io/posts/如何使用stdfunction撰寫callback-function)


```c++
class Thumb: public juce::Component
{
public:
	Thumb();
	
	void paint (juce::Graphics& g) override;
	void mouseDown (const juce::MouseEvent& event) override;
	void mouseDrag (const juce::MouseEvent& event) override;
	int getSize(); // return thumbSize
	// callback function	
	std::function<void(juce::Point<float>)> moveCallback;
private:
	static constexpr int thumbSize = 20;
	juce::ComponentDragger dragger;
	juce::ComponentBoundsConstrainer constrainer;
};
```

在建構子設定 constrainer：

```c++
Thumb::Thumb()
{
	constrainer.setMinimumOnscreenAmounts(thumbSize, thumbSize, thumbSize, thumbSize);
}
```

同樣的覆寫 `paint()`：

```c++
void Thumb::paint (juce::Graphics& g)
{
	g.setColour (juce::Colours::green);
	g.fillEllipse (getLocalBounds().toFloat());
}
```

覆寫 `mouseDrag()` 與 `mouseDown()`

```c++
void Thumb::mouseDown (const juce::MouseEvent& event)
{
	dragger.startDraggingComponent(this, event);
}
```

```c++
void Thumb::mouseDrag (const juce::MouseEvent& event)
{
	dragger.dragComponent(this, event, &constrainer);
	
	if (moveCallback)
		moveCallback (getPosition().toFloat());
}
```

如此一來，我們就完成主要的兩個元件了！接下來，只要將他們之間的互動定義好，並且加入 `ProcessorEditor` 上面就可以了。

###  零件組裝
> Pad 與 Thumb 的關係是：Pad 中包含 Thumb。

首先，我們在 Pad 物件中加入一個 Thumb，並且在 Pad 的建構子中設定這個 Thumb 為可見的 (Visible)。

```c++
class Pad: public juce::Component
{
	...
	
private:
	
	Thumb thumb;
	
	...
};
```

```c++
void Pad::Pad()
{
	addAndMakeVisible(thumb);
}
```


接著，在 Pad 上繪製 Thumb，並且讓 Thumb 保持大小。
```c++
void Pad::resized()
{
	juce::Rectangle<int> centre = getLocalBounds().withSizeKeepingCentre(thumb.getSize(), thumb.getSize());

	thumb.setBounds(centre);

	thumb.setCentrePosition(getLocalBounds().proportionOfWidth(0.5), getLocalBounds().proportionOfHeight(0.3));
}
```

**(optional)** 接著，定義 Thumb 移動時的 callback。舉例來說，我們可以藉由這個 callback 去改變相關的參數，比如說 panner 的大小或音量的遠近。

```c++
void Pad::Pad()
{
	...
	thumb.moveCallback = [&] (juce::Point<float> pos)
	{
		// do something
	}
}
```

最後，將組裝好的 Pad 接到 `PluginEditor` 上：

```c++
class XYPadTutorialEditor : public juce::AudioProcessorEditor
{
public:
	XYPadTutorialEditor();

	...
private:
	...
	Pad pad;
}
```

並將其設定為可視的，並為 Pad 設定邊界位置：

```c++
void XYPadTutorialEditor::XYPadTutorialEditor()
{
	addAndMakeVisible(pad);
	...
}

void XYPadTutorialEditor::resized()
{
	pad.setBounds(getLocalBounds().reduced(20));
}
```

## 大功告成
這樣就在 JUCE 上完成可以移動的 XYPad 了！

![](https://i.imgur.com/18ZThBP.gif)