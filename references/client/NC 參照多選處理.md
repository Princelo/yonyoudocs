# 參照多選處理

## 一、背景

一般表體字段的參照，我們只選擇一個值，默認的參照也只返回一個值，例如客戶、庫存組織、
倉庫等。但物料參照比較特殊需要支持批編輯，即點擊物料參照框，我們可以選擇多個物料一
起返回，那麼就涉及到一個問題，我編輯的某一行的物料字段，選擇了3個物料，那麼UAP物料
參照會返回選擇的3個物料PK，那想一下我們要做甚麼？在該行下方插入2行，填上物料並賦默
認值，最後處理這3行數據的物料編輯後事件。

## 二、實現目標

从某個參照選中多個值後，根據所選值新增多行，選擇值放入每一行的參照字段。
## 三、應用場景

根據業務需求，需要實現多選效果的單據。

## 四、代碼處理

* **工具類**：

nc.ui.pubapp.uif2app.view.util.RefMoreSelectedUtils

* **方法**：

主子表調用 refMoreSelected;

單表調用 refMoreSelectedForBillTable;

* **實現步驟**：

1. 界面初始化時需要將參照（如物料）設置為多選；

```
UIRefPane panel = (UIRefPane) getBillCardPanel().getBodyItem("pk_material").getComponent();
panel.setMultiSelectedEnable(true);
```

2. 編輯後事件中調用工具類，設置按新增或複製來實現多選
