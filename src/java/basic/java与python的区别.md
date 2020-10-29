# java与python的区别

1.java许多方法是强制抛异常，而python则没有这个限制



2.实例变量，成员变量



3.python成员方法调用成员方法的时候要用this，如果是继承的话，this是代表子类；java中this一般省略不写



4.static的区别？



5.变量作用域不一样，java变量作用域在{}，而python在整个方法内部

```python
try:
  a = 10
finnally:
	print(a)
```



```java
try{
  Interger a = 10;
}finnally{
  System.out.println(a);  //错误
}
```



6.继承的区别？



7.接口。java实现功能都要写一个接口，然后再继承接口实现功能；python直接定义一个类去实现



8.python中==表示值相等，is表示地址相等；java中==表示地址相等