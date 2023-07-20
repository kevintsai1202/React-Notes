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

-----

**補充**

原本程式測好好的，沒想到客戶要求要部屬到Tomcat上，沒關係Springboot支援打包成war檔，只要把pom.xml的packaging標籤從jar改成war不就得了!!

好吧，其實沒這麼容易，war只是壓縮格式，還需要引入Tomcat的插件跟設定才能部屬到Tomcat伺服器中，我們先來改pom.xml吧

1. 第一步還是先把打包方式改掉

```
<packaging>war</packaging>
```

2. 接著引入tomcat包

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-tomcat</artifactId>
	<scope>provided</scope>
</dependency>
```

3. 另外因為部屬到Tomcat會用打包的檔名建立目錄，所以可以在build標籤下加上finalName

```
<finalName>avlgroup</finalName>
```

這樣打包後就會存成avlgroup.war

4. 手動加入 ServletInitializer.java
不過這只是壓縮格式，我們的程式目前還是只支援Java啟動(透過main啟動)，要放在Tomcat可要符合Servlet的方式才能在Tomcat中啟動阿
還好Springboot也都幫我們想好了，如果一開始建立專案就勾選War的話，在根路徑除了主程式還會有一個ServletInitializer.java
內容如下

```
public class ServletInitializer extends SpringBootServletInitializer {
	@Override
	protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
		return application.sources(MyAppApplication.class);
	}
}
```
其實這就是部屬在 Tomcat 後會啟動的 Servlet 函式，而程式最後還是加載我們的 Main 主程式

若專案是之後才改成War，記得手動增加這個程式，之後部屬到Tomcat上才能正常執行

接著打包放進 tomcat 的 webapps 目錄，可以順利解壓縮並部屬，趕快輸入網址測試，沒想到畫面一片空白...

檢查 log 才發現問題大了，原來給Tomcat管理的話會多一個路徑

原本網址輸入
```
http://localhost:8080/
```
現在都改成
```
http://localhost:8080/avlgroup
```

5. 修改react路徑
所以原本寫在 react 裡的路徑全都要修改了，有路徑的地方包含 api 還有 router 所設定的網址
最後還有一點很重要的的，就是 build 完產生的 index.html 也需要更改(其實應該可透過參數加上，不過先手動調整吧)
如下面的 code，js 跟 css 的路徑都要加上，若有用 icon 也要記得修改
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <!-- <link rel="icon" type="image/svg+xml" href="/vite.svg" /> -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>GPM管理平台</title>
    <script type="module" crossorigin src="/avlgroup/assets/index-994b621e.js"></script>
    <link rel="stylesheet" href="/avlgroup/assets/index-04490b11.css">
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```
