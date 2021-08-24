---
sort: 9
---
# Composite Pattern 合成模式
當有數個物件的聚合，且它們彼此之間有「部分/整體」的關係，並想要用一致的方式對待其中的每個元素時使用。  
合成樹狀結構，呈現「部分/整體」的階層關係。能讓客戶程式碼以一致的方式處理個別物件，以及合成的物件。也就是說在大多數情況下，我們可以忽略合成以及個別物件之間的差異。

#### 要點
- 合成模式提供一個結構，可同時包容個別物件以及合成物件。
- 合成模式允許客戶對個別物件以及合成物件一視同仁。
- 合成架構內的物件稱為元件，元件可以是合成節點(node)，也可以是葉節點(leaf)。
- 合成模式捨棄單一責任的設計守則，換取透明性(transparency)。讓元件介面同時包含管理子節點的操作以及葉節點的操作，客戶就能對他們一視同仁，客戶看不到一個元素到底是子節點還是葉節點。
- 為了保有透明性，合成內所有物件都必須實踐相同的介面，否則客戶就必須擔心哪個物件是用哪個介面，這就失去了合成模式的意義。
- 這是設計上的抉擇。當然也可以採用另一種：將責任切割開來放在不同介面中，設計上比較安全，但也因此失去了透明性，客戶必須用條件式及instanceof處理不同類型的節點。
- 有許多需要取捨的地方。根據你的需求在透明性和安全性之間取得適當的平衡。
- 某些時候這是觀點的問題。例如add(), remove(), getChild()出現在葉節點似乎不恰當，但是換個角度來看，可以把葉節點視為沒有子類別的節點。

#### 問題
Q: 元件、合成、樹?  
A: 合成包含元件。元件有兩種：合成與樹葉元素。這是一個遞迴式的定義。合成持有一群孩子，這些孩子可以是別的合成或樹葉元素。當用這種方式組織資料時，會得到由上而下的樹狀結構，最頂端的根部是一個合成，而合成的分支逐漸往下延伸，直到樹葉為止。

Q: 有些物件具備一些沒有意義的方法，該怎麼辦?  
A: 可以讓這些方法不做事，或傳回null或false，看哪種在實作中比較合乎邏輯。或是丟出例外讓客戶去處理。

Q: 孩子會不會需要知道父親是誰?  
A: 元件可以用指標記錄他的父親，好讓客戶在其間遊走時更容易。而且若想從樹狀中刪除某個孩子，就可以用這參考直接告訴父親去刪除他。

---

## Java實作

---

元件類別(可以被node或leaf繼承)
``` java
public abstract class MenuComponent {

    // node用的方法
    public void add(MenuComponent menuComponent) {
        throw new UnsupportedOperationException();
    }
    public void remove(MenuComponent menuComponent) {
        throw new UnsupportedOperationException();
    }
    public MenuComponent getChild(int i) {
        throw new UnsupportedOperationException();
    }

    // leaf用的方法
    public double getPrice() {
        throw new UnsupportedOperationException();
    }
    public boolean isVegetarian() {
        throw new UnsupportedOperationException();
    }

    // node, leaf 都會用到的方法
    public String getName() {
        throw new UnsupportedOperationException();
    }
    public String getDescription() {
        throw new UnsupportedOperationException();
    }
    public void print() {
        throw new UnsupportedOperationException();
    }
}
```

node
``` java
public class Menu extends MenuComponent {
    ArrayList menuComponents = new ArrayList();
    String name;
    String description;

    public Menu(String name, String description) {
        this.name = name;
        this.description = description;
    }

    public void add(MenuComponent menuComponent) {
        menuComponents.add(menuComponent);
    }

    public void remove(MenuComponent menuComponent) {
        menuComponents.remove(menuComponent);
    }

    public MenuComponent getChild(int i) {
        return (MenuComponent) menuComponents.get(i);
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    public void print() {
        System.out.print("\n" + getName());
        System.out.println(", " + getDescription());
        System.out.println("---------------------");

        Iterator iterator = menuComponents.iterator();
        while (iterator.hasNext()) {
            MenuComponent menuComponent =
                (MenuComponent) iterator.next();
            menuComponent.print();
        }
    }
}
```

leaf
``` java
public class MenuItem extends MenuComponent {
    String name;
    String description;
    boolean vegetarian;
    double price;

    public MenuItem(String name,
        String description,
        boolean vegetarian,
        double price) {
        this.name = name;
        this.description = description;
        this.vegetarian = vegetarian;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    public double getPrice() {
        return price;
    }

    public boolean isVegetarian() {
        return vegetarian;
    }

    public void print() {
        System.out.print("  " + getName());
        if (isVegetarian()) {
            System.out.print("(v)");
        }
        System.out.println(", " + getPrice());
        System.out.println("     -- " + getDescription());
    }
}
```

女侍
``` java
public class Waitress {
    MenuComponent allMenus;

    public Waitress(MenuComponent allMenus) {
        this.allMenus = allMenus;
    }

    public void printMenu() {
        allMenus.print();
    }
}
```

執行
``` java
public class MenuTestDrive {
    public static void main(String args[]) {
        MenuComponent pancakeHouseMenu =
            new Menu("PANCAKE HOUSE MENU", "Breakfast");
        MenuComponent dinerMenu =
            new Menu("DINER MENU", "Lunch");
        MenuComponent cafeMenu =
            new Menu("CAFE MENU", "Dinner");
        MenuComponent dessertMenu =
            new Menu("DESSERT MENU", "Dessert of course!");
        MenuComponent coffeeMenu = new Menu("COFFEE MENU", "Stuff to go with your afternoon coffee");

        MenuComponent allMenus = new Menu("ALL MENUS", "All menus combined");

        allMenus.add(pancakeHouseMenu);
        allMenus.add(dinerMenu);
        allMenus.add(cafeMenu);

        pancakeHouseMenu.add(new MenuItem(
            "K&B's Pancake Breakfast",
            "Pancakes with scrambled eggs, and toast",
            true,
            2.99));
        pancakeHouseMenu.add(new MenuItem(
            "Regular Pancake Breakfast",
            "Pancakes with fried eggs, sausage",
            false,
            2.99));


        dinerMenu.add(new MenuItem(
            "Vegetarian BLT",
            "(Fakin') Bacon with lettuce & tomato on whole wheat",
            true,
            2.99));
        dinerMenu.add(new MenuItem(
            "BLT",
            "Bacon with lettuce & tomato on whole wheat",
            false,
            2.99));
        dinerMenu.add(new MenuItem(
            "Soup of the day",
            "A bowl of the soup of the day, with a side of potato salad",
            false,
            3.29));

        dinerMenu.add(dessertMenu);

        dessertMenu.add(new MenuItem(
            "Apple Pie",
            "Apple pie with a flakey crust, topped with vanilla icecream",
            true,
            1.59));

        dessertMenu.add(new MenuItem(
            "Cheesecake",
            "Creamy New York cheesecake, with a chocolate graham crust",
            true,
            1.99));
        dessertMenu.add(new MenuItem(
            "Sorbet",
            "A scoop of raspberry and a scoop of lime",
            true,
            1.89));

        cafeMenu.add(new MenuItem(
            "Veggie Burger and Air Fries",
            "Veggie burger on a whole wheat bun, lettuce, tomato, and fries",
            true,
            3.99));
        cafeMenu.add(new MenuItem(
            "Soup of the day",
            "A cup of the soup of the day, with a side salad",
            false,
            3.69));
        cafeMenu.add(new MenuItem(
            "Burrito",
            "A large burrito, with whole pinto beans, salsa, guacamole",
            true,
            4.29));

        cafeMenu.add(coffeeMenu);

        coffeeMenu.add(new MenuItem(
            "Coffee Cake",
            "Crumbly cake topped with cinnamon and walnuts",
            true,
            1.59));
        coffeeMenu.add(new MenuItem(
            "Bagel",
            "Flavors include sesame, poppyseed, cinnamon raisin, pumpkin",
            false,
            0.69));
        coffeeMenu.add(new MenuItem(
            "Biscotti",
            "Three almond or hazelnut biscotti cookies",
            true,
            0.89));

        Waitress waitress = new Waitress(allMenus);

        waitress.printMenu();
    }
}
```

---

修正print方法，讓合成節點的print可以印出其所有子節點的print  

首先在MenuComponent內加入createIterator()
``` java
public abstract class MenuComponent {

    // ...省略 (其他部分不需要修改)

    public abstract Iterator createIterator();

}
```
node類別
``` java
public class Menu extends MenuComponent {

    // ...省略 (其他部分不需要修改)

    public Iterator createIterator() {
        return new CompositeIterator(menuComponents.iterator());
    } // 這個新的反覆器知道如何在合成節點內進行反覆。


    public void print() {
        System.out.print("\n" + getName());
        System.out.println(", " + getDescription());
        System.out.println("---------------------");

        // 改成呼叫子類別的反覆器的print
        // 若遇到子類別是另一個菜單物件，呼叫它的print()將會造成另一個反覆，依此類推
        Iterator iterator = menuComponents.iterator();
        while (iterator.hasNext()) {
            MenuComponent menuComponent =
                (MenuComponent) iterator.next();
            menuComponent.print();
        }
    }
}
```
leaf類別
``` java
public class MenuItem extends MenuComponent {

    // ...省略 (其他部分不需要修改)

    public Iterator createIterator() {
        return new NullIterator();
    }
}
```
新的合成反覆器(使用遞迴)  
他負責反覆地在所有項目內遊走，確保所有子菜單(以及子子菜單...)都不會漏掉
``` java
// 實踐java.utill.Iterator介面
public class CompositeIterator implements Iterator {
    Stack stack = new Stack();

    // 將我們所欲造訪的最上層合成節點的反覆器，丟進堆疊中
    public CompositeIterator(Iterator iterator) {
        stack.push(iterator);
    }

    public Object next() {
        if (hasNext()) {
            // 若還有下一個元素，就從堆疊中取出目前的反覆器，取得他的下一個元素
            Iterator iterator = (Iterator) stack.peek();
            MenuComponent component = (MenuComponent) iterator.next();

            if (component instanceof Menu) {
                // 若元素是一個菜單，等於是有了另一個合成節點，先將他丟進堆疊，稍後會再進一步的深入他
                stack.push(component.createIterator());
            }
            // 不管是不是菜單，都將該元素當回傳值
            return component;
        } else {
            return null;
        }
    }

    public boolean hasNext() {
        if (stack.empty()) {
            return false;
        } else {
            Iterator iterator = (Iterator) stack.peek();
            if (!iterator.hasNext()) {
                stack.pop();
                return hasNext();
            } else {
                return true;
            }
        }
    }

    public void remove() {
        throw new UnsupportedOperationException();
    }
}
```
原本的print()寫法，是用反覆器走訪元件內的每個項目，是在「內部」自行處理反覆的動作。  
但是這個新的反覆器是實踐「外部」的反覆器，有許多需要追蹤的事情。外部反覆器必須紀錄他的位置，以便為你的客戶呼叫hasNext()和next()。所以才會有這樣使用堆疊和遞迴的實作。

#### 空反覆器
若菜單項目內沒什麼可以反覆遊走的，應該怎麼做呢?
1. 傳回null：但這樣客戶的程式碼就需要條件是，判斷是否傳回了null。
2. 傳回一個反覆器，其hasNext()永遠傳回false。我們等於是建立了一個反覆器，其作用是「沒作用」。

``` java
public class NullIterator implements Iterator {

    public Object next() {
        return null;
    }

    public boolean hasNext() {
        return false;
    }

    public void remove() {
        throw new UnsupportedOperationException();
    }
}
```

女侍
``` java
public class Waitress {
    MenuComponent allMenus;

    public Waitress(MenuComponent allMenus) {
        this.allMenus = allMenus;
    }

    public void printMenu() {
        allMenus.print();
    }

    public void printVegetarianMenu() {
        Iterator iterator = allMenus.createIterator();

        System.out.println("\nVEGETARIAN MENU\n----");
        while (iterator.hasNext()) {
            MenuComponent menuComponent =
                (MenuComponent) iterator.next();
            try {
                if (menuComponent.isVegetarian()) {
                    menuComponent.print();
                }
            } catch (UnsupportedOperationException e) {}
        }
    }
}
```
try/catch 是用來做錯誤處理的，而不是作為程式邏輯之用。但還能怎麼做呢? 可以在執行期檢查菜單元件的型態，確定是菜單項目，才呼叫isVegetarian()方法，但這樣就失去了透明性，因為會無法統一處理菜單以及選單項目。  
我們也可以改寫菜單的isVegetarian()，讓他總是回傳false。但這會造成意義上的扭曲，變成:菜單不是素食的。  
我們想要傳達isVegetarian()是菜單沒有支援的方法，因此才選用try/catch。