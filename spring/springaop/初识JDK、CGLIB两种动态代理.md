# 初识JDK、CGLIB两种动态代理



在开始 Spring AOP 源码分析之前，我们先来了解一下其底层 JDK 动态代理和 CGLIB 动态代理两种 AOP 代理的实现，本文也会讲到 Javassist 动态代理的一个简单使用示例。



### 示例

Echo 服务：

```java
public interface EchoService {

    String echo(String message);
}
```

默认实现：

```java
public class DefaultEchoService implements EchoService {

    @Override
    public String echo(String message) {
        return "[ECHO] " + message;
    }
}
```

现在有这么一个需求，当你调用上面的 `echo(String)` 方法的时候，需要打印出方法的执行时间，那么我们可以怎么做？答案就是通过**代理模式**，通过代理类为这个对象提供一种代理以控制对这个对象的访问，在代理类中输出方法的执行时间。

### 静态代理

代理类实现被代理类所实现的接口，同时持有被代理类的引用，新增处理逻辑，当我们调用代理类的方法时候，实际调用被代理类的引用。

代理类：

```java
public class ProxyEchoService implements EchoService {
    /**
     * 被代理对象
     */
    private final EchoService echoService;

    public ProxyEchoService(EchoService echoService) {
        this.echoService = echoService;
    }

    @Override
    public String echo(String message) {
        long startTime = System.currentTimeMillis();
        // 调用被代理对象的方法
        String result = echoService.echo(message);
        long costTime = System.currentTimeMillis() - startTime;
        System.out.println("echo 方法执行的实现：" + costTime + " ms.");
        return result;
    }
}
```

示例：

```java
public class StaticProxyDemo {

    public static void main(String[] args) {
        // 创建代理类，并传入被代理对象，也就是 DefaultEchoService
        EchoService echoService = new ProxyEchoService(new DefaultEchoService());
        echoService.echo("Hello,World");
    }
}
```

控制台会输出以下内容：

> echo 方法执行的实现：0 ms.

得到的结论：

**静态代理**就是通过实现被代理对象所实现的接口，内部保存了被代理对象，在实现的方法中对处理逻辑进行**增强**，实际的方法执行调用了被代理对象的方法。可以看到**静态代理**比较简洁直观，但是在复杂的场景下，需要为每个目标对象创建一个代理类，不易于维护，我们更加注重的应该是业务开发，对于这一层**增强**处理应该抽取出来。



### JDK 动态代理

基于接口代理，通过反射机制生成一个实现代理接口的类，在调用具体方法时会调用 InvokeHandler 来处理。

#### 示例

```java
public class JdkDynamicProxyDemo {

    public static void main(String[] args) {
        // 当前线程的类加载器
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        // $Proxy0
        Object proxy = Proxy.newProxyInstance(classLoader, new Class[]{EchoService.class}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if (EchoService.class.isAssignableFrom(method.getDeclaringClass())) {
                    // 被代理对象
                    EchoService echoService = new DefaultEchoService();
                    long startTime = System.currentTimeMillis();
                    String result = echoService.echo((String) args[0]);
                    long costTime = System.currentTimeMillis() - startTime;
                    System.out.println("echo 方法执行的实现：" + costTime + " ms.");
                    return result;
                }
                return null;
            }
        });
        EchoService echoService = (EchoService) proxy;
        echoService.echo("Hello,World");
    }
}

```

控制台会输出以下内容：

```
echo 方法执行的实现：0 ms.
```

#### 分析

借助 JDK 的 `java.lang.reflect.Proxy` 来创建代理对象，调用 `Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)` 方法可以创建一个代理对象，方法的三个入参分别是：

- `ClassLoader loader`：用于加载代理对象的 Class 类加载器
- `Class<?>[] interfaces`：代理对象需要实现的接口
- `InvocationHandler h`：代理对象的处理器

新生成的代理对象的 Class 对象会继承 `Proxy`，且实现所有的入参 `interfaces` 中的接口，在实现的方法中实际是调用入参 `InvocationHandler` 的 `invoke(..)` 方法。

上面可以看到 InvocationHandler 是直接在入参中创建的，在 `invoke(..)` 方法中拦截 EchoService 的方法。这里的被代理对象是在其内部创建的，实际上我们可以在创建 InvocationHandler 实现类的时候进行设置，这里为了方便直接在内部创建。



#### 新生成的 Class 对象

JDK 动态代理在 JVM 运行时会新生成一个代理对象，那么我们先来看看这个代理对象的字节码，通过添加下面的 JVM 参数可以保存生成的代理对象的 .class 文件

```java
# jdk8及以前版本
-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true
# jdk8+
-Djdk.proxy.ProxyGenerator.saveGeneratedFiles=true
```

在同工程目录下你会发现有一个 .class 文件，如下：

```java
package com.sun.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;
import org.geekbang.thinking.in.spring.aop.overview.EchoService;

public final class $Proxy0 extends Proxy implements EchoService {
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;

    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String echo(String var1) throws  {
        try {
            return (String)super.h.invoke(this, m3, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m3 = Class.forName("org.geekbang.thinking.in.spring.aop.overview.EchoService").getMethod("echo", Class.forName("java.lang.String"));
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}

```

从这个代理对象中你可以看到，它继承了 `java.lang.reflect.Proxy` 这个类，并实现了 EchoService 接口。在实现的 `echo(String)` 方法中，实际上调用的就是父类 `Proxy` 中的 InvocationHandler 的 `incoke(..)` 方法，这个 InvocationHandler 就是创建代理对象时传入的参数。



#### Proxy 底层原理

##### newProxyInstance 方法

JDK 动态代理是通过 `Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)` 方法创建的代理对象，那我们来看看这个方法会做哪些事情

```java
// java.lang.reflect.Proxy.java
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
    throws IllegalArgumentException
{
    // InvocationHandler 不能为空
    Objects.requireNonNull(h);

    final Class<?>[] intfs = interfaces.clone();
    final SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }

    // 获取代理对象的 Class 对象（生成一个代理类）
    Class<?> cl = getProxyClass0(loader, intfs);

    try {
        if (sm != null) {
            // 安全检测，检测是否能够创建对象
            checkNewProxyPermission(Reflection.getCallerClass(), cl);
        }

        // 获取代理类的构造器（入参为 InvocationHandler）
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        final InvocationHandler ih = h;
        // 设置为可访问
        if (!Modifier.isPublic(cl.getModifiers())) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    cons.setAccessible(true);
                    return null;
                }
            });
        }
        // 通过构造器创建一个实例对象，入参是 InvocationHandler 的实现类
        return cons.newInstance(new Object[]{h});
    } catch (IllegalAccessException|InstantiationException e) {
        throw new InternalError(e.toString(), e);
    } catch (InvocationTargetException e) {
        Throwable t = e.getCause();
        if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        } else {
            throw new InternalError(t.toString(), t);
        }
    } catch (NoSuchMethodException e) {
        throw new InternalError(e.toString(), e);
    }
}
```

这个方法的逻辑不复杂，获取到代理类的 Class 对象，然后通过构造器创建一个代理对象，构造器的入参就是 InvocationHandler 的实现类。因为代理类会继承 Proxy这个类，在 Proxy 中就有一个 `Proxy(InvocationHandler h)` 构造方法，所以可以获取到对应的构造器。其中获取代理对象的 Class 对象（生成一个代理类）调用 `getProxyClass0(ClassLoader loader, Class<?>... interfaces)` 方法。



##### getProxyClass0 方法

```java
// java.lang.reflect.Proxy.java

/**
 * a cache of proxy classes
 */
private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
    proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());

private static Class<?> getProxyClass0(ClassLoader loader,
                                       Class<?>... interfaces) {
    // 接口数量不能大于 65535
    if (interfaces.length > 65535) {
        throw new IllegalArgumentException("interface limit exceeded");
    }

    // If the proxy class defined by the given loader implementing
    // the given interfaces exists, this will simply return the cached copy;
    // otherwise, it will create the proxy class via the ProxyClassFactory
    // 
    return proxyClassCache.get(loader, interfaces);
}
```

先尝试从 `proxyClassCache` 缓存中获取对应的代理类，如果不存在则通过 ProxyClassFactory 函数创建一个代理类



##### ProxyClassFactory 代理类工厂

```java
// java.lang.reflect.Proxy.java
private static final class ProxyClassFactory
    implements BiFunction<ClassLoader, Class<?>[], Class<?>>
{
    // prefix for all proxy class names
    private static final String proxyClassNamePrefix = "$Proxy";

    // next number to use for generation of unique proxy class names
    private static final AtomicLong nextUniqueNumber = new AtomicLong();

    @Override
    public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

        Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
        /*
         * 遍历需要实现的接口，进行校验
         */
        for (Class<?> intf : interfaces) {
            // 校验该接口是在存在这个 ClassLoader 中
            Class<?> interfaceClass = null;
            try {
                interfaceClass = Class.forName(intf.getName(), false, loader);
            } catch (ClassNotFoundException e) {
            }
            if (interfaceClass != intf) {
                throw new IllegalArgumentException(
                    intf + " is not visible from class loader");
            }
            // 校验是否真的是接口（因为入参也可以不传接口）
            if (!interfaceClass.isInterface()) {
                throw new IllegalArgumentException(
                    interfaceClass.getName() + " is not an interface");
            }
            
            // 校验是否出现相同的接口（因为入参也可以传相同的接口）
            if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                throw new IllegalArgumentException(
                    "repeated interface: " + interfaceClass.getName());
            }
        }

        String proxyPkg = null;     // package to define proxy class in
        int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

        /*
         * 遍历需要实现的接口，判断是否存在非 `public` 的接口
         * 如果存在，则记录下来，并记录所在的包名
         * 如果存在非 `public` 的接口，且还存在其他包路径下的接口，则抛出异常
         */
        for (Class<?> intf : interfaces) {
            int flags = intf.getModifiers();
            if (!Modifier.isPublic(flags)) {
                accessFlags = Modifier.FINAL;
                String name = intf.getName();
                int n = name.lastIndexOf('.');
                String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
                }
            }
        }

        // 如果不存在非 `public` 的接口，则代理类的名称前缀为 `com.sun.proxy.`
        if (proxyPkg == null) {
            // if no non-public proxy interfaces, use com.sun.proxy package
            proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
        }

        // 生成一个代理类的名称，`com.sun.proxy.$Proxy` + 唯一数字（从 0 开始递增）
        // 对于非 `public` 的接口，这里的名前缀就取原接口包名了，因为不是 `public` 修饰需要保证可访问
        long num = nextUniqueNumber.getAndIncrement();
        String proxyName = proxyPkg + proxyClassNamePrefix + num;

        // 根据代理类的名称、需要实现的接口以及修饰符生成一个字节数组
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
            proxyName, interfaces, accessFlags);
        try {
            // 根据代理类对应的字节数组创建一个 Class 对象
            return defineClass0(loader, proxyName,
                                proxyClassFile, 0, proxyClassFile.length);
        } catch (ClassFormatError e) {
            throw new IllegalArgumentException(e.toString());
        }
    }
}
```

1. 遍历需要实现的接口，进行校验；
   - 校验该接口是在存在这个 ClassLoader 中
   - 校验是否真的是接口（因为入参也可以不传接口）
   - 校验是否出现相同的接口（因为入参也可以传相同的接口）
2. 遍历需要实现的接口，判断是否存在非public的接口，该步骤和生成的代理类的名称有关；
   - 如果存在，则记录下来，并记录所在的包名
   - 如果存在非 `public` 的接口，且还存在其他包路径下的接口，则抛出异常
3. 如果不存在非 `public` 的接口，则代理类的名称前缀为 `com.sun.proxy.`
4. 生成一个代理类的名称，com.sun.proxy.$Proxy+ 唯一数字（从 0 开始递增）
   - 对于非 `public` 的接口，这里的名前缀就取原接口包名了，因为不是 `public` 修饰需要保证可访问
5. 根据代理类的名称、需要实现的接口以及修饰符生成一个字节数组
6. 根据第 `5` 步生成的代理类对应的字节数组创建一个 Class 对象

可以看到就是根据入参中的接口创建一个 Class 对象，实现这些接口，然后创建一个实例对象



### CGLIB 动态代理

JDK 动态代理的目标对象必须是一个接口，在我们日常生活中，无法避免开发人员不写接口直接写类，或者根本不需要接口，直接用类进行表达。这个时候我们就需要通过一些字节码提升的手段，来帮助做这个事情，在运行时，非编译时，来创建新的 Class 对象，这种方式称之为字节码提升。在 Spring 内部有两个字节码提升的框架，ASM（过于底层，直接操作字节码）和 CGLIB（相对于前者更加简便）。

CGLIB 动态代理则是基于类代理（字节码提升），通过 ASM（Java 字节码的操作和分析框架）将被代理类的 class 文件加载进来，通过修改其字节码生成子类来处理。

#### 示例

```java
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class CglibDynamicProxyDemo {

    public static void main(String[] args) {
        // 创建 CGLIB 增强对象
        Enhancer enhancer = new Enhancer();
        // 指定父类，也就是被代理的类
        Class<?> superClass = DefaultEchoService.class;
        enhancer.setSuperclass(superClass);
        // 指定回调接口（拦截器）
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object source, Method method, Object[] args,
                                    MethodProxy methodProxy) throws Throwable {
                long startTime = System.currentTimeMillis();
                Object result = methodProxy.invokeSuper(source, args);
                long costTime = System.currentTimeMillis() - startTime;
                System.out.println("[CGLIB 字节码提升] echo 方法执行的实现：" + costTime + " ms.");
                return result;
            }
        });

        // 创建代理对象
        EchoService echoService = (EchoService) enhancer.create();
        // 输出执行结果
        System.out.println(echoService.echo("Hello,World"));
    }
}
```

控制台会输出以下内容：

```
[CGLIB 字节码提升] echo 方法执行的实现：19 ms.
[ECHO] Hello,World
```

#### 分析

需要借助于 CGLIB 的 `org.springframework.cglib.proxy.Enhancer` 类来创建代理对象，设置以下几个属性：

- `Class<?> superClass`：被代理的类
- `Callback callback`：回调接口

新生成的代理对象的 Class 对象会继承 `superClass` 被代理的类，在重写的方法中会调用 `callback` 回调接口（方法拦截器）进行处理。

上面可以看到 Callback 是直接创建的，在 `intercept(..)` 方法中拦截 DefaultEchoService 的方法。因为 MethodInterceptor 继承了 Callback 回调接口，所以这里传入一个 MethodInterceptor 方法拦截器是没问题的。

> 注意，如果你想设置一个 Callback[] 数组去处理不同的方法，那么需要设置一个 CallbackFilter 筛选器，用于选择这个方法使用数组中的那个 Callback 去处理



#### 新生成的 Class 对象

CGLIB 动态代理在 JVM 运行时会新生成一个代理类，这个代理对象继承了目标类，也就是创建了一个目标类的子类，从而字节码得到提升。那么我们可以看看这个代理类的字节码，通过添加下面的 JVM 参数可以保存生成的代理类的 .class 文件

```java
-Dcglib.debugLocation=${class文件 保存路径}
```

在指定的保存路径下你会发现存在这么一个 .class 文件，如下：

```java
package org.geekbang.thinking.in.spring.aop.overview;

import java.lang.reflect.Method;
import org.springframework.cglib.core.ReflectUtils;
import org.springframework.cglib.core.Signature;
import org.springframework.cglib.proxy.Callback;
import org.springframework.cglib.proxy.Factory;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

public class DefaultEchoService$$EnhancerByCGLIB$$c868af31 extends DefaultEchoService implements Factory {
    private boolean CGLIB$BOUND;
    public static Object CGLIB$FACTORY_DATA;
    private static final ThreadLocal CGLIB$THREAD_CALLBACKS;
    private static final Callback[] CGLIB$STATIC_CALLBACKS;
    private MethodInterceptor CGLIB$CALLBACK_0;
    private static Object CGLIB$CALLBACK_FILTER;
    private static final Method CGLIB$echo$0$Method;
    private static final MethodProxy CGLIB$echo$0$Proxy;
    private static final Object[] CGLIB$emptyArgs;
    private static final Method CGLIB$equals$1$Method;
    private static final MethodProxy CGLIB$equals$1$Proxy;
    private static final Method CGLIB$toString$2$Method;
    private static final MethodProxy CGLIB$toString$2$Proxy;
    private static final Method CGLIB$hashCode$3$Method;
    private static final MethodProxy CGLIB$hashCode$3$Proxy;
    private static final Method CGLIB$clone$4$Method;
    private static final MethodProxy CGLIB$clone$4$Proxy;

    static void CGLIB$STATICHOOK1() {
        CGLIB$THREAD_CALLBACKS = new ThreadLocal();
        CGLIB$emptyArgs = new Object[0];
        Class var0 = Class.forName("org.geekbang.thinking.in.spring.aop.overview.DefaultEchoService$$EnhancerByCGLIB$$c868af31");
        Class var1;
        CGLIB$echo$0$Method = ReflectUtils.findMethods(new String[]{"echo", "(Ljava/lang/String;)Ljava/lang/String;"}, (var1 = Class.forName("org.geekbang.thinking.in.spring.aop.overview.DefaultEchoService")).getDeclaredMethods())[0];
        CGLIB$echo$0$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/String;)Ljava/lang/String;", "echo", "CGLIB$echo$0");
        Method[] var10000 = ReflectUtils.findMethods(new String[]{"equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;"}, (var1 = Class.forName("java.lang.Object")).getDeclaredMethods());
        CGLIB$equals$1$Method = var10000[0];
        CGLIB$equals$1$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/Object;)Z", "equals", "CGLIB$equals$1");
        CGLIB$toString$2$Method = var10000[1];
        CGLIB$toString$2$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/String;", "toString", "CGLIB$toString$2");
        CGLIB$hashCode$3$Method = var10000[2];
        CGLIB$hashCode$3$Proxy = MethodProxy.create(var1, var0, "()I", "hashCode", "CGLIB$hashCode$3");
        CGLIB$clone$4$Method = var10000[3];
        CGLIB$clone$4$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/Object;", "clone", "CGLIB$clone$4");
    }

    final String CGLIB$echo$0(String var1) {
        return super.echo(var1);
    }

    public final String echo(String var1) {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        return var10000 != null ? (String)var10000.intercept(this, CGLIB$echo$0$Method, new Object[]{var1}, CGLIB$echo$0$Proxy) : super.echo(var1);
    }

    final boolean CGLIB$equals$1(Object var1) {
        return super.equals(var1);
    }

    public final boolean equals(Object var1) {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
            Object var2 = var10000.intercept(this, CGLIB$equals$1$Method, new Object[]{var1}, CGLIB$equals$1$Proxy);
            return var2 == null ? false : (Boolean)var2;
        } else {
            return super.equals(var1);
        }
    }

    final String CGLIB$toString$2() {
        return super.toString();
    }

    public final String toString() {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        return var10000 != null ? (String)var10000.intercept(this, CGLIB$toString$2$Method, CGLIB$emptyArgs, CGLIB$toString$2$Proxy) : super.toString();
    }

    final int CGLIB$hashCode$3() {
        return super.hashCode();
    }

    public final int hashCode() {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
            Object var1 = var10000.intercept(this, CGLIB$hashCode$3$Method, CGLIB$emptyArgs, CGLIB$hashCode$3$Proxy);
            return var1 == null ? 0 : ((Number)var1).intValue();
        } else {
            return super.hashCode();
        }
    }

    final Object CGLIB$clone$4() throws CloneNotSupportedException {
        return super.clone();
    }

    protected final Object clone() throws CloneNotSupportedException {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        return var10000 != null ? var10000.intercept(this, CGLIB$clone$4$Method, CGLIB$emptyArgs, CGLIB$clone$4$Proxy) : super.clone();
    }

    public static MethodProxy CGLIB$findMethodProxy(Signature var0) {
        String var10000 = var0.toString();
        switch(var10000.hashCode()) {
        case -1042135322:
            if (var10000.equals("echo(Ljava/lang/String;)Ljava/lang/String;")) {
                return CGLIB$echo$0$Proxy;
            }
            break;
        case -508378822:
            if (var10000.equals("clone()Ljava/lang/Object;")) {
                return CGLIB$clone$4$Proxy;
            }
            break;
        case 1826985398:
            if (var10000.equals("equals(Ljava/lang/Object;)Z")) {
                return CGLIB$equals$1$Proxy;
            }
            break;
        case 1913648695:
            if (var10000.equals("toString()Ljava/lang/String;")) {
                return CGLIB$toString$2$Proxy;
            }
            break;
        case 1984935277:
            if (var10000.equals("hashCode()I")) {
                return CGLIB$hashCode$3$Proxy;
            }
        }

        return null;
    }

    public DefaultEchoService$$EnhancerByCGLIB$$c868af31() {
        CGLIB$BIND_CALLBACKS(this);
    }

    public static void CGLIB$SET_THREAD_CALLBACKS(Callback[] var0) {
        CGLIB$THREAD_CALLBACKS.set(var0);
    }

    public static void CGLIB$SET_STATIC_CALLBACKS(Callback[] var0) {
        CGLIB$STATIC_CALLBACKS = var0;
    }

    private static final void CGLIB$BIND_CALLBACKS(Object var0) {
        DefaultEchoService$$EnhancerByCGLIB$$c868af31 var1 = (DefaultEchoService$$EnhancerByCGLIB$$c868af31)var0;
        if (!var1.CGLIB$BOUND) {
            var1.CGLIB$BOUND = true;
            Object var10000 = CGLIB$THREAD_CALLBACKS.get();
            if (var10000 == null) {
                var10000 = CGLIB$STATIC_CALLBACKS;
                if (var10000 == null) {
                    return;
                }
            }

            var1.CGLIB$CALLBACK_0 = (MethodInterceptor)((Callback[])var10000)[0];
        }

    }

    public Object newInstance(Callback[] var1) {
        CGLIB$SET_THREAD_CALLBACKS(var1);
        DefaultEchoService$$EnhancerByCGLIB$$c868af31 var10000 = new DefaultEchoService$$EnhancerByCGLIB$$c868af31();
        CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
        return var10000;
    }

    public Object newInstance(Callback var1) {
        CGLIB$SET_THREAD_CALLBACKS(new Callback[]{var1});
        DefaultEchoService$$EnhancerByCGLIB$$c868af31 var10000 = new DefaultEchoService$$EnhancerByCGLIB$$c868af31();
        CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
        return var10000;
    }

    public Object newInstance(Class[] var1, Object[] var2, Callback[] var3) {
        CGLIB$SET_THREAD_CALLBACKS(var3);
        DefaultEchoService$$EnhancerByCGLIB$$c868af31 var10000 = new DefaultEchoService$$EnhancerByCGLIB$$c868af31;
        switch(var1.length) {
        case 0:
            var10000.<init>();
            CGLIB$SET_THREAD_CALLBACKS((Callback[])null);
            return var10000;
        default:
            throw new IllegalArgumentException("Constructor not found");
        }
    }

    public Callback getCallback(int var1) {
        CGLIB$BIND_CALLBACKS(this);
        MethodInterceptor var10000;
        switch(var1) {
        case 0:
            var10000 = this.CGLIB$CALLBACK_0;
            break;
        default:
            var10000 = null;
        }

        return var10000;
    }

    public void setCallback(int var1, Callback var2) {
        switch(var1) {
        case 0:
            this.CGLIB$CALLBACK_0 = (MethodInterceptor)var2;
        default:
        }
    }

    public Callback[] getCallbacks() {
        CGLIB$BIND_CALLBACKS(this);
        return new Callback[]{this.CGLIB$CALLBACK_0};
    }

    public void setCallbacks(Callback[] var1) {
        this.CGLIB$CALLBACK_0 = (MethodInterceptor)var1[0];
    }

    static {
        CGLIB$STATICHOOK1();
    }
}

```

你会看到这个代理类继承了 DefaultEchoService 目标类，在重写的 `echo(String)` 方法中会调用 Callback 的 `intercept(..)` 拦截方法进行处理，由于生成的代理类不是那么容易理解，这里就不做分析了，有一个大致的思路就可以，感兴趣的可以研究研究。

#### Enhancer 底层原理

##### create() 方法

CGLIB 动态代理可以通过 `Enhancer.create()` 方法进行字节码提升，该过程比较复杂，不易看懂，我们简单看看做了什么事情

```java
// org.springframework.cglib.proxy.Enhancer.java

public Object create() {
    this.classOnly = false;
    this.argumentTypes = null;
    return this.createHelper();
}

private Object createHelper() {
    // Callback 的校验
    this.preValidate();
    // 创建一个 EnhancerKey 对象，主要设置代理类名称、和 Cllback 的类
    Object key = KEY_FACTORY.newInstance(this.superclass != null ? this.superclass.getName() : null,
                                         ReflectUtils.getNames(this.interfaces), 
                                         this.filter == ALL_ZERO ? null : new WeakCacheKey(this.filter), 
                                         this.callbackTypes, 
                                         this.useFactory, 
                                         this.interceptDuringConstruction, 
                                         this.serialVersionUID);
    this.currentKey = key;
    // 创建一个代理对象（代理类的子类）
    Object result = super.create(key);
    return result;
}
```



##### preValidate 方法

```java
// org.springframework.cglib.proxy.Enhancer.java

private static final CallbackFilter ALL_ZERO = new CallbackFilter() {
    public int accept(Method method) {
        return 0;
    }
};

private void preValidate() {
    // 设置 Callback 的类型
    if (this.callbackTypes == null) {
        this.callbackTypes = CallbackInfo.determineTypes(this.callbacks, false);
        this.validateCallbackTypes = true;
    }

    // 如果 Callback 筛选器为空，则设置为取第一个
    // `filter` 用于筛选拦截方法所对应的 Callback
    if (this.filter == null) {
        if (this.callbackTypes.length > 1) {
            throw new IllegalStateException("Multiple callback types possible but no filter specified");
        }

        this.filter = ALL_ZERO;
    }
}
```



##### create(Object) 方法

```java
protected Object create(Object key) {
    try {
        // 获取类加载器
        ClassLoader loader = this.getClassLoader();
        Map<ClassLoader, AbstractClassGenerator.ClassLoaderData> cache = CACHE;
        // 获取 ClassLoaderData 类加载器数据
        AbstractClassGenerator.ClassLoaderData data = (AbstractClassGenerator.ClassLoaderData)cache.get(loader);
        if (data == null) {
            Class var5 = AbstractClassGenerator.class;
            synchronized(AbstractClassGenerator.class) {
                cache = CACHE;
                data = (AbstractClassGenerator.ClassLoaderData)cache.get(loader);
                if (data == null) {
                    Map<ClassLoader, AbstractClassGenerator.ClassLoaderData> newCache = new WeakHashMap(cache);
                    data = new AbstractClassGenerator.ClassLoaderData(loader);
                    newCache.put(loader, data);
                    CACHE = newCache;
                }
            }
        }

        this.key = key;
        // 通过类加载器根据 `key` 创建一个 Class 对象（例如 FastClassByCGLIB）
        Object obj = data.get(this, this.getUseCache());
        // 通过 Class 对象创建一个代理对象
        return obj instanceof Class ? this.firstInstance((Class)obj) : this.nextInstance(obj);
    } catch (Error | RuntimeException var9) {
        throw var9;
    } catch (Exception var10) {
        throw new CodeGenerationException(var10);
    }
}
```



整个过程比较复杂，上面没有深入分析，因为笔者实在是看不懂~😢，可以先理解为创建一个目标类的子类😈

可参考 [cglib动态代理的使用和分析](https://www.cnblogs.com/ZhangZiSheng001/p/11917086.html) 这篇文章



### Javassist 动态代理

[Javassist](http://www.javassist.org/) 和 CGLIB 一样，是一个操作 Java 字节码的类库，支持 JVM 运行时创建 Class 对象，实现动态代理当然不在话下。例如 Dubbo 就是默认使用 Javassist 来进行动态代理的

#### 使用示例

我们来看一下 Javassist 的使用示例：

```java
package com.study.proxy;

import javassist.*;

import java.lang.reflect.Method;

public class JavassistTest {

    public static void main(String[] args) throws Exception {
        // 获取默认的单例 ClassPool 对象
        ClassPool cp = ClassPool.getDefault();
        // 使用 ClassPool 生成指定名称的 CtClass 对象，用于创建 Class 对象
        CtClass clazz = cp.makeClass("com.study.proxy.Person");

        // 创建一个字段，设置类型和名称
        CtField field = new CtField(cp.get("java.lang.String"), "name", clazz);
        // 设置该字段的修饰符
        field.setModifiers(Modifier.PRIVATE);

        // 为 CtClass 添加两个方法，上面 `field` 字段的 setter、getter 方法
        clazz.addMethod(CtNewMethod.setter("setName", field));
        clazz.addMethod(CtNewMethod.getter("getName", field));
        // 将 `field` 字段添加到 CtClass 对象中，并设置默认值
        clazz.addField(field, CtField.Initializer.constant("jingping"));

        // 创建一个构造方法，指定参数类型、所属类
        CtConstructor ctConstructor = new CtConstructor(new CtClass[]{}, clazz);
        StringBuffer body = new StringBuffer();
        body.append("{\n this.name=\"liujingping\"; \n}");
        // 设置构造器的内容
        ctConstructor.setBody(body.toString());
        // 将构造器添加到 CtClass 对象中
        clazz.addConstructor(ctConstructor);

        // 创建一个方法，指定返回类型、名称、参数类型、所属类
        CtMethod ctMethod = new CtMethod(CtClass.voidType, "say", new CtClass[]{}, clazz);
        // 设置方法的修饰符
        ctMethod.setModifiers(Modifier.PUBLIC);
        body = new StringBuffer();
        body.append("{\n System.out.println(\"say: hello, \" + this.name); \n}");
        // 设置方法的内容
        ctMethod.setBody(body.toString());
        // 将方法添加到 CtClass 对象中
        clazz.addMethod(ctMethod);
        // 设置 .class 文件的保存路径
        clazz.writeFile(".");

        // 通过 CtClass 对象创建一个 Class 对象，无法继续修改 Class 对象
        Class<?> c = clazz.toClass();
        // 创建一个实例对象
        Object obj = c.newInstance();
        Method get = obj.getClass().getMethod("getName");
        // 输出 liujingping
        System.out.println(get.invoke(obj));
        Method say = obj.getClass().getDeclaredMethod("say");
        // 输出 say: hello, liujingping
        say.invoke(obj);
        Method set = obj.getClass().getDeclaredMethod("setName", String.class);
        set.invoke(obj, "liujingping2");
        // 输出 liujingping2
        System.out.println(get.invoke(obj));
        // 输出 say: hello, liujingping2
        say.invoke(obj);
    }
}
```

上面会创建一个 `com.study.proxy.Person` Class 对象，如下：

```java
package com.study.proxy;

public class Person {
    private String name = "jingping";

    public void setName(String var1) {
        this.name = var1;
    }

    public String getName() {
        return this.name;
    }

    public Person() {
        this.name = "liujingping";
    }

    public void say() {
        System.out.println("say: hello, " + this.name);
    }
}
```

#### 动态代理

```java
package com.study.proxy;

import javassist.util.proxy.ProxyFactory;

import java.lang.reflect.Method;

public class JavassistProxy {

    public void say() {
        System.out.println("hello, liujingping");
    }

    public static void main(String[] args) throws Exception {
        ProxyFactory factory = new ProxyFactory();
        // 指定代理类的父类
        factory.setSuperclass(JavassistProxy.class);
        // 设置方法过滤器，用于判断是否拦截方法
        factory.setFilter((Method method) -> {
            // 拦截 `say` 方法
            return "say".equals(method.getName());
        });
        factory.getClass().getDeclaredField("writeDirectory").set(factory, ".");
        JavassistProxy proxy = (JavassistProxy) factory.create(new Class<?>[]{}, new Object[]{}, (o, method, method1, objects) -> {
            System.out.println("前置处理");
            Object result = method1.invoke(o, objects);
            System.out.println("后置处理");
            return result;
        });
        proxy.say();
    }
}

```

控制台会输出：

```
前置处理
hello, liujingping
后置处理
```

#### 新生成的 Class 对象

上面我通过反射设置了 `ProxyFactory` 的 `writeDirectory` 属性为 `.`，没找到这个属性的写入方法😂，那么会在当前工程保存一个同包名的 .class 文件，如下：

```java
package com.study.proxy;

import java.io.ObjectStreamException;
import java.lang.reflect.Method;
import javassist.util.proxy.MethodHandler;
import javassist.util.proxy.ProxyObject;
import javassist.util.proxy.RuntimeSupport;

public class JavassistProxy_$$_jvst475_0 extends JavassistProxy implements ProxyObject {
    private MethodHandler handler;
    public static byte[] _filter_signature;
    public static final long serialVersionUID;
    private static Method[] _methods_;

    public JavassistProxy_$$_jvst475_0() {
        this.handler = RuntimeSupport.default_interceptor;
        super();
    }

    public final void _d8say() {
        super.say();
    }

    public final void say() {
        Method[] var1 = _methods_;
        this.handler.invoke(this, var1[16], var1[17], new Object[0]);
    }

    static {
        Method[] var0 = new Method[26];
        Class var1 = Class.forName("com.fullmoon.study.proxy.JavassistProxy_$$_jvst475_0");
        RuntimeSupport.find2Methods(var1, "say", "_d8say", 16, "()V", var0);
        _methods_ = var0;
        serialVersionUID = -1L;
    }

    public void setHandler(MethodHandler var1) {
        this.handler = var1;
    }

    public MethodHandler getHandler() {
        return this.handler;
    }

    Object writeReplace() throws ObjectStreamException {
        return RuntimeSupport.makeSerializedProxy(this);
    }
}
```

可以看到会继承 JavassistProxy 目标类，并实现 Javassist 的 ProxyObject 接口，在 `say()` 方法中会调用 MethodHandler 处理类进行拦截处理，这里我们仅做了解，因为 Spring AOP 中没有使用到这种动态代理。



### 总结

本文对 JDK 动态代理和 CGLIB 动态代理进行了浅显的分析，可以得出以下结论：

- **静态代理**通过实现被代理类所实现的接口，内部保存被代理类的引用，在实现的方法中对处理逻辑进行**增强**，真正的方法执行调用被代理对象的方法。**静态代理**比较简洁直观，不过每个目标对象都需要创建一个代理类，在复杂的场景下需要创建大量的代理类，不易于维护，也不易于扩展，我们更加注重的应该是业务开发，对于这一层**增强**处理应该抽取出来。
- **JDK 动态代理**基于接口代理，在 JVM 运行时通过反射机制生成一个实现代理接口的类，在调用具体方法时会调用 InvokeHandler 来进行处理。
- **CGLIB 动态代理**基于类代理（字节码提升），通过 ASM（Java 字节码的操作和分析框架）将被代理类的 class 文件加载进来，通过修改其字节码生成一个子类来处理。













