---
title: 刪除 SmartGit license!
date: 2018-02-21 19:23:59
tags:
---

由於我是 Ubuntu 使用者，在 Git gui 的免費方案中目前是選用 **SmartGit**。

SmartGit 跟 SourceTree 的介面資訊和操作大同小異，但若使用 30 天的免費方案，在 30 天後得需要從路徑中刪除 SmartGit 的設定檔，以便再次啟用免費服務。

於是我撰寫了一個 script 腳本，可以協助我達到快速刪除。希望也能幫助有需之人，請享用。

```sh
TAGERT_FILE="settings.xml"
SMARTGIT_PATH="${HOME}/.smartgit"

VERSION_REGEX="^[0-9]+$"
LATEST_VERSION=`ls ${SMARTGIT_PATH} | grep -P ${VERSION_REGEX} | sort -n -r -k 1 | head -1`

TAGERT_FILEPATH="${SMARTGIT_PATH}/${LATEST_VERSION}/${TAGERT_FILE}"

rm ${TAGERT_FILEPATH}
```

Reference on StackOverflow: [How to change SmartGit's licensing option after 30 days of commercial use on ubuntu?](https://stackoverflow.com/questions/10972289/how-to-change-smartgits-licensing-option-after-30-days-of-commercial-use-on-ub)
