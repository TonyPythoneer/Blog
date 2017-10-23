# 快速起手簡易教學

## 指令

ref: https://hexo.io/zh-cn/docs/writing.html

* 新頁面

```sh
$ hexo new page <title>
```

* 發文

```sh
$ hexo new post <title>
```

* 草稿

```sh
$ hexo new draft <title>
```

* 將草稿移動成發文

```
$ hexo publish post <title>
```

* 建立模版

```
$ hexo new photo <title>
```

在執行這行指令時，Hexo 會嘗試在 `scaffolds` 資料夾中找尋 `photo.md`，並根據其內容建立文章

## 俺式發文流程管理術

首先進入 blog 根目錄

在命名文章的標題前，先用當下日期作為前綴詞，是考量日後發文篇數過多，方便管理自己的日記，指令如下：

```sh
$ hexo new post "2017-10-20-<title>"
```

log 訊息會顯示，於 `<blog_root>/_posts/2017-10-20-<title>.md` 產生檔案。內容如下：

```txt
---
title: 2017-10-20-<title>
date: 2017-10-20 17:49:50
tags:
---
```

目前階段還是在 local 端產生文章，在佈署前還是有編輯的機會，針對 `title` 屬性把日期前綴詞刪除。
確認自己文章完成後，那時候再佈署即可，型式如下：

```txt
---
title: <title>
date: 2017-10-20 17:49:50
tags:
---

Your content...
```

## 額外

1. 添加「閱讀更多」

只要在 markdown 加上這層註解，會被 Hexo 顯示為「閱讀更多」

```txt
<!-- more -->
```

2. 「分類」和「標籤」會自動化處理

只要按照官方文件上的說明，在 markdown 上標示屬性為對應的 type，即使內文為空， Hexo 會根據 post 的 categories 或 tags 作自動分類。這就大可放心了。

這是 `categories/index.md` 的範例：

```
---
title: categories
date: 2017-10-20 16:47:47
type: "categories"
comments: false
---
```

這是 `tags/index.md` 的範例：

```
---
title: tags
date: 2017-10-20 16:47:07
type: "tags"
comments: false
---
```

# 佈署

Demo page:

http://tonypythoneer.github.io/blog

Offical page:

http://tonypythoneer.github.io
