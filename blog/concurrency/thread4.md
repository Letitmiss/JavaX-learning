# 主线程钩子

```
package Hook;

public class ExitCapture {

    public static void main(String[] args) {

        Runtime.getRuntime().addShutdownHook(new Thread( () -> {
            try {
                Thread.sleep(1_000);
                System.out.println("appliaction will exit , will write data and  flush");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }));
        int i = 0 ;
        while (true) {
            try {
                Thread.sleep(1_000);
                System.out.println("I am working ");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            i++;
            if (i > 5) {
                throw new RuntimeException("error");
            }
        }
    }
}

```
