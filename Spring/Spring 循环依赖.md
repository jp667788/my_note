
allowCircularReferences = true 是否允许循环依赖

``` java
	protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {  
	   Assert.notNull(singletonFactory, "Singleton factory must not be null");  
	 	synchronized (this.singletonObjects) {  
		  if (!this.singletonObjects.containsKey(beanName)) {  
			 	this.singletonFactories.put(beanName, singletonFactory);  
	 			this.earlySingletonObjects.remove(beanName);  
	 			this.registeredSingletons.add(beanName);  
	 		}  
	   	}  
	}
```


### @Bean 处理流程