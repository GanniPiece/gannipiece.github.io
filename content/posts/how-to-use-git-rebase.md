---
title: "如何使用 git-rebase-與其他分支合併"
date: 2022-01-20T11:43:53+08:00
categories:
- "技術"
tags:
- "技術"
- "Git"
- "Rebase"
keywords:
- "GanniPiece"
- "Git"
- "Rebase"
draft: false
---

<!--more-->

## 前言
最近 Plug-In 開發得差不多了，接下來要與夥伴改進與測試目前的合成器，首先遇到的問題就是如何將我們兩個的修改版本進行整合。[Git](https://git-scm.com "git") 這個版本控制的工具提供了兩種容易上手的方法，分別是 merge 與 rebase 。

在這篇文章中，我會先說明 rebase 的使用方法，並以我們在開發中的使用情境提供一個簡單的範例。

## 方法
Rebase 的意思即是以某個 branch 作為基準來做整合，假設今天有兩條 branches **A** 和 **B**，並且在這兩條 branches 中存在版本差異，那我們就必須對差異的部分進行整合。

1. 如果我們希望以 A 分支作為基準，我們首先需要將 branch 切到 A 分支上

- 	```shell {linenos=false, linenostart=1}
	❯ git checkout A
	```

2. 確認 HEAD 所在的分支

- 	```shell
	❯ git status
	On branch A
	nothing to commit, working tree clean
	``` 

3. B rebase A

-	```shell
	❯ # (on branch B)
	❯ git rebase A
	```	
4. 若兩個版本間有衝突，依照 git 輸出的訊息逐一對衝突處進行修改
5. 修改完後將檔案加入 stage

-	```shell
	❯ git add <modified file name> 
	```
6. 重複步驟 4, 5，並繼續 rebase 直至所有衝突解決

-	```shell
	❯ git rebase --continue
	```


## 範例
今天要新增兩個獨立的功能，於是我和夥伴一人各開一條 branch 進行開發，分別在 branch A 和 branch B。此
時夥伴已將功能 A 開發完成，而我想要先在B branch 上顯示夥伴完成的介面功能，就可以用 rebase 的方式將他
的功能加入我當前的分支。

以簡單的例子來看，原先在 develop 上面有 d.cpp 這個檔案。在開發的過程中，我們分別對 A, B 加入 a.cpp,
b.cpp 兩個檔案。

1.  最初的 d.cpp (branch 'develop')

```c++
#include <iostream>
int main()
{
	std::cout << "D" << std::endl;
}
```

2. 在 branch A 修改 d.cpp
	
```c++
#include <iostream>
int main()
{
	std::cout << "A" << std::endl;
}
```

3. 在 branch B 修改 d.cpp

```c++
#include <iostream>
int main()
{
	std::cout << "B" << std::endl;
}
```
	
此時，如果我們要將以 A 為基準來 rebase B，就會遇到檔案衝突的問題。

```shell
Auto-merging d.cpp
CONFLICT (content): Merge conflict in d.cpp
error: could not apply 1b2cb82... Update B
Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
Could not apply 1b2cb82... Update B
```

由訊息得知我們在 `d.cpp` 這個檔案中有衝突，必須先將其解決，再使用 `git rebase --continue` 指令完成這
次的 rebase。

4. 開啟 `d.cpp`，解決其中衝突

```c++
#include <iostream>

int main()
{
<<<<<<< HEAD
    std::cout << "A" << std::endl;
=======
    std::cout << "B" << std::endl;
>>>>>>> 1b2cb82 (Update B)

}
```

5. 選擇要留下的版本後，加入 stage
6. `git rebase --continue` 後，重複步驟5 直至所有衝突解決
7. 完成

[1]: https://git-scm.com
