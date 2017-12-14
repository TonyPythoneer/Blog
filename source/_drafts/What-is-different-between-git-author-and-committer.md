---
title: What is different between git author and committer?
tags:
---

解釋為什麼 commit 的節點會顯示兩個人的頭貼。

```sh
git rebase develop  # 整合目前 develop 的 commit 節點。獲得成員 PR merge 後的新 commits，作為新的 base
git push origin feature/sendy-integration -f  # 將整合後的 branch 強制推上去
```

commit 的節點會顯示兩個人的頭貼是，一個是 author 資訊，一個是 committer 資訊。因為是我去對 branch 做更新，所以 branch 會將我辨認為 committer 的角色並加入 commit 訊息中，這個動作不會影響 author 對 commit changes 的紀錄，甚至覆寫 author 的 changes 行為(避免搶奪 author 的貢獻)。

ref: [git 官方文件](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History)
> You may be wondering what the difference is between author and committer. The author is the person who originally wrote the patch, whereas the committer is the person who last applied the patch. So, if you send in a patch to a project and one of the core members applies the patch, both of you get credit — you as the author and the core member as the committer.

為什麼是用 force push 作 update remote branch？

原因是其他已 merge 進 develop 的 PR，如果從 github 上的介面操作，「Merge pull request」的按鈕預設是「create a merge commit」這個行為，所以當 PR merge 執行之後，develop 會產生一個新的 commit 節點。因為這個 commit 節點不是 remote feature/fix branch 擁有的，使用 force push 是為了將這個 merge commit 節點更新到 remote feature/fix branch ，確保 remote feature/fix branch 和 develop branch 的 commit 基礎是一致的，這樣 git 才會允許 remote feature/fix branch 合併至 remote develop branch 。
