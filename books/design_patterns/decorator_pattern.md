---
sort: 3
---
# Decorator Pattern 裝飾者模式
動態地將責任加諸於物件上。若要擴充功能，裝飾者提供了比繼承更有彈性的選擇。

#### 要點
- 裝飾者模式意味著一群裝飾者類別，這些類別用來封裝具象元件。
- 每個裝飾者「有一個」(即封裝一個)實體變數，參考到某個component。
- 利用繼承達到「型態相符」，而非利用繼承取得相同的「行為」
- 裝飾者可以加上新的方法。通常，新的行為會利用舊的行為當作基礎，在舊的行為前面或(與)後面作一些變化。
- 將裝飾者和元件合成時，就是加入新的行為。所得到的新行為，並非繼承自超類別，而是由合成物件得來的。
- 如果依賴繼承，那麼類別的行為在編譯時就靜態的決定了，行為不是來自超類別，就是次類別override後的版本。反之，利用合成，可以把裝飾者混合著用，而且是在執行期動態地進行。
- 可以在任何時候，實踐新的裝飾者增加新的行為。如果依賴繼承，每當需要新行為時，還得修改既有的程式碼。
- 你可以用無數個裝飾者包裝一個元件。
- 使用者不需要知道裝飾者的存在，除非程式中有依賴特定的具象型態(這時一旦導入裝飾者，就會出bug了)。
- 裝飾者會導致程式中出現許多小類別，如果過度使用裝飾者模式，會讓程式變得很複雜。
- 開放關閉原則: 沒有閒工夫把整個系統都這麼設計(就算做得到，也可能只是一種浪費)。你需要專注在系統設計中，找出最有可能改變的地方，才採用開放關閉原則，而不是所有地方都用。

---

## Java實作

「元件」介面或抽象類別
``` java
public abstract class Beverage {
    String description = "Unknown Beverage";

    public String getDescription() {
        return description;
    }

    public abstract double cost();
}
```

元件實作
``` java
public class DarkRoast extends Beverage {
    public DarkRoast() {
        description = "Dark Roast Coffee";
    }

    public double cost() {
        return .99;
    }
}
```

裝飾者是一個元件
``` java
public abstract class CondimentDecorator extends Beverage {
    public abstract String getDescription();
}
```

裝飾者實作
``` java
public class Mocha extends CondimentDecorator {
    Beverage beverage;

    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", Mocha";
    }

    public double cost() {
        return .20 + beverage.cost();
    }
}
```

``` java
public class Whip extends CondimentDecorator {
    Beverage beverage;

    public Whip(Beverage beverage) {
        this.beverage = beverage;
    }

    public String getDescription() {
        return beverage.getDescription() + ", Whip";
    }

    public double cost() {
        return .10 + beverage.cost();
    }
}
```

執行
``` java
public class StarbuzzCoffee {

    public static void main(String args[]) {
        Beverage beverage = new Espresso();
        System.out.println(beverage.getDescription() +
            " $" + beverage.cost());

        Beverage beverage2 = new DarkRoast();
        beverage2 = new Mocha(beverage2);
        beverage2 = new Mocha(beverage2);
        beverage2 = new Whip(beverage2);
        System.out.println(beverage2.getDescription() +
            " $" + beverage2.cost());
    }
}
```
