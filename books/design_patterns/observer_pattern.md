---
sort: 2
---
# Observer Pattern 觀察者模式
定義了物件之間的一對多關係，當可觀察者改變狀態，會通知觀察者並自動更新。  
觀察者也可以接收被觀察者物件，主動拉取資料。

#### 要點
- 可觀察者更新觀察者的方式，是透過一個共同的介面。
- 採用鬆綁方式結合。因為可觀察者不知道觀察者的細節，只知道他有實踐特定的觀察者介面。
- 可從被觀察者處推送或拉取資料。(推送被認為是較正確的作法)
- 有多個觀察者時，不可以依賴特定的通知次序。
- 可以在被觀察者加入旗標狀態，呼叫notify時先檢查旗標才進行更新。

#### 問題
Q: 這和一對多關係有何關聯?  
A: 主題是具有狀態的物件，且可以控制這些狀態。觀察者依賴主題告訴他們這些狀態，這就產生了關係。

Q: 依賴性如何產生?  
A: 主題是真正擁有資料的人，觀察者依賴主題來更動自己。才不會讓大家都去控制同一份資料，造成混亂。

Q: 為何觀察者內要記錄主題物件?建構完後似乎用不著了啊?  
A: 的確如此，但以後可能要取消註冊，所以留著比較方便。

---

## Java實作

---

由被觀察者推送資料

``` java
public interface Subject {
    public void registerObserver(Observer o);
    public void removeObserver(Observer o);
    public void notifyObservers();
}
```
``` java
public class WeatherData implements Subject {
    private ArrayList observers;
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {
        observers = new ArrayList();
    }

    public void registerObserver(Observer o) {
        observers.add(o);
    }

    public void removeObserver(Observer o) {
        int i = observers.indexOf(o);
        if (i >= 0) {
            observers.remove(i);
        }
    }

    public void notifyObservers() {
        for (int i = 0; i < observers.size(); i++) {
            Observer observer = (Observer) observers.get(i);
            observer.update(temperature, humidity, pressure);
        }
    }

    public void measurementsChanged() {
        notifyObservers();
    }

    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }
}
```
``` java
public interface Observer {
    public void update(float temp, float humidity, float pressure);
}
```
``` java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
    private float temperature;
    private float humidity;
    private Subject weatherData;

    public CurrentConditionsDisplay(Subject weatherData) {
        this.weatherData = weatherData;
        weatherData.registerObserver(this);
    }

    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }

    public void display() {
        System.out.println("Current conditions: " + temperature +
            "F degrees and " + humidity + "% humidity");
    }
}
```
``` java
public class WeatherStation {

    public static void main(String[] args) {
        WeatherData weatherData = new WeatherData();

        CurrentConditionsDisplay currentDisplay =
            new CurrentConditionsDisplay(weatherData);
        StatisticsDisplay statisticsDisplay = new StatisticsDisplay(weatherData);
        ForecastDisplay forecastDisplay = new ForecastDisplay(weatherData);

        weatherData.setMeasurements(80, 65, 30.4 f);
        weatherData.setMeasurements(82, 70, 29.2 f);
        weatherData.setMeasurements(78, 90, 29.2 f);
    }
}
```
---

由觀察者主動拉取資料

``` java
public class WeatherData extends Observable {
    private float temperature;
    private float humidity;
    private float pressure;

    public WeatherData() {}

    public void measurementsChanged() {
        setChanged();
        notifyObservers();
    }

    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }

    public float getTemperature() {
        return temperature;
    }

    public float getHumidity() {
        return humidity;
    }

    public float getPressure() {
        return pressure;
    }
}
```
``` java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
    Observable observable;
    private float temperature;
    private float humidity;

    public CurrentConditionsDisplay(Observable observable) {
        this.observable = observable;
        observable.addObserver(this);
    }

    public void update(Observable obs, Object arg) {
        if (obs instanceof WeatherData) {
            WeatherData weatherData = (WeatherData) obs;
            this.temperature = weatherData.getTemperature();
            this.humidity = weatherData.getHumidity();
            display();
        }
    }

    public void display() {
        System.out.println("Current conditions: " + temperature +
            "F degrees and " + humidity + "% humidity");
    }
}
```

---

setChanged細節

``` java
setChanged(){
    changed = true
}

notifyObservers(Object arg){
    if(changed){
        for every obervers on the list{
            call update(this, arg)
        }
        changed=false
    }
}

notifyObservers(){
    notifyObservers(null)
}
```