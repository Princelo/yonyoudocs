# 單據按鈕的可用性處理
### 原作者：谷允金 2013年3月16日

## 一、按鈕的配置

模板中我們一般根據需要來選擇按鈕，一般按鈕的顯示狀態分為編輯狀態和非編輯狀態。
一般打開一個模板，默認為非編輯狀態，當你選中某個文件或某條記錄進行編輯或修改
時，就進入了編輯狀態。兩種狀態下可根據需要顯示不同的按鈕。

下面的圖片為模板剛打開時，即非編輯狀態：

此時可以進行新增、修改或刪除、複製菜單的操作，由於還沒有新增菜單，此時，啟用
菜單無法操作。當點擊新增後，進入編輯狀態，出現如下界面：

你會發現，有些按鈕消失了，又出現了新的按鈕。這兩種狀態之間的按鈕切換，是通過
客戶端的xml配置文件來實現的。

兩種狀態下配置的按鈕：

![button-xml](https://github.com/Princelo/yonyoudocs/blob/master/references/client/images/button-xml.png?raw=true)

每個按鈕對應的類，默認是 pubapp.uif2app 目錄下對應的 action，如果該 action 不
能滿足需求，要自己寫 action 方法。

在上圖中，屬性“action”下的列表是非編輯狀態，即一打開或保存以後，配置文件中的五
個按鈕就可以在頁面顯示出來：
點擊新增或修改操作時，進入編輯狀態，這五個按鈕就會消失，而顯示“editActions”下
的四個按鈕。這裡所指的顯示，是指能看到，能不能進行點擊操作，還要根據具體的業務
具體判斷。如下圖所示，雖然三個按鈕顯示了，但是卻不能進行點擊操作。

## 二、按鈕的顯示處理

以添加按鈕為例，我們來看下代碼：

![button-code1](https://github.com/Princelo/yonyoudocs/blob/master/references/client/images/button-code1.png?raw=true)

當點擊添加按鈕時，StandardBillAddAction 中的 doAction 方法被觸發，該方法將當前
UIState 狀態改為 ADD 狀態。關於 UIState 的狀態類型，可以參見下圖：
StandardBillAddAction 中的 isActionEnable() 方法，是從父類繼承的。返回值為 boolean
類型，是用來判斷當前狀態的。如果是返回 true，頁面則該按鈕為可操作狀態，否則，
返回 false，頁面按鈕可操作。
你可以根據自己的需求來定義按鈕是否可用，比如上面的例子中，如果需要該按鈕只在
NOT_EDIT 和 ADD 狀態下可用，則該方法應該這樣寫：

    ```
    protected boolean isActionEnable() {
        return this.model.getUiState() == UIState.NOT_EDIT
            || this.model.getUiState() == UIState.ADD;
    }
    ```

如果有更複雜的情況，要進行邏輯判斷，只有在滿足需要的情況下才能返回 true，否則
返回 false。
