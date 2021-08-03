---
sort: 7
---
# Error handling
不要讓錯誤處理影響你的程式邏輯閱讀

---

## 用例外，而不要用回傳狀態碼
以前會用的辦法：設置錯誤旗標或是回傳代表錯誤的狀態碼。  
這方法的問題在於，呼叫者必須立即處理錯誤，但常會忘記。且馬上要處理錯誤會讓錯誤處理的程式碼蓋過本來流程。  
修改前:
``` java
public class DeviceController {
    ...
    public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1);
        // Check the state of the device
        if (handle != DeviceHandle.INVALID) {
            // Save the device status to the record field
            retrieveDeviceRecord(handle);
            // If not suspended, shut down
            if (record.getStatus() != DEVICE_SUSPENDED) {
                pauseDevice(handle);
                clearDeviceWorkQueue(handle);
                closeDevice(handle);
            } else {
                logger.log("Device suspended.  Unable to shut down");
            }
        } else {
            logger.log("Invalid handle for: " + DEV1.toString());
        }
    }
    ...
}
```
修改後：
``` java
public class DeviceController {
    ...
    public void sendShutDown() {
        try {
            tryToShutDown();
        } catch (DeviceShutDownError e) {
            logger.log(e);
        }
    }
    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);
        pauseDevice(handle);
        clearDeviceWorkQueue(handle);
        closeDevice(handle);
    }
    private DeviceHandle getHandle(DeviceID id) {
        ...
        throw new DeviceShutDownError("Invalid handle for: " + id.toString());
        ...
    }
    ...
}
```
它把裝置關閉和錯誤處理區分開來了。

---

## 先寫try-catch-finally區塊
定義catch能先寫好預期會出現什麼例外，寫完測試後，再把剩下的邏輯流程補完。  
try-catch就像開一個transaction，寫finally可以幫助讀者知道這個區塊最終還是要做什麼事。  
先寫測試(預期可能出現什麼例外)：
``` java
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
    sectionStore.retrieveSection("invalid - file");
}
```
實作(這時還沒寫拋出例外)
``` java
public List < RecordedGrip > retrieveSection(String sectionName) {
    // dummy return until we have a real implementation
    return new ArrayList < RecordedGrip > ();
}
```
加入會拋出的例外
``` java
public List < RecordedGrip > retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName)
    } catch (Exception e) {
        throw new StorageException("retrieval error", e);
    }
    return new ArrayList < RecordedGrip > ();
}
```
縮窄catch的例外範圍，之後就能補完流程了(會放在FileInputStream和close()中間)
``` java
public List < RecordedGrip > retrieveSection(String sectionName) {
    try {
        FileInputStream stream = new FileInputStream(sectionName);
        stream.close();
    } catch (FileNotFoundException e) {
        throw new StorageException("retrieval error", e);
        }
        return new ArrayList < RecordedGrip > ();
    }
```

---

## 不要用Checked Exception
定義checked Exception的代價是違反開放閉合原則：若你拋出了checked exception，而方法外面還有三層catch，代表那三層都必須定義且知道這個 checked exception，導致低層級的改變卻影響到了高層級的程式。  
checked Exception可能會用在撰寫關鍵函式庫時（記得必須catch它們）。除此之外在一般情況下，其產生依賴的成本都大於了使用它所帶來的效益。

---

## 在Exception中提供上下文
在拋出的exception裡加入能得知發生什麼錯誤與發生在哪裡的資訊、並足夠能在catch裡面log起來。

---

## 根據呼叫者的需要，來定義例外類別
定義例外類別時最重要的是：它們將會如何被catch?
來看一個糟糕例子
``` java
ACMEPort port = new ACMEPort(12);
try {
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
    logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
    reportPortError(e);
    logger.log("Unlock exception", e);
} catch (GMXError e) {
    reportPortError(e);
    logger.log("Device response exception");
} finally {
    ...
}
```
因為我們catch後做的事幾乎一樣，所以可以粗略的包成通用的例外:
``` java
LocalPort port = new LocalPort(12);
try {
    port.open();
} catch (PortDeviceFailure e) {
    reportError(e);
    logger.log(e.getMessage(), e);
} finally {
    ...
}
```
LocalPort類別把ACMEPort類別和例外處理包了起來：
``` java
public class LocalPort {
    private ACMEPort innerPort;
    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }
    public void open() {
        try {
            innerPort.open();
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }
    ...
}
```
這種wrappers是很有用的。事實上將第三方API包起來是一種很好的實作，包起來的行為即是將對第三方的依賴減小，未來也較容易替換內容、易於測試。  
在上面的例子，我們自定義了一個exception，寫出了更乾淨的程式碼。
通常用單一的例外對應一個區塊的程式碼就夠了，可以藉著例外中夾帶的訊息分辨是哪種錯誤。只有當你要允許某些例外通過、某些catch住時，才使用不一樣的例外類別。

---

## 不要讓錯誤處理打斷你的商業邏輯
目前為止，你包裹了外部api並拋出你自定義的例外、你也寫了能處理終止計算的handler。但有時你並不希望程式中止在那裡。  
這裡有段尷尬的程式碼，業務邏輯跟錯誤處理混雜在一起了。
``` java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch (MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}
```
可以用SPECIAL CASE PATTERN來處理：建立或設定一個物件來處理特殊情況：改變ExpenseReportDAO讓它總是回傳MealExpense物件。  
我們的程式就會變成:
``` java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();
```
其中
``` java
public class PerDiemMealExpenses implements MealExpenses {
    public int getTotal() {
        // return the per diem default
    }
}
```

---

## 不要回傳null
以下是一段不好的程式碼
``` java
public void registerItem(Item item) {
    if (item != null) {
        ItemRegistry registry = peristentStore.getItemRegistry();
        if (registry != null) {
            Item existing = registry.getItem(item.getID());
            if (existing.getBillingPeriod().hasRetailOwner()) {
                existing.register(item);
            }
        }
    }
}
```
當我們回傳null，等於是替自己創造額外的工作。可以考慮改為拋出例外或回傳 SPECIAL CASE object。當呼叫第三方API，其會回傳null時，也可以考慮用這兩種方法wrap它。  
用SPECIAL CASE object的情況，如果回傳的是物件list，那麼可以改成回傳空list:
``` java
List < Employee > employees = getEmployees();
for (Employee e: employees) {
    totalPay += e.getPay();
}
```

---

## 不要把null作為參數傳入方法裡
除非某個API要求你傳null進它的方法，不然不要在任何地方這麼做。  
假設有段程式
``` java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        return (p2.x - p1.x) * 1.5;
    }
}
```
如果你為了檢查null而這樣寫：
``` java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        if (p1 == null || p2 == null) {
            throw InvalidArgumentException(
                "Invalid argument for MetricsCalculator.xProjection");
        }
        return (p2.x - p1.x) * 1.5;
    }
}
```
是有可能比空值例外好，但外面的人還是要處理InvalidArgumentException  
有一個替代方案：用assert
``` java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2) {
        assert p1 != null: "p1 should not be null";
        assert p2 != null: "p2 should not be null";
        return (p2.x - p1.x) * 1.5;
    }
}
```
但還是沒有解決問題，外面傳進null還是會拋出執行期例外。  
大部分的程式語言都沒有很好的方法來處理這種狀況。理性的方法是在引數列預設禁止null，讓你在撰寫程式時能跟隨指示來避免犯錯。