---
sort: 6
---
# Command Pattern 命令模式
將「請求」封裝成物件，將動作和接收者包進命令物件中，只暴露出一個execute()方法，調用者不知道接收者是誰、做了什麼動作，只知道呼叫execute()就能完成任務。  
可使用不同的請求、佇列、或日誌，參數化其他物件、也支援可復原的作業。

#### 要點
- 命令模式將發出請求的物件和執行請求的物件之間鬆綁。
- 在被鬆綁的兩者之間，是藉由命令物件進行溝通的。命令物件封裝了接收者以及一個或一組動作。
- 調用者藉由呼叫命令物件的execute()發出請求，這會使得接收者的動作被調用。

- 調用者可以接收命令當作參數，甚至在執行期動態地進行。
- 命令可以支援復原，做法是實踐一個undo()方法，來回到execute()被執行前的狀態。
- 巨集命令是普通命令的一種簡單延伸，允許調用多個方法。巨集方法也可以支援復原。
- 實務上，很常見使用「聰明」命令物件，也就是直接執行了請求，而不是將工作交由接收者進行。
- 命令也可以用來實踐登錄日誌裡的交易機制。

#### 問題
Q: 接收者一定有必要存在嗎? 為何命令物件不直接實踐execute()方法的細節?  
A: 一般來說，我們盡量設計「笨蛋」命令物件，它只懂得調用一個接收者的一個行為。然而，有許多「聰明」命令物件會實踐許多邏輯、直接完成一個請求。當然你可以設計聰明的命令物件，只是這樣一來，調用者和接收者之間的鬆綁程度是比不上「笨蛋」命令物件的，而且如此一來，你也不能把接收者當作參數傳給命令。

Q: 如何實踐多層次的復原操作，按下多次復原按鈕，來復原到很早很早以前的狀態?  
A: 不只紀錄最後一個被執行的命令，而是使用一個堆疊記錄操作過程的每一個命令。按下復原時，從堆疊取出最上層的命令，呼叫它的undo()方法。

Q: 可以在PartyCommand的execute()裡面呼叫其他的命令，來實踐集合形式嗎?  
A: 這樣就等於寫死在PartyCommand中，捨棄了用集合的優點。利用巨集命令會更有彈性、更優雅。

#### 情境
1. 餐廳點餐：顧客是Client，訂單是Command，takeOrder()=SetCommand()，女侍是Invoker，OrderUp()=execute()，廚師是Receiver
2. 遙控器：使用者是Client，遙控器是Invoker，電燈是Receiver

#### 空物件
當需要一個沒有任何意義值時，空物件就很有用，也可以將處理null的責任轉移給空物件。許多設計模式都會看到空物件的使用，甚至有時空物件也被視為是一種設計模式。

---

## Java實作

---

``` java
public interface Command {
    public void execute();
}
```
``` java
public class Light {

    public Light() {}

    public void on() {
        System.out.println("Light is on");
    }

    public void off() {
        System.out.println("Light is off");
    }
}
```
``` java
public class LightOnCommand implements Command {
    Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    public void execute() {
        light.on();
    }
}
```
``` java
public class SimpleRemoteControl {
    Command slot;

    public SimpleRemoteControl() {}

    public void setCommand(Command command) {
        slot = command;
    }

    public void buttonWasPressed() {
        slot.execute();
    }
}
```
``` java
public class RemoteControlTest {
    public static void main(String[] args) {
        SimpleRemoteControl remote = new SimpleRemoteControl();
        Light light = new Light();
        GarageDoor garageDoor = new GarageDoor();
        LightOnCommand lightOn = new LightOnCommand(light);
        GarageDoorOpenCommand garageOpen =
            new GarageDoorOpenCommand(garageDoor);

        remote.setCommand(lightOn);
        remote.buttonWasPressed();
        remote.setCommand(garageOpen);
        remote.buttonWasPressed();
    }
}
```

---

### 巨集

``` java
public class MacroCommand implements Command {
    Command[] commands;

    public MacroCommand(Command[] commands) {
        this.commands = commands;
    }

    public void execute() {
        for (int i = 0; i < commands.length; i++) {
            commands[i].execute();
        }
    }

    /**
     * NOTE:  these commands have to be done backwards to ensure proper undo functionality
     */
    public void undo() {
        for (int i = commands.length - 1; i >= 0; i--) {
            commands[i].undo();
        }
    }
}
```
``` java
public class RemoteLoader {

    public static void main(String[] args) {

        RemoteControl remoteControl = new RemoteControl();

        Light light = new Light("Living Room");
        TV tv = new TV("Living Room");
        Stereo stereo = new Stereo("Living Room");
        Hottub hottub = new Hottub();

        LightOnCommand lightOn = new LightOnCommand(light);
        StereoOnCommand stereoOn = new StereoOnCommand(stereo);
        TVOnCommand tvOn = new TVOnCommand(tv);
        HottubOnCommand hottubOn = new HottubOnCommand(hottub);
        LightOffCommand lightOff = new LightOffCommand(light);
        StereoOffCommand stereoOff = new StereoOffCommand(stereo);
        TVOffCommand tvOff = new TVOffCommand(tv);
        HottubOffCommand hottubOff = new HottubOffCommand(hottub);

        Command[] partyOn = {
            lightOn,
            stereoOn,
            tvOn,
            hottubOn
        };
        Command[] partyOff = {
            lightOff,
            stereoOff,
            tvOff,
            hottubOff
        };

        MacroCommand partyOnMacro = new MacroCommand(partyOn);
        MacroCommand partyOffMacro = new MacroCommand(partyOff);

        remoteControl.setCommand(0, partyOnMacro, partyOffMacro);

        System.out.println(remoteControl);
        System.out.println("--- Pushing Macro On---");
        remoteControl.onButtonWasPushed(0);
        System.out.println("--- Pushing Macro Off---");
        remoteControl.offButtonWasPushed(0);
    }
}
```

---

### undo()的細節

``` java
public class NoCommand implements Command {
    public void execute() {}
    public void undo() {}
}
```
``` java
//
// This is the invoker
//
public class RemoteControlWithUndo {
    Command[] onCommands;
    Command[] offCommands;
    Command undoCommand;

    public RemoteControlWithUndo() {
        onCommands = new Command[7];
        offCommands = new Command[7];

        Command noCommand = new NoCommand();
        for (int i = 0; i < 7; i++) {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
        undoCommand = noCommand;
    }

    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }

    public void onButtonWasPushed(int slot) {
        onCommands[slot].execute();
        undoCommand = onCommands[slot];
    }

    public void offButtonWasPushed(int slot) {
        offCommands[slot].execute();
        undoCommand = offCommands[slot];
    }

    public void undoButtonWasPushed() {
        undoCommand.undo();
    }
}
```
``` java
public class CeilingFanHighCommand implements Command {
    CeilingFan ceilingFan;
    int prevSpeed;

    public CeilingFanHighCommand(CeilingFan ceilingFan) {
        this.ceilingFan = ceilingFan;
    }

    public void execute() {
        prevSpeed = ceilingFan.getSpeed();
        ceilingFan.high();
    }

    public void undo() {
        if (prevSpeed == CeilingFan.HIGH) {
            ceilingFan.high();
        } else if (prevSpeed == CeilingFan.MEDIUM) {
            ceilingFan.medium();
        } else if (prevSpeed == CeilingFan.LOW) {
            ceilingFan.low();
        } else if (prevSpeed == CeilingFan.OFF) {
            ceilingFan.off();
        }
    }
}
```
