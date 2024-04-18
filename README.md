# m3u8 影片線上下載工具([English version](https://github.com/SAOJSM/m3u8-downloader-cht/blob/master/README-EN.md))

![界面](https://i.imgur.com/3leDymH.jpg)
### [線上工具網址](https://m3u8-downloader-cht.glitch.me)，推薦使用 chrome 瀏覽器。

### 研發背景
- m3u8影片格式簡介
    - m3u8影片格式原理：將完整的影片拆分成多個 .ts 影片片段，.m3u8 文件詳細記錄每個影片片段的地址。
    - 影片播放時，會先讀取 .m3u8 文件，再逐個下載播放 .ts 影片片段。
    - 常用於直播業務，也常用該方法規避影片竊取的風險。加大影片竊取難度。
- 鑒於 m3u8 以上特點，無法簡單通過影片連結下載，需使用特定下載軟件。
    - 但軟件下載過程繁瑣，試錯成本高。
    - 使用軟件的下載情況不穩定，常出現瀏覽器正常播放，但軟件下載速度慢，甚至無法正常下載的情況。
    - 軟件被編譯打包，無法了解內部運行機制，不清楚裡面到底發生了什麼。
- 基於以上原因，開發了本工具。

### 工具特點
- 無需安裝，打開網頁即可用。
- 強制下載現有片段，無需等待完整影片下載完成。
- 操作直觀，精確到影片片段的操作。


### 功能說明
【M3U8解析/下載】輸入 M3U8 連結，點擊下載影片。
【跨域複製代碼】當資源出現跨域限制時，點擊複製頁面代碼，在影片頁面的控制台輸入。將工具注入到影片頁面中，解決跨域問題。
【重新下載錯誤片段】當部分影片片段下載失敗時，點擊該按鈕，重新下載錯誤片段。
【強制下載現有片段】將已經下載好的影片片段強制整合下載。可以提前觀看已經下載的片段。該操作不影響當前下載進程。
【片段Icon】對應每一個 .ts 影片片段的下載情況。「灰色」：待下載，「綠色」：下載成功，「紅色」：下載失敗。點擊紅色 Icon 可重新下載對應錯誤片段。

### 使用說明
- 打開影片目標網頁，鼠標右鍵「檢查」，或者「開發者工具」，或者按下鍵盤的「F12」鍵
- 找到 network，輸入 m3u8，過濾 m3u8 文件。
- 刷新頁面，監聽 m3u8 文件。

- 找到目標M3U8文件，查看文件內容，是否符合格式。
    - 如下為索引文件，不是真正的影片 m3u8 文件

    - 一般內容有許多 ts 結尾的文件才是我們需要的影片 m3u8 文件。

- 複製這個 m3u8 文件的連結。

- 打開工具頁面，輸入連結，點擊「解析下載」。
- 出現片段 Icon，則證明操作成功，耐心等待影片下載。
- 片段全部下載成功，將觸發瀏覽器自動下載，下載整合後的完整影片。
- 如果有片段下載失敗，則點擊對應片段，或點擊「重新下載錯誤片段」按鈕。重新下載錯誤片段。

### 異常情況
【無法下載，沒有顯示片段Icon】
  - 一般由於跨域造成。
  - 點擊「跨域複製代碼」按鈕。
  - 打開影片目標網頁的「開發者工具界面」，找到 console 欄。
  - 貼上剛剛複製的內容，按下Enter。
  - 滾動頁面到底部，發現工具顯示在底部。然後在注入的工具中正常使用。

【下載後的影片資源不可看】
  - 網站對影片源進行了加密操作。不同的影片網站有不同的算法操作。無法通用處理。
  - 一般網站不會有這種情況。愛奇藝，騰訊等大影片網站才會有該安全措施。

### 實現思路
【下載並解析 m3u8 文件】
- 直接通過 ajax 的 get 請求 m3u8 文件。得到 m3u8 文件的內容字符串。讀取字符串進行解析。
- 需要注意的是，m3u8 文件不是 json 格式，不能將 dataType 設置為 json。
【隊列下載 ts 影片片段】
- 同樣使用 ajax 的 get 請求影片片段，一個 ajax 請求一個 ts 影片片段，但關鍵點在於，下載的是影片文件，屬於二進制數據，需要將 responseType 請求頭設置為 arraybuffer。```xhr.responseType = 'arraybuffer'```
- 使用隊列下載，是因為影片片段太多，不可能一次性請求全部。需要分批下載。
- 同時由於瀏覽器同源並發限制，影片同時請求數不能過多。本工具設置為並發下載數為 10。
【組合 ts 影片片段】
- 看似很難，但其實使用 [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 對象即可將多個 ts 文件整合成一個文件。new Blob()，傳入 ts 文件數組。
- 這里有個小細節需要注意，需要在 new Blob 的第二個參數中設置文件的 MIME 類型，否則將默認為 txt 文件。 ```const fileBlob = new Blob(fileDataList, { type: 'video/MP2T' }) ```
【自動下載】
- 下載，當然先要獲得文件連結，即剛生成的 Blob 文件連結。
- 使用 [URL.createObjectURL](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL)，即可得到瀏覽器內存中，Blob 的文件連結。```URL.createObjectURL(fileBlob)```
- 最後，使用 a 標簽的 [a.download](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a) 屬性，將 a 標簽設置為下載功能。主動調用 click 事件```a.click()```。完成文件自動下載。

### 核心代碼
【整合及自動下載】

```
    // 下載整合後的TS文件
    downloadFile(fileDataList, fileName, fileType) {
      this.tips = 'ts 片段整合中，請留意瀏覽器下載'
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

除了 vue.js 文件，本工具代碼均包含在 index.html 文件裡面。包括換行，一共 540 行代碼，其中 css 樣式 190 行，html 標簽 30 行。JS 邏輯代碼 300 行。

羅列這些代碼量只是想表明，本工具運用到的都只是 JS 的常見知識，並不覆雜。鼓勵大家多嘗試閱讀原始碼，其實看原始碼並沒有想象中的那麼困難。

### [原始碼連結](https://github.com/SAOJSM/m3u8-downloader-cht/blob/master/index.html)

### AES 常規解密功能
- 借助「aes-decryptor.js」，該文件來至 [hls.js](https://github.com/video-dev/hls.js)

### MP4 轉碼功能
- 借助「mux-mp4.js」，原始碼來至 [mux.js](https://github.com/videojs/mux.js#mp4)
- 但 mux.js 存在一個無法計算影片長度的 bug
- 本人已 fork 該項目，並修復該 bug，修復後的項目[連結在這里](https://github.com/Momo707577045/mux.js)

### 第三方接入
- 在 url 中通過 source 參數拼接下載地址即可，如：```https://m3u8-downloader.glitch.me?source=https://vod-normal-global-cdn-z01.afreecatv.com/v3/mapped-vod/save01/afreeca/station/2021/0811/16/1628668696129310.php/live/0/index-f3-v1-a1.m3u8```
- 系統將自動解析該參數

### 油猴插件

- 「跳轉下載」即新開頁面，打開本工具頁面，自動攜帶並解析目標地址
- 「注入下載」為解決跨域而生，直接將代碼注入到當前影片網站，進行影片下載
- [插件原始碼點這里]( https://github.com/SAOJSM/m3u8-downloader-cht/blob/master/tamper-monkey.js)
- 手動添加油猴插件步驟
  - 點擊 tamper-monkey「油猴」icon，點擊「添加新腳本」

  - 在當前位置，貼上上述連結中的原始碼

  - 點擊「文本」，「保存」

  - 得到如下結果，即為添加成功

### 完結撒花，感謝閱讀。
