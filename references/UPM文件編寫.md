# UPM 文件編寫
### 原作者：谷允金 2012年3月15日

## 一、槪述

目前我們編寫的代碼主要放在三個文件夾下：public、private 和 client。服務器代碼放在
private 文件夾下；客戶端代碼放在 client 文件夾下；服務器代碼和客戶端代碼不能相互
訪問和依賴，服務端和客戶端都要依賴的代碼，比如服務接口，放到 public 文件夾下。
如下圖：

![upm-dir](https://github.com/Princelo/yonyoudocs/blob/master/references/public/images/upm-dir.png?raw=true)

以上面的圖片為例，IHelloNC 是一個接口類，在 public 文件夾下，HelloNC 是它的實現類,
在 private 文件夾下。假如我們要用到 IHelloNC 裡面的某個方法，是怎樣調用的呢？我們
一般通過 upm 文件來註冊接口類實現類之間的關係，客戶端代碼通過 NCLocator 類獲取本地
的服務代理。

## 二、通過操作簡單認識一下 upm

### 1. 新建 HelloNC 項目

如上圖所示，新建一個 HelloNC 項目及相關目錄下的類。各個類代碼如下：

**IHelloNC**:

```
public interface IHelloNC {
    String sayHello();
}
```

**HelloNC**:

```
public class HelloNC implements IHelloNC {
    @Override
    public String sayHello() {
        return "Hello NC";
    }
}
注：這只是簡單演示了例子，真正工作中要在該實現類的方法中進行異常處理（詳見本文3.2.3異常處理）
```

**ClientUI**:

```
public class ClientUI extends AbstractFunclet {
    public void init() {
        IHelloNC helloNC = NCLocator.getInstance().lookup(IHelloNC.class);
        String msg = helloNC.sayHello();
        System.out.println(msg);
    }
}
```

### 2.增加配置信息

在 META-INF 目錄下新建 HelloNC.upm

```
<?xml version="1.0" encoding="gb2312"?>
<module name="HelloEJB">
    <public>
        <component singleton="true" remote="true" tx="CMT">
            <interface>nc.itf.demo.hello.IHelloNC</interface>
            <implementation>nc.impl.demo.hello.HelloNC</implementation>
        </component>
    </public>
```

## 三、組件配置信息

### 1.參數介紹

上面通過一個例子，讓大家對組件的配置有了初步的瞭解。下面我們來詳細介紹一下upm文件
中的組件配置信息：
模塊組件的配置主要分為 public 和 private 兩大塊，public 為公业的組件，而 private
為模塊的私有組件。公共的組件能夠被 NCLocator 查找，而私有的組件只能被該模塊內的組
件通過 ComponentContext 查找。
每個 Component 配置主要有如下的屬性和元素：

|名稱 | 意義 |
|----|:--------------:|
|name | 屬性，組件的名稱，可以為空  |
|interface | 元素，組件的接口類，如果不是遠程的或者獨立發版需要可以為空  |
|inplementation | 元素，組件的實現類，一般來說必須提供  |
|remote | 屬性，值為 true 或者 false，表示該組件是否被公佈為遠程組件，默認為false，只對公共組件有效  |
|singleton | 屬性，值為 true 或者 false，表示該組件是否為組件級別的單例，默認為true  |
|construct | 元素，指出構造該組件的構造函數  |
|factory-method | 元素，construct與它只能用一個，用於構造組件  |
|factory-method/method | factory-method的屬性，用於表明構造對象的方法  |
|factory-method/provider | factory-method的方法提供者，如果用引用那麼會用該引用的實例創建對象，否則用一個類的靜態方法。  |
|property | 元素，組件的属性，按照javabean的規範進行  |
|tx | 屬性CMT，表示事務型，其他為非事務，只對公共組件有效  |
|cluster | 屬性，默認為空，SP標示運行於主節點，其他表示與某個服務運行在同一個節點  |

通過開發環境提供的插件你可以把該組件進行發佈，發佈後配置組件在對應的配置文件的 upm文件中，
內容如下：

```
<public>
    <component name="nc.itf.uap.template.TemplateEJB" priority="0" singleton="true" remote="true" tx="CMT" supportAlias="true">
        <interface>nc.itf.uap.billtemplate.IBillTemplateBase</interface>
        <interface>nc.itf.uap.pf.IPFTemplate</interface>
        ...
        <implementation>nc.impl.uap.template.TemplateEJB</implementation>
        ...
        <property name="iBillTemplateBase">
            <ref>IBillTemplateBase</ref>
        </property>
    </component>
</public>
```

這裡我們制定了tx屬性來表示發佈為一個事務型發佈，事務型发佈 tx 屬性必須為CMT。從該配置我們還可以
看出，該接口同時發佈為遠程的(remote 屬性為 true)，一個組件可能同時為遠程組件和事務型組件。
複雜的遠程組件的部署方式也可以採用這種方式（我們稱為私有/發佈模式，發佈後的組件必須在公业區域）
進行，這樣可以減少遠程代理的樹木，提供遠程組件的查找效率。
注意我們使用了 supportAlias 屬性，指出我們可以通過兩個接口中的任何接口進行遠程組件的查找。
supportAlias 為 true 只有在鐮組件中才起作用。

### 3. 配置規範及約定

#### 1.配置規範

關於 NC 部署文件的編制規定
原則：公有(public)全名化，私有(private)別名化

例如以下範例：
公有的組件不使用“name”屬性，組件的名稱默認為實現的名稱；
私有的組件名稱在“name=”後面進行設置；

```
<public>
    <component priority="0" singleton="true" remote="true" tx="NONE">
        <interface>nc.itf.uap.busibean.ISysInitQry</interface>
        <implementation>nc.impl.uap.busibean.para.SysInitQryImpl</implementation>
    </component>
</public>
<private>
    <component name="ISysInit" priority="0" singleton="true">
        <implementation>nc.bs.pub.para.SysInitImpl</implementation>
    </component>
    <component name="ISysInitUpdate" priority="0" singleton="true">
        <implementation>nc.bs.pub.para.update.SysInitUpdateImpl</implementation>
    </component>
</private>
```

#### 2. 約定

約定如下：
所有的屬性中不能出現(除系統生成的proxy外)
    supportAlias="true|false" priority屬性只對活動組件有用

公共組件約定
公共組件的屬性中不能出現：
    name=""

公共組件可以出現以下屬性：
    tx="NONE"
    singleton="true" //根據不同情況進行決定，建議採用 true
    remote="true|false"

私有組件約定
私有組件的節點不能出現：
    <interface>...</interface>
不能出現以下屬性：
    tx=""
    remote=""
可以出現以下屬性：
    singleton="true" //根據不同情況進行決定，建議採用 true
    name=名稱以接口名稱為準（不包含包名稱）

#### 3.異常處理

upm 中註冊的所有實現類都叫做邊界類，就是這個類是調用後台的入口，所以這些接口都需要拋出異常，
並且實現類 try catch 保證事務的完整性。以運輸單據為例：

```
<?xml?>
<module name="SCM_DMDELIVBILL">
    <public>
        <component priority="0" singleton="true" remote="true" tx="CMT" supportAlias="true">
            <interface>nc.itf.dm.m4804.IDelivBill</interface>
            <implementation>nc.impl.dm.m4804.DelivBillImpl</implementation>
        </component>
```

實現類中用 ExceptionUtils.marsh(ex)
這個 marsh 方法能將異常解析為最原始的異常信息返回到頂層。

```
public class DelivBillImpl implements IDelivBill {
    @Override
    public DelivBillAggVO[] approveDelivBill(DelivBillAggVO[] bills) throws BusinessException {
        DelivBillAggVO[] retvos = null;
        try {
            DelivBillApproveAction action = new DelivBillApproveAction();
            retvos = action.approve(bills);
        } catch (Exception ex) {
            ExceptionUtils.marsh(ex);
        }
        return retvos;
    }
}
```
