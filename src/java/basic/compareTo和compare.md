# compareTo和compare

* `compareTo(Object o)`方法是java.lang.Comparable<T>接口中的方法，当需要对某个类的对象进行排序时，该类需要实现Comparable<T>接口的，必须重写public int compareTo(T o)方法，比如MapReduce中Map函数和Reduce函数处理的 <key,value>，其中需要根据key对键值对进行排序，所以，key实现了WritableComparable<T>接口，实现这个接口可同时用于序列化和反序列化。WritableComparable<T>接口(用于序列化和反序列化)是Writable接口和Comparable<T>接口的组合

* `compare(Object o1,Object o2)`方法是java.util.Comparator<T>接口的方法，它实际上用的是待比较对象的compareTo(Object o)方法

## 使用compareTo

```java
public class Demo01 {
    public static void main(String[] args) {
        TreeSet<User> users = new TreeSet<>();
        users.add(new User("zs", 12));
        users.add(new User("ls", 13));
        users.add(new User("wu", 14));
        users.add(new User("zs", 13));
        users.add(new User("zs", 12));
        System.out.println(users);  // [User{name='ls', age=13}, User{name='wu', age=14}, User{name='zs', age=12}]
    }
}


class User implements Comparable{
    public String name;
    public Integer age;

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Object o) {
        User user = (User) o;
        return this.name.compareTo(((User) o).name);
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}

```

## 使用compare

```java
public class Demo01 {
    public static void main(String[] args) {
        
        Comparator comparator = new Comparator() {
            @Override
            public int compare(Object o1, Object o2) {
                User user1 = (User) o1;
                User user2 = (User) o2;
                return user1.name.compareTo(user2.name);
            }
        };

        TreeSet<User> userTreeSet = new TreeSet<>(comparator);
        userTreeSet.add(new User("zs", 12));
        userTreeSet.add(new User("ls", 13));
        userTreeSet.add(new User("wu", 14));
        userTreeSet.add(new User("zs", 13));
        userTreeSet.add(new User("zs", 12));

        System.out.println(userTreeSet);


    }

}


class User{
    public String name;
    public Integer age;

    public User(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

