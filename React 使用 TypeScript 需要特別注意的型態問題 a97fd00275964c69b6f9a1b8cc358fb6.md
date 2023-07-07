# React 使用 TypeScript 需要特別注意的型態問題

TypeScript 顧名思義就是所有的變數都需要先宣告，即使在開發環境能開啟並測試，等要打包時還是無法編譯，雖然還是能強制編譯，不過使用 TypeScript 就是要避免更多無法預知的錯誤，所以還是要想辦法將語法上的錯誤先解決在進行編譯

> PS. 如想快速開發並看到成果還是先使用JavaScript版本，用了 TypeScript 真的花不少時間在解決形態上的錯誤
> 

接下來特別列出幾個比較難處理的型態問題，希望能幫到你

## **forwardRef 的型態問題**

寫 js 時forwardRef 都不會管型態，但在 TypeScript 時父階取得 ref 並呼叫函式卻會出現錯誤

![Untitled](React%20%E4%BD%BF%E7%94%A8%20TypeScript%20%E9%9C%80%E8%A6%81%E7%89%B9%E5%88%A5%E6%B3%A8%E6%84%8F%E7%9A%84%E5%9E%8B%E6%85%8B%E5%95%8F%E9%A1%8C%20a97fd00275964c69b6f9a1b8cc358fb6/Untitled.png)

測試時都能正常執行，但編譯卻過不了，以下是處理的方式

1. 宣告型態

由於我們是要掛載程給其他組件用，所以型態內需要放函式，另外forwardRef 的泛型需要設定 ref 跟 props 的型態，所以 props 的 type 也一起宣告

```jsx
type Props={
  rowSelectType:RowSelectionType,
  setMaster: (arg:PlmCompanyDataType)=>void,
  setAvls: (arg:PlmCompanyDataType[])=>void,
}

export interface Ref {
	cleanSelected:()=>void,
	reload:()=>void,
};
```

1. 在 forwardRef 加上泛型

```jsx
export default forwardRef<RefType, PropsType>((props: PropsType, ref)={
```

1. 在父階組件使用 useRef 也要加入泛型

```jsx
const masterDataRef = useRef<RefType>(null);
const avlsDataRef = useRef<RefType>(null);
```

以上都完成就不會報錯了

## **useState若放的是物件也需要設定泛型**

useState 設定泛型跟 useRef 一樣，都是加在 useState 後就好

```jsx
const [master, setMaster] = useState<PlmCompanyDataType>({});
const [avls, setAvls] = useState<PlmCompanyDataType[]>([]);
```

需要特別注意 array 的寫法，另外通常用 array 後都會在使用 map 進行處理，不過 map 內的變數就不需特別指定了，它會使用一樣的型態

```jsx
avls.map((avl) => {
	newAVLGroup = {
		master: { ...master },
		avl: { ...avl },
	}
	createAVLGroup(newAVLGroup)
})
```

## **部分組件出現錯誤**

![Untitled](React%20%E4%BD%BF%E7%94%A8%20TypeScript%20%E9%9C%80%E8%A6%81%E7%89%B9%E5%88%A5%E6%B3%A8%E6%84%8F%E7%9A%84%E5%9E%8B%E6%85%8B%E5%95%8F%E9%A1%8C%20a97fd00275964c69b6f9a1b8cc358fb6/Untitled%201.png)

這是使用 Ant Design 的 Select 以及其底下的 Option，不知為何 Option 就是有錯誤，由於 Option只是為了顯示內容，最後只好使用標準組件 option 代替(小寫)

## 沒用到的變數會出現警告

![Untitled](React%20%E4%BD%BF%E7%94%A8%20TypeScript%20%E9%9C%80%E8%A6%81%E7%89%B9%E5%88%A5%E6%B3%A8%E6%84%8F%E7%9A%84%E5%9E%8B%E6%85%8B%E5%95%8F%E9%A1%8C%20a97fd00275964c69b6f9a1b8cc358fb6/Untitled%202.png)

這比較無傷大雅，真的用不到就移除吧

不過我在 ProTable 中 onChange 帶入兩個變數，第一個沒用的參數也出現警告

![Untitled](React%20%E4%BD%BF%E7%94%A8%20TypeScript%20%E9%9C%80%E8%A6%81%E7%89%B9%E5%88%A5%E6%B3%A8%E6%84%8F%E7%9A%84%E5%9E%8B%E6%85%8B%E5%95%8F%E9%A1%8C%20a97fd00275964c69b6f9a1b8cc358fb6/Untitled%203.png)

最後只好將他打印出來 XD

![Untitled](React%20%E4%BD%BF%E7%94%A8%20TypeScript%20%E9%9C%80%E8%A6%81%E7%89%B9%E5%88%A5%E6%B3%A8%E6%84%8F%E7%9A%84%E5%9E%8B%E6%85%8B%E5%95%8F%E9%A1%8C%20a97fd00275964c69b6f9a1b8cc358fb6/Untitled%204.png)
