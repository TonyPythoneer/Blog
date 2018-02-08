---
title: 公司組織與跨人才養成
date: 2018-02-07 18:07:13
tags:
- "Topic:Workplace"
---

https://cdn.dribbble.com/users/751348/screenshots/2329465/open-position-frontend-developer-dribbble-2-preview_1x.png

```
這是一個選擇，沒有所謂的對錯，

多數公司當然覺得自己不如 airbnb，所以固守目前的組織規劃
相對也必須知道，這麼做，不會有接近 airbnb 的可能性

```

<!-- more -->


原來這個話題已經累積高深度的回答：Do you need coding skills to be a UI Designer?
https://www.quora.com/Do-you-need-coding-skills-to-be-a-UI-Designer

如果 UI 上，太依賴 designer 給的套版
在 FE 的損失是，這個東西是只關注視覺層次，卻不見得有顧慮到 UX 和互動的層次(因為這沒有顯而易見的指標，可以檢測 designer 是否在 UX 方面做的好)

在工程面：
這個套版可能無法被自動化測試(E2E)給檢測，例如是否能應用於各種版本的裝置上
無法建立 SPA，因為設計師有習慣的作業工具和內建的產生器，會直接是一個 html，對於 FE 轉換成本極大，因為 FE 得自己考慮 JS 互動是否能活化 UI 上的角色和 CSS 相容度是否有問題

designer 經常是交付一頁式的東西，無法像 FE 有 web component 的東西，可以單一交付和呈現
在產生器可能會因此產生大量無用的代碼再裏面

CSS 是否能符合工程的高複用性可能也是一個問題，但FE無從去檢測，假如那個 CSS 不知為何 MINIFY 居然還有 300 KB以上？

備註：按照市面普遍都是 100 KB 起跳

因為 UI 負責的工作是屬性更高級的抽象層次，無法用工程的角度去評估他們的價值...

如果想讓 UI 也知道一點 FE
最大的問題是，組織架構和個人規劃...
1. 組織架構
在薪資結構要改變，讓 UI 覺得他是因為這個優勢而趨使，而踏入 CODING 的領域
當非工程人員有跨技能的行為，是否願意給予鼓勵空間，來達到上述行為，尤其是傳產
2. 個人
總而言之，這是一個選擇，不是對錯，看當事人願不願意為了組織工作需求而改變
或許他真的是覺得 coding 是個困難，最終選擇成為一個 photoshoper 說不定，但這個太赤裸了，不太好讓人回答

現實狀況：
我只知道 airbnb 沒有所謂的 UI designer
只有 UX researcher 和 web designer
或許他們意識到，不會寫 coding 只會產生套版的人在跨團隊/職能的交付上是會耗費極大的轉換成本
所以會要求 web designer 要 code，好讓 FE 直接取用，接著在 refine 與 BE 互動的細節

我覺得那些回答者若只考量 designer 本身，那些論調都對啦
但拋開職能應有的角色，組織和個人是否能願意多做一點鼓勵，我覺得才是問題所在 -- 鼓勵跨領域的人才出現

即使是 Eng，在國外，好像也有類似的問題 QQ
已經愈來愈少所謂的 junior dev/eng
而是只有 software eng 和 Sr xxx dev 了

看職能描述，software eng 感覺是最基礎打雜工
需要知道各種領域一點點的事情
最後知道自己要做啥，才會從通才轉成專才

對老外來說，在職能描述已經模糊了
無論是 Sr dev/eng，都是要求是特定領域的高熟練者，有能力交付公司業務的需求

看了一下回答者的身份
最高身份的 co-founder 或 CEO，都有正反兩種說法 XD

同時這兩位是一, 二名的閱讀次數
第1名是  co-founder
兩位都是作 UX 起家

接續老外對於 dev 的職階規劃
Sr xxx dev 到最後會變成 architect/tech director/CTO
又從專長變成通才了 XD

哎，人生變化真大

技術面的通才，但是水準是最高點 -> architect
(該死的)人與技術之間的通才 -> tech director/CTO
