---
sort: 6
---
# Command Pattern 命令模式
將「請求」封裝成物件，以便使用不同的請求、佇列、或者日誌，參數化其他物件。命令模式也支援可復原的作業。

將動作和接收者包進命令物件中，只暴露出一個execute()方法。調用者不知道接收者是誰，做了什麼動作，只知道呼叫execute()就能完成任務。



##### 優點
- 像全域變數一樣方便，又沒有全域變數的缺點
- 不用在啟動時就建立物件，可以拖延實體化

##### 缺點
- 多執行緒時可能發生問題，要小心選擇實作方式
- 不適合被繼承
- 同時做了兩件事，違背Single responsibility Principle
- unit test會很困難

##### 使用情境
- 控制全域變數
- 開發人員日常使用
- 執行緒池、快取區、對話盒、log、處理偏好設定與登錄的物件、登入系統的物件、和驅動程式溝通的物件
- 常與工廠模式一起用

---

## Java實作
1. 將建構式設成私有
2. 建立一個私有類別變數儲存該獨體
3. 建立一個公開類別方法取得該獨體

``` java
/* 在公開取得方法加上synchronized
但其實只有第一次呼叫時需要同步，之後每次呼叫都會拖慢效能 */

public class Singleton {

    private static Singleton uniqueInstance;

    // other useful instance variables here

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();  
        }
        return uniqueInstance;
    }

    // other useful methods here
}
```
``` java
/* 在載入類別初始化時直接建立該獨體
不使用拖延實體化的方法 */

public class Singleton {

    private static Singleton uniqueInstance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return uniqueInstance;
    }
}
```
``` java
/* 雙重檢查上鎖，先檢查是否已存在實體，若沒有才同步 */

public class Singleton {

    private volatile static Singleton uniqueInstance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```
---
## Python實作
推薦
[使用metaclass](https://stackoverflow.com/questions/6760685/creating-a-singleton-in-python)
- 不需要繼承
- 不用擔心__new__被子類別覆寫
- 類別不需要意識到它是singleton

``` python
class Singleton(type):
    __instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls.__instances:
            cls.__instances[cls] = super(Singleton, cls).__call__(*args, **kwargs)
        # 若想要每次都執行__init__就加以下這個else
        else:
            cls.__instances[cls].__init__(*args, **kwargs)
        return cls.__instances[cls]

class Logger(metaclass=Singleton):
    pass
```
``` python
# 注意 python3已經不適用這樣寫
class Logger(object):
    __metaclass__ = Singleton
```

直接用模組
``` python
# Modules are imported only once, everything else is overthinking.
# Don't use singletons and try not to use globals.

class Foo(object):
    pass

some_global_variable = Foo()
```

特殊作法：  
[共享狀態，而非限定一個實體](https://www.geeksforgeeks.org/singleton-method-python-design-patterns/?ref=rp)

---

其他不推薦的實作方式  

``` python
# 修改__new__()這個類別方法
# 壞處
#   若有多處需要用singleton，同樣程式碼要寫很多遍

class Singleton:

    __instance = None 

    def __new__(cls, *args, **kwargs): 
        if not cls.__instance: 
            cls.__instance = super().__new__(cls) 
        return cls.__instance 
```

``` python
# 類似上面，但改為繼承
# 壞處
#   __new__可能被子類別覆寫
#   繼承需寫在最左邊

class Singleton:

    __instance = None

    def __new__(cls, *args, **kwargs):
        if not cls.__instance:
            cls.__instance = object.__new__(cls)
        return cls.__instance

class MyClass(Singleton):
    pass
```

``` python
# Decorator
# 好處
#   比繼承更直覺
# 壞處
#   出來的是function而不是class，你不能呼叫其class方法

def singleton(class_):

    instances = {}

    def getinstance(*args, **kwargs):
        if class_ not in instances:
            instances[class_] = class_(*args, **kwargs)
        return instances[class_]

    return getinstance


@singleton
class MyClass(BaseClass):
    pass
```
