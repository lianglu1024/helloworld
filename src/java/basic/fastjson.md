# fastjson

导入依赖

```xml
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.60</version>
</dependency>
```

在fastjson包中主要有3个类，JSON，JSONArray，JSONObject。三者之间的关系如下，JSONObject和JSONArray继承JSON。

- JSONObject（实现了Map的接口）：解析Json对象，获取对象中的值，通常是使用类中的get()，getString()方法。
- JSONArray（实现了List的接口）：JSON对象数组，通常是通过迭代器取得其中的JSONObject，再利用JSONObeject的get()方法进行取值。
- JSON：主要是实现json对象，json对象数组，javabean对象，json字符串之间的相互转化。 转换之后取值还是按各自的方法进行。

## JSON

JSON这个类主要用于**转换**：

- 将Java对象序列化为JSON字符串
- 将JSON字符串反序列化为Java对象

以下五种方法常用：

```
String text = JSON.toJSONString(object);

JSONObject jsonObject = JSON.parseObject(text)

XXX xxx = JSON.parseObject(text, XXX.class)

JSONArrat jsonArray = JSON.parseArray(text);

List<XXX> users = JSON.parseArray(text, XXX.class);
```

示例：

```java
public class User {
    private Integer age;

    private String name;

    public User(Integer age, String name) {
        this.age = age;
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

使用：

```java
public class Demo {

    public static void main(String[] args) {
        User user1 = new User(12, "zhangsan");

        String s = JSON.toJSONString(user1);  //{"age":12,"name":"zhangsan"}

        JSONObject jsonObject = JSON.parseObject(s);

        System.out.println(jsonObject);  //{"age":12,"name":"zhangsan"}

        System.out.println(jsonObject.get("name"));  // "zhangsan"是一个Object类

        User user = JSON.parseObject(s, User.class);
        System.out.println(user);  // com.zstu.fastjson.User@64a294a6

        String JSON_ARRAY_STR = "[{\"studentName\":\"lily\",\"studentAge\":12}, {\"studentName\":\"lucy\",\"studentAge\":15}]";
        JSONArray objects = JSON.parseArray(JSON_ARRAY_STR);
        for (Object object : objects) {
            JSONObject o = (JSONObject) object;
            System.out.println((String) o.get("studentName"));
        }
    }
}
```

复杂JSON格式字符串—》JSONObject

```java
public class Demo {

    public static void main(String[] args) {

        String  COMPLEX_JSON_STR = "{\"teacherName\":\"crystall\"," +
                "\"teacherAge\":27,\"course\":{\"courseName\":\"english\",\"code\":1270},\"students\":[{\"studentName\":\"lily\",\"studentAge\":12},{\"studentName\":\"lucy\",\"studentAge\":15}]}";

        JSONObject jsonObject = JSON.parseObject(COMPLEX_JSON_STR);
        // 获取简单对象
        String teacherName = jsonObject.getString("teacherName");
        Integer teacherAge = jsonObject.getInteger("teacherAge");
        System.out.println("teacherName: " + teacherName + ",teacherAge " + teacherAge);
        // 获取JSONObject对象
        JSONObject course = jsonObject.getJSONObject("course");
        // 获取JSONObject中的数据
        String courseName = course.getString("courseName");
        Integer code = course.getInteger("code");
        System.out.println("courseName:  " + courseName + "   code:  " + code);
        // 获取JSONArray对象
        JSONArray students = jsonObject.getJSONArray("students");
        // 获取JSONArray的中的数据
        Iterator<Object> iterator = students.iterator();
        while (iterator.hasNext()){
            JSONObject jsonObject1 = (JSONObject) iterator.next();
            System.out.println("studentName: " + jsonObject1.getString("studentName") + ",StudentAge: "
                    + jsonObject1.getInteger("studentAge"));
        }
    }
}
```

JavaBean-List —》json字符串数组

```java
public class Demo {
    public static void main(String[] args) {
       List<User> list = new ArrayList<>();
       list.add(new User(12, "zs"));
       list.add(new User(13, "ls"));
        String s = JSON.toJSONString(list);
        System.out.println(s);  //[{"age":12,"name":"zs"},{"age":13,"name":"ls"}]
    }
}
```

json字符串数组 —》JavaBean-List 

```java
public class Demo {
    public static void main(String[] args) {
       String s = "[{\"age\":12,\"name\":\"zs\"},{\"age\":13,\"name\":\"ls\"}]";
       List<User> users = JSON.parseArray(s, User.class);
        for (User user : users) {
            System.out.println(user);
        }
    }
}
```

