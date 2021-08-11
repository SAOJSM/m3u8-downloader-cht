# m3u8 視頻在線提取工具([English version](https://github.com/Momo707577045/m3u8-downloader/blob/master/README-EN.md))

![界面](http://upyun.luckly-mjw.cn/Assets/m3u8-download/01.jpeg)
### [工具在線地址](https://m3u8-downloader.glitch.me)，推薦使用 chrome 瀏覽器。

### 研發背景
- m3u8視頻格式簡介
    - m3u8視頻格式原理：將完整的視頻拆分成多個 .ts 視頻碎片，.m3u8 文件詳細記錄每個視頻片段的地址。
    - 視頻播放時，會先讀取 .m3u8 文件，再逐個下載播放 .ts 視頻片段。
    - 常用於直播業務，也常用該方法規避視頻竊取的風險。加大視頻竊取難度。
- 鑒於 m3u8 以上特點，無法簡單通過視頻鏈接下載，需使用特定下載軟件。
    - 但軟件下載過程繁瑣，試錯成本高。
    - 使用軟件的下載情況不穩定，常出現瀏覽器正常播放，但軟件下載速度慢，甚至無法正常下載的情況。
    - 軟件被編譯打包，無法了解內部運行機制，不清楚里面到底發生了什麽。
- 基於以上原因，開發了本工具。

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/09.jpeg)

### 工具特點
- 無需安裝，打開網頁即可用。
- 強制下載現有片段，無需等待完整視頻下載完成。
- 操作直觀，精確到視頻碎片的操作。


### 功能說明
![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/02.jpeg)
【解析下載】輸入 m3u8 鏈接，點擊下載視頻。
【跨域覆制代碼】當資源出現跨域限制時，點擊覆制頁面代碼，在視頻頁面的控制台輸入。將工具注入到視頻頁面中，解決跨域問題。
【重新下載錯誤片段】當部分視頻片段下載失敗時，點擊該按鈕，重新下載錯誤片段。
【強制下載現有片段】將已經下載好的視頻片段強制整合下載。可以提前觀看已經下載的片段。該操作不影響當前下載進程。
【片段Icon】對應每一個 .ts 視頻片段的下載情況。「灰色」：待下載，「綠色」：下載成功，「紅色」：下載失敗。點擊紅色 Icon 可重新下載對應錯誤片段。

### 使用說明
- 打開視頻目標網頁，鼠標右鍵「檢查」，或者「開發者工具」，或者按下鍵盤的「F12」鍵
- 找到 network，輸入 m3u8，過濾 m3u8 文件。
- 刷新頁面，監聽 m3u8 文件。

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/03.jpeg)
- 找到目標m3u8文件，查看文件內容，是否符合格式。
    - 如下為索引文件，不是真正的視頻 m3u8 文件

        ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/04.jpeg)
    - 一般內容有許多 ts 字眼的文件才是我們需要的視頻 m3u8 文件。

         ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/05.jpeg)
- 拷貝這個 m3u8 文件的鏈接。

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/06.jpeg)
- 打開工具頁面，輸入鏈接，點擊「解析下載」。
- 出現片段 Icon，則證明操作成功，耐心等待視頻下載。
- 片段全部下載成功，將觸發瀏覽器自動下載，下載整合後的完整視頻。
- 如果有片段下載失敗，則點擊對應片段，或點擊「重新下載錯誤片段」按鈕。重新下載錯誤片段。

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/08.jpeg)

### 異常情況
【無法下載，沒有顯示片段Icon】
  - 一般由於跨域造成。
  - 點擊「跨域覆制代碼」按鈕。
  - 打開視頻目標網頁的「開發者工具界面」，找到 console 欄。

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/10.jpeg)
  - 粘貼剛剛覆制的內容，回車。
  - 滾動頁面到底部，發現工具顯示在底部。然後在注入的工具中正常使用。

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/11.jpeg)

【下載後的視頻資源不可看】
  - 網站對視頻源進行了加密操作。不同的視頻網站有不同的算法操作。無法通用處理。
  - 一般網站不會有這種情況。愛奇藝，騰訊等大視頻網站才會有該安全措施。

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/12.jpeg)

### 實現思路
【下載並解析 m3u8 文件】
- 直接通過 ajax 的 get 請求 m3u8 文件。得到 m3u8 文件的內容字符串。讀取字符串進行解析。
- 需要注意的是，m3u8 文件不是 json 格式，不能將 dataType 設置為 json。
【隊列下載 ts 視頻片段】
- 同樣使用 ajax 的 get 請求視頻碎片，一個 ajax 請求一個 ts 視頻碎片，但關鍵點在於，下載的是視頻文件，屬於二進制數據，需要將 responseType 請求頭設置為 arraybuffer。```xhr.responseType = 'arraybuffer'```
- 使用隊列下載，是因為視頻碎片太多，不可能一次性請求全部。需要分批下載。
- 同時由於瀏覽器同源並發限制，視頻同時請求數不能過多。本工具設置為並發下載數為 10。
【組合 ts 視頻片段】
- 看似很難，但其實使用 [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 對象即可將多個 ts 文件整合成一個文件。new Blob()，傳入 ts 文件數組。
- 這里有個小細節需要注意，需要在 new Blob 的第二個參數中設置文件的 MIME 類型，否則將默認為 txt 文件。 ```const fileBlob = new Blob(fileDataList, { type: 'video/MP2T' }) ```
【自動下載】
- 下載，當然先要獲得文件鏈接，即剛生成的 Blob 文件鏈接。
- 使用 [URL.createObjectURL](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL)，即可得到瀏覽器內存中，Blob 的文件鏈接。```URL.createObjectURL(fileBlob)```
- 最後，使用 a 標簽的 [a.download](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a) 屬性，將 a 標簽設置為下載功能。主動調用 click 事件```a.click()```。完成文件自動下載。

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/13.jpeg)


### 核心代碼
【整合及自動下載】

```
    // 下載整合後的TS文件
    downloadFile(fileDataList, fileName, fileType) {
      this.tips = 'ts 碎片整合中，請留意瀏覽器下載'
      const fileBlob = new Blob(fileDataList, { type: 'video/MP2T' }) // 創建一個Blob對象，並設置文件的 MIME 類型
      const a = document.createElement('a')
      a.download = fileName + '.' + fileType
      a.href = URL.createObjectURL(fileBlob)
      a.style.display = 'none'
      document.body.appendChild(a)
      a.click()
      a.remove()
    },
```

是的，涉及新知識點的部分只有上面一小段，其他的都是 JS 的基礎應用。

除了 vue.js 文件，本工具代碼均包含在 index.html 文件里面。包括換行，一共 540 行代碼，其中 css 樣式 190 行，html 標簽 30 行。JS 邏輯代碼 300 行。

羅列這些代碼量只是想表明，本工具運用到的都只是 JS 的常見知識，並不覆雜。鼓勵大家多嘗試閱讀源碼，其實看源碼並沒有想象中的那麽困難。

### [源碼鏈接](https://github.com/Momo707577045/m3u8-downloader/blob/master/index.html)

### AES 常規解密功能
- 借助「aes-decryptor.js」，該文件來至 [hls.js](https://github.com/video-dev/hls.js)

### MP4 轉碼功能
- 借助「mux-mp4.js」，源碼來至 [mux.js](https://github.com/videojs/mux.js#mp4)
- 但 mux.js 存在一個無法計算視頻長度的 bug
- 本人已 fork 該項目，並修覆該 bug，修覆後的項目[鏈接在這里](https://github.com/Momo707577045/mux.js)

### 第三方接入
- 在 url 中通過 source 參數拼接下載地址即可，如：```https://m3u8-downloader.glitch.me?source=http://1257120875.vod2.myqcloud.com/0ef121cdvodtransgzp1257120875/3055695e5285890780828799271/v.f230.m3u8```
- 系統將自動解析該參數

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/16.jpeg)


### 油猴插件

![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/15.jpeg)

- 「跳轉下載」即新開頁面，打開本工具頁面，自動攜帶並解析目標地址
- 「注入下載」為解決跨域而生，直接將代碼注入到當前視頻網站，進行視頻下載
- [插件源碼點這里](https://github.com/Momo707577045/m3u8-downloader/blob/master/tamper-monkey.js)
- 手動添加油猴插件步驟
  - 點擊 tamper-monkey「油猴」icon，點擊「添加新腳本」

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/21.jpeg)

  - 在當前位置，粘貼上述鏈接中的源碼

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/17.jpeg)

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/18.jpeg)

  - 點擊「文本」，「保存」

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/19.jpeg)

  - 得到如下結果，即為添加成功

    ![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/20.jpeg)



### 完結撒花，感謝閱讀。
![](http://upyun.luckly-mjw.cn/Assets/m3u8-download/14.jpeg)
