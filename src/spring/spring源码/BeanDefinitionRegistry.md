BeanDefinitionRegistry是一个顶层接口

BeanDefinitionRegistry 的方法定义了如何对 BeanDefinition 进行管理。SimpleBeanDefinitionRegistry 是 BeanDefinitionRegistry 的一个简单实现，可通过它来了解下具体实现。

```java
public class SimpleBeanDefinitionRegistry extends SimpleAliasRegistry implements BeanDefinitionRegistry {

	/** Map of bean definition objects, keyed by bean name. */
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(64);


	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "'beanName' must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");
		this.beanDefinitionMap.put(beanName, beanDefinition);
	}

	@Override
	public void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
		if (this.beanDefinitionMap.remove(beanName) == null) {
			throw new NoSuchBeanDefinitionException(beanName);
		}
	}

	@Override
	public BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException {
		BeanDefinition bd = this.beanDefinitionMap.get(beanName);
		if (bd == null) {
			throw new NoSuchBeanDefinitionException(beanName);
		}
		return bd;
	}

	@Override
	public boolean containsBeanDefinition(String beanName) {
		return this.beanDefinitionMap.containsKey(beanName);
	}

	@Override
	public String[] getBeanDefinitionNames() {
		return StringUtils.toStringArray(this.beanDefinitionMap.keySet());
	}

	@Override
	public int getBeanDefinitionCount() {
		return this.beanDefinitionMap.size();
	}

	@Override
	public boolean isBeanNameInUse(String beanName) {
		return isAlias(beanName) || containsBeanDefinition(beanName);
	}

}
```

