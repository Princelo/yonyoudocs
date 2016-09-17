# NC SQL 編程規範
## 一、槪述
本手冊側重於代碼編寫過程中 SQL 語句的編寫規範問題，內容涉及書寫風格，性能優化，多數據適配等方面。
文檔中用 ★ 標示的內容為必須遵守的條例，其餘的可視為建議。
## 二、書寫風格
    1. SQL 語句全部使用小寫。★
    * 引用字符時用單引號。如: `update testable set idcol = 'abcd'`。★
    * 連接符或運算符or、in、and、=、<=、>=、+、— 等前後加上一個空格。
    * 嚴禁使用 select * …… 形式的語句，必須指出 select 的具體字段，即`select col1, col2, ... from tablea where ...`★
    * 嚴禁使用`insert into table values(?,?,?)`，必須指出具體要賦值的字段，即`insert into tablea(col1, col2, ...) values (?, ?, ...)`★
    * SQL 語句包含多表連接時，建議對每個表命名別名，對每個字段的使用都要帶上表別名，即`select a.col1, a.col2, b.col3 from tablea a, tableb b where a.col4 = b.col5`
    * 避免隱含的類型轉換。例如在 where 子句中 numeric 型和 int 型的列的比較或相加。★
    * 讀取指通過 JDBC 讀到的數據格式，保存是指保存在VO中的數據格式。插入或者更新是指 insert 或者 update 語句中的數據格式。
        * 整型字段：讀取時根據字段設置保存為 Integer 或者 Long。
        * 數字型字段：讀取為 BigDecimal，並保存為 UFDouble，插入或者更新時為 BigDecimal。
        * 字符型字段：讀取為 String，並保存為 String，插入或者更新為 String。
        * 布爾型字段：讀取為 String('Y' OR 'N')，並保存為UFBoolean，插入或者更新時為 String('Y' OR 'N')。
        * 時間字段：讀取為 String，並保存為UFDateTime，插入或者更新時的時間格式由中間件統一處理，有單獨需求的要申請後才能決定。
    * 在子查詢中前後必須加上括號，`select col1, col2 from tablea where col3 in (select col4 from tableb where col4 > 0)`★
    * 避免在 where 使用‘1 = 1’，‘1 = 2’這種表達式作為部分條件，如`select col1, col2 from tablea where 1 = 1 and col1 > 0`。
    * 禁止使用視圖。★
## 三、性能優化
    1. 儘量使用 prepareStatement，利用預處理功能。
    * 在進行多條記錄的增加、刪除時，建議使用批處理功能，批處理的次數以整個 SQL 語句不超過相應數據庫的 SQL 語句大小的限制為準。
    * 建議每條 SQL 語句中的 in 中的元素個數在 500 以下，如果個數超過時，應拆分為多條 SQL 語句。禁止使用 xx in (...) or xx in (...)。★
    * 禁止使用 or 超過 500，如 `xx = '123' or xx = '456'`。★
    * 儘量不使用外連接。
    * 禁止使用 not in 語句，建議用 not exist。★
    * 禁止使用 union，如果有業務需要，請拆分為兩個查詢。★
    * 禁止在一條 SQL 語句中使用3層以上的嵌套查詢，如果有，請考慮使用臨時表或中間結果集。★
    * 儘量避免在一條 SQL 語句中從 >= 4 個表中同時取數，對於僅是作為過濾條件關聯，但不涉及取數的表，不參與表個數計算；如果必須關聯4個或4個以上表，儘量採用子查詢的方式。
    * 儘量避免使用 order by 和 group by 排序操作，因為大量的排序操作影響系統性能。如必須使用排序操作，儘量建立在有索引的列上。
    * 對索引列的比較，儘量避免使用 not 或 !=，可以考慮拆分為幾個條件。如 col1 是索引列，條件 col != 0 可以拆分為 col1 > 0 or col2 < 0。
    * 任何對列的操作都將導致表掃描，所以應儘量將數據庫函數、計算表達式寫在邏輯操作符右邊。
    * 在對 char 類型比較時，建議不要使用 rtrim() 函數，應該在程序中將不足的長度補齊。
    * 用多表連接代替 exists 子句。
    * 如果有多表連接時，應該有主從之分，並儘量從一個表取數，如`select a.col1, a.col2 a join b on a.col3 = b.col4 where b.col5 = 'a'`。
    * 在 where 子句中，如果有多個過濾條件，應將索引列或過濾記錄數最多的條件應該放在前面。
    * 在使用 like 時，建議 like 的一邊是字符串，表列在一邊出現。
## 四、多數據庫的考慮
    1. 字符串連接必須用“||”符號，不使用“+”。注意：在 Oracle 中一個 null 值與非 null 值連接，結果為非 null 值，在 DB2 和 SqlServer 中相反。使用 isnull() 對 null 轉換為''。★
    * 通配符不能使用 '[a-c]%' 這種列式，應寫成如：`select col1, col2 from table_name where col1 like '[a]% or col1 like '[b]%' or col1 like '[c]%'`。★
    * 在 case when 語句中只能出現 =、>=、<= 以及 is null 運算符，不能出現 <、>、<>、!= 以及 is not null 運算符。否則在 Oracle 的 decode 函數無法表達。★
    ```
    當必須使用 <、>、!=、is not null 時，建議採用如下變通方法：
    1. 使用 != 時：例如 case when a != b then e，
        可改為：case when a = b then e1 else e (間接實現 a != b)
    2. 使用 < 時： 例如 case when a < b then e,
        可改為：case a >= b then case a = b then e1 else e (間接實現 a < b)
            或 case a >= b then e1 else e (間接實現 a < b)
    3. 使用 > 時：例如 case when a > b then e,
        可改為：case a >= b case a = b then e1 else e (間接實現 a > b)
            或：cashe a <= b then e1 else e  (間接實現 a < b)
    4. 使用 is not null 時：例如 case when a is not null then e,
        可改為 case a is null then e1 else e  (間接實現 a is not null)
    ++特別說明++：當執行大數據量的操作時，Sql Server 對 case when 的持衡效率極低，甚至可能會死機，因此希望大家儘量不要使用 case when。
    ```
    * 左連接的寫法必須帶 “outer” 關鍵字。例如：`select f1 from t1 left outer join t2 on t1.f1 = t2.f1`，而不是`select f1 from t1 left join t2 on t1.f1 = t2.f1`。★
    * 只能使用以下函數，如要使用新的函數必須申報，審批後才能使用。函數：
    ```
    coalesce, cast, len, left, replace, right, substring, lower, upper, ltrim, rtrim, abs, acos, asin, atan, cos, ceiling, exp, floor, log, power, round, sign, sin, square, sqrt, tan, count, max, min, sum, avg
    ```★
    * substring 函數中起始位置為1，表示從頭開始。★
    * 對於一些 char/varchar 的字段的值，即使是 0，1，2 等值，也必須表達為 '0', '1', '2' 。★
