---
title: git rebase 的心流
date: 2018-05-17 15:34:18
tags:
- "Else:Git"
---

![message-queue-workflow](/images/2018/5/git-rebase.svg)

如果要確保你的 git branch 是基於最新的 develop/master 上伸展的，`git rebase` 是你的好朋友！
如果你的 git branch 要即將合併到主幹，但太多瑣碎的訊息需要加以整理，`git rebase` 是你的好朋友！

<!-- more -->

## 建議方法

### 向主幹取得基底更新

推荐指令：

```sh
git rebase develop
```

通常進行開發時，會對 `develop` branch 做 checkout and create branch，命名為 `feature`；開發人員會把追加新功能的 commmit 疊加在新的 branch 上。

若這段期間有其他同事的 branch 已經合併至 `develop`、主幹有了變化，此時你的 `feature` branch 跟 `develop` 有些差異且被 Git 標記為「已過時」，要確保你的 code 是建築於最新的主幹下進行開發，可以使用 `git rebase` 來更新你舊的 `develop` 的節點基礎。

尤其是你的 branch 有 conflict 時，若是使用 merge，整理完後會讓你的 branch 增加 `develop merged into XXX commit` 的節點紀錄，但通常不是 `develop` 沒必要有這個紀錄存在，因為你的 branch 是分支，不是主幹。不需要因 merge 而讓 commit 無謂的增加，不如先自己整理過比較好。

### 合併至主幹前，進行 commit 整理

```sh
git rebase -i HEAD~X
```

此處 X 是 HEAD 為起點的前 X 個 commits。

若在進行 code review 之前或是之後，在開發時總是會有瑣碎的 change 和 message 被 commit 至 `feature` branch，使得 branch 變得非常冗長，無論對於 reviewer 或是回頭看主幹 commits 的人都是很大的閱讀負擔。

你可以對自己的 commit 稍做整理，這邊附加的參數 `-i` 是進入互動模式，文字介面如下：

```txt
pick 9b175d0 Update guide
pick b49bb75 Move "20180228-communication-time-is-money" to draft
pick 3e0bca6 Add tag of "20180221-remove-smartgit-license"
pick dfa3c43 Refine post of "20180302-celery-signal"
pick b167d00 Refine post of "20180511-emoji-converter"
pick f501041 Refine post of "20180511-django-and-emoji"
pick e793fda Fix tag of "20180511-ORM-vs-MySQL-builtin-function"
pick e9fa9ee Update guide
pick 70838db Refine post of "20180511-django-and-emoji"
pick f15365c Add new post of "20180517-zone-of-git-rebase"

# Rebase 47f0d79..f15365c onto 47f0d79
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

上面是你指定的 `rebase` 範圍，下面是操作指令。每個段落結構為 command, commit id, commit message。

* 常用操作說明如下：
  * pick: 預設操作。保持該 commit 的狀態
  * move: 非指令操作，是用剪下、貼上進行調整。你可以搬動自己的 commit 節點，複製特定段落內容到合適的位置。
  * squash: 合併至前個 commit。若某個 commit id 的 change 跟前個 commit 大同小異，可以利用此操作來 `減少訊息數量過於分散`和 `提升訊息密度`。
  * edit: 重新編輯 message。若當時的 commit 只是靈光一現，知道做什麼，卻沒有用合適的說明來表述，這時有可以 `允許再次編輯`，讓該 commmit 有個合理的表達。

另外，請切忌自己 rebase 也會有 conflict 的風險，請確認 change 的檔案對象在 commit 順序是否變成有「新的」比「舊的」還要前面，此時會發生 conflict 的問題。請仔細審閱和了解自己的 commit 在做什麼事情，否則請在開發前，把該做的 task 切小，讓事情變得可描述化。

若是不曉得自己新的 commit 數量有幾個，可以用以下指令查詢，來對比推荐指令：

```sh
git rev-list develop..new_post --count
```

意思是，以 `develop` 為基底，而 `new_post` 是新的 branch，這兩者的 diff commit 計數其數量。

## 結論

善用 git 指令，待人待己都能得到十足友善。
