# 观察者模式

```
package Observertest;

import java.util.ArrayList;
import java.util.List;

/**
 * 主题: 如果数据有变化,通知所有Observer
 */
public class Subject {

    private List<Observer> observerList = new ArrayList<>();

    private int state;

    public int getState() {
        return this.state;
    }

    public void setState (int state) {
        if (state == this.state) {
            return;
        }
        this.state = state;
        notifyObserver();
    }

    public void attach(Observer observer) {
        observerList.add(observer);
    }


    private void notifyObserver() {
        observerList.forEach(Observer::update);
    }
}

package Observertest;

/**
 * 抽象的Observer
 */
public abstract  class Observer {

    protected Subject subject;

    public Observer(Subject subject) {
        this.subject = subject;
        this.subject.attach(this);
    }

     abstract void update();
}

package Observertest;

public class BinaryObserver extends Observer {

    public BinaryObserver(Subject subject) {
        super(subject);
    }

    @Override
    void update() {
        System.out.println("Binary String : " + Integer.toBinaryString(subject.getState()));

    }
}


package Observertest;

public class OctalObserver extends Observer {

    public OctalObserver(Subject subject) {
        super(subject);
    }

    @Override
    void update() {
        System.out.println("Octal string : " + Integer.toOctalString(subject.getState()));
    }
}


package Observertest;

public class ObseverClient {

    public static void main(String[] args) {

        final Subject subject = new Subject();

       BinaryObserver binaryObserver = new BinaryObserver(subject);
       OctalObserver octalObserver = new OctalObserver(subject);

       System.out.println("---------------");
       subject.setState(10);

       System.out.println("---------------");
       subject.setState(16);
        
    }
}

```
