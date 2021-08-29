---
sort: 11
---
# Proxy Pattern 代理人模式
讓某個物件具有一個替身，藉以控制外界對此物件的接觸。被代理的物件可以是遠端的、建立成本高的、或需要安全控管的。

#### 要點
- 和其他的包裝者(wrapper)一樣，代理人會造成你的設計中類別數目增加
- 代理人的結構類似裝飾者，但是目的不同。裝飾者模式為物件加上行為，而代理人則是控制存取。
- 轉接器模式會改變物件的介面，代理人則實踐相同的介面
- 遠端代理人控制存取遠端物件。遠端代理人是「遠端物件的本地代表」。遠端物件是活在不同的JVM記憶體堆積區(heap)中，(在不同的記憶體位址空間執行的遠端物件)。本地代表是可以在本地端調用的物件，其行為會被轉到遠端物件中施行
- 虛擬代理人控制存取建立成本昂貴的資源
- 保護代理人根據權限控制對資源的存取
- Firewall Proxy: 控制網路資源的存取
- Reference Proxy: 當物件被參考到時，進行額外的動作，例如計算被參考次數
- Caching Proxy: 將耗費資源的運算結果暫存起來
- Synchronization Proxy: 提供安全的存取，讓多個執行緒存取同一物件時不會出錯
- Complexity Proxy: 對一群複雜的類別，進行複雜度隱藏以及存取控制。也稱Facade Proxy。和表象模式不一樣，代理人控制存取、表象模式只提供另一組介面
- Copy-On-Write Proxy: 拖延動作的進行，直到客戶真的需要為止才執行

#### 問題
Q: 動態代理人動態在哪裡?  
A: 因為在執行期才將他的類別建立出來。程式開始執行時，還沒有proxy類別

---

## Java實作

---
虛擬代理人，目的在將從網路上取回圖片的動作不要卡住主執行流程

原本的圖片元件:
``` java
class ImageComponent extends JComponent {
    private Icon icon;

    public ImageComponent(Icon icon) {
        this.icon = icon;
    }

    public void setIcon(Icon icon) {
        this.icon = icon;
    }

    public void paintComponent(Graphics g) {
        super.paintComponent(g);
        int w = icon.getIconWidth();
        int h = icon.getIconHeight();
        int x = (800 - w) / 2;
        int y = (600 - h) / 2;
        icon.paintIcon(this, g, x, y);
    }
}
```
代理人，當要渲染圖片時，會開一個執行緒去取得圖片，在此之前就先顯示等待中
``` java
class ImageProxy implements Icon {
    ImageIcon imageIcon;
    URL imageURL;
    Thread retrievalThread;
    boolean retrieving = false;

    public ImageProxy(URL url) {
        imageURL = url;
    }

    public int getIconWidth() {
        if (imageIcon != null) {
            return imageIcon.getIconWidth();
        } else {
            return 800;
        }
    }

    public int getIconHeight() {
        if (imageIcon != null) {
            return imageIcon.getIconHeight();
        } else {
            return 600;
        }
    }

    public void paintIcon(final Component c, Graphics g, int x, int y) {
        if (imageIcon != null) {
            imageIcon.paintIcon(c, g, x, y);
        } else {
            g.drawString("Loading CD cover, please wait...", x + 300, y + 190);
            if (!retrieving) {
                retrieving = true;

                retrievalThread = new Thread(new Runnable() {
                    public void run() {
                        try {
                            imageIcon = new ImageIcon(imageURL, "CD Cover");
                            c.repaint();
                        } catch (Exception e) {
                            e.printStackTrace();
                        }
                    }
                });
                retrievalThread.start();
            }
        }
    }
}
```
執行測試
``` java
public class ImageProxyTestDrive {
    ImageComponent imageComponent;
    JFrame frame = new JFrame("CD Cover Viewer");
    JMenuBar menuBar;
    JMenu menu;
    Hashtable cds = new Hashtable();

    public static void main(String[] args) throws Exception {
        ImageProxyTestDrive testDrive = new ImageProxyTestDrive();
    }

    public ImageProxyTestDrive() throws Exception {
        cds.put("Ambient: Music for Airports", "http://images.amazon.com/images/P/B000003S2K.01.LZZZZZZZ.jpg");
        cds.put("Buddha Bar", "http://images.amazon.com/images/P/B00009XBYK.01.LZZZZZZZ.jpg");
        cds.put("Ima", "http://images.amazon.com/images/P/B000005IRM.01.LZZZZZZZ.jpg");
        cds.put("Karma", "http://images.amazon.com/images/P/B000005DCB.01.LZZZZZZZ.gif");
        cds.put("MCMXC A.D.", "http://images.amazon.com/images/P/B000002URV.01.LZZZZZZZ.jpg");
        cds.put("Northern Exposure", "http://images.amazon.com/images/P/B000003SFN.01.LZZZZZZZ.jpg");
        cds.put("Selected Ambient Works, Vol. 2", "http://images.amazon.com/images/P/B000002MNZ.01.LZZZZZZZ.jpg");

        URL initialURL = new URL((String) cds.get("Selected Ambient Works, Vol. 2"));
        menuBar = new JMenuBar();
        menu = new JMenu("Favorite CDs");
        menuBar.add(menu);
        frame.setJMenuBar(menuBar);

        for (Enumeration e = cds.keys(); e.hasMoreElements();) {
            String name = (String) e.nextElement();
            JMenuItem menuItem = new JMenuItem(name);
            menu.add(menuItem);
            menuItem.addActionListener(new ActionListener() {
                public void actionPerformed(ActionEvent event) {
                    imageComponent.setIcon(new ImageProxy(getCDUrl(event.getActionCommand())));
                    frame.repaint();
                }
            });
        }

        // set up frame and menus

        Icon icon = new ImageProxy(initialURL);
        imageComponent = new ImageComponent(icon);
        frame.getContentPane().add(imageComponent);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(800, 600);
        frame.setVisible(true);

    }

    URL getCDUrl(String name) {
        try {
            return new URL((String) cds.get(name));
        } catch (MalformedURLException e) {
            e.printStackTrace();
            return null;
        }
    }
}
```
