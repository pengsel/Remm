## IOC

### ApplicationContext

实际上是BeanFactory和BeanDefinitionReader的结合，BeanFactory主要实现的是通过反射从BeanDefinition中得到相应实例，BeanDefinitionReader是从配置文件中解析出BeanDefinition信息，注册到BeanDefinition的map中

## AOP

AOP的实现有两种，一种是基于JDK动态代理。

被代理类的方法为切面，实现了自定义的接口ReflectiveMethodInvocation即advice通知，AdviceSupport将两者结合起来，其中包含TargetSource被代理对象和

MethodInterceptor拦截器，

当进行代理调用时，首先JdkDynamicAopProxy会拦截所有的方法，并调用它的invoke方法，invoke方法首先获取到自定义的拦截器MethodInterceptor，ReflectiveMethodInvocation代表真正的方法调用，它将通过反射调用被代理对象的方法，可以在自定义拦截器的invoke方法中调用ReflectiveMethodInvocation的proceed方法，并在方法的前后实现代理功能，增添自定义的内容。

按照源码解释：

#### MethodInvocation

> ```
> Description of an invocation to a method, given to an interceptor
> upon method-call.
> ```

MethodInvocation是一个方法调用，在ReflectiveMethodInvocation这里就是反射调用，核心方法是proceed()，一个简单的MethodInvocation实现：

```java
public class ReflectiveMethodInvocation implements MethodInvocation {

   private Object target;

   private Method method;

   private Object[] args;

   public ReflectiveMethodInvocation(Object target, Method method, Object[] args) {
      this.target = target;
      this.method = method;
      this.args = args;
   }

   @Override
   public Method getMethod() {
      return method;
   }

   @Override
   public Object[] getArguments() {
      return args;
   }

   @Override
   public Object proceed() throws Throwable {
      return method.invoke(target, args);
   }

   @Override
   public Object getThis() {
      return target;
   }

   @Override
   public AccessibleObject getStaticPart() {
      return method;
   }
}
```

#### MethodInterceptor

```
* Intercepts calls on an interface on its way to the target. These
* are nested "on top" of the target.
*
* <p>The user should implement the {@link #invoke(MethodInvocation)}
* method to modify the original behavior.
```

MethodInterceptor是一个拦截器，在某个接口被调用时工作，可以通过实现invoke方法改变原本的方法执行。

一个实例：

```java
public class TimerInterceptor implements MethodInterceptor {

   @Override
   public Object invoke(MethodInvocation invocation) throws Throwable {
      long time = System.nanoTime();
      System.out.println("Invocation of Method " + invocation.getMethod().getName() + " start!");
      Object proceed = invocation.proceed();
      System.out.println("Invocation of Method " + invocation.getMethod().getName() + " end! takes " + (System.nanoTime() - time)
            + " nanoseconds.");
      return proceed;
   }
}
```

#### 怎么管理切面

通过AspecJ来检查某个类是否是切面表达式所代表的类。

#### PointParser

可以通过切面解析器将AspectJ的切面表达式解析成为相应的PointExpression

```
* A PointcutParser can be used to build PointcutExpressions for a user-defined subset of AspectJ's pointcut language
```



#### PointExpression

可以判断某个类或者某个方法是否在切面上

```
* Represents an AspectJ pointcut expression and provides convenience methods to determine
* whether or not the pointcut matches join points specified in terms of the
* java.lang.reflect interfaces.
```

具体实现其实是调用了AspectJ已经封装好的方法：

```java
/*判断类是否match*/
@Override
public boolean matches(Class targetClass) {
   checkReadyToMatch();
   return pointcutExpression.couldMatchJoinPointsInType(targetClass);
}
/*判断方法是否match*/
@Override
public boolean matches(Method method, Class targetClass) {
   checkReadyToMatch();
   ShadowMatch shadowMatch = pointcutExpression.matchesMethodExecution(method);
   if (shadowMatch.alwaysMatches()) {
      return true;
   } else if (shadowMatch.neverMatches()) {
      return false;
   }
   // TODO:其他情况不判断了！见org.springframework.aop.aspectj.RuntimeTestWalker
   return false;
}
```

#### 获取切面bean改为获取bean的代理对象

为了在方法执行时，调用代理对象，执行相应的advice，在获取被代理对象时，替换成新生成的绑定了拦截器的代理对象，并set进BeanDefinitions，方便下次直接从其中取。

如何对BeanFactory实现无侵入式的拓展？

```java
public interface BeanPostProcessor {

   Object postProcessBeforeInitialization(Object bean, String beanName) throws Exception;

   Object postProcessAfterInitialization(Object bean, String beanName) throws Exception;

}
```

BeanFactory现在有一个BeanPostProcessor，在getBean时对Bean进初始化时，会根据BeanPostProcessor对生成的bean进行预处理，如果发现他不是通知Advisor或者MethodInterceptor，并且Class在切面表达式中，就生成它的一个代理。这里的切面的粒度在类。

```java
@Override
public Object postProcessAfterInitialization(Object bean, String beanName) throws Exception {
   if (bean instanceof AspectJExpressionPointcutAdvisor) {
      return bean;
   }
       if (bean instanceof MethodInterceptor) {
           return bean;
       }
   List<AspectJExpressionPointcutAdvisor> advisors = beanFactory
         .getBeansForType(AspectJExpressionPointcutAdvisor.class);
   for (AspectJExpressionPointcutAdvisor advisor : advisors) {
      if (advisor.getPointcut().getClassFilter().matches(bean.getClass())) {
         AdvisedSupport advisedSupport = new AdvisedSupport();
         advisedSupport.setMethodInterceptor((MethodInterceptor) advisor.getAdvice());
         advisedSupport.setMethodMatcher(advisor.getPointcut().getMethodMatcher());

         TargetSource targetSource = new TargetSource(bean, bean.getClass().getInterfaces());
         advisedSupport.setTargetSource(targetSource);

         return new JdkDynamicAopProxy(advisedSupport).getProxy();
      }
   }
   return bean;
}
```

具体执行时，会判断函数是否match相应的aspectJ的切面表达式，满足执行生成代理实例的方法，否则执行被代理实例的方法。这里切面的粒度在方法。

```java
@Override
public Object invoke(final Object proxy, final Method method, final Object[] args) throws Throwable {
   MethodInterceptor methodInterceptor = advised.getMethodInterceptor();
   if (advised.getMethodMatcher() != null
         && advised.getMethodMatcher().matches(method, advised.getTargetSource().getTarget().getClass())) {
      return methodInterceptor.invoke(new ReflectiveMethodInvocation(advised.getTargetSource().getTarget(),
            method, args));
   } else {
      return method.invoke(advised.getTargetSource().getTarget(), args);
   }
}
```

这里的MethodInterceptor便是我们要自定义实现的。

BeanPostProcessor在哪初始化的，答 在refresh()函数中，创建一个ApplicationContext时将会初始化它

```javascript
public void refresh() throws Exception {
    //这个是一个抽象方法，可以自定义字节的Reader，从Xml或者其他文件解析出BeanDefinition
   loadBeanDefinitions(beanFactory);
    //注册BeanPostProcessors，扫描有哪些是BeanPostProcessors类型，直接添加
   registerBeanPostProcessors(beanFactory);
    //里面调用了preInstantiateSingletons
   onRefresh();
}
```

### cglib代理

基本上是一致的，不同的是它的拦截是实现net.sf.cglib.proxy.MethodInterceptor的intercept()方法，如何封装这个方法看自己，这里为了和上面兼容，依然包装了一个org.aopalliance.intercept.MethodInterceptor。

```java
public class Cglib2AopProxy extends AbstractAopProxy {

   public Cglib2AopProxy(AdvisedSupport advised) {
      super(advised);
   }

   @Override
   public Object getProxy() {
      Enhancer enhancer = new Enhancer();
      enhancer.setSuperclass(advised.getTargetSource().getTargetClass());
      enhancer.setInterfaces(advised.getTargetSource().getInterfaces());
      enhancer.setCallback(new DynamicAdvisedInterceptor(advised));
      Object enhanced = enhancer.create();
      return enhanced;
   }

   private static class DynamicAdvisedInterceptor implements MethodInterceptor {

      private AdvisedSupport advised;

      private org.aopalliance.intercept.MethodInterceptor delegateMethodInterceptor;

      private DynamicAdvisedInterceptor(AdvisedSupport advised) {
         this.advised = advised;
         this.delegateMethodInterceptor = advised.getMethodInterceptor();
      }

      @Override
      public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
         if (advised.getMethodMatcher() == null
               || advised.getMethodMatcher().matches(method, advised.getTargetSource().getTargetClass())) {
            return delegateMethodInterceptor.invoke(new CglibMethodInvocation(advised.getTargetSource().getTarget(), method, args, proxy));
         }
         return new CglibMethodInvocation(advised.getTargetSource().getTarget(), method, args, proxy).proceed();
      }
   }

   private static class CglibMethodInvocation extends ReflectiveMethodInvocation {

      private final MethodProxy methodProxy;

      public CglibMethodInvocation(Object target, Method method, Object[] args, MethodProxy methodProxy) {
         super(target, method, args);
         this.methodProxy = methodProxy;
      }

      @Override
      public Object proceed() throws Throwable {
         return this.methodProxy.invoke(this.target, this.arguments);
      }
   }

}
```

####  稍微看下Cglib对于其中类和方法的解释

Enhancer



为了实现方法拦截，生成了动态子类。可以对一个具体的子类进行代理，而不仅仅是实现了接口的类。生成的子类重写了父类的 non-final 方法，提供了自定义钩子函数的接口。

可以通过CallbackFilter()来控制在一个方法上的callback。

```java
Callback
/**
 * Map methods of subclasses generated by {@link Enhancer} to a particular
 * callback. The type of the callbacks chosen for each method affects
 * the bytecode generated for that method in the subclass, and cannot
 * change for the life of the class.
 */
public interface CallbackFilter {
    /**
     * Map a method to a callback.
     * @param method the intercepted method
     * @return the index into the array of callbacks (as specified by {@link Enhancer#setCallbacks}) to use for the method, 
     */
    int accept(Method method);

    /**
     * The <code>CallbackFilter</code> in use affects which cached class
     * the <code>Enhancer</code> will use, so this is a reminder that
     * you should correctly implement <code>equals</code> and
     * <code>hashCode</code> for custom <code>CallbackFilter</code>
     * implementations in order to improve performance.
    */
    boolean equals(Object o);
}
```

enhancer.create()方法

```java
/**
 * Generate a new class if necessary and uses the specified
 * callbacks (if any) to create a new object instance.
 * Uses the no-arg constructor of the superclass.
 * @return a new instance
 */
```