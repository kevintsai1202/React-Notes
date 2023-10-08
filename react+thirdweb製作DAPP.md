![圖片](https://github.com/kevintsai1202/PIZZA-DAPP/assets/12706683/81316d92-47db-46af-b176-03b81ab25216)
# PIZZA-DAPP
本篇教學是給有 React 基礎的前端工程師參考的，React 如何使用不在本篇教學範圍

直接進入主題，以前要開發一個 DAPP 非常麻煩，雖然已有 Ethers.js 或是 web3.js 這類函式庫，但是光一個錢包的處理就非常繁瑣，直到最近看到 Thirdweb 的教學才發現寫個 DAPP 這麼容易，除了有各種 Hook 外，還有連結錢包跟調用智能合約的元件可供使用，大大縮短了 DAPP 的開發時程

至於為什麼我採用 React 框架？因為 Vue 開發的 DAPP 很容易被當成對岸的項目（容易跑路😂），廢話不多說，下面就開始寫程式吧

## 建立API Key
![Untitled](https://github.com/kevintsai1202/PIZZA-DAPP/assets/12706683/2ac03541-f576-4585-b70f-422074aac859)

跟其他函式庫不同，Thirdweb 需要在平台註冊並取得 api key 才能正常使用，這部分有點類似ChatGPT，使用時需要將 id 寫在 Provider 裡才能正常連線到 web3，這部分要特別注意

## 部屬智能合約
![Untitled](https://github.com/kevintsai1202/PIZZA-DAPP/assets/12706683/047ca013-2b3e-44cf-900f-246595f9eb94)
![Untitled](https://github.com/kevintsai1202/PIZZA-DAPP/assets/12706683/7c0ef80f-89a0-4896-bf01-2f8f636468d9)
想開發通用的智能合約可以透過平台提供的範本部署，若智能合約已經部屬在鏈上，我們只需要將智能合約的地址匯入 Thirdweb 即可使用 api 調用，匯入方式在下圖點選 Import Contract 再輸入合約地址即可，不管是部署還是匯入合約，完成後都能在 Thirdweb 提供的介面直接調用或查看內容，這部分比在區塊鏈瀏覽器還好用

## 程式範本
![Untitled](https://github.com/kevintsai1202/PIZZA-DAPP/assets/12706683/61d5d9c6-645a-4def-9943-6cdec0c678a8)

點選智能合約函式在按下Code Snippets會出現使用範例，這部分對第一次使用的工程師非常友好

## 安裝Thirdweb及引用

完成部署後就是在 VS Code 安裝及引用Thirdweb，安裝跟其他函式庫一樣在終端機輸入安裝指令即可，我是用yarn的方式安裝，執行內容如下

```bash
yarn add @thirdweb-dev/react @thirdweb-dev/sdk ethers@^5
```

安裝完成就能Import，我們先完成第一個步驟-在App 中將 ThirdwebProvider 放在最外層，這是React很常見的用法，記得要將 申請的 api id 填入設定中，之後就能能正常使用 thirdweb 元件

```jsx
import { ThirdwebProvider, useContract } from "@thirdweb-dev/react";

function App() {
  return (
    <ThirdwebProvider 
      activeChain="binance" 
      clientId="YOUR_CLIENT_ID" // You can get a client id from dashboard settings
    >
      <Component />
    </ThirdwebProvider>
  )
}
```

## 錢包連線

與web3 連線的第一步就是使用小狐狸錢包登入，這部分 Thirdweb 封裝了一個 ConnectWallet，點選按鈕會直接跳出錢包選擇、區塊鏈選擇跟帳號切換等功能，錢包在連線狀態下還能自動連線，連線後也能顯示錢包地址，這些完全不用自己動手，此外還提供客制功能，能顯示自訂圖片或說明，這麼複雜的功能只需要一個標籤即可完成，是不是覺得特別幸福啊，而且在手機版也能正常開啟小狐狸錢包連線，並且會使用小狐狸錢包內建瀏覽器來開啟 DAPP 網頁，這樣的相容性是最好的

```tsx
import { ConnectWallet } from "@thirdweb-dev/react";

function App() {
  return <ConnectWallet modalSize="compact"/>;
}
```
這麼短短幾行就能完成錢包連線的所有功能，而且還支援多種錢包，也能自行設定那些錢包要出現
![Untitled](https://github.com/kevintsai1202/PIZZA-DAPP/assets/12706683/2e7e941a-a6b6-46c3-b1d5-04d64b89c188)
![Untitled](https://github.com/kevintsai1202/PIZZA-DAPP/assets/12706683/1b7e9165-b1cd-4081-b0c8-ae8a4d79e4de)


## 調用函式

連線後當然就是調用智能合約函式了，前面提過平台提供各的撰寫範本，函式又分為 Read 跟 Write，差別就在有沒有將數據寫入到鏈上，有的話執行時還會出現瓦斯費用的確認

Thirdweb 還提供呼叫函式的按鈕，通常 Write 函式會搭配按鈕使用，按下按鈕才會執行並跳出瓦斯確認，而 Read 因為只讀取不耗費瓦斯，通常會直接顯示在網頁上

```tsx
	//以下為Read Function的調用方式，可以在進入函式就先讀取，之後再放在要顯示的地方
	const { data: tvlBalance } = useContractRead(contract, "getBalance")
  const { data : myMiners} = useContractRead(contract, "hatcheryMiners", [props.address])
  const { data : myEggs} = useContractRead(contract, "getEggsSinceLastHatch" ,[props.address])
  const { data : myRewards} = useContractRead(contract, "calculateEggSell", [myEggs])
```

```tsx
//下面是Web3Button結合Write函式的用法，通常會按鈕後進行費用確認才開始執行
<Web3Button className="btn"
            style={{  
										width: '100%', 
										fontSize: '1.2rem', 
										borderRadius: '30px', 
										fontWeight:'bold'
	                }}
            contractAddress={contractAddress}
            action={async ()=>{
				                        await contract?.call("bakePizza",[myReferrer],{
													                        value: ethers.utils.parseEther(value)
															                      })
		                }}
            onSuccess={()=>{
			                        setValue("0.01")
			                        message.success("BAKE Pizza Success!")
			                 }}
            onError={(error)=>{
			                        setValue("0.01")
			                        message.error(error.message)
                    }}
>BAKE Pizza
</Web3Button>
```

## 錢包餘額讀取

一個DAPP除了要調用智能合外，還需要讀取錢包的代幣餘額，這部分稍微複雜一點，因為取得Balance是個Promise函式，所以還需要撰寫成功的擷取和失敗的顯示內容，程式如下

```tsx
//放在useEffect中是希望切換錢包時就能重新讀取
useEffect(()=>{
        wallet?.getBalance()
        .then(
          (result)=>setWalletBalance(parseFloat(result.displayValue).toFixed(10))
        )
        .catch(
          (reason)=>(message.error(reason))
        )
    },[props])
```

正常來說，回傳值result.displayValue是String類型，可以直接輸出在網頁上，但區塊鏈代幣精度通常是小數點下18位，顯示在畫面上有點過長，可以如上先使用parseFloat轉為浮點數再擷取小數點下十位
