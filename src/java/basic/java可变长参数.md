# 可变长参数

调用方法的时候，对于同一个方法，我们可以传入可变长的参数

```java
public class TestDemo {

    @Test
    public void test01() {
        method("a", "b");
        method("a", "b","c");
    }


    public void method(String... s){
        for (String s1 : s) {
            System.out.println(s1);
        }
    }
}
```

方法method中的局部变量s会接收实际参数转变成数组

注意：一个方法只能有一个可变参数，且可变参数应为最后一个参数

```java
public class TestDemo {

    @Test
    public void test01() {
        method(10,"a", "b");

        method(20,"a", "b","c");
    }


    public void method(Integer i, String... s){
        System.out.println(i);
        for (String s1 : s) {
            System.out.println(s1);
        }
    }
}
```

