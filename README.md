# m3u8 影片線上下載工具([English version](https://github.com/SAOJSM/m3u8-downloader-cht/blob/master/README-EN.md))

### [線上工具網址](https://m3u8-download-cht.glitch.me)，推薦使用 Chrome 瀏覽器。
### ※此工具僅供展示使用，請勿濫用，Glitch超過流量就無法展示專案網頁，如需要個人使用請下載整個專案，開啟index.html即可使用

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
- 支援跨域請求，可以處理大多數網站的影片。
- 支援下載特定範圍的片段。

### 功能說明
【M3U8解析/下載】輸入 M3U8 連結，點擊下載影片。
【跨域代碼注入】當資源出現跨域限制時，點擊複製代碼，在影片頁面的控制台輸入。將代碼注入到影片頁面中，解決跨域問題。
【下載範圍選擇】可以選擇只下載特定範圍的影片片段。
【片段Icon】對應每一個影片片段的下載情況。「灰色」：待下載，「綠色」：下載成功，「紅色」：下載失敗。點擊紅色 Icon 可重新下載對應錯誤片段。

### 使用說明
- 打開影片目標網頁，鼠標右鍵「檢查」，或者「開發者工具」，或者按下鍵盤的「F12」鍵
- 找到 network，輸入 m3u8，過濾 m3u8 文件。
- 刷新頁面，監聽 m3u8 文件。
- 找到目標M3U8文件，查看文件內容，是否符合格式。
- 複製這個 m3u8 文件的連結。
- 打開工具頁面，輸入連結，點擊「下載原格式」或「下載 MP4」。
- 如果遇到跨域問題，點擊「複製跨域代碼」按鈕，並將代碼貼到視頻頁面的控制台中執行。
- 出現片段 Icon，則證明操作成功，耐心等待影片下載。
- 片段全部下載成功，將觸發瀏覽器自動下載，下載整合後的完整影片。
- 如果有片段下載失敗，則點擊對應片段重新下載錯誤片段。

### 異常情況
【無法下載，沒有顯示片段Icon】
  - 一般由於跨域造成。
  - 點擊「複製跨域代碼」按鈕。
  - 打開影片目標網頁的「開發者工具界面」，找到 console 欄。
  - 貼上剛剛複製的內容，按下Enter。
  - 在原始頁面重新嘗試下載。

【下載後的影片資源不可看】
  - 網站對影片源進行了加密操作。不同的影片網站有不同的算法操作。無法通用處理。
  - 一般網站不會有這種情況。愛奇藝，騰訊等大影片網站才會有該安全措施。

### 實現思路
【下載並解析 m3u8 文件】
- 直接通過 fetch 請求 m3u8 文件。得到 m3u8 文件的內容字符串。讀取字符串進行解析。

【隊列下載影片片段】
- 使用 fetch 請求影片片段，下載的是影片文件，屬於二進制數據。
- 使用隊列下載，是因為影片片段太多，不可能一次性請求全部。需要分批下載。
- 同時由於瀏覽器同源並發限制，影片同時請求數不能過多。本工具設置為並發下載數為 3。

【組合影片片段】
- 使用 [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob) 對象將多個文件整合成一個文件。
- 需要在 new Blob 的第二個參數中設置文件的 MIME 類型。

【自動下載】
- 使用 [URL.createObjectURL](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL) 得到瀏覽器內存中的 Blob 文件連結。
- 使用 a 標簽的 [download](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a) 屬性設置下載功能。

### 核心代碼
【整合及自動下載】

```javascript
async mergeAndDownload() {
    this.tips = '正在合併檔案...';
    
    const sortedData = Array.from(this.mediaCache.entries())
        .sort(([a], [b]) => a - b)
        .map(([_, value]) => value.data);
    
    try {
        const blob = new Blob(sortedData, { 
            type: this.isGetMP4 ? 'video/mp4' : 'video/MP2T' 
        });
        
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `video_${this.formatTime(this.beginTime)}${this.isGetMP4 ? '.mp4' : '.ts'}`;
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
        URL.revokeObjectURL(url);
        
        this.mediaCache.clear();
        this.downloading = false;
        this.tips = 'M3U8 下載工具';
    } catch (error) {
        console.error('合併檔案時發生錯誤:', error);
        this.alertError('合併檔案時發生錯誤');
    }
}
```

### AES 解密功能
- 使用「aes-decryptor.js」，該文件來至 [hls.js](https://github.com/video-dev/hls.js)

### MP4 轉碼功能
- 使用「mux-mp4.js」，來自 [mux.js](https://github.com/videojs/mux.js#mp4)

### 第三方接入
- 在 url 中通過 source 參數拼接下載地址即可，如：```https://m3u8-download-cht.glitch.me?source=https://example.com/video.m3u8```
- 系統將自動解析該參數

### 完結撒花，感謝閱讀。
