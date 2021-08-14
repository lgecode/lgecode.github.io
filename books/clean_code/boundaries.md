---
sort: 8
---
# Boundaries
---

## 當使用第三方程式碼
提供者想要顧及到廣泛的應用情境，但使用者想要滿足他指定的需求，這個衝突可能會造成點問題。  
拿java.util.Map舉例，如果將Map物件在程式間傳遞，任何拿到的人都可以clear()，或是存入你並不希望這個Map放入的物件型態。  
你可能會這樣寫：
``` java
Map sensors = new HashMap();
```
拿到的人就必須這樣取值
``` java
Sensor s = (Sensor)sensors.get(sensorId );
```
這造成了客戶端必須有"從中取得物件並轉成正確型態"的步驟。這並不是clean code，而且這程式碼也不能很好的帶有說明性。  
可以用泛型很好的解決：  
``` java
Map<Sensor> sensors = new HashMap<Sensor>();
...
Sensor s = sensors.get(sensorId);
```
但這沒有解決問題。如果今天不用Map型別了，程式裡的很多地方都要改寫。  
乾淨的作法是，讓使用者不用顧慮型別，讓它變成實作細節：
``` java
public class Sensors {
    private Map sensors = new HashMap();

    public Sensor getById(String id) {
        return (Sensor) sensors.get(id);
    }
    //snip
}
```
interface的boundary(Map)被隱藏了。型別轉換管理都被藏在Sensors class裡。這個interface也更容易被app使用，更容易了解且不會被誤用。  
並不是建議你全部都要像這樣封裝，而是建議不要傳遞maps，不要在公開API中接收或傳遞這種類型。

---

## Learning Tests
我們可以藉由撰寫測試來同時探索第三方API如何使用，這種測試稱為learning tests。在其中，我們呼叫第三方API、做控制實驗來了解我們想從它們獲得什麼。
當第三方API有更新時，這些測試也能幫助了解是否有不一樣的結果。  
無論你有沒有要用learning tests來學習第三方API，都該有一組測試它的程式，若沒有這樣的邊界測試，可能會導致一直使用舊版本的第三方API(因為怕更新就出錯了)。

---

## 使用還不存在的程式碼


``` java
```

``` java
```
