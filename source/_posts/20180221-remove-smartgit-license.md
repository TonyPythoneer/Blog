---
title: 刪除 SmartGit license!
date: 2018-02-21 19:23:59
tags:
- "Else:Tool"
---

如果你是 Ubuntu 使用者，在 Git gui 是選用 **SmartGit**，希望能給你幫助。

解決目標是：SmartGit 的免費方案使用期間為 30 天，若要再次使用則需要從路徑中刪除 SmartGit 的設定檔。

於是我撰寫了一個 script 腳本，省略其瑣碎的步驟並達到快速刪除，參考如下：

```sh
TAGERT_FILE="settings.xml"
SMARTGIT_PATH="${HOME}/.smartgit"

VERSION_REGEX="^[0-9]+$"
LATEST_VERSION=`ls ${SMARTGIT_PATH} | grep -P ${VERSION_REGEX} | sort -n -r -k 1 | head -1`

TAGERT_FILEPATH="${SMARTGIT_PATH}/${LATEST_VERSION}/${TAGERT_FILE}"

rm ${TAGERT_FILEPATH}
```

Reference on StackOverflow: [How to change SmartGit's licensing option after 30 days of commercial use on ubuntu?](https://stackoverflow.com/questions/10972289/how-to-change-smartgits-licensing-option-after-30-days-of-commercial-use-on-ub)
