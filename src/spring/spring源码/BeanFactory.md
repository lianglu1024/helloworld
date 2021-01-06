![image-20201223160226311](/Users/Liang/Library/Application Support/typora-user-images/image-20201223160226311.png)



![image-20201223162336425](https://tva1.sinaimg.cn/large/0081Kckwly1glxv85pt5uj30l60l8dka.jpg)

![image-20201223163621480](https://tva1.sinaimg.cn/large/0081Kckwly1glxvlehnu9j30rk0gwtex.jpg)

![image-20201223163751368](https://tva1.sinaimg.cn/large/0081Kckwly1glxvmy37n2j30yw0g4afx.jpg)

#  BeanFactory

The root interface for accessing a Spring bean container

# BeanFactoryPostProcessor

可用于用户扩展

Factory hook that allows for custom modification of an application context's bean definitions, adapting the bean property values of the context's underlying bean factory.

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {

	/**
	 * Modify the application context's internal bean factory after its standard
	 * initialization. All bean definitions will have been loaded, but no beans
	 * will have been instantiated yet. This allows for overriding or adding
	 * properties even to eager-initializing beans.
	 * @param beanFactory the bean factory used by the application context
	 * @throws org.springframework.beans.BeansException in case of errors
	 */
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

```



12