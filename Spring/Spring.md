# Spring

## IOC和AOP概念

IOC利用java**反射机制**，AOP利用**代理模式**

### IOC（控制反转）

**依赖注入，控制反转**

使用工厂模式，把本来通过new方式创建对象的方法变更为交给容器管理，在spring容器启动的时候，spring会把你在配置文件中配置的bean都初始化好，然后在你需要调用的时候，就把它已经初始化好的那些bean分配给你需要调用这些bean的类。 依赖注入就是输入我们类的原型，有四种方式：

- 注解
  - 用`@Component`注解标识将对象放到「IOC容器」中，用`@Autowired`注解将对象注入
- XML
- JavaConfig
- 基于Groovy DSL配置

### AOP（切面编程）

把重复的代码抽取，在运行的时候往业务方法上**动态植入**“切面类代码”，实现原理有两种：

- 动态代理：也就是在执行你想执行的方法时动态执行一些额外的操作
- 静态植入：引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码，属于静态代理

### 循环依赖怎么处理的

A依赖B，B依赖A，在Spring中有两种循环依赖：

- **构造器循环依赖**

  - ```java
    @Service
    public class A {
        public A(B b) {  }
    }
    
    @Service
    public class B {
        public B(C c) {
        }
    }
    
    @Service
    public class C {
        public C(A a) {  }
    }
    ```

    启动失败

- **field属性依赖**

  - ```java
    @Service
    public class A1 {
        @Autowired
        private B1 b1;
    }
    
    @Service
    public class B1 {
        @Autowired
        public C1 c1;
    }
    
    @Service
    public class C1 {
        @Autowired  public A1 a1;
    }
    ```

    启动成功（单例）

  - ```java
    @Service
    @Scope("prototype")
    public class A1 {
        @Autowired
        private B1 b1;
    }
    
    @Service
    @Scope("prototype")
    public class B1 {
        @Autowired
        public C1 c1;
    }
    
    @Service
    @Scope("prototype")
    public class C1 {
        @Autowired  public A1 a1;
    }
    ```

    启动失败（多例）

对于循环依赖的场景，构造器注入和prototype类型的属性注入都会初始化Bean失败。因为@Service默认是单例的，所以单例的属性注入是可以成功的。

为什么单例的属性注入能成功？

![preview](https://picb.zhimg.com/v2-8dda72f4f7ae6f8dcdf4ad84f62d0565_r.jpg)

![preview](https://pic4.zhimg.com/v2-73ffee93119ec21f1a630a792c5f0607_r.jpg)

可以看到，核心就是一个对象的初始化分为构造和属性注入两个步骤，构造完了就放入三级缓存，然后属性注入的对象初始化时就能从三级缓存拿到原始对象。

[通过循环依赖问题彻底理解 Spring IOC 的精华](https://zhuanlan.zhihu.com/p/132299743)



### BeanFactory和ApplicationContext

Spring的两大核心接口：**BeanFactory**和**ApplicationContext**

BeanFactory，直译Bean工厂（com.springframework.beans.factory.BeanFactory），咱们通常称BeanFactory为IoC容器，提供最基础的注册于获取方法；而称ApplicationContext为应用上下文，是BeanFactory的子接口，主要是实现了getBean方法，其中定义了refresh方法。

ApplicationContext会预先初始化全部的Singleton Bean，并不是懒加载方式。

Spring装配Bean的三种方式：

- xml文件配置加载
- 注解加载，如@Configuration和@Bean
- 隐式加载和自动装配，@Configuration、@Component、@ComponentScan（Spring会自动发现应用上下文中所建立的bean。）

IoC 在 Spring 里，只需要低级容器就可以实现，2 个步骤：

1. 加载配置文件，解析成 BeanDefinition 放在 Map 里。
2. 调用 getBean 的时候，从 BeanDefinition 所属的 Map 里，拿出 Class 对象进行实例化，同时，如果有依赖关系，将递归调用 getBean 方法 —— 完成依赖注入。

上面就是 Spring 低级容器（BeanFactory）的 IoC。

至于高级容器 ApplicationContext，他包含了低级容器的功能，当他执行 refresh 模板方法的时候，将刷新整个容器的 Bean。同时其作为高级容器，包含了太多的功能。一句话，他不仅仅是 IoC。他支持不同信息源头，支持 BeanFactory 工具类，支持层级容器，支持访问文件资源，支持事件发布通知，支持接口回调等等。

### Bean的加载流程

入口：

```javascript
public class Application {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        Person person = context.getBean("person", Person.class);
        System.out.println(person.toString());
    }
}
```

![quicker_c37fdb93-6ae9-46f0-be83-820a17c6a783.png](https://i.loli.net/2020/07/04/sBEaZjmfq91tY7c.png)

如上图所示，Spring的启动过程主要可以分为两部分：

- 第一步：**解析成BeanDefinition**：将bean定义信息解析为BeanDefinition类，不管bean信息是定义在xml中，还是通过@Bean注解标注，都能通过不同的BeanDefinitionReader转为BeanDefinition类。
  - 这里分两种BeanDefinition，RootBeanDefintion和BeanDefinition。RootBeanDefinition这种是系统级别的，是启动Spring必须加载的6个Bean。BeanDefinition是我们定义的Bean。
- 第二步：参照BeanDefintion定义的类信息，**通过BeanFactory生成bean实例存放在缓存中**。
  - 这里的BeanFactoryPostProcessor是一个拦截器，在BeanDefinition实例化后，BeanFactory生成该Bean之前，可以对BeanDefinition进行修改。
  - BeanFactory根据BeanDefinition定义使用反射实例化Bean，实例化和初始化Bean的过程中就涉及到Bean的生命周期了，典型的问题就是Bean的循环依赖。接着，Bean实例化前会判断该Bean是否需要增强，并决定使用哪种代理来生成Bean。

![Bean 加载流程图](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9vYmxlZS5vc3MtY24taGFuZ3pob3UuYWxpeXVuY3MuY29tL3NwcmluZy9CZWFuJTIwJUU1JThBJUEwJUU4JUJEJUJEJUU2JUI1JTgxJUU3JUE4JThCJUU1JTlCJUJFLnBuZw?x-oss-process=image/format,png)

可以看到getBean主要分为几个阶段：

- **获取 BeanName**：对传入的 name 进行解析，转化为可以从 Map 中获取到 BeanDefinition 的 bean name。
- **合并 Bean 定义**：对父类的定义进行合并和覆盖，如果父类还有父类，会进行递归合并，以获取完整的 Bean 定义信息。
- **实例化**：使用构造或者工厂方法创建 Bean 实例。
- **属性填充**：寻找并且注入依赖，依赖的 Bean 还会递归调用 getBean 方法获取。
- **初始化**：调用自定义的初始化方法。
- **获取最终的 Bean**：如果是 FactoryBean 需要调用 getObject 方法，如果需要类型转换调用 TypeConverter 进行转化。

**获取 BeanName**：

> 通过getBean传入的name可能有三种类型，所以需要对传入的name转换为beanName：
>
> - bean name：直接获取BeanDefinition定义
> - alias name：别名，需要转化
> - factorybean name，带 `&` 前缀：通过它获取 BeanDefinition 的时候需要去除 & 前缀。

**合并 RootBeanDefinition**

> 从配置文件读取到的 BeanDefinition 是 **GenericBeanDefinition**。它的记录了一些当前类声明的属性或构造参数，但是对于父类只用了一个 `parentName` 来记录。然而在后续实例化 Bean 的时候，使用的 BeanDefinition 是 **RootBeanDefinition** 类型而非 **GenericBeanDefinition**，因为`GenericBeanDefinition` 在有继承关系的情况下， 存储的是 **增量信息** 而不是 **全量信息**，定义的信息不足，需要进行**合并父类定义**。
>
> 合并的时候如果父类还有父类定义，则用递归定义，最后用本类定义覆盖父类返回的定义进行合并为完整的定义。

**循环依赖解决**

1. 原型模式

> Bean在创建过程中使用了一个 ThreadLocal 变量`prototypesCurrentlyInCreation`来记录正在创建中的Bean对象，创建前写创建完成后删，，在单个元素时 `prototypesCurrentlyInCreation`只记录 String 对象，在多个依赖元素后改用 Set 集合。这里是 Spring 使用的一个节约内存的小技巧。
>
> 循环依赖有两种：**构造函数依赖**与**属性依赖**，两种方式调用链实现不同
>
> 构造函数使用`BeanDefinitionValueResolver.resolveReference`方法调用`getBean`
>
> ```java
> private Object resolveReference(Object argName, RuntimeBeanReference ref) {
> 	...
> 	Object bean = this.beanFactory.getBean(refName);
> 	...
> }
> ```
>
> 属性依赖的话就是直接调用无参构造函数，不需要检索构造参数的引用，直接实例化成功，然后在属性填充阶段使用`AbtractBeanFactory.populateBean`调用`getbean`
>
> ```java
> public Object resolveCandidate(String beanName, Class<?> requiredType, BeanFactory beanFactory)throws BeansException {
> 	return beanFactory.getBean(beanName, requiredType);
> }
> ```
>
> 每次调用`getBean`实例化前后都会操作`prototypesCurrentlyInCreation`，调用判定的函数在`AbstractBeanFactory.doGetBean`中
>
> ```java
> if (isPrototypeCurrentlyInCreation(beanName)) {
> 	throw new BeanCurrentlyInCreationException(beanName);
> }
> ```

2. 单例模式

> Spring 也不支持单例模式的构造循环依赖。检测到构造循环依赖也会抛出 `BeanCurrentlyInCreationException` 异常。
>
> 单例模式也用一种数据结构来存储正在创建中的Bean，以下均存在于`DefaultSingletonBeanRegistry`：
>
> ```java
> private final Set<String> singletonsCurrentlyInCreation =
> 			Collections.newSetFromMap(new ConcurrentHashMap<String, Boolean>(16));
> ```
>
> ```java
> public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
> 		...
> 		// 记录正在加载中的 beanName
> 		beforeSingletonCreation(beanName);
> 		...
> 		// 通过 singletonFactory 创建 bean
> 		singletonObject = singletonFactory.getObject();
> 		...
> 		// 删除正在加载中的 beanName
> 		afterSingletonCreation(beanName);
> 		
> }
> ```
>
> ```java
> protected void beforeSingletonCreation(String beanName) {
> 		if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
> 			throw new BeanCurrentlyInCreationException(beanName);
> 		}
> 	}
> ```
>
> 会尝试往 `singletonsCurrentlyInCreation` 记录当前实例化的 bean。我们知道 singletonsCurrentlyInCreation 的数据结构是 **Set**，是不允许重复元素的，**所以一旦前面记录了，这里的 add 操作将会返回失败**。

> 单例模式中使用三级缓存来解决属性依赖：
>
> - **singletonObjects**，单例缓存，存储已经实例化完成的单例。
> - **singletonFactories**，生产单例的工厂的缓存，存储工厂。
> - **earlySingletonObjects**，提前暴露的单例缓存，这时候的单例刚刚创建完，但还会注入依赖。
>
> 这个 `earlySingletonObjects` 的好处是，如果此时又有其他地方尝试获取未初始化的单例，可以从 `earlySingletonObjects` 直接取出而不需要再调用 `getEarlyBeanReference`。

**创建实例**

> 获取到完整的 RootBeanDefintion 后，就可以拿这份定义信息来实例具体的 Bean。
>
> 通过`DefaultListableBeanFactory`的`getBean()`方法去初始化，实际由`AbstractAutowireCapableBeanFactory`的`doCreateBean`去完成。
>
> 具体实例创建见 `AbstractAutowireCapableBeanFactory.createBeanInstance` ，返回 Bean 的包装类 `BeanWrapper`，一共有三种策略：
>
> - 使用工厂方法创建，instantiateUsingFactoryMethod 
> - 使用有参构造函数创建，autowireConstructor。
> - 使用无参构造函数创建，instantiateBean。
>
> 这三个实例化方式，最后都会走 `getInstantiationStrategy().instantiate`：
>
> ```java
> public Object instantiate ... {
> 	if (bd.getMethodOverrides().isEmpty()) {
> 		...
> 		return BeanUtils.instantiateClass(constructorToUse);
> 	}
> 	else {
> 		// Must generate CGLIB subclass.
> 		return instantiateWithMethodInjection(bd, beanName, owner);
> 	}
> }
> ```
>
> 虽然拿到了构造函数，并没有立即实例化。因为用户使用了 replace 和 lookup 的配置方法，用到了动态代理加入对应的逻辑。如果没有的话，直接使用反射来创建实例。创建实例后，就可以开始注入属性和初始化等操作。
>
> 但这里的 Bean 还不是最终的 Bean。返回给调用方使用时，如果是 FactoryBean 的话需要使用 getObject 方法来创建实例。见 `AbstractBeanFactory.getObjectFromBeanInstance` ，会执行到 `doGetObjectFromFactoryBean` ：
>
> ```java
> private Object doGetObjectFromFactoryBean ... {
> 	...
> 	object = factory.getObject();
> 	...
> 	return object;
> }
> ```

**属性填充**

> ```java
> protected void populateBean ... {
> 	PropertyValues pvs = mbd.getPropertyValues();
> 	
> 	...
> 	// InstantiationAwareBeanPostProcessor 前处理
> 	for (BeanPostProcessor bp : getBeanPostProcessors()) {
> 		if (bp instanceof InstantiationAwareBeanPostProcessor) {
> 			InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
> 			if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
> 				continueWithPropertyPopulation = false;
> 				break;
> 			}
> 		}
> 	}
> 	...
> 	
> 	// 根据名称注入
> 	if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME) {
> 		autowireByName(beanName, mbd, bw, newPvs);
> 	}
> 
> 	// 根据类型注入
> 	if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
> 		autowireByType(beanName, mbd, bw, newPvs);
> 	}
> 
> 	... 
> 	// InstantiationAwareBeanPostProcessor 后处理
> 	for (BeanPostProcessor bp : getBeanPostProcessors()) {
> 		if (bp instanceof InstantiationAwareBeanPostProcessor) {
> 			InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
> 			pvs = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
> 			if (pvs == null) {
> 				return;
> 			}
> 		}
> 	}
> 	
> 	...
> 	
> 	// 应用属性值
> 	applyPropertyValues(beanName, mbd, bw, pvs);
> }
> ```
>
> - 应用 `InstantiationAwareBeanPostProcessor` 处理器，在属性注入前后进行处理。假设我们使用了 @Autowire 注解，这里会调用到 `AutowiredAnnotationBeanPostProcessor` 来对依赖的实例进行检索和注入的，它是 `InstantiationAwareBeanPostProcessor` 的子类。
> - 根据名称或者类型进行自动注入，存储结果到 PropertyValues 中。
> - 应用 PropertyValues，填充到 BeanWrapper。这里在检索依赖实例的引用的时候，会递归调用 BeanFactory.getBean 来获得。

**初始化**

> 如果 Bean 需要容器的一些资源，比如需要获取到 BeanFactory、ApplicationContext 等等，如果判断 Bean 实现了这几个 **Aware 系列接口**，就会**往 Bean 中注入它关心的资源**。
>
> - BeanFactoryAware，用来获取 BeanFactory。
> - ApplicationContextAware，用来获取 ApplicationContext。
> - ResourceLoaderAware，用来获取 ResourceLoaderAware。
> - ServletContextAware，用来获取 ServletContext。
>
> 在 Bean 的初始化前或者初始化后，如果需要进行一些增强操作比如打日志、做校验、属性修改、耗时检测等等，只要 Bean 实现了 BeanPostProcessor 接口，加载的时候**都会被 Spring 注册到 beanPostProcessors 中**，然后在 Bean 实例化前后，Spring 会去调用我们已经注册的 beanPostProcessors 把处理器都执行一遍。
>
> 最后出发自定义方法，自定义方法有两种：
>
> - 实现 InitializingBean。提供了一个很好的机会，在属性设置完成后再加入自己的初始化逻辑。
> - 定义 init 方法。自定义的初始化逻辑。

**类型转换**

> Bean 已经加载完毕，属性也填充好了，初始化也完成了，在返回给调用者之前，还留有一个机会对 Bean 实例进行类型的转换。

### BeanFactory 和 FactoryBean 

**BeanFactory 是什么**

Spring Bean 容器的根接口，**它是 IOC 的基本容器，负责管理和加载 Bean，它为其他具体的IOC容器提供了最基本的规范**，比如 `DefaultListableBeanFactory` 和 `ConfigurableBeanFactory`，BeanFactory 也提供了用于读取 XML 配置文件的实现，比如 `XMLBeanFactory`。

ApplicationContext 接口是 BeanFactory 的扩展，它最主要的实现就是 `ClassPathXmlApplicationContext`，用来读取XML 配置文件，现在我们用的更多的是 ClassPathXmlApplicationContext 而不是 XMLBeanFactory 了。

使用示例：

```java
ApplicationContext beanFactory = new ClassPathXmlApplicationContext("spring-beans.xml");
HelloBean helloBean = (HelloBean) beanFactory.getBean("helloBean");
helloBean.printMsg();
```

**FactoryBean 是什么**

FactoryBean 是一个接口，它本身就是一个对象工厂，**如果bean 实现了这个接口，它被用作公开的对象工厂，而不是作为直接将bean暴露的实例**。该接口在框架内部大量使用，例如 AOP ProxyFactoryBean 或者 JndiObjectFactoryBean。也能自定义组件；然而，这仅适用于基础框架代码。**FactoryBeans 支持单例或多例，并且可以根据需要懒加载创建对象**，也可以在启动时急切创建对象

接口很简单，只有三个方法：

- `getObject`: 返回一个工厂生产出来的对象，这个对象将要使用在Spring IOC 容器中
- `getObjectType` : 顾名思义就是返回工厂生产出来对象的类型
- `isSingleton`: 表示生产出来的对象是否是单例的

可以用来代理一个对象，对该对象的所有方法做一个拦截，

### 注解的实现原理

注解本质是继承自`Annotation`接口的接口，**一个注解准确意义上来说，只不过是一种特殊的注释而已，如果没有解析它的代码，它可能连注释都不如。**注解携带的是元数据，其次，它可能会引起一些和元数据相关的操作，但**不会对被注释的代码逻辑产生影响**。

而解析一个类或者方法的注解往往有两种形式

- **编译期直接的扫描**
  - 编译器的扫描指的是编译器在对 java 代码编译字节码的过程中会检测到某个类或者方法被一些注解修饰，这时它就会对于这些注解进行某些处理。如 @Override，一旦编译器检测到某个方法被修饰了 @Override 注解，编译器就会检查当前方法的方法签名是否真正重写了父类的某个方法，也就是比较父类中是否具有一个同样的方法签名。
  - 仅适用于JDK内置的注解
    - 元注解：
      - @Target：注解的作用目标，如允许作用在方法或者属性上
      - @Retention：注解的生命周期
      - @Documented：注解是否应当被包含在 JavaDoc 文档中
      - @Inherited：是否允许子类继承该注解
    - 内置三大注解
      - @Override：它没有任何的属性，所以并不能存储任何其他信息。它只能作用于方法之上，编译结束后将被丢弃。一种典型的『标记式注解』，仅被编译器可知，编译器在对 java 文件进行编译成字节码的过程中，一旦检测到某个方法上被修饰了该注解，就会去匹对父类中是否具有一个同样方法签名的函数，如果不是，自然不能通过编译。
      - @Deprecated
      - @SuppressWarnings
- **运行期反射**
  - 注解本质上是继承了 Annotation 接口的接口，而当你通过反射，也就是 getAnnotation 方法去获取一个注解类实例的时候，其实 JDK 是通过动态代理机制生成一个实现我们注解（接口）的代理类。

Spring Boot中注解示例

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Indexed
public @interface Component {
    String value() default "";
}
```

最后我们再总结一下整个反射注解的工作原理：

首先，我们通过键值对的形式可以为注解属性赋值，像这样：@Hello（value = "hello"）。

接着，你用注解修饰某个元素，编译器将在编译期扫描每个类或者方法上的注解，会做一个基本的检查，你的这个注解是否允许作用在当前位置，最后会将注解信息写入元素的属性表。

然后，当你进行反射的时候，虚拟机将所有生命周期在 RUNTIME 的注解取出来放到一个 map 中，并创建一个 AnnotationInvocationHandler 实例，把这个 map 传递给它。

最后，虚拟机将采用 JDK 动态代理机制生成一个目标注解的代理类，并初始化好处理器。

那么这样，一个注解的实例就创建出来了，它本质上就是一个代理类，你应当去理解好 AnnotationInvocationHandler 中 invoke 方法的实现逻辑，这是核心。一句话概括就是，**通过方法名返回注解属性值**

### 解决依赖冲突

依赖冲突可能有两种情况：

1. 项目同一依赖应用，存**在多版本，每个版本同一个类，可能存在差异**
2. 项目不同依赖应用，**存在包名，类名完全一样的类**。

先学习下Maven的依赖机制：**仲裁机制**

1. 优先按照依赖管理元素中指定的版本声明进行仲裁时，下面的两个原则都无效了
2. 短路径优先
3. 若路径距离相同，看 pom 中声明的顺序。

再学习下Maven的Scope属性，maven项目可以分为三个阶段：**编译阶段，测试阶段，运行阶段**，通过设置Scope属性可以判断依赖应用是否参与上述阶段，`Maven` 提供 6 种 `scope` ：

- `compile`：默认属性，将会使依赖包参与项目的编译，测试，运行阶段。当然，项目打包之后将会包含该依赖

- `provided`：依赖仅参与项目编译，测试的阶段

  > 若有如下依赖关系：
  >
  > ```log
  > A----->B----->C
  > ```
  >
  > C 的 `scope` 为`provided`，C 将会参与 B 的编译，测试阶段，但是 C 不会传递给 A。如果 A 运行过程需要 C，需要自己直接引入 C 依赖。典型如 `Servlet API`，因为 `Tomcat` 等容器内部会提供。

- `runtime`：不再参与项目编译阶段，只参与测试，运行阶段

  > 若依赖不参与编译阶段，这种情况 IDE 中是无法导入相应的类的。若存在依赖类，编译过程中将会报错。
  >
  > 典型的例子是 `JDBC` 驱动包，如 `mysql` :
  >
  > ```xml
  > <dependency>
  >     <groupId>mysql</groupId>
  >     <artifactId>mysql-connector-java</artifactId>
  >     <version>6.0.6</version>
  >     <scope>runtime</scope>
  > </dependency>
  > ```
  >
  > 知识点：这个好处在于，只能使用 `JDBC` 标准接口，这样就不会与特定的数据库绑定。后续若切换数据库，只需要更换 `pom`，然后修改相应的参数即可。

- `test`：仅参与测试阶段的工作，典型的例子为 `junit`

- `system`

- `import`;不会参与以上阶段运行。其只能在 `dependencyManagement`下使用，且 `type` 需要为 `pom`。典型的例子为 Spring-boot 依赖。

  > ```xml
  >     <dependencyManagement>
  >         <dependencies>
  >             <!-- Spring Boot -->
  >             <dependency>
  >                 <groupId>org.springframework.boot</groupId>
  >                 <artifactId>spring-boot-dependencies</artifactId>
  >                 <version>2.1.6.RELEASE</version>
  >                 <type>pom</type>
  >                 <scope>import</scope>
  >             </dependency>
  >         </dependencies>
  >     </dependencyManagement>
  > ```
  >
  > 知识点：通过这种方式，**解决单继承问题**，也可以更好将依赖分类。

另外 `Maven scope` 将会影响依赖传递。

![image-20200216185916932](https://img2018.cnblogs.com/blog/1419561/202002/1419561-20200224085635091-1646213722.jpg)

> 如果依赖关系为： **A--->B--->C**,A 依赖 B，B 依赖 C。最左列代表 B 的 `scope` 属性，第一行代表 C 的 `scope` 属性

如上所示，当 C 的 `scope` 为 **provided/test**, C 只在 B 中起作用，不会通过间接依赖传递给 A。

当且仅当 B 的 `scope` 为 `compile`，且 C `scope` 为 `runtime` ，A 将会间接依赖 C，且 `scope` 为 `runtime`。其他情况下，C 的 scope 将会与 B 的 scope 一致。



对于第一种Jar包冲突问题，通常的做法是**用<excludes>排除不需要的版本**，但这种做法带来的问题是每次引入带有传递性依赖的Jar包时，都需要一一进行排除，非常麻烦。maven为此提供了集中管理依赖信息的机制，即依赖管理元素**<dependencyManagement>**，对依赖Jar包进行统一版本管理，一劳永逸。通常的做法是，在parent模块的pom文件中尽可能地声明所有相关依赖Jar包的版本，并在子pom中简单引用该构件即可。

来看个示例，当开发时确定使用的httpclient版本为4.5.1时，可在父pom中配置如下：

```xml
...
    <properties>
      <httpclient.version>4.5.1</httpclient.version>
    </properties>
    <dependencyManagement>
      <dependencies>
        <dependency>
          <groupId>org.apache.httpcomponents</groupId>
          <artifactId>httpclient</artifactId>
          <version>${httpclient.version}</version>
        </dependency>
      </dependencies>
    </dependencyManagement>
...
```

然后各个需要依赖该Jar包的子pom中配置如下依赖：

```xml
...
    <dependencies>
      <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
      </dependency>
    </dependencies>
...
```

除了排除依赖，我们可以通过合理的设置 `scope` 属性，不让依赖传播下去。比如说，A 需要是使用 `Spring-beans` 包中某些类。如果其他项目铁定会使用 Spring，那么我们可以将 A 中 `Spring-beans` `scope` 设置为 `provided`，让其他项目自己选择引入 `Spring-beans` 的版本。

第二种情况可以使用maven冲突检查插件**maven-enforcer-plugin**

[[程序员需要了解依赖冲突的原因以及解决方案](https://www.cnblogs.com/goodAndyxublog/p/12355528.html)](https://www.cnblogs.com/goodAndyxublog/p/12355528.html)

### Bean的生命周期

![img](https://pic4.zhimg.com/80/v2-baaf7d50702f6d0935820b9415ff364c_720w.jpg?source=1940ef5c)

1. **实例化Bean**

对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。 
对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。 
容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入。 
实例化对象被包装在BeanWrapper对象中，BeanWrapper提供了设置对象属性的接口，从而避免了使用反射机制设置属性。

2. **设置对象属性（依赖注入）**

实例化后的对象被封装在BeanWrapper对象中，并且此时对象仍然是一个原生的状态，并没有进行依赖注入。 
紧接着，Spring根据BeanDefinition中的信息进行依赖注入。 
并且通过BeanWrapper提供的设置属性的接口完成依赖注入。

3.  **注入Aware接口**

紧接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean。

4. **BeanPostProcessor**

当经过上述几个步骤后，bean对象已经被正确构造，但如果你想要对象被使用前再进行一些自定义的处理，就可以通过BeanPostProcessor接口实现。 
该接口提供了两个函数：

- postProcessBeforeInitialzation( Object bean, String beanName ) 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会先于InitialzationBean执行，因此称为前置处理。 
  所有Aware接口的注入就是在这一步完成的。
- postProcessAfterInitialzation( Object bean, String beanName ) 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会在InitialzationBean完成后执行，因此称为后置处理。

5. **InitializingBean与init-method**

当BeanPostProcessor的前置处理完成后就会进入本阶段。 
InitializingBean接口只有一个函数：

- afterPropertiesSet()

这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。 
若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行afterPropertiesSet函数。

当然，Spring为了降低对客户代码的侵入性，给bean的配置提供了init-method属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。init-method本质上仍然使用了InitializingBean接口。

6. **DisposableBean和destroy-method**

和init-method一样，通过给destroy-method指定函数，就可以在bean销毁前执行指定的逻辑。



## Spring事务

### 编程式事务管理

需要在代码中显式调用beginTransaction()、commit()、rollback()等事务管理相关的方法，这就是编程式事务管理。通过 Spring 提供的事务管理 API，我们可以在代码中灵活控制事务的执行。在底层，Spring 仍然将事务操作委托给底层的持久化框架来执行

### Spring 声明式事务管理

就Spring 声明式事务而言，无论其基于 <tx> 命名空间的实现还是基于 @Transactional 的实现，其本质都是 **Spring AOP 机制**的应用；其本质是**对目标方法前后进行拦截，并在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务**。即通过以@Transactional的方式或者XML配置文件的方式**向业务组件中的目标业务方法插入事务增强处理并生成相应的代理对象**

Spring并不直接管理事务，而是提供了多种事务管理器，他们将事务管理的职责委托给[hibernate](http://lib.csdn.net/base/javaee)或者JTA等持久化机制所提供的相关平台框架的事务来实现。 

### 七种事务传播行为

1. `PROPAGATION_REQUIRED`：必须运行在事务中，如果没有事务则开启一个新的事务。
2. `PROPAGATION_SUPPORTS`：不必须运行在事务中，如果没有事务就算了。
3. `PROPAGATION_MANDATORY`：必须运行在事务中，没有就抛异常。
4. `PROPAGATION_REQUIRES_NEW`：不管有没有事务都要开启一个新事务，有旧事务则旧事务会挂起。
   - **内层事务提交后，外层事务无法将其回滚**
5. `PROPAGATION_NOT_SUPPORTED`：不能运行在事务中，有旧事务则挂起旧事务
6. `PROPAGATION_NEVER`：不能运行在事务中，有旧事务则抛异常
7. `PROPAGATION_NESTED`：如果一个活动的事务存在，则运行在一个嵌套的事务中. 如果没有活动事务, 则按TransactionDefinition.PROPAGATION_REQUIRED 属性执行
   - **外层事务的回滚会引起内层事务的回滚**

此外事务可以设置**只读**、**事务超时**与**回滚规则**等属性

## Spring中运用到的设计模式

- **工厂设计模式** : Spring使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建 bean 对象。
- **代理设计模式** : Spring AOP 功能的实现。
- **单例设计模式** : Spring 中的 Bean 默认都是单例的。
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
- **包装器设计模式** : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
- **观察者模式:** Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配`Controller`。

**工厂模式**

Spring使用工厂模式可以通过 `BeanFactory` 或 `ApplicationContext` 创建 bean 对象。**通过 `ConcurrentHashMap` 实现单例注册表的特殊方式实现单例模式**

**单例模式**

Spring 中 bean 的默认作用域就是 singleton(单例)的。 单例好处：

- 对于频繁使用的对象，可以省略创建对象所花费的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销；
- 由于 new 操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻 GC 压力，缩短 GC 停顿时间。

**代理模式**

**Spring AOP 就是基于动态代理的**，如果要代理的对象，实现了某个接口，那么Spring AOP会使用**JDK Proxy**，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用**Cglib** ，这时候Spring AOP会使用 **Cglib** 生成一个被代理对象的子类来作为代理

Spring AOP 和 AspectJ AOP 有什么区别?

- **Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

**模板方法**

模板方法模式是一种行为设计模式，它定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。 模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤的实现方式。

Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。一般情况下，我们都是使用继承的方式来实现模板模式，但是 Spring 并没有使用这种方式，而是使用Callback 模式与模板方法模式配合，既达到了代码复用的效果，同时增加了灵活性。

**观察者模式**

表示的是一种对象与对象之间具有依赖关系，当一个对象发生改变的时候，这个对象所依赖的对象也会做出反应。Spring 事件驱动模型就是观察者模式很经典的一个应用。Spring 事件驱动模型非常有用，在很多场景都可以解耦我们的代码。比如我们每次添加商品的时候都需要重新更新商品索引，这个时候就可以利用观察者模式来解决这个问题。

**装饰者模式**

动态地给对象添加一些额外的属性或行为。相比于使用继承，装饰者模式更加灵活。简单点儿说就是当我们需要修改原有的功能，但我们又不愿直接去修改原有的代码时，设计一个Decorator套在原有代码外面。

Spring 中配置 DataSource 的时候，DataSource 可能是不同的数据库和数据源。我们能否根据客户的需求在少修改原有类的代码下动态切换不同的数据源？这个时候就要用到装饰者模式(这一点我自己还没太理解具体原理)。Spring 中用到的包装器模式在类名上含有 `Wrapper`或者 `Decorator`。这些类基本上都是动态地给一个对象添加一些额外的职责

[面试官:“谈谈Spring中都用到了那些设计模式?”。](https://juejin.im/post/5ce69379e51d455d877e0ca0)

##  SpringMVC

**工作原理**：

![img](https://upload-images.jianshu.io/upload_images/3301869-21660c3eed8e557c.JPG?imageMogr2/auto-orient/strip|imageView2/2/w/860/format/webp)

- DispatcherServlet前置控制器，分配请求给页面控制器
- HandlerMapping将请求映射为HandlerExecutionChain对象(包含Handler处理器对象（页面控制器），多个HandlerInterceptor对象即拦截器)，再返回给DispatcherServlet
- DispatcherServlet再次发送请求给HandlerAdapter
- HandlerAdapter将处理器包装为适配器，调用处理器相应功能处理方法，Handler返回ModelAnView给HandlerAdapter
- HandlerAdapter发送给DispatcherServlet进行视图的解析（ViewResolver），ViewResolver将逻辑视图解析为具体的视图，返回给DispatcherServlet
- 进行视图的渲染（View），返回给DispatcherServlet，最后通过DispatcherServlet将视图返回给用户。

3.分工职责

1. 前置控制器**DispatcherServlet**:接收请求  返回结果
2. 映射处理器 **HandlerMapping**:根据请求映射为HandlerExecutionChain对象，查找对应的Handler
3. 处理器适配 **HandlerAdapter**:调用处理器相对应的处理方法，返回ViewAndModel
4. 视图解析器 **ViewResolver**
5. 视图的渲染 **View**

## Mybatis

### sql与mapper如何对应

**mybatis会读取xml文件, 并获取xml和interface的映射, 将需要执行的sql绑定在interface上, 并构造代理注入spring, 在调用时通过反射获取当前调用的interface以及method, 然后在注册好的映射map中获取具体执行的sql并执行**

以xml为例，调用sql之前需要通过`SqlSessionFactoryBuilder`工厂创建`sqlsession`，然后调用`getMapper`方法

```java
ActivityCzMapper am = sqlSession.getMapper(ActivityCzMapper.class);
        ActivityCz activityCz = am.selectByPrimaryKey(1);

//------------------------------------------------------------------------------
public <T> T getMapper(Class<T> type) {
        return this.configuration.getMapper(type, this);
    }

//------------------------------------------------------------------------------
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
        return this.mapperRegistry.getMapper(type, sqlSession);
    }

//------------------------------------------------------------------------------
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
        MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory)this.knownMappers.get(type);
        if(mapperProxyFactory == null) {
            throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
        } else {
            try {
                return mapperProxyFactory.newInstance(sqlSession);
            } catch (Exception var5) {
                throw new BindingException("Error getting mapper instance. Cause: " + var5, var5);
            }
        }
    }

```

可以看到return的是一个newInstance，说明利用了动态代理，这个`mapperProxyFactory`是从`knownMappers`这个map中get到的，key为我们的接口class对象。跟踪这个newInstance方法

```java
//实现了InvocationHandler，说明利用了JDK的动态代理，把三个参数初始化的MapperProxy代理类传到下一个重载方法。
public class MapperProxy<T> implements InvocationHandler, Serializable{}

private Map<Method, MapperMethod> methodCache = new ConcurrentHashMap();  

public T newInstance(SqlSession sqlSession) {
        MapperProxy<T> mapperProxy = new MapperProxy(sqlSession, this.mapperInterface, this.methodCache);
        return this.newInstance(mapperProxy);
    }

//-------------------------------------------------------------------------
protected T newInstance(MapperProxy<T> mapperProxy) {
        return Proxy.newProxyInstance(this.mapperInterface.getClassLoader(), new Class[]{this.mapperInterface}, mapperProxy);
 }
```

可以得知利用jdk的动态代理模式Proxy.newProxyInstance方法，里面的三个参数显而易见，对于第三个参数mapperProxy就是 实现了InvocationHandler接口的MapperProxy类。