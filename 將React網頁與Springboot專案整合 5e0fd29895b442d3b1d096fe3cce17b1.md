# 將React網頁與Springboot專案整合

初期前後端通常會分別開發，因為開發環境不同，啟動的 port 甚至 url 都不一樣
但最後部屬時若沒 Cloud 需求，前後端將會放在同一個容器運行，接下來就看看如何將 React 打包後的網頁放入 Springboot 專案中

第一步就需要將 React 編譯成正式網頁以及 js 檔
我們只需在終端機輸入 yarn run build (使用 npm 就輸入 npm run build)

![請先忽略size，目前沒做任何調整，所有的元件編譯起來非常臃腫](%E5%B0%87React%E7%B6%B2%E9%A0%81%E8%88%87Springboot%E5%B0%88%E6%A1%88%E6%95%B4%E5%90%88%205e0fd29895b442d3b1d096fe3cce17b1/Untitled.png)

請先忽略size，目前沒做任何調整，所有的元件編譯起來非常臃腫

完成後在專案下會新增一個 dist 子目錄

![Untitled](%E5%B0%87React%E7%B6%B2%E9%A0%81%E8%88%87Springboot%E5%B0%88%E6%A1%88%E6%95%B4%E5%90%88%205e0fd29895b442d3b1d096fe3cce17b1/Untitled%201.png)

我們只需將 dist 下的資料，全部複製到 Springboot 專案的 static 目錄下即可
之後啟動 Springboot 並輸入 Springboot 設定的網址，預設就會開啟 static 目錄下的 index.html

不過問題來了，我們在開發時 API 的 hostname 都是輸入 localhost，例如

```html
http://localhost:8080/api/something
```

等部屬到伺服器這網址勢必要更改，而且每當換環境我們就需要修改網址
接著又要

> 重新 build 網頁 ⇒ 複製到 springboot 專案 ⇒ build springboot 專案 ⇒ 最後把 jar 檔複製到新環境執行
> 

這樣也太辛苦，部屬前一刻才知道網址，我們就得重新 build 網頁嗎?

這裡教大家一個好方法，當前端網頁開始整合進 Springboot 專案測試時
可以將所有呼叫 API 的 URL 改成如下

```jsx
let url: string = http://${window.location.host}/api/something;
```

${window.location.host} 這個變數會取得瀏覽器目前的 hostname，這樣不管部屬到甚麼環境，都能抓到正確的網址，也不需要重新 build 網頁了