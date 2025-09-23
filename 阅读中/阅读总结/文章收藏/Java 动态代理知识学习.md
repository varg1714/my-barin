---
source: "[[彻底搞懂动态代理]]"
create: 2025-09-23
---

## 1. 笔记：动态代理核心概念与实现：JDK vs Cglib

### 1.1. 什么是代理模式？

代理模式（Proxy Pattern）的核心思想是为一个对象提供一个代理（或占位符），以控制对这个对象的访问。在不直接访问目标对象的情况下，代理对象可以执行一些额外的操作，如权限校验、日志记录、性能监控、事务管理等。这是一种典型的结构型设计模式，能够有效地对原有代码进行功能增强，符合开闭原则。

在 Java 中，实现代理模式主要有两种方式：静态代理和动态代理。动态代理在运行时才创建代理类，更具灵活性，主要分为 **JDK 动态代理** 和 **Cglib 动态代理** 两种。

### 1.2. JDK 动态代理

#### 1.2.1. 核心原理

JDK 动态代理是 Java 官方提供的代理实现方式，其核心是利用**反射机制**。它要求**被代理的类必须实现一个或多个接口**。在运行时，JDK 会动态地创建一个新的代理类，这个代理类会实现目标类所实现的接口，并继承自 `java.lang.reflect.Proxy`。所有对代理对象接口方法的调用，都会被转发到 `InvocationHandler` 接口的 `invoke` 方法中进行统一处理。

#### 1.2.2. 实现步骤

1. **定义接口**：创建一个接口，定义需要被代理的方法。

    ```java
    public interface People {
        void speak();
    }
    ```

2. **创建目标类**：实现上述接口。

    ```java
    public class Me implements People {
        @Override
        public void speak() {
            System.out.println("我要一张JAY演唱会门票");
        }
    }
    ```

3. **创建 `InvocationHandler`**：实现 `InvocationHandler` 接口，这是代理逻辑的核心。`invoke` 方法会在代理对象的方法被调用时触发。

    ```java
    import java.lang.reflect.InvocationHandler;
    import java.lang.reflect.Method;
    
    public class HuangNiu implements InvocationHandler {
        private Object target; // 真实的目标对象
    
        public HuangNiu(Object target) {
            this.target = target;
        }
    
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("代理中...（前置增强）");
            // 通过反射调用真实对象的方法
            Object result = method.invoke(target, args);
            System.out.println("代理处理完毕。（后置增强）");
            return result;
        }
    }
    ```

4. **生成代理对象**：使用 `Proxy.newProxyInstance()` 方法创建代理实例。

    ```java
    People me = new Me();
    People proxyInstance = (People) Proxy.newProxyInstance(
        me.getClass().getClassLoader(), // 目标类的类加载器
        me.getClass().getInterfaces(),   // 目标类实现的接口数组
        new HuangNiu(me)                 // InvocationHandler 实例
    );
    proxyInstance.speak();
    ```

#### 1.2.3. 底层机制

*  JDK 在运行时会生成一个名为 `$Proxy0`（数字会递增）的字节码文件。
*  这个 `$Proxy0` 类继承了 `Proxy` 类，并实现了 `People` 接口。
*  接口中的 `speak()` 方法在 `$Proxy0` 中被重写，其内部实现是调用 `InvocationHandler` 的 `invoke` 方法，从而将执行权交给了我们自定义的代理逻辑。
*  最终在 `invoke` 方法内部，通过 `method.invoke(target, args)` 反射调用了真实目标对象的方法。

#### 1.2.4. 关键限制

*  **必须基于接口**：被代理的类必须实现接口，这是 JDK 动态代理最大的限制。

### 1.3. Cglib 动态代理

#### 1.3.1. 核心原理

Cglib (Code Generation Library) 是一个强大的、高性能的代码生成库。它不依赖接口，而是通过**继承**的方式实现动态代理。其底层使用了 **ASM** 这个字节码操作框架，在运行时动态地生成一个被代理类的**子类**，并重写父类中所有非 `final` 的方法，从而在子类中织入增强逻辑。

#### 1.3.2. 实现步骤

1. **创建目标类**：一个普通的 Java 类，无需实现接口。

    ```java
    public class Me {
        public void speak() {
            System.out.println("我要一张JAY演唱会门票");
        }
    }
    ```

2. **创建 `MethodInterceptor`**：实现 `MethodInterceptor` 接口，它等同于 JDK 代理中的 `InvocationHandler`。

    ```java
    import net.sf.cglib.proxy.MethodInterceptor;
    import net.sf.cglib.proxy.MethodProxy;
    import java.lang.reflect.Method;
    
    public class HuangNiu implements MethodInterceptor {
        @Override
        public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
            System.out.println("代理中...（前置增强）");
            // 调用父类（真实对象）的方法
            Object result = proxy.invokeSuper(obj, args);
            System.out.println("代理处理完毕。（后置增强）");
            return result;
        }
    }
    ```

3. **生成代理对象**：使用 Cglib 的 `Enhancer` 类来创建代理实例。

    ```java
    import net.sf.cglib.proxy.Enhancer;
    
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(Me.class); // 设置父类
    enhancer.setCallback(new HuangNiu()); // 设置方法拦截器
    Me proxyInstance = (Me) enhancer.create(); // 创建代理对象
    proxyInstance.speak();
    ```

#### 1.3.3. `intercept` 方法参数详解

`public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy)`

*  `Object obj`: Cglib 生成的**代理对象**本身。
*  `Method method`: 被拦截的**方法**的反射对象。
*  `Object[] args`: 方法的**参数**数组。
*  `MethodProxy proxy`: **方法代理**，用于调用父类（即原始类）的方法。
    *  `proxy.invokeSuper(obj, args)`: **推荐使用**。调用父类的原始方法，性能高，且不会导致递归调用。
    *  `proxy.invoke(target, args)`: 不推荐在拦截器内部使用，它可能再次触发拦截，导致无限循环。

#### 1.3.4. 关键限制

*  **不能代理 `final` 类**：因为 Cglib 的原理是继承，所以无法为 `final` 修饰的类创建子类。
*  **不能代理 `final` 或 `private` 方法**：子类无法重写父类的 `final` 或 `private` 方法，因此这些方法无法被代理。


### 1.4. Cglib 高性能揭秘：FastClass 机制

Cglib 在方法**执行**时通常比 JDK 动态代理更快，其秘诀在于 **FastClass 机制**，它成功地**避免了反射调用**。

1. **工作原理**:
    *  在创建代理对象时，Cglib 会为目标类和代理类分别额外生成一个 `FastClass` 类（如 `Service$$FastClassByCGLIB$$...`）。
    *  这个 `FastClass` 会为类中的每个方法建立一个**整数索引（index）**。
    *  当代理方法被调用时，`MethodProxy.invokeSuper` 内部会通过这个 `FastClass` 来执行。

2. **执行流程**:
    *  **获取索引 (`getIndex`)**: `FastClass` 首先通过方法签名（方法名+参数）在一个内部的 `switch-case` 结构中快速匹配，得到该方法对应的 `index`。
    *  **直接调用 (`invoke`)**: 接着，`FastClass` 的 `invoke` 方法会再次使用 `switch-case`，根据传入的 `index` 直接跳转到对应的代码块，执行一个**标准的、硬编码的方法调用**（例如 `target.speak()`），而不是 `method.invoke(...)`。

3. **性能优势**:
    *  **避免反射开销**: 完全绕过了 Java 反射机制中耗时的权限检查和动态方法查找过程。
    *  **JIT 优化友好**: 生成的字节码是直接的方法调用，这种代码极易被 Java 的即时编译器（JIT）进行**方法内联**等深度优化，其性能几乎与手写的原生代码无异。


### 1.5. 总结与对比

| 特性 | JDK 动态代理 | Cglib 动态代理 |
| :--- | :--- | :--- |
| **实现原理** | 基于**接口**和 **Java 反射**机制。 | 基于**继承**和 **ASM 字节码**操作。 |
| **代理对象** | 代理类与目标类实现相同的接口。 | 代理类是目标类的子类。 |
| **前提条件** | 目标类**必须实现接口**。 | 目标类**不能是 `final`**，被代理方法不能是 `final` 或 `private`。 |
| **性能** | **创建快，执行慢**。每次调用都涉及反射，有性能开销。 | **创建慢，执行快**。首次创建复杂，但后续调用通过 FastClass 机制，性能高。 |
| **依赖** | JDK 原生支持，无需额外依赖。 | 需要引入 `cglib` 第三方库。 |