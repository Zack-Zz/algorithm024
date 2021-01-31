```java
public class UseDeque {
    /**
    * main function
    */
    public static void main(String[] args) {
        Deque<String> deque = new LinkedList<>();
        deque.addLast("a");
        deque.addLast("b");
        deque.addLast("c");
        System.out.println(deque);

        String last = deque.peekLast();
        System.out.println(last);
        System.out.println(deque);

        while (deque.peekLast() != null) {
            System.out.println(deque.pop());
        }
        System.out.println(deque);
    }
}
```