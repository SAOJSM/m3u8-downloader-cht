# M3U8 Video Downloader

### [Online Tool](https://m3u8-downloader-cht.glitch.me/index-en.html), Chrome browser is recommended.

### Background
- About M3U8 Video Format
    - M3U8 video format splits a complete video into multiple video segments. The M3U8 file records the address of each video segment.
    - When playing the video, it first reads the M3U8 file, then downloads and plays video segments one by one.
    - This method is commonly used in live streaming and helps prevent video theft.
- Due to these characteristics, M3U8 videos cannot be downloaded directly through video links.
    - Traditional download software is often complicated to use and unreliable.
    - Software downloads can be unstable, with slow speeds or failures even when browser playback works fine.
    - Most software is compiled and packaged, making it hard to understand how they work.
- This tool was developed to address these issues.

### Features
- No installation required, works directly in your browser.
- Force download available segments without waiting for the complete video.
- Intuitive operation with precise control over video segments.
- Support for cross-origin requests.
- Support for downloading specific ranges of segments.

### Functions
【Download】Input the M3U8 link and click to download the video.
【Cross-domain Code】When encountering cross-domain restrictions, click to copy the code and paste it into the video page's console.
【Download Range】Choose to download only specific segments of the video.
【Segment Icons】Shows the status of each video segment. "Gray": pending, "Green": successful, "Red": failed. Click red icons to retry failed segments.

### Instructions
1. Open the video page and access Developer Tools (F12)
2. Find 'Network' tab, filter for "m3u8"
3. Refresh the page to capture M3U8 files
4. Find the target M3U8 file and copy its URL
5. Open this tool, paste the URL, click "Download Original" or "Download MP4"
6. If you encounter cross-domain issues, use the "Copy Cross-domain Code" feature
7. Wait for the download to complete
8. If any segments fail, click them to retry

### Troubleshooting
【No Segment Icons Showing】
  - Usually due to cross-domain restrictions
  - Use the "Copy Cross-domain Code" feature
  - Paste the code in the video page's console
  - Try downloading again on the original page

【Downloaded Video Won't Play】
  - The video might be encrypted
  - This is common with major streaming platforms like Netflix, Hulu, etc.

### Technical Implementation
【M3U8 File Processing】
- Uses fetch to download and parse M3U8 files

【Segment Download Queue】
- Uses fetch to download video segments as binary data
- Implements queue system for batch downloading
- Limits concurrent downloads to 3 to avoid browser restrictions

【Segment Combination】
- Uses [Blob](https://developer.mozilla.org/en-US/docs/Web/API/Blob) to combine video segments
- Sets appropriate MIME type for the final file

【Automatic Download】
- Uses [URL.createObjectURL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL)
- Implements download using the [download](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a) attribute

### Core Code
【Merge and Download】

```javascript
async mergeAndDownload() {
    this.tips = 'Merging files...';
    
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
        this.tips = 'M3U8 Downloader';
    } catch (error) {
        console.error('Error merging files:', error);
        this.alertError('Error merging files');
    }
}
```

### AES Decryption
- Uses aes-decryptor.js from [hls.js](https://github.com/video-dev/hls.js)

### MP4 Conversion
- Uses mux-mp4.js from [mux.js](https://github.com/videojs/mux.js#mp4)

### Third-party Integration
- Add source parameter to URL: ```https://m3u8-downloader.glitch.me?source=https://example.com/video.m3u8```
- The system will automatically process the parameter

### Thank you for reading!










































































