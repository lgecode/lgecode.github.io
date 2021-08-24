---
sort: 9
---
# Iterator Pattern 反覆器模式
讓我們能夠取得一個聚集內的每一個元素，而不需要此聚集將其實踐方式暴露出來。  
反覆器負責在元素之間遊走，而不是由聚集在元素之間遊走。

#### 要點
- 將反覆遊走的動作封裝進一個反覆器物件中。
- 允許存取聚集內的元素，而不需要暴露出內部的結構。
- 使用反覆器時，我們依賴聚集提供正確的反覆器。
- 反覆器提供了共同的介面，當我們使用聚集內的項目時，就可以利用多型機制。

#### 問題
Q: 對於沒有明確排序的聚合怎麼辦?  
A: 反覆器只是取出所有的元素，並不代表元素的先後順序。除非某聚合的文件有特別說明，否則不可以對反覆器所取出的元素大小順序做出假設。

Q: 反覆器可以被設計成向前移動嗎?  
A: 可以，可能要加上hasPrevious()和previous()之類的方法。

Q: 什麼是「內部的」和「外部的」反覆器?  
A: 我們實踐的是外部的：客人藉由呼叫next()取得下一個元素。而內部反覆器是自己控制，你必須將操作手續傳入給反覆器，會較為沒有彈性。

Q: 我們可以讓聚集除了實踐它們內部的聚合，同時也實作相關的操作和反覆的方法嗎?(除了增加方法的個數外還有什麼壞處?)  
A: 這樣會有兩個理由去改變這個類別，聚合改變、反覆的方式改變。違反單一責任原則。  
A: 可是後來java的collection還是加了啊?

---

## Java實作

---

早餐菜單
``` java
public class PancakeHouseMenu implements Menu {
    ArrayList menuItems;

    public PancakeHouseMenu() {
        menuItems = new ArrayList();

        addItem("K&B's Pancake Breakfast",
            "Pancakes with scrambled eggs, and toast",
            true,
            2.99);

        addItem("Regular Pancake Breakfast",
            "Pancakes with fried eggs, sausage",
            false,
            2.99);

        addItem("Blueberry Pancakes",
            "Pancakes made with fresh blueberries, and blueberry syrup",
            true,
            3.49);

        addItem("Waffles",
            "Waffles, with your choice of blueberries or strawberries",
            true,
            3.59);
    }

    public void addItem(String name, String description,
        boolean vegetarian, double price) {
        MenuItem menuItem = new MenuItem(name, description, vegetarian, price);
        menuItems.add(menuItem);
    }

    // 我們不再使用這個會傳出ArrayList固定型別的方法
    public ArrayList getMenuItems() {
        return menuItems;
    }

    // 改為新增一個傳出反覆器的方法(ArrayList本身就有)
    public Iterator createIterator() {
        return menuItems.iterator();
    }
}
```

晚餐菜單
``` java
public class DinerMenu implements Menu {
    static final int MAX_ITEMS = 6;
    int numberOfItems = 0;
    MenuItem[] menuItems;

    public DinerMenu() {
        menuItems = new MenuItem[MAX_ITEMS];

        addItem("Vegetarian BLT",
            "(Fakin') Bacon with lettuce & tomato on whole wheat", true, 2.99);
        addItem("BLT",
            "Bacon with lettuce & tomato on whole wheat", false, 2.99);
        addItem("Soup of the day",
            "Soup of the day, with a side of potato salad", false, 3.29);
        addItem("Hotdog",
            "A hot dog, with saurkraut, relish, onions, topped with cheese",
            false, 3.05);
        addItem("Steamed Veggies and Brown Rice",
            "A medly of steamed vegetables over brown rice", true, 3.99);
        addItem("Pasta",
            "Spaghetti with Marinara Sauce, and a slice of sourdough bread",
            true, 3.89);
    }

    public void addItem(String name, String description,
        boolean vegetarian, double price) {
        MenuItem menuItem = new MenuItem(name, description, vegetarian, price);
        if (numberOfItems >= MAX_ITEMS) {
            System.err.println("Sorry, menu is full!  Can't add item to menu");
        } else {
            menuItems[numberOfItems] = menuItem;
            numberOfItems = numberOfItems + 1;
        }
    }

    // 我們不再使用這個會傳出MenuItem陣列固定型別的方法
    public MenuItem[] getMenuItems() {
        return menuItems;
    }

    // 改為新增一個傳出反覆器的方法(陣列沒有，故需實作一個反覆器類別)
    public Iterator createIterator() {
        return new DinerMenuIterator(menuItems);
    }
}
```

晚餐菜單的反覆器實作類別
``` java
public class DinerMenuIterator implements Iterator {
    MenuItem[] list;
    int position = 0;

    public DinerMenuIterator(MenuItem[] list) {
        this.list = list;
    }

    public Object next() {
        MenuItem menuItem = list[position];
        position = position + 1;
        return menuItem;
    }

    public boolean hasNext() {
        if (position >= list.length || list[position] == null) {
            return false;
        } else {
            return true;
        }
    }

    public void remove() {
        if (position <= 0) {
            throw new IllegalStateException("You can't remove an item until you've done at least one next()");
        }
        if (list[position - 1] != null) {
            for (int i = position - 1; i < (list.length - 1); i++) {
                list[i] = list[i + 1];
            }
            list[list.length - 1] = null;
        }
    }
}
```

女侍
``` java
public class Waitress {
    Menu pancakeHouseMenu;
    Menu dinerMenu;
    Menu cafeMenu;

    public Waitress(Menu pancakeHouseMenu, Menu dinerMenu, Menu cafeMenu) {
        this.pancakeHouseMenu = pancakeHouseMenu;
        this.dinerMenu = dinerMenu;
        this.cafeMenu = cafeMenu;
    }

    public void printMenu() {
        Iterator pancakeIterator = pancakeHouseMenu.createIterator();
        Iterator dinerIterator = dinerMenu.createIterator();
        Iterator cafeIterator = cafeMenu.createIterator();

        System.out.println("MENU\n----\nBREAKFAST");
        printMenu(pancakeIterator);
        System.out.println("\nLUNCH");
        printMenu(dinerIterator);
        System.out.println("\nDINNER");
        printMenu(cafeIterator);
    }

    private void printMenu(Iterator iterator) {
        while (iterator.hasNext()) {
            MenuItem menuItem = (MenuItem) iterator.next();
            System.out.print(menuItem.getName() + ", ");
            System.out.print(menuItem.getPrice() + " -- ");
            System.out.println(menuItem.getDescription());
        }
    }

    public void printVegetarianMenu() {
        System.out.println("\nVEGETARIAN MENU\n---------------");
        printVegetarianMenu(pancakeHouseMenu.createIterator());
        printVegetarianMenu(dinerMenu.createIterator());
        printVegetarianMenu(cafeMenu.createIterator());
    }

    public boolean isItemVegetarian(String name) {
        Iterator pancakeIterator = pancakeHouseMenu.createIterator();
        if (isVegetarian(name, pancakeIterator)) {
            return true;
        }
        Iterator dinerIterator = dinerMenu.createIterator();
        if (isVegetarian(name, dinerIterator)) {
            return true;
        }
        Iterator cafeIterator = cafeMenu.createIterator();
        if (isVegetarian(name, cafeIterator)) {
            return true;
        }
        return false;
    }


    private void printVegetarianMenu(Iterator iterator) {
        while (iterator.hasNext()) {
            MenuItem menuItem = (MenuItem) iterator.next();
            if (menuItem.isVegetarian()) {
                System.out.print(menuItem.getName() + ", ");
                System.out.print(menuItem.getPrice() + " -- ");
                System.out.println(menuItem.getDescription());
            }
        }
    }

    private boolean isVegetarian(String name, Iterator iterator) {
        while (iterator.hasNext()) {
            MenuItem menuItem = (MenuItem) iterator.next();
            if (menuItem.getName().equals(name)) {
                if (menuItem.isVegetarian()) {
                    return true;
                }
            }
        }
        return false;
    }
}
```

``` java

```

``` java

```

``` java

```