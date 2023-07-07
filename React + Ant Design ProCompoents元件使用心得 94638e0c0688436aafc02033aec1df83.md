# React + Ant Design ProCompoents元件使用心得

[Image 20230706-002.mp4](React%20+%20Ant%20Design%20ProCompoents%E5%85%83%E4%BB%B6%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%2094638e0c0688436aafc02033aec1df83/Image_20230706-002.mp4)

## 前言

**Ant Design** 除了標準元件外，還出了 **Ant Design Pro** 跟 **ProCompoents** 函式庫

**Ant Design Pro** 雖然功能強大，可是太龐大了，若非做一個大型系統沒必要用這麼大的框架，需要移除的設定可能比要寫的還多

所以最後選擇 **ProCompoents** 做為系統開發的主要元件庫，當然一些小元件還是使用 **Ant Design** 搭配使用

你可能會說怎麼不選全世界最多人用的 **MUI** 呢？一來是省錢二來則是 **Ant Design** 功能真得更強一些，省錢的部分是 **MUI** 也有推出 Pro 組件，不過這是付費功能，功能更強的部份則是 **Ant Design** 的 Table 組件只要改個設定就能做到 Inline Edit，而 MUI 則要使用 Grid 組件才有 Inline Edit

本文主要是做個紀錄，希望以後在使用 **React + Ant Design** 時能縮短開發時間，另外有關**ProCompoents** 的介紹真的太少，一些問題也能給大家做個參考

## 設計概念

目前這個程式的功能不算多，主要有兩個頁面

**第一個頁面**用來呈現設定好的資料，資料庫存的是 **foreign key**，所以其他資訊需要關聯到另一個表格，API 返回的 JSON 大概如下

```jsx
{
	id: 1
		master:{
			id: 1,
			name: 'abc'
		},
		sliver:{
			id: 2,
			name: 'def'
		}
}
```

表格需要能將 master跟 sliver 的 name 呈現出來，且最後有個欄位可以刪除單筆資料

**第二個頁面**則是用來新增資料用的，會有兩個步驟，第一個挑選master(單選)，第二個挑選sliver(複選)，待資料送出後就能在第一個頁面查詢

下面就是最後的成果

![第一個頁面呈現設定好的資訊，右上角的搜尋是自己包裝的，並非用ProTable的標準搜尋Form](React%20+%20Ant%20Design%20ProCompoents%E5%85%83%E4%BB%B6%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%2094638e0c0688436aafc02033aec1df83/Untitled.png)

第一個頁面呈現設定好的資訊，右上角的搜尋是自己包裝的，並非用ProTable的標準搜尋Form

![新增的部分功能複雜很多，兩個步驟雖然都是同樣的組件，但須依參數顯示單選或複選，最後 submit 建立資料後還須將 Table 勾選的資料清空](React%20+%20Ant%20Design%20ProCompoents%E5%85%83%E4%BB%B6%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%2094638e0c0688436aafc02033aec1df83/Untitled%201.png)

新增的部分功能複雜很多，兩個步驟雖然都是同樣的組件，但須依參數顯示單選或複選，最後 submit 建立資料後還須將 Table 勾選的資料清空

## **ProCompoents** 看起來功能強大，但…

**ProCompoents** 雖然幫我們組合了一堆元件，縮短開發時程，看起來很美好，但實際運行起來還是有些問題，尤其我是以後端開發為主，一些看似簡單的功能我也費了很大的功夫才搞出來，文章中介紹的內容有甚麼更好的方法也請讓我知道

**ProCompoents** 也是 **Ant Design Pro** 的底層的元件，算是加強版的 **Ant Design**，以 **ProTable** 來說，只要將 API 呼叫函式指定給 request 參數，網頁載入後就會自動獲取數據

```jsx
const requestCompanyData = async () => {
	return await fetch("[http://localhost/companies](http://localhost/plmcompanies)")
		.then((response) => response.json())
		.then((json) => {
		return Promise.resolve(json);
	});
};

<ProTable request={requestCompanyData} />
```

如上所示，只須把 fetch 函式指定給 request 參數，系統就能自動幫我們將資料渲染在網頁上

![Untitled](React%20+%20Ant%20Design%20ProCompoents%E5%85%83%E4%BB%B6%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%2094638e0c0688436aafc02033aec1df83/Untitled%202.png)

上圖是官網的一個範例，可以看到搜尋/分頁/編輯都能做到，不過當然許多功能都還得自己實做出來，UI 組件只是加快你完成 Layout 的部分，讓我們這些沒有美工底子的工程師也能做出還能看的網頁

不過幾個問題要特別注意

1. API 產生的 JSON 格式需包含以下幾個 fields
    1. data: 搜尋的資料必須以Array的形式放在data下
    2. success: 成功要傳回true，否則資料會顯示異常，若是後端資料有問題需傳回false
    3. total: 若要系統自動幫你分頁，這裡就要由後端拋出總筆數
2. fetch 若傳回失敗並非後端伺服器錯誤，而是比較底層的錯誤，例如網路異常，此時可丟出網站更新中或是網路繁忙中的錯誤提示，而不是直接顯示一個 500 錯誤
3. 有注意到底部分頁前的文字嗎？沒錯，這是簡體字，而且沒有參數能直接修改，後面在教大家如何調整（沒資料時也顯示簡體）

總之組合元件可以幫我們快速產生堪用的畫面，但要交付出去還是需要一些細部的調整

## 簡體字的調整

先說說如何拿掉表格的預設資訊吧，這裡的預設顯示簡體字真的蠻討厭的，也難怪老外用的人不多

### 1. 修改 ProTable 總筆數資訊

```jsx
pagination={{
	showTotal: ()=>``,
	pageSize: 10,
}}
```

如需調整顯示內容需要在 ProTable 中設定 pagination 屬性

**pageSize** 是每頁顯示的筆數，可自行調整

**showTotal** 則是總筆資訊的內容，需要放一個能返回字串的函式，若不想顯示，可以讓函式回傳空字串，若想顯示資料可以放一個模板字串，傳入兩個變數，第一個 total 用來顯示總筆數，第二個 range 是 array，表示目前頁面的區間，例如

```jsx
showTotal: (total, range)=>`從${range[0]}到${range[1]}，總共 ${total} 筆資料`
```

### 2. 置換沒資料時的顯示內容

![Untitled](React%20+%20Ant%20Design%20ProCompoents%E5%85%83%E4%BB%B6%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%2094638e0c0688436aafc02033aec1df83/Untitled%203.png)

上圖就是 ProTable 沒資料時顯示的內容，只需要在 ProTable 增加下面設定就能改掉預設的文字

```jsx
locale={{emptyText: '無數據'}}
```

其實更正確的做法是設定多國語系，不過只是一兩個地方想修改，而且沒有跨國的需求，所以就直接改屬性了，下面是改完後的顯示內容

![修改後 ICON 也消失了，不過實在找不到如何設定 ICON，就先不管它了](React%20+%20Ant%20Design%20ProCompoents%E5%85%83%E4%BB%B6%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%2094638e0c0688436aafc02033aec1df83/Untitled%204.png)

修改後 ICON 也消失了，不過實在找不到如何設定 ICON，就先不管它了

## PageContainer 的迷失

有寫過視窗程式大概都很難理解切換Tab後還需要自己指定顯示的內容，不過網頁框架這部分就真的需要自己操作，Tab 跟顯示的內容基本上是兩個物件，點選Tab後的流程大致如下

1. 切換 Tab 後觸發 onTabChange 事件
2. onTabChange 中可取得不同 Tab 的 Key，需自己判斷不同的 Key 該產生甚麼內容
3. 切換不同內容在前端框架幾乎都是搭配 Router 來實作(除非你想自己用DOM換掉內容)，我們可以把 Key 設成要跳轉的 Route path，這樣在 onTabChange 中就能直接使用 navigate 切換內容

來看看程式怎麼寫

下面是組件的渲染內容(就是return後的標籤)

```jsx
<PageContainer 
	style={{
		height: window.innerHeight,
		width: '100%',
	}}
	tabList={[
		{
			tab: '清單1',
			key: '/Tab1',
		},
		{
			tab: '清單2',
			key: '/Tab2',
		},
	]}
	tabProps={{
		type: 'card',
		defaultActiveKey: "/",
	}}
	onTabChange={tabChangeHandle}
>
	<Routes>
    <Route path='/' element={<Page1 />} />
    <Route path='/Tab1' element={<Page1 />} />
    <Route path='/Tab2' element={<Page2 />} />
  </Routes>
</PageContainer>
```

Key 是實際會跳轉的路徑，對應到 Route 的 Path，而 tabChangeHandle 就是使用 navigate 來切換路徑及顯示內容

```jsx
const navigate = useNavigate();
const tabChangeHandle = (path: string) => navigate(path, { replace: false })
```

另外有一點要特別注意，Routes外面其實還要需要有 BrowserRouter 標籤，但因我在 App 中只載入 LandingPage，所以 BrowserRouter 需要放在 App 包裹整個 LandingPage

```jsx
ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
	<React.StrictMode>
		<BrowserRouter>
			<LandingPage />
		</BrowserRouter>
	</React.StrictMode>,
)
```

最後有個蠻討厭的地方，當我跳轉到某個頁面後若按下 F5，雖然該路徑的內容會更新，不過 Tab 卻會因為重新渲染而回到初始值，這部分我是使用 useEffect 來讓我按下 F5 後強制跳回初始路徑

```jsx
useEffect(() => {
	navigate("/", { replace: false })
},[]);
```

記得 useEffect 第二參數一定要給一個空陣列，這樣更新後就只會執行一次(第二個參數是監控變數，因為永遠為空不會發生變化，所以第一次執行後就不會再執行了)，否則 State 一變動就會重新執行，變成無窮迴圈

若使用其他的 Navigator 或是 Manu 組件也都是差不多的做法，反正跳轉網頁就是使用 Router 來實作

## 習慣物件導向程式會喜歡用 ref

對於熟悉 JavaScript 的前端工程師，把函式當成參數丟給其他組件執行是很理所當然的事，但對習慣物件導向語言的工程師來說卻很難理解，不過 **React** 有了 ref 後，也能取得組件的實例，可以用物件的方式來操作組件

```jsx
const inputRef = useRef();
<Input ref = {inputRef }></Input>
```

透過上面的設定 inputRef 就能控制 Input 組件，例如要設定輸入框的內容或是將其Disable，都可透過 inputRef 來操作

另外我還透過下面的方式取得 ProTable 的 ref

```jsx
<ProTable<CompanyDataType>
        actionRef={actionRef}
```

透過 actionRef 能做的操作非常多，我目前在刪除以及搜尋時都會 reload 資料，只需呼叫actionRef.current?.reload() 就能重新執行 ProTable 的 request 函式

![這裡是所有actionRef能呼叫的函式](React%20+%20Ant%20Design%20ProCompoents%E5%85%83%E4%BB%B6%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%2094638e0c0688436aafc02033aec1df83/Untitled%205.png)

這裡是所有actionRef能呼叫的函式

但是自己寫的函式組件卻沒有 ref 參數，難道還得將 Function 當作參數丟給父組件控制嗎?

### 透過forwardRef所有的函式組件都能有ref

別怕，**React** 幫我們想到了，只要用 forwardRef 將函式組件封裝起來，就可以帶有 ref 參數

不過實際寫起來有點小複雜，下面是將組件增加 ref 的兩個步驟

1. 將函式組件使用 forwardRef 封裝

原本的函式組件寫法

```jsx
export default MyComponent: React.FC = (props: any) => {
```

forwardRef 封裝寫法

```jsx
export default forwardRef((props: any, ref: any) => {
```

1. 公開函式需加進useImperativeHandle下

```jsx
useImperativeHandle(ref, () => ({
	cleanSelected() {
		tableCleanSelected();
	},
	
	init(){
		initAllData();
	}
}));
```

完成以上動作就能用 ref 來操作 cleanSelected 跟 init 函式了，比較常見的場景是表單下有自己寫的組件，待表單 submit 後要清空數據，這時就能透過 ref 將子組件內容清空

```jsx
refObj.current.cleanSelected()
```

## ProTable 中需要特別注意的部分

如果JSON中包著其他物件，要在表格中顯示的值是物件裡的內容就需要特別指定

```jsx
{
	id: 1
		obj1:{
			id: 1,
			name: abc
		},
		obj2:{
			id: 2,
			name: def
		}
}
```

以上面的 JSON 為例，若我需要在表格第一個 Column 顯示 id，第二個 Coumn 顯示 obj1 的name，第三個 Column 顯示 obj2 的 name，設定如下

```jsx
const columns: ProColumns<ObjType>[] = [
{
	title: "ID",
	key: "ID",
	dataIndex: "id",
},
{
	title: "物件1名稱",
	key: "Obj1Name",
	render: (_, row) => row?.obj1?.name,
},
{
	title: "物件2名稱",
	key: "Obj2Name",
	render: (_, row) => row?.obj2?.name,
}]
```

如果只有一層的話 dataIndex 直接放上 field name 就能顯示內容，若是 Object，dataIndex 就無法指定了，需要自訂 render 顯示想要的內容

若是更進階的用法，例如在欄位上增加一個刪除的 Button，且按下後顯示是否刪除，可以參考下面寫法

```jsx
{
  title: 'Actions',
  key: 'action',
  render: (_, row) => (
    <Popconfirm 
			title="是否刪除?" 
			onConfirm={
				() => deleteDataById(String(row?.id))
			}> 
			<Button key="delete">
				刪除
			</Button>
		</Popconfirm>
  )
},
```

由於新增頁面下面是我們自己寫的組件，需要在勾選後執行 props 傳入的函式，這樣父組件才能取得勾選的資料，在 ProTable 可透過下面方式自動觸發函式

```jsx
<ProTable<CompanyDataType>
        actionRef={actionRef}
        columns={columns}
        rowSelection={{
          type: rowSelectType,
          onChange(selectedRowKeys,selectedRows) {
            console.log("selectedRows", selectedRows)
            rowSelectType === 'radio' ? setMaster(selectedRows[0]) : setAvls(selectedRows)
          },
        }}
        tableAlertRender={({ onCleanSelected }) => {
          tableCleanSelected = onCleanSelected; //將清空函式掛載出去，之後透過useImperativeHandle以及forwardRef讓父階組件在送出資料後可以清除選項
          return false;
        }}
        request={requestPlmCompanyData} //fetch包裝函式
```

其中 rowSelecttion 會在勾選後呼叫 onChange，這裡要特別注意參數的順序，第一個 key 雖然沒用到，還是要放個變數，不然只會取到 key 而不是物件

tableAlertRender 則會在勾選資料後於表格上方出現想顯示的結果，例如取消選擇以及已選擇的筆數，若不想顯示可以直接 return false，其實整個 tableAlertRender 都不寫也行，但這裏我們可以取得 onCleanSelected 函式，呼叫就能清空選擇，只要將函式掛在 useImperativeHandle 下就能讓父組件控制清空選擇內容

![Untitled](React%20+%20Ant%20Design%20ProCompoents%E5%85%83%E4%BB%B6%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%2094638e0c0688436aafc02033aec1df83/Untitled%206.png)

## 使用率最高的組件 Message

開發時我們可以透過 console.log 來查看變數，但正式上線總不能要使用者也按 F12 查看 log 吧，這時一些重要的訊息就需要透過 Message 顯示給用戶

原以為直接呼叫函式即可 

```jsx
message.success(’123')
```

畫面上也順利跳出成功的 message，可是看一下 log 卻出現一個警告

![Untitled](React%20+%20Ant%20Design%20ProCompoents%E5%85%83%E4%BB%B6%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%2094638e0c0688436aafc02033aec1df83/Untitled%207.png)

具體原因我就不特別說了，總之要避免警告訊息需要做以下幾個步驟

1. 在需要顯示訊息的組件使用 useMessage 取得函式及一個掛載變數 contextHolder
    
    ```jsx
    const [messageApi, contextHolder] = message.useMessage();
    ```
    
2. 將 contextHolder 變數放在渲染得根節點內
    
    ```jsx
    return (
    <>
    	{contextHolder}
    	<Row>
    		<Col span="8" offset="12">
    			<Search>
    ```
    
3. 之後呼叫就不會有錯誤了
    
    ```jsx
    messageApi.info("這是一般訊息")
    messageApi.error("這是錯誤訊息")
    messageApi.warning("這是警告訊息")
    messageApi.loading("這是載入中訊息")
    ```
    
    ![Untitled](React%20+%20Ant%20Design%20ProCompoents%E5%85%83%E4%BB%B6%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%2094638e0c0688436aafc02033aec1df83/Untitled%208.png)