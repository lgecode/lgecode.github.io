---
sort: 1
---
# Strategy Pattern 策略模式
定義了演算法家族，個別封裝起來，讓它們之間可以互相替換，此模式讓演算法的變動不會影響到使用演算法的程式。

##### 優點
- 開放閉合原則：輕易就能增加新的策略而不改變原來程式碼。
- 分離性：可以從客戶端程式碼中分離特定實作細節。
- 封裝：用來實作演算法的資料結構是封裝在策略類別裡面，不會影響到使用它的外類別。
- 執行期轉換：可在執行期間動態改變使用的策略。


##### 缺點
- 產生額外物件：產生多個策略會有多的實體要維護。
- 使用者意識：使用者必須知道每個策略間的差異與實作內容以便選擇最適合的。
- 增加複雜度：若最後只使用一兩個策略，則浪費了建構策略模式實作的成本。


##### 使用情景
- 很多相似類別：當有很多不同實作方式的類別時。
- 隔離：當要把業務邏輯與演算法實作分開時。
- [商品套用各種折扣](https://www.geeksforgeeks.org/strategy-method-python-design-patterns/)
- 遊戲內使用不同武器攻擊敵人

---

## 問題思考

有一套鴨子類別，各種類鴨子都繼承了父類別鴨子  
Q :要讓鴨子會飛，這時程式要怎麼修改呢？  

A1:在父類別加上飛行方法fly()  
X :並不是每種鴨子都會飛，如果每個不會飛的子類別鴨子都要修改fly()，以後會很難維護  

A2:將飛行改為介面，每個要飛的鴨子要去實作飛行  
X :每次鴨子都要實作重複的飛行方法，程式碼無法再利用，以後要修改也麻煩  

## 解法

- 將會變動的飛行獨立出來，宣告飛行行為這個屬性(抽象類別或介面)，如此可使用多型，在執行時依據不同的實作而有不同的行為。
- 當需要飛的行為時，就產生飛的類別實作 FlyBehavier，需要用翅膀飛的鴨子就可以用 FlyWithWings，而不會飛的鴨子就用 FlyNoWay。
- 如此可以讓飛的行為被其他鴨子物件再利用，因為這些行為跟鴨子類別沒關係了。就算我們再新增其他飛的行為，也不會影響到鴨子類別。也可以在程式執行時期動態改變飛行行為。

## Python實作

``` python
# 飛行行為

from abc import ABC, abstractmethod

class FlyBehavior(ABC):
    
    @abstractmethod
    def fly(self):
        pass


class FlyWithWings(FlyBehavior):

    def fly(self):
        print("I'm flying!!")


class FlyNoWay(FlyBehavior):

    def fly(self):
        print("I can't fly")
```
``` python
# 鴨子類別

class Duck():

    def __init__(self, fly_behavior:FlyBehavior):
        self.fly_behavior = fly_behavior

    def perform_fly(self):
        self.fly_behavior.fly()


class MallardDuck(Duck):

    def __init__(self):
        super().__init__(FlyWithWings())
        

class ModelDuck(Duck):

    def __init__(self):
        super().__init__(FlyNoWay())
```
``` python
if __name__ == "__main__":

    mallard = MallardDuck()
    model = ModelDuck()

    mallard.perform_fly()
    model.perform_fly()

    model.fly_behavior = FlyWithWings()
    model.perform_fly()
```