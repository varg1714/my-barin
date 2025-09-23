---
source: https://zhuanlan.zhihu.com/p/447842296
create: 2025-09-23 12:27
read: true
knowledge: true
knowledge-date: 2025-09-23
tags:
  - Java
summary: "[[Java 动态代理知识学习]]"
---
## 什么是 CGLIB

CGLIB 是一个强大的、高性能的代码生成库。采用非常底层的字节码技术，对指定目标类生成一个子类，并对子类进行增强，其被广泛应用于 AOP 框架（Spring、dynaop）中，用以提供方法拦截操作。CGLIB 作为一个开源项目，其代码托管在 github，地址为：[https://github.com/cglib/cglib](https://github.com/cglib/cglib)

CGLIB 代理主要通过对字节码的操作，以控制对象的访问。 CGLIB 底层使用了 ASM（一个短小精悍的字节码操作框架）来操作字节码生成新的类。 CGLIB 相比于 JDK 动态代理更加强大： Java 动态代理使用 Java 原生的反射 API 进行操作（运行期），在生成类上比较高效。 CGLIB 使用 ASM 框架直接对字节码进行操作（编译期），在类的执行过程中比较高效

![](https://pic3.zhimg.com/v2-23e457a4fa7f99680bc393d5ff600cfe_r.jpg)

## 为什么使用 CGLIB

CGLIB 代理主要通过对字节码的操作，为对象引入间接级别，以控制对象的访问。我们知道 Java 中有一个动态代理也是做这个事情的，那我们为什么不直接使用 Java 动态代理，而要使用 CGLIB 呢？答案是 CGLIB 相比于 JDK 动态代理更加强大，JDK 动态代理虽然简单易用，但是其有一个致命缺陷是，只能对接口进行代理。如果要代理的类为一个普通类、没有接口，那么 Java 动态代理就没法使用了。

### CGLib 的基本使用

引入 cglib 的依赖

```
<dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
        <version>2.2.2</version>
</dependency>
```

1、目标类

```
/**
 * 被代理类（目标类）
 */
public class Service {

    /**
     *  final 方法不能被子类覆盖
     */
    public final void finalMethod() {
        System.out.println("Service.finalMethod 执行了");
    }

    public void publicMethod() {
        System.out.println("Service.publicMethod 执行了");
    }
}
```

2、代理类

```
/**
 * 代理类
 */
public class CglibDynamicProxy implements MethodInterceptor {

    /**
     * 目标对象（也被称为被代理对象）
     */
    private Object target;

    public CglibDynamicProxy(Object target) {
        this.target = target;
    }

    /**
     * 如何增强
     * @param obj 代理对象引用
     * @param method 被代理对象的方法的描述引用
     * @param args 方法参数
     * @param proxy 代理对象 对目标对象的方法的描述
     * @return
     * @throws Throwable
     */
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("CglibDynamicProxy intercept 方法执行前-------------------------------");

        System.out.println("obj = " + obj.getClass());
        System.out.println("method = " + method);
        System.out.println("proxy = " + proxy);

        Object object = proxy.invoke(target, args);
        System.out.println("CglibDynamicProxy intercept 方法执行后-------------------------------");
        return object;
    }

    /**
     * 获取被代理接口实例对象
     *
     * 通过 enhancer.create 可以获得一个代理对象，它继承了 target.getClass() 类
     *
     * @param <T>
     * @return
     */
    public <T> T getProxy() {
        Enhancer enhancer = new Enhancer();
        //设置被代理类
        enhancer.setSuperclass(target.getClass());
        // 设置回调
        enhancer.setCallback(this);
        // create方法正式创建代理类
        return (T) enhancer.create();
    }
}
```

3、测试方法

```
public static void main(String[] args) {
        // 1. 构造目标对象
        Service target = new Service();

        // 2. 根据目标对象生成代理对象
        CglibDynamicProxy proxy = new CglibDynamicProxy(target);

        // 获取 CGLIB 代理类
        Service proxyObject = proxy.getProxy();

        // 调用代理对象的方法
        proxyObject.finalMethod();
        proxyObject.publicMethod();
    }
```

输出结果

Service.finalMethod 执行了 CglibDynamicProxy intercept 方法执行前 ------------------------------- obj = class com.demo.cglib.Service\$\$EnhancerByCGLIB\$\$ bdf5414d method = public void com.demo.cglib.Service.publicMethod() proxy = net.sf.cglib.proxy.MethodProxy@1ed6993a Service.publicMethod 执行了 CglibDynamicProxy intercept 方法执行后 -------------------------------  

### 基本原理

CGLIB 动态代理的实现机制是生成目标类的子类，通过调用父类（目标类）的方法实现，在调用父类方法时再代理中进行增强。

实现 CGLIB 动态代理必须实现 MethodInterceptor(方法拦截器) 接口，源码如下:

```
public interface MethodInterceptor extends Callback{

 public Object intercept(Object obj, java.lang.reflect.Method method, Object[] args,
                               MethodProxy proxy) throws Throwable;

}
```

这个接口只有一个 intercept() 方法，这个方法有 4 个参数：

1）obj 表示增强的对象，即实现这个接口类的一个对象；

2）method 表示要被拦截的方法；

3）args 表示要被拦截方法的参数；

4）proxy 表示要触发父类的方法对象；

## 反编译 class

在 main 方法内的第一行我们也可以添加如下设置，来存储代理类的 class 文件。

```
// 代理类class文件存入本地磁盘，可反编译查看源码
System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "d:\\temp");
```

再次执行程序，我们可以看到在对应目录下生成三个 class 文件：

![](https://pic4.zhimg.com/v2-6c8843ea260e130a39916be3b6529883_r.jpg)

反编译后

```
//  CGLIB 生成的代理类
public class Service$$EnhancerByCGLIB$$bdf5414d extends Service implements Factory {
    private boolean CGLIB$BOUND;
    private static final ThreadLocal CGLIB$THREAD_CALLBACKS;
    private static final Callback[] CGLIB$STATIC_CALLBACKS;
    private MethodInterceptor CGLIB$CALLBACK_0; // 拦截器
    private static final Method CGLIB$publicMethod$0$Method; // 被代理方法
    private static final MethodProxy CGLIB$publicMethod$0$Proxy; // 代理方法
    private static final Object[] CGLIB$emptyArgs;
    private static final Method CGLIB$finalize$1$Method;
    private static final MethodProxy CGLIB$finalize$1$Proxy;
    private static final Method CGLIB$equals$2$Method;
    private static final MethodProxy CGLIB$equals$2$Proxy;
    private static final Method CGLIB$toString$3$Method;
    private static final MethodProxy CGLIB$toString$3$Proxy;
    private static final Method CGLIB$hashCode$4$Method;
    private static final MethodProxy CGLIB$hashCode$4$Proxy;
    private static final Method CGLIB$clone$5$Method;
    private static final MethodProxy CGLIB$clone$5$Proxy;

    static void CGLIB$STATICHOOK1() {
        CGLIB$THREAD_CALLBACKS = new ThreadLocal();
        CGLIB$emptyArgs = new Object[0];
        // 代理类
        Class var0 = Class.forName("com.demo.cglib.Service$$EnhancerByCGLIB$$bdf5414d");
        // 被代理类
        Class var1;
        Method[] var10000 = ReflectUtils.findMethods(new String[]{"finalize", "()V", "equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;"}, (var1 = Class.forName("java.lang.Object")).getDeclaredMethods());
        CGLIB$finalize$1$Method = var10000[0];
        CGLIB$finalize$1$Proxy = MethodProxy.create(var1, var0, "()V", "finalize", "CGLIB$finalize$1");
        CGLIB$equals$2$Method = var10000[1];
        CGLIB$equals$2$Proxy = MethodProxy.create(var1, var0, "(Ljava/lang/Object;)Z", "equals", "CGLIB$equals$2");
        CGLIB$toString$3$Method = var10000[2];
        CGLIB$toString$3$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/String;", "toString", "CGLIB$toString$3");
        CGLIB$hashCode$4$Method = var10000[3];
        CGLIB$hashCode$4$Proxy = MethodProxy.create(var1, var0, "()I", "hashCode", "CGLIB$hashCode$4");
        CGLIB$clone$5$Method = var10000[4];
        CGLIB$clone$5$Proxy = MethodProxy.create(var1, var0, "()Ljava/lang/Object;", "clone", "CGLIB$clone$5");
        CGLIB$publicMethod$0$Method = ReflectUtils.findMethods(new String[]{"publicMethod", "()V"}, (var1 = Class.forName("com.demo.cglib.Service")).getDeclaredMethods())[0];
        CGLIB$publicMethod$0$Proxy = MethodProxy.create(var1, var0, "()V", "publicMethod", "CGLIB$publicMethod$0");
    }

    // 代理方法（methodProxy.invokeSuper会调用）
    final void CGLIB$publicMethod$0() {
        super.publicMethod();
    }

    // 被代理方法(methodProxy.invoke 会调用)
    public final void publicMethod() {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
            var10000.intercept(this, CGLIB$publicMethod$0$Method, CGLIB$emptyArgs, CGLIB$publicMethod$0$Proxy);
        } else {
            super.publicMethod();
        }
    }

    final void CGLIB$finalize$1() throws Throwable {
        super.finalize();
    }

    protected final void finalize() throws Throwable {
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (var10000 == null) {
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }

        if (var10000 != null) {
            var10000.intercept(this, CGLIB$finalize$1$Method, CGLIB$emptyArgs, CGLIB$finalize$1$Proxy);
        } else {
            super.finalize();
        }
    }
```

可以看出代理类是目标类的子类，调用 intercept() 方法，intercept() 方法由自定义的 CglibDynamicProxy 实现，所以最后调用 CglibDynamicProxy 中的 intercept() 方法，从而完成了由代理对象访问到目标对象的动态代理实现。

MethodProxy 非常关键，我们分析一下它具体做了什么。

```
public class MethodProxy {
    private Signature sig1;
    private Signature sig2;
    private CreateInfo createInfo;

    private final Object initLock = new Object();
    private volatile FastClassInfo fastClassInfo;

    /**
     *
     * @param c1  被代理对象 Class
     * @param c2  CGLIB 代理对象 Class
     * @param desc 入参类型
     * @param name1 有拦截器逻辑的方法名 publicMethod
     * @param name2 直接调用父类方法的方法名 CGLIB$publicMethod$0
     * @return
     */
    public static MethodProxy create(Class c1, Class c2, String desc, String name1, String name2) {
        MethodProxy proxy = new MethodProxy();
        proxy.sig1 = new Signature(name1, desc);
        proxy.sig2 = new Signature(name2, desc);
        proxy.createInfo = new CreateInfo(c1, c2);
        return proxy;
    }

    private void init() {
          CreateInfo ci = createInfo;
          FastClassInfo fci = new FastClassInfo();
          // 1. 生成被代理类 FastClass，在这里是 Service$$FastClassByCGLIB$$fdf36b96
          fci.f1 = helper(ci, ci.c1);
          // 2. 生成代理类 FastClass，在这里是 Service$$EnhancerByCGLIB$$bdabbd96$$FastClassByCGLIB$$a112cdb2
          fci.f2 = helper(ci, ci.c2);
          // 3. 得到被代理类 FastClass 中的 publicMethod 方法签名
          fci.i1 = fci.f1.getIndex(sig1);
          // 4. 得到代理类 FastClass 中的 CGLIB$publicMethod$0 方法签名
          fci.i2 = fci.f2.getIndex(sig2);
          fastClassInfo = fci;
          createInfo = null;
    }

    private static class FastClassInfo {
        FastClass f1; //被代理类 FastClass，在这里是 Service$$FastClassByCGLIB$$fdf36b96
        FastClass f2; //代理类 FastClass，在这里是 Service$$EnhancerByCGLIB$$bdabbd96$$FastClassByCGLIB$$a112cdb2
        int i1; // 被代理类 FastClass 中的 publicMethod方法签名(index)
        int i2; // 代理类 FastClass 中的 CGLIB$publicMethod$0 的方法签名
    }

    /**
     * 在相同类型的不同对象上调用原始方法。如果传入的是代理对象，则调用的是 CGLIB 代理类重写的方法     
     * @param obj 方法调用的对象，如果使用 MethodInterceptor 的 intercept 方法的第一个参数，将导致递归
     * @param args 参数列表
     */
    public Object invoke(Object obj, Object[] args) throws Throwable {
        try {
            init();
            FastClassInfo fci = fastClassInfo;
            // 调用 Service$$FastClassByCGLIB$$fdf36b96 类中的 publicMethod 方法
            return fci.f1.invoke(fci.i1, obj, args);
        } catch (InvocationTargetException e) {
            throw e.getTargetException();
        } catch (IllegalArgumentException e) {
            if (fastClassInfo.i1 < 0)
                throw new IllegalArgumentException("Protected method: " + sig1);
            throw e;
        }
    }

    /**
     * 在代理对象上调用父类的原始方法
     * @param obj CGLIB 生成的代理对象
     * @param args 参数列表
     */
    public Object invokeSuper(Object obj, Object[] args) throws Throwable {
        try {
            init();
            FastClassInfo fci = fastClassInfo;
           //调用Service$$EnhancerByCGLIB$$bdabbd96$$FastClassByCGLIB$$a112cdb2类中的CGLIB$publicMethod$0 方法
            return fci.f2.invoke(fci.i2, obj, args);
        } catch (InvocationTargetException e) {
            throw e.getTargetException();
        }
    }
}
```

### FastClass 机制

CGLIB 动态代理执行代理方法效率之所以比 JDK 动态代理高，是因为 CGLIB 采用了 FastClass 机制。

FastClass 的原理简单来说就是：为代理类和被代理类各生成一个 Class，这个 Class 会为代理类或被代理类的方法分配一个 index，根据这个 index，FastClass 就可以直接定位要调用的方法直接进行调用，这样**省去了反射调用**，所以调用效率比 JDK 动态代理通过反射调用高。

Service\$\$FastClassByCGLIB\$\$ e18dee0a.class 是被代理类的 FastClass。

Service\$\$ EnhancerByCGLIB\$\$bdf5414d \$\$FastClassByCGLIB\$\$97a6d33a.class 是代理类的 FastClass。

先来看一下被代理类的 FastClass:

```
public class Service$$FastClassByCGLIB$$e18dee0a extends FastClass {
    public Service$$FastClassByCGLIB$$e18dee0a(Class var1) {
        super(var1);
    }

    // 根据方法签名，获取到 FastClass 中的方法 index 值
    public int getIndex(Signature var1) {
        String var10000 = var1.toString();
        switch(var10000.hashCode()) {

        case -560613858:
            if (var10000.equals("finalMethod()V")) {
                return 1;
            }
            break;
        case -433379701:
            if (var10000.equals("publicMethod()V")) {
                return 0;
            }
            break;


        return -1;
    }

    public int getIndex(String var1, Class[] var2) {
        switch(var1.hashCode()) {
        case -1776922004:
            if (var1.equals("toString")) {
                switch(var2.length) {
                case 0:
                    return 6;
                }
            }
            break;
        case -1295482945:
            if (var1.equals("equals")) {
                switch(var2.length) {
                case 1:
                    if (var2[0].getName().equals("java.lang.Object")) {
                        return 5;
                    }
                }
            }
            break;

        case 147696667:
            if (var1.equals("hashCode")) {
                switch(var2.length) {
                case 0:
                    return 7;
                }
            }
            break;
        case 348732586:
            if (var1.equals("publicMethod")) {
                switch(var2.length) {
                case 0:
                    return 0;
                }
            }
            break;
        case 1900285431:
            if (var1.equals("finalMethod")) {
                switch(var2.length) {
                case 0:
                    return 1;
                }
            }
            break;

        case 1950568386:
            if (var1.equals("getClass")) {
                switch(var2.length) {
                case 0:
                    return 8;
                }
            }
        }

        return -1;
    }

    public int getIndex(Class[] var1) {
        switch(var1.length) {
        case 0:
            return 0;
        default:
            return -1;
        }
    }
   /**
     * 调用 FastClass 中的方法
     * @param var1 方法 index 值
     * @param var2 方法所在的类的对象
     * @param var3 参数列表
     */
    public Object invoke(int var1, Object var2, Object[] var3) throws InvocationTargetException {
        // 强制转换 Object 为 Service 类型，从而可以直接调用 Service 中定义的方法
        Service var10000 = (Service)var2;
        int var10001 = var1;

        try {
            switch(var10001) {
            case 0:
                var10000.publicMethod();
                return null;
            case 1:
                var10000.finalMethod();
                return null;
            case 2:
                var10000.wait();
                return null;
            case 3:
                var10000.wait(((Number)var3[0]).longValue(), ((Number)var3[1]).intValue());
                return null;
            case 4:
                var10000.wait(((Number)var3[0]).longValue());
                return null;
            case 5:
                return new Boolean(var10000.equals(var3[0]));
            case 6:
                return var10000.toString();
            case 7:
                return new Integer(var10000.hashCode());
            case 8:
                return var10000.getClass();
            case 9:
                var10000.notify();
                return null;
            case 10:
                var10000.notifyAll();
                return null;
            }
        } catch (Throwable var4) {
            throw new InvocationTargetException(var4);
        }

        throw new IllegalArgumentException("Cannot find matching method/constructor");
    }

    public Object newInstance(int var1, Object[] var2) throws InvocationTargetException {
        Service var10000 = new Service;
        Service var10001 = var10000;
        int var10002 = var1;

        try {
            switch(var10002) {
            case 0:
                var10001.<init>();
                return var10000;
            }
        } catch (Throwable var3) {
            throw new InvocationTargetException(var3);
        }

        throw new IllegalArgumentException("Cannot find matching method/constructor");
    }

    public int getMaxIndex() {
        return 10;
    }
}
```

再来看一下代理类的 FastClass：

```
public class Service$$EnhancerByCGLIB$$bdf5414d$$FastClassByCGLIB$$97a6d33a extends FastClass {
    public Service$$EnhancerByCGLIB$$bdf5414d$$FastClassByCGLIB$$97a6d33a(Class var1) {
        super(var1);
    }
    // 根据方法签名，获取到 FastClass 中的方法 index 值 
    public int getIndex(Signature var1) {
        String var10000 = var1.toString();
        switch(var10000.hashCode()) {

        case -894172689:
            if (var10000.equals("newInstance(Lnet/sf/cglib/proxy/Callback;)Ljava/lang/Object;")) {
                return 3;
            }
            break;
        case -623122092:
            if (var10000.equals("CGLIB$findMethodProxy(Lnet/sf/cglib/core/Signature;)Lnet/sf/cglib/proxy/MethodProxy;")) {
                return 6;
            }
            break;
        case -560613858:
            if (var10000.equals("finalMethod()V")) {
                return 21;
            }
            break;
        case -433379701:
            if (var10000.equals("publicMethod()V")) {
                return 7;
            }
            break;
        case -419626537:
            if (var10000.equals("setCallbacks([Lnet/sf/cglib/proxy/Callback;)V")) {
                return 9;
            }
            break;

        case 1984935277:
            if (var10000.equals("hashCode()I")) {
                return 2;
            }
            break;
        case 2011844968:
            if (var10000.equals("CGLIB$clone$5()Ljava/lang/Object;")) {
                return 17;
            }
        }

        return -1;
    }

    public int getIndex(String var1, Class[] var2) {
        switch(var1.hashCode()) {
        case -2083498450:
            if (var1.equals("CGLIB$finalize$1")) {
                switch(var2.length) {
                case 0:
                    return 14;
                }
            }
            break;
        case -1776922004:
            if (var1.equals("toString")) {
                switch(var2.length) {
                case 0:
                    return 1;
                }
            }
            break;
        case -1295482945:
            if (var1.equals("equals")) {
                switch(var2.length) {
                case 1:
                    if (var2[0].getName().equals("java.lang.Object")) {
                        return 0;
                    }
                }
            }
            break;
        case -1053468136:
            if (var1.equals("getCallbacks")) {
                switch(var2.length) {
                case 0:
                    return 10;
                }
            }
            break;

        case -683767239:
            if (var1.equals("CGLIB$publicMethod$0")) {
                switch(var2.length) {
                case 0:
                    return 20;
                }
            }
            break;
        case -124978608:
            if (var1.equals("CGLIB$equals$2")) {
                switch(var2.length) {
                case 1:
                    if (var2[0].getName().equals("java.lang.Object")) {
                        return 15;
                    }
                }
            }
            break;
        case -60403779:
            if (var1.equals("CGLIB$SET_STATIC_CALLBACKS")) {
                switch(var2.length) {
                case 1:
                    if (var2[0].getName().equals("[Lnet.sf.cglib.proxy.Callback;")) {
                        return 12;
                    }
                }
            }
            break;
        case -29025554:
            if (var1.equals("CGLIB$hashCode$4")) {
                switch(var2.length) {
                case 0:
                    return 16;
                }
            }
            break;

        case 85179481:
            if (var1.equals("CGLIB$SET_THREAD_CALLBACKS")) {
                switch(var2.length) {
                case 1:
                    if (var2[0].getName().equals("[Lnet.sf.cglib.proxy.Callback;")) {
                        return 13;
                    }
                }
            }
            break;
        case 147696667:
            if (var1.equals("hashCode")) {
                switch(var2.length) {
                case 0:
                    return 2;
                }
            }
            break;

        case 348732586:
            if (var1.equals("publicMethod")) {
                switch(var2.length) {
                case 0:
                    return 7;
                }
            }
            break;
        case 495524492:
            if (var1.equals("setCallbacks")) {
                switch(var2.length) {
                case 1:
                    if (var2[0].getName().equals("[Lnet.sf.cglib.proxy.Callback;")) {
                        return 9;
                    }
                }
            }
            break;
        case 1154623345:
            if (var1.equals("CGLIB$findMethodProxy")) {
                switch(var2.length) {
                case 1:
                    if (var2[0].getName().equals("net.sf.cglib.core.Signature")) {
                        return 6;
                    }
                }
            }
            break;
        case 1543336190:
            if (var1.equals("CGLIB$toString$3")) {
                switch(var2.length) {
                case 0:
                    return 18;
                }
            }
            break;
        case 1811874389:
            if (var1.equals("newInstance")) {
                switch(var2.length) {
                case 1:
                    String var10001 = var2[0].getName();
                    switch(var10001.hashCode()) {
                    case -845341380:
                        if (var10001.equals("net.sf.cglib.proxy.Callback")) {
                            return 3;
                        }
                        break;
                    case 1730110032:
                        if (var10001.equals("[Lnet.sf.cglib.proxy.Callback;")) {
                            return 4;
                        }
                    }
                case 2:
                default:
                    break;
                case 3:
                    if (var2[0].getName().equals("[Ljava.lang.Class;") && var2[1].getName().equals("[Ljava.lang.Object;") && var2[2].getName().equals("[Lnet.sf.cglib.proxy.Callback;")) {
                        return 5;
                    }
                }
            }
            break;
        case 1817099975:
            if (var1.equals("setCallback")) {
                switch(var2.length) {
                case 2:
                    if (var2[0].getName().equals("int") && var2[1].getName().equals("net.sf.cglib.proxy.Callback")) {
                        return 8;
                    }
                }
            }
            break;
        case 1900285431:
            if (var1.equals("finalMethod")) {
                switch(var2.length) {
                case 0:
                    return 21;
                }
            }
            break;

        case 1905679803:
            if (var1.equals("getCallback")) {
                switch(var2.length) {
                case 1:
                    if (var2[0].getName().equals("int")) {
                        return 11;
                    }
                }
            }
            break;
        case 1950568386:
            if (var1.equals("getClass")) {
                switch(var2.length) {
                case 0:
                    return 25;
                }
            }
            break;
        case 1951977611:
            if (var1.equals("CGLIB$clone$5")) {
                switch(var2.length) {
                case 0:
                    return 17;
                }
            }
        }

        return -1;
    }

    public int getIndex(Class[] var1) {
        switch(var1.length) {
        case 0:
            return 0;
        default:
            return -1;
        }
    }

      /**
     * 调用 FastClass 中的方法
     * @param var1 方法 index 值
     * @param var2 方法所在的类的对象
     * @param var3 参数列表
     */
    public Object invoke(int var1, Object var2, Object[] var3) throws InvocationTargetException {
        bdf5414d var10000 = (bdf5414d)var2;
        int var10001 = var1;

        try {
            switch(var10001) {
            case 0:
                return new Boolean(var10000.equals(var3[0]));
            case 1:
                return var10000.toString();
            case 2:
                return new Integer(var10000.hashCode());
            case 3:
                return var10000.newInstance((Callback)var3[0]);
            case 4:
                return var10000.newInstance((Callback[])var3[0]);
            case 5:
                return var10000.newInstance((Class[])var3[0], (Object[])var3[1], (Callback[])var3[2]);
            case 6:
                return bdf5414d.CGLIB$findMethodProxy((Signature)var3[0]);
            case 7:
                var10000.publicMethod();
                return null;
            case 8:
                var10000.setCallback(((Number)var3[0]).intValue(), (Callback)var3[1]);
                return null;
            case 9:
                var10000.setCallbacks((Callback[])var3[0]);
                return null;
            case 10:
                return var10000.getCallbacks();
            case 11:
                return var10000.getCallback(((Number)var3[0]).intValue());
            case 12:
                bdf5414d.CGLIB$SET_STATIC_CALLBACKS((Callback[])var3[0]);
                return null;
            case 13:
                bdf5414d.CGLIB$SET_THREAD_CALLBACKS((Callback[])var3[0]);
                return null;
            case 14:
                var10000.CGLIB$finalize$1();
                return null;
            case 15:
                return new Boolean(var10000.CGLIB$equals$2(var3[0]));
            case 16:
                return new Integer(var10000.CGLIB$hashCode$4());
            case 17:
                return var10000.CGLIB$clone$5();
            case 18:
                return var10000.CGLIB$toString$3();
            case 19:
                bdf5414d.CGLIB$STATICHOOK1();
                return null;
            case 20:
                var10000.CGLIB$publicMethod$0();
                return null;
            case 21:
                var10000.finalMethod();
                return null;
            case 22:
                var10000.wait();
                return null;
            case 23:
                var10000.wait(((Number)var3[0]).longValue(), ((Number)var3[1]).intValue());
                return null;
            case 24:
                var10000.wait(((Number)var3[0]).longValue());
                return null;
            case 25:
                return var10000.getClass();
            case 26:
                var10000.notify();
                return null;
            case 27:
                var10000.notifyAll();
                return null;
            }
        } catch (Throwable var4) {
            throw new InvocationTargetException(var4);
        }

        throw new IllegalArgumentException("Cannot find matching method/constructor");
    }

    public Object newInstance(int var1, Object[] var2) throws InvocationTargetException {
        bdf5414d var10000 = new bdf5414d;
        bdf5414d var10001 = var10000;
        int var10002 = var1;

        try {
            switch(var10002) {
            case 0:
                var10001.<init>();
                return var10000;
            }
        } catch (Throwable var3) {
            throw new InvocationTargetException(var3);
        }

        throw new IllegalArgumentException("Cannot find matching method/constructor");
    }

    public int getMaxIndex() {
        return 27;
    }
}
```

### CGLIB 创建动态代理类过程

*   查找目标类上的所有非 final 的 public 类型的方法定义；
*   将符合条件的方法定义转换成字节码；
*   将组成的字节码转换成相应的代理的 class 对象；
*   实现 MethodInterceptor 接口，用来处理对代理类上所有方法的请求。

### JDK 动态代理与 CGLIB 对比

JDK 动态代理：基于 Java 反射机制实现，必须要实现了接口的业务类才生成代理对象。

CGLIB 动态代理：基于 ASM 机制实现，通过生成业务类的子类作为代理类。

JDK Proxy 的优势：

最小化依赖关系、代码实现简单、简化开发和维护、JDK 原生支持，比 CGLIB 更加可靠，随 JDK 版本平滑升级。而字节码类库通常需要进行更新以保证在新版 Java 上能够使用。

基于 CGLIB 的优势：

无需实现接口，达到代理类无侵入，只操作关心的类，而不必为其他相关类增加工作量。高性能。

### CGLib 动态创建对象和属性

引入依赖

```
<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib-nodep</artifactId>
    <version>3.2.5</version>
</dependency>
public class CglibBean {

    /**
     * 实体Object
     */
    public Object object = null;

    /**
     * 属性map
     */
    public BeanMap beanMap = null;

    public CglibBean() {
        super();
    }

    @SuppressWarnings("unchecked")
    public CglibBean(Map propertyMap) {
        this.object = generateBean(propertyMap);
        this.beanMap = BeanMap.create(this.object);
    }

    /**
     * 给bean属性赋值
     * @param property 属性名
     * @param value 值
     */
    public void setValue(String property, Object value) {
        beanMap.put(property, value);
    }

    /**
     * 通过属性名得到属性值
     * @param property 属性名
     * @return 值
     */
    public Object getValue(String property) {
        return beanMap.get(property);
    }

    /**
     * 得到该实体bean对象
     * @return
     */
    public Object getObject() {
        return this.object;
    }

    @SuppressWarnings("unchecked")
    private Object generateBean(Map propertyMap) {
        BeanGenerator generator = new BeanGenerator();
        Set keySet = propertyMap.keySet();
        for (Iterator i = keySet.iterator(); i.hasNext();) {
            String key = (String) i.next();
            generator.addProperty(key, (Class) propertyMap.get(key));
        }
        return generator.create();
    }
}
```

测试类

```
public static void main(String[] args) throws ClassNotFoundException {

        // 设置类成员属性
        HashMap propertyMap = new HashMap();

        propertyMap.put("id", Class.forName("java.lang.Integer"));
        propertyMap.put("name", Class.forName("java.lang.String"));
        propertyMap.put("address", Class.forName("java.lang.String"));

        // 生成动态 Bean
        CglibBean bean = new CglibBean(propertyMap);

        // 给 Bean 设置值
        bean.setValue("id", new Integer(123));
        bean.setValue("name", "454");
        bean.setValue("address", "789");

        // 从 Bean 中获取值，当然了获得值的类型是 Object

        System.out.println("  >> id      = " + bean.getValue("id"));
        System.out.println("  >> name    = " + bean.getValue("name"));
        System.out.println("  >> address = " + bean.getValue("address"));

        // 获得bean的实体
        Object object = bean.getObject();

        // 查看属性名
        Field[] declaredFields = clazz.getDeclaredFields();
        for (int i = 0; i < declaredFields.length; i++) {
            System.out.println(declaredFields[i].getName());
        }

        // 通过反射查看所有方法名
        Class clazz = object.getClass();
        Method[] methods = clazz.getDeclaredMethods();
        for (int i = 0; i < methods.length; i++) {
            System.out.println(methods[i].getName());
        }
    }
```

输出结果：

```
id      = 123 
name    = 454 
address = 789
$cglib_prop_address
$cglib_prop_name
$cglib_prop_id
getAddress 
getName 
getId 
setName 
setAddress 
setId
```

动态代理在 Java 开发中是非常常见的，在日志，监控，事务中都有着广泛的应用，同时在大多主流框架中的核心组件中也是少不了使用的，掌握其要点，不管是开发还是阅读其他框架源码时，都是必须的。