---
layout:     post
title:      "Spring-Aop"
subtitle:   ""
date:       2018-05-12 12:00:00
author:     "闫祥"
header-img: "img/spring/spring-logo.png"
tags:       Spring
---

# Spring-Aop

## AOP 能做好什么？
首先我们来做一个demo来实现在Dao执行sql前后打印当前系统时间
[源码链接](https://github.com/yan11xiang/accumulation/tree/master/src/main/java/com/cbrothercoder/spring/aop)

``` java
/**
 * @author yx http://cbrothercoder.com
 */
public interface TestDao {
    void insert();

    void update();

    void delete();
}

/**
 * @author yx http://cbrothercoder.com
 */
public class TestDaoImpl implements TestDao {
    @Override
    public void insert() {
        System.out.println("TestDaoImpl insert()");
    }

    @Override
    public void update() {
        System.out.println("TestDaoImpl update()");
    }

    @Override
    public void delete() {
        System.out.println("TestDaoImpl delete()");
    }
}
```
第一种实现的方法
``` java
/**
 * 普通编程方法打印dao执行时间
 *
 * @author yx http://cbrothercoder.com
 */
public class ServiceImpl {
    private TestDao testDao = new TestDaoImpl();

    public void insert() {
        System.out.println("insert() 开始时间:" + System.currentTimeMillis());
        testDao.insert();
        System.out.println("insert() 结束时间:" + System.currentTimeMillis());
    }

    public void update() {
        System.out.println("update() 开始时间:" + System.currentTimeMillis());
        testDao.update();
        System.out.println("update() 结束时间:" + System.currentTimeMillis());
    }

    public void deleted() {
        System.out.println("delete() 开始时间:" + System.currentTimeMillis());
        testDao.delete();
        System.out.println("delete() 结束时间:" + System.currentTimeMillis());
    }

}
```
使用Service包装一层可以实现目的，但是代码无法复用，如果想对其他dao也打印执行时间，则需要使用写对应的Service。

``` java
/**
 * 使用装饰器模式
 * TestDao testDao = new LogDao(new TestDaoImpl);
 * 缺点是无法复用日志输入
 * 如果需要增加deleted的日志输出，依然需要更改代码
 *
 * @author yx http://cbrothercoder.com
 */
public class LogDao implements TestDao {

    private TestDao testDao;

    public LogDao(TestDao testDao) {
        this.testDao = testDao;
    }

    @Override
    public void insert() {
        System.out.println("insert() 开始时间:" + System.currentTimeMillis());
        testDao.insert();
        System.out.println("insert() 结束时间:" + System.currentTimeMillis());
    }

    @Override
    public void update() {
        System.out.println("update() 开始时间:" + System.currentTimeMillis());
        testDao.update();
        System.out.println("update() 结束时间:" + System.currentTimeMillis());
    }

    @Override
    public void delete() {
        testDao.delete();
    }
}
```
使用装饰器模式，缺点是无法复用日志输入。

``` java
public class LogInvocationHandler implements InvocationHandler {

    private Object obj;

    public LogInvocationHandler(Object object) {
        this.obj = object;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        if ("insert".equals(methodName) || "update".equals(methodName)) {
            System.out.println(methodName + "()方法开始时间：" + System.currentTimeMillis());
            Object result = method.invoke(obj, args);
            System.out.println(methodName + "()方法结束时间：" + System.currentTimeMillis());
            return result;
        }
        return method.invoke(obj, args);
    }

    /**
     * 结果演示
     * 这种方式的优点为：
     * <p>
     * 输出日志的逻辑被复用起来，如果要针对其他接口用上输出日志的逻辑，只要在newProxyInstance的时候的第二个参数增加Class<?>数组中的内容即可
     * 这种方式的缺点为：
     * <p>
     * JDK提供的动态代理只能针对接口做代理，不能针对类做代理
     * 代码依然有耦合，如果要对delete方法调用前后打印时间，得在LogInvocationHandler中增加delete方法的判断
     *
     * @param args
     */
    public static void main(String[] args) {
        TestDao testDao = new TestDaoImpl();
        TestDao dao = (TestDao) Proxy.newProxyInstance(LogInvocationHandler.class.getClassLoader(), new Class[]{TestDao.class}, new LogInvocationHandler(testDao));
        dao.insert();
        System.out.println("--------------------");
        dao.update();
        System.out.println("--------------------");
        dao.delete();
    }
}
```
``` java
/**
 * @author yx http://cbrothercoder.com
 */
public class DaoProxy implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        String methodName = method.getName();
        if ("insert".equals(methodName) || "update".equals(methodName)) {
            System.out.println(methodName + "()方法开始时间：" + System.currentTimeMillis());
            methodProxy.invokeSuper(o, objects);
            System.out.println(methodName + "()方法结束时间：" + System.currentTimeMillis());
            return o;
        }

        methodProxy.invokeSuper(o, objects);
        return o;
    }

    public static void main(String[] args) {
        DaoProxy daoProxy = new DaoProxy();

        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(TestDaoImpl.class);
        enhancer.setCallback(daoProxy);

        TestDao dao = (TestDaoImpl)enhancer.create();
        dao.insert();
        System.out.println("--------------------");
        dao.delete();
        System.out.println("--------------------");
        dao.update();
    }
}
```
使用ciglib拦截TestDaoImpl，如果为方法名为 insert 和update ，则增强方法，增加打印时间方法输出
执行main方法输出：
```
insert()方法开始时间：1526310192562
TestDaoImpl insert()
insert()方法结束时间：1526310192633
--------------------
TestDaoImpl delete()
--------------------
update()方法开始时间：1526310192633
TestDaoImpl update()
update()方法结束时间：1526310192633
```
如上，当使用Proxy 和 ciglib时，代码可以被复用起来，但是使用时还是会有些不方便，对业务代码的侵入性太强。
那么如果使用AOP呢？
``` java
/**
 * @author yx http://cbrothercoder.com
 */
public class TimeHandler {
    public void printTime(ProceedingJoinPoint pjp) {
        Signature signature = pjp.getSignature();
        if (signature instanceof MethodSignature) {
            MethodSignature methodSignature = (MethodSignature) signature;
            Method method = methodSignature.getMethod();
            System.out.println(method.getName() + "()方法开始时间:" + System.currentTimeMillis());
            try {
                pjp.proceed();
                System.out.println(method.getName() + "()方法结束时间:" + System.currentTimeMillis());
            } catch (Throwable throwable) {
            }

        }
    }
}
/**
 * @author yx http://cbrothercoder.com
 */
@ContextConfiguration("/spring/spring-aop.xml")
@RunWith(SpringJUnit4ClassRunner.class)
public class AopTest {

    @Autowired
    private TestDao testDao;

    /**
     * 不使用用ContextConfiguration 和 RunWith注解时可以直接调用
     */
    @Test
    public void testAop() {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring/spring-aop.xml");
        context.refresh();
        TestDao testDao = (TestDao) context.getBean("testDao");
        testDao.insert();
    }

    @Test
    public void testAopByName() {
        testDao.insert();
    }
}

```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-3.0.xsd"
       default-autowire="byName">

    <bean id="testDao" class="com.cbrothercoder.spring.aop.dao.impl.TestDaoImpl"/>
    <bean id="timeHandler" class="com.cbrothercoder.spring.aop.TimeHandler"/>

    <aop:config>
        <aop:pointcut id="testDaoPointcut" expression="execution(* com.cbrothercoder.spring.aop.dao.TestDao.*(..))"/>
        <aop:aspect id="time" ref="timeHandler">
            <aop:around method="printTime" pointcut-ref="testDaoPointcut"/>
        </aop:aspect>
    </aop:config>

</beans>
```
上面是通过配置文件来实现的 Spring AOP ， 执行 AopTest testAop()  或者 testAopByName() 方法，代理方法执行。

> 使用总结：使用Spring AOP代理只需要配置xml，即可实现对目标方法的代理，降低了对代码的侵入性。

## 源码分析
Spring对AOP做了特殊的处理，才能让我们调用示例化的Dao时实际调用的是增强后的Dao。代码定位到DefaultBeanDefinitionDocumentReader，来追踪下Spring 到底做了什么魔法。

``` java
/**
 * Parse the elements at the root level in the document:
 * "import", "alias", "bean".
 * @param root the DOM root element of the document
 */
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
  if (delegate.isDefaultNamespace(root)) {
    NodeList nl = root.getChildNodes();
    for (int i = 0; i < nl.getLength(); i++) {
      Node node = nl.item(i);
      if (node instanceof Element) {
        Element ele = (Element) node;
        if (delegate.isDefaultNamespace(ele)) {
          parseDefaultElement(ele, delegate);
        }
        else {
          delegate.parseCustomElement(ele);
        }
      }
    }
  }
  else {
    delegate.parseCustomElement(root);
  }
}
```
正常来说，遇到<bean id="daoImpl"...>、<bean id="timeHandler"...>这两个标签的时候，都会执行 parseDEfaultElement ,因为<bean>标签是默认的Namespace。但是在遇到后面的<aop:config>标签的时候就不一样了，<aop:config>并不是默认的Namespace，因此会执行 delegate.parseCustomElement 的代码
``` java
public BeanDefinition parseCustomElement(Element ele, BeanDefinition containingBd) {
  String namespaceUri = getNamespaceURI(ele);
  NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
  if (handler == null) {
    error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
    return null;
  }
  return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
}
```
![aop-handler](/img/spring/spring-aop-handler.png)
接着看一下 <aop:config> 这个Node 的 Namespace="http://www.springframework.org/schema/aop" 的 handler从哪里来，使用 idea debug 时可以看到 handler 是 AopNamespaceHandler.
看一下这个类，有几个parser  
+ config-->ConfigBeanDefinitionParser
+ aspectj-autoproxy-->AspectJAutoProxyBeanDefinitionParser
+ scoped-proxy-->ScopedProxyBeanDefinitionDecorator
+ spring-configured-->SpringConfiguredBeanDefinitionParser

接下来，我们深入 AopNamespaceHandler parse 源代码去看一下具体实现
``` java
public BeanDefinition parse(Element element, ParserContext parserContext) {
  return findParserForElement(element, parserContext).parse(element, parserContext);
}
```
首先是找到 Element对应的 parser (ConfigBeanDefinitionParser) ,继续看一下 parse 方法
``` java
	@Override
	public BeanDefinition parse(Element element, ParserContext parserContext) {
		CompositeComponentDefinition compositeDef =
				new CompositeComponentDefinition(element.getTagName(), parserContext.extractSource(element));
		parserContext.pushContainingComponent(compositeDef);

		configureAutoProxyCreator(parserContext, element);

		List<Element> childElts = DomUtils.getChildElements(element);
		for (Element elt: childElts) {
			String localName = parserContext.getDelegate().getLocalName(elt);
			if (POINTCUT.equals(localName)) {
				parsePointcut(elt, parserContext);
			}
			else if (ADVISOR.equals(localName)) {
				parseAdvisor(elt, parserContext);
			}
			else if (ASPECT.equals(localName)) {
				parseAspect(elt, parserContext);
			}
		}

		parserContext.popAndRegisterContainingComponent();
		return null;
	}
```
ConfigBeanDefinitionParser 在 parse 方法中对子标签进行了分别处理。看一下处理 aop:aspect 标签的方法
``` java

	private void parseAspect(Element aspectElement, ParserContext parserContext) {
		String aspectId = aspectElement.getAttribute(ID);
		String aspectName = aspectElement.getAttribute(REF);

		try {
			this.parseState.push(new AspectEntry(aspectId, aspectName));
			List<BeanDefinition> beanDefinitions = new ArrayList<BeanDefinition>();
			List<BeanReference> beanReferences = new ArrayList<BeanReference>();

			List<Element> declareParents = DomUtils.getChildElementsByTagName(aspectElement, DECLARE_PARENTS);
			for (int i = METHOD_INDEX; i < declareParents.size(); i++) {
				Element declareParentsElement = declareParents.get(i);
				beanDefinitions.add(parseDeclareParents(declareParentsElement, parserContext));
			}

			// We have to parse "advice" and all the advice kinds in one loop, to get the
			// ordering semantics right.
			NodeList nodeList = aspectElement.getChildNodes();
			boolean adviceFoundAlready = false;
			for (int i = 0; i < nodeList.getLength(); i++) {
				Node node = nodeList.item(i);
				if (isAdviceNode(node, parserContext)) {
					if (!adviceFoundAlready) {
						adviceFoundAlready = true;
						if (!StringUtils.hasText(aspectName)) {
							parserContext.getReaderContext().error(
									"<aspect> tag needs aspect bean reference via 'ref' attribute when declaring advices.",
									aspectElement, this.parseState.snapshot());
							return;
						}
						beanReferences.add(new RuntimeBeanReference(aspectName));
					}
					AbstractBeanDefinition advisorDefinition = parseAdvice(
							aspectName, i, aspectElement, (Element) node, parserContext, beanDefinitions, beanReferences);
					beanDefinitions.add(advisorDefinition);
				}
			}

			AspectComponentDefinition aspectComponentDefinition = createAspectComponentDefinition(
					aspectElement, aspectId, beanDefinitions, beanReferences, parserContext);
			parserContext.pushContainingComponent(aspectComponentDefinition);

			List<Element> pointcuts = DomUtils.getChildElementsByTagName(aspectElement, POINTCUT);
			for (Element pointcutElement : pointcuts) {
				parsePointcut(pointcutElement, parserContext);
			}

			parserContext.popAndRegisterContainingComponent();
		}
		finally {
			this.parseState.pop();
		}
	}

	private boolean isAdviceNode(Node aNode, ParserContext parserContext) {
    	if (!(aNode instanceof Element)) {
    		return false;
    	}
    	else {
    		String name = parserContext.getDelegate().getLocalName(aNode);
    		return (BEFORE.equals(name) || AFTER.equals(name) || AFTER_RETURNING_ELEMENT.equals(name) ||
    				AFTER_THROWING_ELEMENT.equals(name) || AROUND.equals(name));
    	}
    }
```
parseAspect 方法使用 isAdviceNode(node, parserContext) 判断是否是 advice 标签，如果是 则执行 parseAdvice 方法，直接定位到 parseAdvice 方法中的 createAdviceDefinition,方法
``` java
private AbstractBeanDefinition createAdviceDefinition(
    Element adviceElement, ParserContext parserContext, String aspectName, int order,
    RootBeanDefinition methodDef, RootBeanDefinition aspectFactoryDef,
    List<BeanDefinition> beanDefinitions, List<BeanReference> beanReferences) {

  RootBeanDefinition adviceDefinition = new RootBeanDefinition(getAdviceClass(adviceElement, parserContext));
  adviceDefinition.setSource(parserContext.extractSource(adviceElement));

  adviceDefinition.getPropertyValues().add(ASPECT_NAME_PROPERTY, aspectName);
  adviceDefinition.getPropertyValues().add(DECLARATION_ORDER_PROPERTY, order);

  if (adviceElement.hasAttribute(RETURNING)) {
    adviceDefinition.getPropertyValues().add(
        RETURNING_PROPERTY, adviceElement.getAttribute(RETURNING));
  }
  if (adviceElement.hasAttribute(THROWING)) {
    adviceDefinition.getPropertyValues().add(
        THROWING_PROPERTY, adviceElement.getAttribute(THROWING));
  }
  if (adviceElement.hasAttribute(ARG_NAMES)) {
    adviceDefinition.getPropertyValues().add(
        ARG_NAMES_PROPERTY, adviceElement.getAttribute(ARG_NAMES));
  }

  ConstructorArgumentValues cav = adviceDefinition.getConstructorArgumentValues();
  cav.addIndexedArgumentValue(METHOD_INDEX, methodDef);

  Object pointcut = parsePointcutProperty(adviceElement, parserContext);
  if (pointcut instanceof BeanDefinition) {
    cav.addIndexedArgumentValue(POINTCUT_INDEX, pointcut);
    beanDefinitions.add((BeanDefinition) pointcut);
  }
  else if (pointcut instanceof String) {
    RuntimeBeanReference pointcutRef = new RuntimeBeanReference((String) pointcut);
    cav.addIndexedArgumentValue(POINTCUT_INDEX, pointcutRef);
    beanReferences.add(pointcutRef);
  }

  cav.addIndexedArgumentValue(ASPECT_INSTANCE_FACTORY_INDEX, aspectFactoryDef);

  return adviceDefinition;
}

```

> 参考资料：
>
> [五月的仓颉博客-Spring3：AOP](http://www.cnblogs.com/xrq730/p/4919025.html)
>
> [五月的仓颉博客-AOP源码解析-上](http://www.cnblogs.com/xrq730/p/6753160.html)
>
> [五月的仓颉博客-AOP源码解析-下](http://www.cnblogs.com/xrq730/p/6757608.html)

*****
[记录并分享自己的学习成长过程](http://cbrothercoder.com/)
