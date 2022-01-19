---
title: "如何使用 std::function 撰寫 Callback Function"
categories:
- "技術"
tags:
- "c-plus-plus"
- "std"
- "callback function"
keywords:
- "GanniPiece"
- "技術文章"
- "c++"
- "std::function"
date: 2022-01-18T00:53:58+08:00
draft: false
---

<!--more-->

## 前言

[std::function][1] 是 c++11 後的新功能，定義於 header \<functional> 中。有點像是 c 語言中的 function pointer，除此之外，也有更廣泛的應用，任何 [*CopyConstructable*][2] 的 [*Callable*][3] 物件皆可以 std::function 儲存、複製以及調用。

舉例來說，function, lambda expression, bind expression 是這類型的物件，另外，像是成員函數 (member function) 、資料成員 (member data) 的指標，也是 `std::function` 的範疇。

其中儲存的可呼叫的物件即是 `std::function` 的目標 (target)。假如今天我們沒有給定任何目標，則此種情形稱作 **empty**，並且會丟出 `std::bad_function_call` 的 exception。

而 callback function 中文通常翻譯成回調函式或回呼函式，是指一個能藉由參數傳遞通往另外一個函式的函式。



## 方法

首先，我們先來看 `std::function` 的定義

```c++
template<class R, class... args>
class function<R(args)>
```

由此可見，如果要使用 `function` 這個類別，在初始化時我們要先定義傳入的類別型態 `args` 與傳出的類別型態 `R`。這兩個參數對應了 *Callable* 物件中的參數與回傳型態，如果沒有回傳值或傳入值，則以 `void` 表示。以下是一個簡單的例子，

```c++
std::function<void(int, float)> callback;
```

我們定義了一個實例了一個名叫 callback 的 `std::function`，在這個 `std::function` 中，傳入的參數分別是型態為 `int` 與 `float` 的兩個參數，因為沒有回傳值，所以將 `R` 設為 `void`。

實例化此 `std::function` 後，我們需要將實際被呼叫的函式存入其中。`std::function` 中覆寫了等號的運算子 `operator=` 作為賦予新目標的方法。也就是如果我們今天我們今天有一個函式叫做 `func`，其傳入的兩個參數分別是 `int`、`float`，我們可以直接以等號賦值。

```c++
void func(int a, float b);
callback = func;
```



## 範例程式

```c++
#include <iostream>
#include <functional>

std::function<void(int, float)> callback;

void add(int a, float b)
{
  	std::cout << a + b << std::endl;
}

void sub(int a, float b)
{
	  std::cout << a - b << std::endl;
}

void func(int a, float b, std::function<void(int, float)> c)
{
	  c(a, b);
}

int main()
{
	  int a = 5;
  	float b = 0.3

    // 5 + 0.3
    callback = add;
 		func(a, b, callback);
  
    // 5 - 0.3
    callback = sub;
    func(a, b, callback);
}
```





## 

[1]: https://en.cppreference.com/w/cpp/utility/functional/function
[2]: https://en.cppreference.com/w/cpp/named_req/CopyConstructible
[3]: https://en.cppreference.com/w/cpp/named_req/Callable