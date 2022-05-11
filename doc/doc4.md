## 4. 彩蛋

> spring-data-jpa原理

### 4.1 思考

#### <img src="https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511134553451.png" alt="image-20220511134553451" style="zoom:80%;" />

### 4.2 源码分析

#### 4.2.1 Spring Data Jpa 自动配置

![image-20220511134910241](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511134910241.png)

 ` @Import  org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesRegistrar`

![image-20220511091903320](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511091903320.png)

在该类中配置了 @EnableJpaRepositories

![](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511091942768.png)



![image-20220511093031190](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511093031190.png)

 `@Import org.springframework.data.jpa.repository.config.JpaRepositoriesRegistrar`



集成`RepositoryBeanDefinitionRegistrarSupport`

```
class JpaRepositoriesRegistrar extends RepositoryBeanDefinitionRegistrarSupport {

	/*
	 * (non-Javadoc)
	 * @see org.springframework.data.repository.config.RepositoryBeanDefinitionRegistrarSupport#getAnnotation()
	 */
	@Override
	protected Class<? extends Annotation> getAnnotation() {
		return EnableJpaRepositories.class;
	}

	/*
	 * (non-Javadoc)
	 * @see org.springframework.data.repository.config.RepositoryBeanDefinitionRegistrarSupport#getExtension()
	 */
	@Override
	protected RepositoryConfigurationExtension getExtension() {
		return new JpaRepositoryConfigExtension();
	}
}
```

#### 4.2.2. 创建JpaRepositoryFactoryBean

`RepositoryBeanDefinitionRegistrarSupport`实现`ImportBeanDefinitionRegistrar`

```java
package org.springframework.data.repository.config;

import java.lang.annotation.Annotation;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanNameGenerator;
import org.springframework.context.EnvironmentAware;
import org.springframework.context.ResourceLoaderAware;
import org.springframework.context.annotation.ConfigurationClassPostProcessor;
import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
import org.springframework.core.env.Environment;
import org.springframework.core.io.ResourceLoader;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.lang.NonNull;
import org.springframework.util.Assert;

public abstract class RepositoryBeanDefinitionRegistrarSupport implements ImportBeanDefinitionRegistrar, ResourceLoaderAware, EnvironmentAware {
    @NonNull
    private ResourceLoader resourceLoader;
    @NonNull
    private Environment environment;

    public RepositoryBeanDefinitionRegistrarSupport() {
    }

    public void setResourceLoader(ResourceLoader resourceLoader) {
        this.resourceLoader = resourceLoader;
    }

    public void setEnvironment(Environment environment) {
        this.environment = environment;
    }

    /** @deprecated */
    @Deprecated
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        this.registerBeanDefinitions(metadata, registry, ConfigurationClassPostProcessor.IMPORT_BEAN_NAME_GENERATOR);
    }

    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry, BeanNameGenerator generator) {
        Assert.notNull(metadata, "AnnotationMetadata must not be null!");
        Assert.notNull(registry, "BeanDefinitionRegistry must not be null!");
        Assert.notNull(this.resourceLoader, "ResourceLoader must not be null!");
        if (metadata.getAnnotationAttributes(this.getAnnotation().getName()) != null) {
            AnnotationRepositoryConfigurationSource configurationSource = new AnnotationRepositoryConfigurationSource(metadata, this.getAnnotation(), this.resourceLoader, this.environment, registry, generator);
            RepositoryConfigurationExtension extension = this.getExtension();
            RepositoryConfigurationUtils.exposeRegistration(extension, registry, configurationSource);
            RepositoryConfigurationDelegate delegate = new RepositoryConfigurationDelegate(configurationSource, this.resourceLoader, this.environment);
            // 委托类注册
            delegate.registerRepositoriesIn(registry, extension);
        }
    }

    protected abstract Class<? extends Annotation> getAnnotation();

    protected abstract RepositoryConfigurationExtension getExtension();
}

```

最终生成代理类

![image-20220511101643918](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511101643918.png)

![image-20220511101648015](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511101648015.png)

![image-20220511101655585](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511101655585.png)

![image-20220511101803945](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511101803945.png)

![image-20220511101819136](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511101819136.png)

![image-20220511101908120](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511101908120.png)





#### 4.2.3. 代理类`SimpleJpaRepository`

上一步我们确定了，jps是通过创建factory动态创建repository的，先看一看`JpaRepositoryFactoryBean` 源码

```
public void afterPropertiesSet() {
        this.factory = this.createRepositoryFactory();
        this.factory.setQueryLookupStrategyKey(this.queryLookupStrategyKey);
        this.factory.setNamedQueries(this.namedQueries);
        this.factory.setEvaluationContextProvider((QueryMethodEvaluationContextProvider)this.evaluationContextProvider.orElseGet(() -> {
            return QueryMethodEvaluationContextProvider.DEFAULT;
        }));
        this.factory.setBeanClassLoader(this.classLoader);
        this.factory.setBeanFactory(this.beanFactory);
        if (this.publisher != null) {
            this.factory.addRepositoryProxyPostProcessor(new EventPublishingRepositoryProxyPostProcessor(this.publisher));
        }

        RepositoryFactorySupport var10001 = this.factory;
        this.repositoryBaseClass.ifPresent(var10001::setRepositoryBaseClass);
        this.repositoryFactoryCustomizers.forEach((customizer) -> {
            customizer.customize(this.factory);
        });
        RepositoryFragments customImplementationFragment = (RepositoryFragments)this.customImplementation.map((xva$0) -> {
            return RepositoryFragments.just(new Object[]{xva$0});
        }).orElseGet(RepositoryFragments::empty);
        RepositoryFragments repositoryFragmentsToUse = ((RepositoryFragments)this.repositoryFragments.orElseGet(RepositoryFragments::empty)).append(customImplementationFragment);
        this.repositoryMetadata = this.factory.getRepositoryMetadata(this.repositoryInterface);
        this.repository = Lazy.of(() -> {
            return (Repository)this.factory.getRepository(this.repositoryInterface, repositoryFragmentsToUse);
        });
        this.mappingContext.ifPresent((it) -> {
            it.getPersistentEntity(this.repositoryMetadata.getDomainType());
        });
        if (!this.lazyInit) {
            this.repository.get();
        }

    }
```

顺着getRepository方法继续追踪

```java
public <T> T getRepository(Class<T> repositoryInterface, RepositoryFragments fragments) {

		if (logger.isDebugEnabled()) {
			logger.debug(LogMessage.format("Initializing repository instance for %s…", repositoryInterface.getName()));
		}

		Assert.notNull(repositoryInterface, "Repository interface must not be null!");
		Assert.notNull(fragments, "RepositoryFragments must not be null!");

		ApplicationStartup applicationStartup = getStartup();

		StartupStep repositoryInit = onEvent(applicationStartup, "spring.data.repository.init", repositoryInterface);

		repositoryBaseClass.ifPresent(it -> repositoryInit.tag("baseClass", it.getName()));

		StartupStep repositoryMetadataStep = onEvent(applicationStartup, "spring.data.repository.metadata",
				repositoryInterface);
		RepositoryMetadata metadata = getRepositoryMetadata(repositoryInterface);
		repositoryMetadataStep.end();

		StartupStep repositoryCompositionStep = onEvent(applicationStartup, "spring.data.repository.composition",
				repositoryInterface);
		repositoryCompositionStep.tag("fragment.count", String.valueOf(fragments.size()));

		RepositoryComposition composition = getRepositoryComposition(metadata, fragments);
		
		//指定RepositoryBaseClass为SimpleJpaRepository
		RepositoryInformation information = getRepositoryInformation(metadata, composition);

		/// ...

		return repository;
	}
```



```java
	private RepositoryInformation getRepositoryInformation(RepositoryMetadata metadata,
			RepositoryComposition composition) {

		RepositoryInformationCacheKey cacheKey = new RepositoryInformationCacheKey(metadata, composition);

		return repositoryInformationCache.computeIfAbsent(cacheKey, key -> {

            // 获取指定的Repository类
			Class<?> baseClass = repositoryBaseClass.orElse(getRepositoryBaseClass(metadata));

			return new DefaultRepositoryInformation(metadata, baseClass, composition);
		});
	}
```

RepositoryFactorySupport

```java
	/**
	 * Returns the base class backing the actual repository instance. Make sure
	 * {@link #getTargetRepository(RepositoryInformation)} returns an instance of this class.
	 *
	 * @param metadata
	 * @return
	 */
	protected abstract Class<?> getRepositoryBaseClass(RepositoryMetadata metadata);
```

JpaRepositoryFactory

```java
public class JpaRepositoryFactory extends RepositoryFactorySupport {
  // ...
  @Override
  protected Class<?> getRepositoryBaseClass(RepositoryMetadata metadata) {
     return SimpleJpaRepository.class;
  }
  // ...
}
```

#### 4.2.4. Spring Data JPA 是怎么匹配Repository接口的？

RepositoryConfigurationExtensionSupport

```java
	protected boolean isStrictRepositoryCandidate(RepositoryMetadata metadata) {

		if (noMultiStoreSupport) {
			return false;
		}

		// 这里就是获取标识|（识别）类型
		Collection<Class<?>> types = getIdentifyingTypes();
		Collection<Class<? extends Annotation>> annotations = getIdentifyingAnnotations();
		String moduleName = getModuleName();

		if (types.isEmpty() && annotations.isEmpty()) {
			if (!noMultiStoreSupport) {
				logger.warn(LogMessage.format("Spring Data %s does not support multi-store setups!", moduleName));
				noMultiStoreSupport = true;
				return false;
			}
		}
```

JpaRepositoryConfigExtension

```java
@Override
protected Collection<Class<?>> getIdentifyingTypes() {
   return Collections.<Class<?>> singleton(JpaRepository.class);
}
```

### 4.3 总结

![image-20220511135637658](https://hp-blog-img.oss-cn-beijing.aliyuncs.com/markdown/image-20220511135637658.png)


