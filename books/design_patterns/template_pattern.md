---
sort: 8
---
# Template Pattern 樣板方法模式
將一個演算法的骨架定義在一個方法中，而演算法本身會用到的一些方法，則是定義在次類別中。樣板方法讓次類別在不改變演算法架構的情況下，重新定義演算法中的某些步驟。

#### 要點
- 樣板方法讓我們達到程式碼再用性的目的。
- 樣板方法的抽象類別可以定義具象方法、抽象方法、以及掛勾。
- 掛勾是一種方法，它不做事，或只做預設的事，次類別可以選擇要不要去推翻它。
- 好萊塢守則告訴我們，將決策權放在高階模組中，以便決定如何、以及何時呼叫低階模組。
- 策略模式以及樣板方法模式都是用來封裝演算法，一個用合成，一個用繼承。
- 工廠方法是樣板方法的一種特殊版本。

#### 問題
Q: 如何知道何時該用抽象方法，何時使用掛勾?  
A: 當你的次類別「必須」提供演算法用在某個方法或步驟的實踐時，就使用抽象方法。如果演算法的這個部分是選擇性的，就用掛勾。

Q: 掛鉤的真正目的是什麼?  
A: 除了讓次類別實踐演算法中選擇性的部分，或掛勾對次類別並不重要時，可以置之不理。另一個使用時機是，讓次類別能夠有機會對於樣板方法中某個即將發生的(或剛剛發生的)步驟做出反應。讓刺費別有能力對其抽象的超類別做出一些決定。

A: 不要讓演算法內的步驟切割得太細，但若步驟太少會比較沒有彈性。適時地將選擇性的步驟實踐成掛勾，以減輕次類別的負荷。

Q: 好萊塢守則跟依賴反轉守則差別為何?  
A: 依賴反轉教我們避免使用具象類別，多使用抽象，避免相依性。好萊塢守則是用在建立框架或元件，好讓低階的元件能夠被掛勾進入系統中，且又不會讓高階的元件太依賴低階元件。

Q: 也就是說低階的元件不可以呼叫高階袁建中的方法?  
A: 也不盡然。事實上低階的元件在結束時，常常會呼叫從超類別中繼承來的方法。我們所要做的事，避免讓高階和低階之間有明顯的相互依賴。

---

## Java實作

---

``` java
public abstract class CaffeineBeverageWithHook {

    void prepareRecipe() {
        boilWater();
        brew();
        pourInCup();
        if (customerWantsCondiments()) {
            addCondiments();
        }
    }

    abstract void brew();

    abstract void addCondiments();

    void boilWater() {
        System.out.println("Boiling water");
    }

    void pourInCup() {
        System.out.println("Pouring into cup");
    }

    boolean customerWantsCondiments() {
        return true;
    }
}
```

``` java
public class CoffeeWithHook extends CaffeineBeverageWithHook {

    public void brew() {
        System.out.println("Dripping Coffee through filter");
    }

    public void addCondiments() {
        System.out.println("Adding Sugar and Milk");
    }

    public boolean customerWantsCondiments() {

        String answer = getUserInput();

        if (answer.toLowerCase().startsWith("y")) {
            return true;
        } else {
            return false;
        }
    }

    private String getUserInput() {
        String answer = null;

        System.out.print("Would you like milk and sugar with your coffee (y/n)? ");

        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
        try {
            answer = in .readLine();
        } catch (IOException ioe) {
            System.err.println("IO error trying to read your answer");
        }
        if (answer == null) {
            return "no";
        }
        return answer;
    }
}
```
``` java
public class TeaWithHook extends CaffeineBeverageWithHook {

    public void brew() {
        System.out.println("Steeping the tea");
    }

    public void addCondiments() {
        System.out.println("Adding Lemon");
    }

    public boolean customerWantsCondiments() {

        String answer = getUserInput();

        if (answer.toLowerCase().startsWith("y")) {
            return true;
        } else {
            return false;
        }
    }

    private String getUserInput() {
        // get the user's response
        String answer = null;

        System.out.print("Would you like lemon with your tea (y/n)? ");

        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));
        try {
            answer = in .readLine();
        } catch (IOException ioe) {
            System.err.println("IO error trying to read your answer");
        }
        if (answer == null) {
            return "no";
        }
        return answer;
    }
}
```
