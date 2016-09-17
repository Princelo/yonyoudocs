# 新增按鈕後單據默認值，新增表體行時默認值，典型處理
### 原作者：付強 2012年3月19日

## 一、槪述
    新增通常指單據由瀏覽狀態變為可編輯狀態，並且界面如有數據要清空，僅僅是
界面變化，通常不涉及到後台操作。

## 二、如何在新增或增行時設置默認值
### 1. 單表批量操作默認值
    增行按鈕繼承 nc.ui.uif2.actions.batch.BatchAddLineWithDefValueAction，
重寫setDefaultValue(Object obj)方法，對 obj 設置值就可以了，obj為當前單據
的VO類型。
### 2. 主子表新增時表頭默認值
    重寫 nc.ui.uif2.editor.BillForm 的 setDefaultValue() 方法，在此方法中，
可以直接操作 BillCardPanel 來設置默認值。
### 3. 主子表表體增行默認值
* 增行按鈕繼承 nc.ui.pubapp.uif2app.actions.BodyAddLineAction
* 插入行按鈕 nc.ui.pubapp.uif2app.actions.BodyInsertLineAction 重寫
getInsertVO() 方法，直接返回要插入的行VO，默認值就在這個VO中設置好。
