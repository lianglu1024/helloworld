Java中的Properties类属于配置文件，以键值对的方式存储，可以看做是属性集。

Properties类（Java.util.Properties）继承Hashtable（Java.util.Hashtable）

主要方法
* `getProperty(String key)`：用指定的键在此属性列表中搜索属性。也就是通过参数 key ，得到 key 所对应的 value。
* `load(InputStream inStream)`：从输入流中读取属性列表（键和值）。通过对指定的文件（比如test.properties 文件）进行装载来获取该文件中的所有键值对。以供 getProperty ( String key) 来搜索。
* `setProperty(String key, String value)`：他通过调用父类的put方法来设置键值对。
* `store(OutputStream out, String comments)`：将此 Properties 表中的属性列表（键和值）写入输出流。与 load 方法相反，该方法将键值对写入到指定的文件中去。
* `clear()`：清除所有装载的键值对。该方法由父类中提供。
* `Enumeration<?> propertyNames()`：返回Properties中的key值。

用例
```java
package com.zstu.demo;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

public class PropertiesDemo {

    public static void main(String[] args) throws IOException {
        Properties properties = new Properties();
        InputStream in = ClassLoader.getSystemResourceAsStream("test.properties");
        properties.load(in);
        System.out.println(properties.getProperty("user"));
    }
}
```
