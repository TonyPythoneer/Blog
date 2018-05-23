

因為敝公司非常嚴重依賴第 3 方的套件做檔案驗證，當傳入的檔案非常大，大到超過記憶體上限，卻仍把整份檔案讀取至記憶體時，在 Python 會顯示 MemoryError，意即 OS 的 free memory 都被吃完，OS 不得不啟動 out-of-memory(OOM) 機制把這個 process 給殺掉。

可能現在的硬體成本便宜，無論是大的記憶體或是硬碟都能以低門檻的入手成本輕易取得，若當資源有限時，可能要思考怎麼發揮現有資源並完成目的，而 CS 基本功是很重要的一環。

<!-- more -->

我們的情境如下，希望用戶上傳的是 `mp3` 檔案，當用戶上傳完時，需要做檔案驗證。

雖然上傳時，有做檔案限制是 150 MB，但是我們的程式卻做整筆讀進記憶體的處理

FF FB
49 44 33
ref: https://en.wikipedia.org/wiki/List_of_file_signatures

```py
with open('xxx.mp3', 'r') as f:
    file_signature = f.read(3)
    hex_str = file_signature.encode('hex')
    if 'fffb' == hex_str[:4] or '494433' == hex_str:
        print 'This is a valid mp3 file'
```
