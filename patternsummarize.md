1.单例模式（Singleton Pattern）

确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。

2.工厂模式

定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法使一个类的实例化延迟到其子类。

3.抽象工厂模式（Abstract Factory Pattern）

为创建一组相关或相互依赖的对象提供一个接口，而且无须指定它们的具体类。

4.模板方法模式（Template Method Pattern）

定义一个操作中的算法的框架，而将一些步骤延迟到子类中。使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

5.代理模式（Proxy Pattern）

为其他对象提供一种代理以控制对这个对象的访问。

6.原型模式（Prototype Pattern）

用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

7.装饰模式（Decorator Pattern）

动态地给一个对象添加一些额外的职责。就增加功能来说，装饰模式相比生成子类更为灵活。

8.策略模式（Strategy Pattern）

定义一组算法，将每个算法都封装起来，并且使它们之间可以互换。

9.适配器模式（Adapter Pattern）

将一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作。

10.观察者模式（Observer Pattern）

定义对象间一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并被自动更新。

##### AOP

```
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    if (delegate.isDefaultNamespace(root)) {
        NodeList nl = root.getChildNodes();

        for(int i = 0; i < nl.getLength(); ++i) {
            Node node = nl.item(i);
            if (node instanceof Element) {
                Element ele = (Element)node;
                if (delegate.isDefaultNamespace(ele)) {
                    this.parseDefaultElement(ele, delegate);
                } else {
                    delegate.parseCustomElement(ele);
                }
            }
        }
    } else {
        delegate.parseCustomElement(root);
    }

}

  @Nullable
    public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
        String namespaceUri = this.getNamespaceURI(ele);
        if (namespaceUri == null) {
            return null;
        } else {
            NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
            if (handler == null) {
                this.error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
                return null;
            } else {
                return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
            }
        }
```

#### ioc

```
public void refresh() throws BeansException, IllegalStateException {
    Object var1 = this.startupShutdownMonitor;
    synchronized(this.startupShutdownMonitor) {
        this.prepareRefresh();
        ConfigurableListableBeanFactory beanFactory = this.obtainFreshBeanFactory();
        this.prepareBeanFactory(beanFactory);

        try {
            this.postProcessBeanFactory(beanFactory);
            this.invokeBeanFactoryPostProcessors(beanFactory);
            this.registerBeanPostProcessors(beanFactory);
            this.initMessageSource();
            this.initApplicationEventMulticaster();
            this.onRefresh();
            this.registerListeners();
            this.finishBeanFactoryInitialization(beanFactory);
            this.finishRefresh();
        } catch (BeansException var9) {
            if (this.logger.isWarnEnabled()) {
                this.logger.warn("Exception encountered during context initialization - cancelling refresh attempt: " + var9);
            }

            this.destroyBeans();
            this.cancelRefresh(var9);
            throw var9;
        } finally {
            this.resetCommonCaches();
        }

    }
}
```

##### DI

```
public Object createBean(Class<?> beanClass, int autowireMode, boolean dependencyCheck) throws BeansException {
    RootBeanDefinition bd = new RootBeanDefinition(beanClass, autowireMode, dependencyCheck);
    bd.setScope("prototype");
    return this.createBean(beanClass.getName(), bd, (Object[])null);
}

public Object autowire(Class<?> beanClass, int autowireMode, boolean dependencyCheck) throws BeansException {
    RootBeanDefinition bd = new RootBeanDefinition(beanClass, autowireMode, dependencyCheck);
    bd.setScope("prototype");
    if (bd.getResolvedAutowireMode() == 3) {
        return this.autowireConstructor(beanClass.getName(), bd, (Constructor[])null, (Object[])null).getWrappedInstance();
    } else {
        Object bean;
        if (System.getSecurityManager() != null) {
            bean = AccessController.doPrivileged(() -> {
                return thisx.getInstantiationStrategy().instantiate(bd, (String)null, this);
            }, this.getAccessControlContext());
        } else {
            bean = this.getInstantiationStrategy().instantiate(bd, (String)null, this);
        }

        this.populateBean(beanClass.getName(), bd, new BeanWrapperImpl(bean));
        return bean;
    }
}

public void autowireBeanProperties(Object existingBean, int autowireMode, boolean dependencyCheck) throws BeansException {
    if (autowireMode == 3) {
        throw new IllegalArgumentException("AUTOWIRE_CONSTRUCTOR not supported for existing bean instance");
    } else {
        RootBeanDefinition bd = new RootBeanDefinition(ClassUtils.getUserClass(existingBean), autowireMode, dependencyCheck);
        bd.setScope("prototype");
        BeanWrapper bw = new BeanWrapperImpl(existingBean);
        this.initBeanWrapper(bw);
        this.populateBean(bd.getBeanClass().getName(), bd, bw);
    }
}
```