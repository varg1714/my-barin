---
source: https://pdai.tech/md/interview/x-interview.html#_8-5-elasticsearch
create: 2024-08-19 09:35
read: false
---

*   [Java 全栈知识点问题汇总（上）](#java-%E5%85%A8%E6%A0%88%E7%9F%A5%E8%AF%86%E7%82%B9%E9%97%AE%E9%A2%98%E6%B1%87%E6%80%BB%E4%B8%8A)
    *   [1 Java 基础](#1-java-%E5%9F%BA%E7%A1%80)
        *   [1.1 语法基础](#11-%E8%AF%AD%E6%B3%95%E5%9F%BA%E7%A1%80)
            *   [面向对象特性？](#%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%89%B9%E6%80%A7)
            *   [a = a + b 与 a += b 的区别](#a--a--b-%E4%B8%8E-a--b-%E7%9A%84%E5%8C%BA%E5%88%AB)
            *   [3*0.1 == 0.3 将会返回什么? true 还是 false?](#301--03-%E5%B0%86%E4%BC%9A%E8%BF%94%E5%9B%9E%E4%BB%80%E4%B9%88-true-%E8%BF%98%E6%98%AF-false)
            *   [能在 Switch 中使用 String 吗?](#%E8%83%BD%E5%9C%A8-switch-%E4%B8%AD%E4%BD%BF%E7%94%A8-string-%E5%90%97)
            *   [对 equals() 和 hashCode() 的理解?](#%E5%AF%B9equals%E5%92%8Chashcode%E7%9A%84%E7%90%86%E8%A7%A3)
            *   [final、finalize 和 finally 的不同之处?](#finalfinalize-%E5%92%8C-finally-%E7%9A%84%E4%B8%8D%E5%90%8C%E4%B9%8B%E5%A4%84)
            *   [String、StringBuffer 与 StringBuilder 的区别？](#stringstringbuffer%E4%B8%8Estringbuilder%E7%9A%84%E5%8C%BA%E5%88%AB)
            *   [接口与抽象类的区别？](#%E6%8E%A5%E5%8F%A3%E4%B8%8E%E6%8A%BD%E8%B1%A1%E7%B1%BB%E7%9A%84%E5%8C%BA%E5%88%AB)
            *   [this() & super() 在构造方法中的区别？](#this--super%E5%9C%A8%E6%9E%84%E9%80%A0%E6%96%B9%E6%B3%95%E4%B8%AD%E7%9A%84%E5%8C%BA%E5%88%AB)
            *   [Java 移位运算符？](#java%E7%A7%BB%E4%BD%8D%E8%BF%90%E7%AE%97%E7%AC%A6)
        *   [1.2 泛型](#12-%E6%B3%9B%E5%9E%8B)
            *   [为什么需要泛型？](#%E4%B8%BA%E4%BB%80%E4%B9%88%E9%9C%80%E8%A6%81%E6%B3%9B%E5%9E%8B)
            *   [泛型类如何定义使用？](#%E6%B3%9B%E5%9E%8B%E7%B1%BB%E5%A6%82%E4%BD%95%E5%AE%9A%E4%B9%89%E4%BD%BF%E7%94%A8)
            *   [泛型接口如何定义使用？](#%E6%B3%9B%E5%9E%8B%E6%8E%A5%E5%8F%A3%E5%A6%82%E4%BD%95%E5%AE%9A%E4%B9%89%E4%BD%BF%E7%94%A8)
            *   [泛型方法如何定义使用？](#%E6%B3%9B%E5%9E%8B%E6%96%B9%E6%B3%95%E5%A6%82%E4%BD%95%E5%AE%9A%E4%B9%89%E4%BD%BF%E7%94%A8)
            *   [泛型的上限和下限？](#%E6%B3%9B%E5%9E%8B%E7%9A%84%E4%B8%8A%E9%99%90%E5%92%8C%E4%B8%8B%E9%99%90)
            *   [如何理解 Java 中的泛型是伪泛型？](#%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3java%E4%B8%AD%E7%9A%84%E6%B3%9B%E5%9E%8B%E6%98%AF%E4%BC%AA%E6%B3%9B%E5%9E%8B)
        *   [1.3 注解](#13-%E6%B3%A8%E8%A7%A3)
            *   [注解的作用？](#%E6%B3%A8%E8%A7%A3%E7%9A%84%E4%BD%9C%E7%94%A8)
            *   [注解的常见分类？](#%E6%B3%A8%E8%A7%A3%E7%9A%84%E5%B8%B8%E8%A7%81%E5%88%86%E7%B1%BB)
        *   [1.4 异常](#14-%E5%BC%82%E5%B8%B8)
            *   [Java 异常类层次结构?](#java%E5%BC%82%E5%B8%B8%E7%B1%BB%E5%B1%82%E6%AC%A1%E7%BB%93%E6%9E%84)
            *   [可查的异常（checked exceptions）和不可查的异常（unchecked exceptions）区别？](#%E5%8F%AF%E6%9F%A5%E7%9A%84%E5%BC%82%E5%B8%B8checked-exceptions%E5%92%8C%E4%B8%8D%E5%8F%AF%E6%9F%A5%E7%9A%84%E5%BC%82%E5%B8%B8unchecked-exceptions%E5%8C%BA%E5%88%AB)
            *   [throw 和 throws 的区别？](#throw%E5%92%8Cthrows%E7%9A%84%E5%8C%BA%E5%88%AB)
            *   [Java 7 的 try-with-resource?](#java-7-%E7%9A%84-try-with-resource)
            *   [异常的底层？](#%E5%BC%82%E5%B8%B8%E7%9A%84%E5%BA%95%E5%B1%82)
        *   [1.5 反射](#15-%E5%8F%8D%E5%B0%84)
            *   [什么是反射？](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%8F%8D%E5%B0%84)
            *   [反射的使用？](#%E5%8F%8D%E5%B0%84%E7%9A%84%E4%BD%BF%E7%94%A8)
            *   [getName、getCanonicalName 与 getSimpleName 的区别?](#getnamegetcanonicalname%E4%B8%8Egetsimplename%E7%9A%84%E5%8C%BA%E5%88%AB)
        *   [1.6 SPI 机制](#16-spi%E6%9C%BA%E5%88%B6)
            *   [什么是 SPI 机制？](#%E4%BB%80%E4%B9%88%E6%98%AFspi%E6%9C%BA%E5%88%B6)
            *   [SPI 机制的应用？](#spi%E6%9C%BA%E5%88%B6%E7%9A%84%E5%BA%94%E7%94%A8)
            *   [SPI 机制的简单示例？](#spi%E6%9C%BA%E5%88%B6%E7%9A%84%E7%AE%80%E5%8D%95%E7%A4%BA%E4%BE%8B)
    *   [2 Java 集合](#2-java-%E9%9B%86%E5%90%88)
        *   [2.1 Collection](#21-collection)
            *   [集合有哪些类？](#%E9%9B%86%E5%90%88%E6%9C%89%E5%93%AA%E4%BA%9B%E7%B1%BB)
            *   [ArrayList 的底层？](#arraylist%E7%9A%84%E5%BA%95%E5%B1%82)
            *   [ArrayList 自动扩容？](#arraylist%E8%87%AA%E5%8A%A8%E6%89%A9%E5%AE%B9)
            *   [ArrayList 的 Fail-Fast 机制？](#arraylist%E7%9A%84fail-fast%E6%9C%BA%E5%88%B6)
        *   [2.2 Map](#22-map)
            *   [Map 有哪些类？](#map%E6%9C%89%E5%93%AA%E4%BA%9B%E7%B1%BB)
            *   [JDK7 HashMap 如何实现？](#jdk7-hashmap%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0)
            *   [JDK8 HashMap 如何实现？](#jdk8-hashmap%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0)
            *   [HashSet 是如何实现的？](#hashset%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%9A%84)
            *   [什么是 WeakHashMap?](#%E4%BB%80%E4%B9%88%E6%98%AFweakhashmap)
    *   [3 Java 并发](#3-java-%E5%B9%B6%E5%8F%91)
        *   [3.1 并发基础](#31-%E5%B9%B6%E5%8F%91%E5%9F%BA%E7%A1%80)
            *   [多线程的出现是要解决什么问题的? 本质什么?](#%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%87%BA%E7%8E%B0%E6%98%AF%E8%A6%81%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98%E7%9A%84-%E6%9C%AC%E8%B4%A8%E4%BB%80%E4%B9%88)
            *   [Java 是怎么解决并发问题的?](#java%E6%98%AF%E6%80%8E%E4%B9%88%E8%A7%A3%E5%86%B3%E5%B9%B6%E5%8F%91%E9%97%AE%E9%A2%98%E7%9A%84)
            *   [线程安全有哪些实现思路?](#%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E6%9C%89%E5%93%AA%E4%BA%9B%E5%AE%9E%E7%8E%B0%E6%80%9D%E8%B7%AF)
            *   [如何理解并发和并行的区别?](#%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3%E5%B9%B6%E5%8F%91%E5%92%8C%E5%B9%B6%E8%A1%8C%E7%9A%84%E5%8C%BA%E5%88%AB)
            *   [线程有哪几种状态? 分别说明从一种状态到另一种状态转变有哪些方式?](#%E7%BA%BF%E7%A8%8B%E6%9C%89%E5%93%AA%E5%87%A0%E7%A7%8D%E7%8A%B6%E6%80%81-%E5%88%86%E5%88%AB%E8%AF%B4%E6%98%8E%E4%BB%8E%E4%B8%80%E7%A7%8D%E7%8A%B6%E6%80%81%E5%88%B0%E5%8F%A6%E4%B8%80%E7%A7%8D%E7%8A%B6%E6%80%81%E8%BD%AC%E5%8F%98%E6%9C%89%E5%93%AA%E4%BA%9B%E6%96%B9%E5%BC%8F)
            *   [通常线程有哪几种使用方式?](#%E9%80%9A%E5%B8%B8%E7%BA%BF%E7%A8%8B%E6%9C%89%E5%93%AA%E5%87%A0%E7%A7%8D%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F)
            *   [基础线程机制有哪些?](#%E5%9F%BA%E7%A1%80%E7%BA%BF%E7%A8%8B%E6%9C%BA%E5%88%B6%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [线程的中断方式有哪些?](#%E7%BA%BF%E7%A8%8B%E7%9A%84%E4%B8%AD%E6%96%AD%E6%96%B9%E5%BC%8F%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [线程的互斥同步方式有哪些? 如何比较和选择?](#%E7%BA%BF%E7%A8%8B%E7%9A%84%E4%BA%92%E6%96%A5%E5%90%8C%E6%AD%A5%E6%96%B9%E5%BC%8F%E6%9C%89%E5%93%AA%E4%BA%9B-%E5%A6%82%E4%BD%95%E6%AF%94%E8%BE%83%E5%92%8C%E9%80%89%E6%8B%A9)
            *   [线程之间有哪些协作方式?](#%E7%BA%BF%E7%A8%8B%E4%B9%8B%E9%97%B4%E6%9C%89%E5%93%AA%E4%BA%9B%E5%8D%8F%E4%BD%9C%E6%96%B9%E5%BC%8F)
        *   [3.2 并发关键字](#32-%E5%B9%B6%E5%8F%91%E5%85%B3%E9%94%AE%E5%AD%97)
            *   [Synchronized 可以作用在哪里?](#synchronized%E5%8F%AF%E4%BB%A5%E4%BD%9C%E7%94%A8%E5%9C%A8%E5%93%AA%E9%87%8C)
            *   [Synchronized 本质上是通过什么保证线程安全的?](#synchronized%E6%9C%AC%E8%B4%A8%E4%B8%8A%E6%98%AF%E9%80%9A%E8%BF%87%E4%BB%80%E4%B9%88%E4%BF%9D%E8%AF%81%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84)
            *   [Synchronized 使得同时只有一个线程可以执行，性能比较差，有什么提升的方法?](#synchronized%E4%BD%BF%E5%BE%97%E5%90%8C%E6%97%B6%E5%8F%AA%E6%9C%89%E4%B8%80%E4%B8%AA%E7%BA%BF%E7%A8%8B%E5%8F%AF%E4%BB%A5%E6%89%A7%E8%A1%8C%E6%80%A7%E8%83%BD%E6%AF%94%E8%BE%83%E5%B7%AE%E6%9C%89%E4%BB%80%E4%B9%88%E6%8F%90%E5%8D%87%E7%9A%84%E6%96%B9%E6%B3%95)
            *   [Synchronized 由什么样的缺陷? Java Lock 是怎么弥补这些缺陷的?](#synchronized%E7%94%B1%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E7%BC%BA%E9%99%B7-java-lock%E6%98%AF%E6%80%8E%E4%B9%88%E5%BC%A5%E8%A1%A5%E8%BF%99%E4%BA%9B%E7%BC%BA%E9%99%B7%E7%9A%84)
            *   [Synchronized 和 Lock 的对比，和选择?](#synchronized%E5%92%8Clock%E7%9A%84%E5%AF%B9%E6%AF%94%E5%92%8C%E9%80%89%E6%8B%A9)
            *   [Synchronized 在使用时有何注意事项?](#synchronized%E5%9C%A8%E4%BD%BF%E7%94%A8%E6%97%B6%E6%9C%89%E4%BD%95%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9)
            *   [Synchronized 修饰的方法在抛出异常时, 会释放锁吗?](#synchronized%E4%BF%AE%E9%A5%B0%E7%9A%84%E6%96%B9%E6%B3%95%E5%9C%A8%E6%8A%9B%E5%87%BA%E5%BC%82%E5%B8%B8%E6%97%B6%E4%BC%9A%E9%87%8A%E6%94%BE%E9%94%81%E5%90%97)
            *   [多个线程等待同一个 Synchronized 锁的时候，JVM 如何选择下一个获取锁的线程?](#%E5%A4%9A%E4%B8%AA%E7%BA%BF%E7%A8%8B%E7%AD%89%E5%BE%85%E5%90%8C%E4%B8%80%E4%B8%AAsynchronized%E9%94%81%E7%9A%84%E6%97%B6%E5%80%99jvm%E5%A6%82%E4%BD%95%E9%80%89%E6%8B%A9%E4%B8%8B%E4%B8%80%E4%B8%AA%E8%8E%B7%E5%8F%96%E9%94%81%E7%9A%84%E7%BA%BF%E7%A8%8B)
            *   [synchronized 是公平锁吗？](#synchronized%E6%98%AF%E5%85%AC%E5%B9%B3%E9%94%81%E5%90%97)
            *   [volatile 关键字的作用是什么?](#volatile%E5%85%B3%E9%94%AE%E5%AD%97%E7%9A%84%E4%BD%9C%E7%94%A8%E6%98%AF%E4%BB%80%E4%B9%88)
            *   [volatile 能保证原子性吗?](#volatile%E8%83%BD%E4%BF%9D%E8%AF%81%E5%8E%9F%E5%AD%90%E6%80%A7%E5%90%97)
            *   [32 位机器上共享的 long 和 double 变量的为什么要用 volatile?](#32%E4%BD%8D%E6%9C%BA%E5%99%A8%E4%B8%8A%E5%85%B1%E4%BA%AB%E7%9A%84long%E5%92%8Cdouble%E5%8F%98%E9%87%8F%E7%9A%84%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E7%94%A8volatile)
            *   [volatile 是如何实现可见性的?](#volatile%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E5%8F%AF%E8%A7%81%E6%80%A7%E7%9A%84)
            *   [volatile 是如何实现有序性的?](#volatile%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E6%9C%89%E5%BA%8F%E6%80%A7%E7%9A%84)
            *   [说下 volatile 的应用场景?](#%E8%AF%B4%E4%B8%8Bvolatile%E7%9A%84%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF)
            *   [所有的 final 修饰的字段都是编译期常量吗?](#%E6%89%80%E6%9C%89%E7%9A%84final%E4%BF%AE%E9%A5%B0%E7%9A%84%E5%AD%97%E6%AE%B5%E9%83%BD%E6%98%AF%E7%BC%96%E8%AF%91%E6%9C%9F%E5%B8%B8%E9%87%8F%E5%90%97)
            *   [如何理解 private 所修饰的方法是隐式的 final?](#%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3private%E6%89%80%E4%BF%AE%E9%A5%B0%E7%9A%84%E6%96%B9%E6%B3%95%E6%98%AF%E9%9A%90%E5%BC%8F%E7%9A%84final)
            *   [说说 final 类型的类如何拓展?](#%E8%AF%B4%E8%AF%B4final%E7%B1%BB%E5%9E%8B%E7%9A%84%E7%B1%BB%E5%A6%82%E4%BD%95%E6%8B%93%E5%B1%95)
            *   [final 方法可以被重载吗?](#final%E6%96%B9%E6%B3%95%E5%8F%AF%E4%BB%A5%E8%A2%AB%E9%87%8D%E8%BD%BD%E5%90%97)
            *   [父类的 final 方法能不能够被子类重写?](#%E7%88%B6%E7%B1%BB%E7%9A%84final%E6%96%B9%E6%B3%95%E8%83%BD%E4%B8%8D%E8%83%BD%E5%A4%9F%E8%A2%AB%E5%AD%90%E7%B1%BB%E9%87%8D%E5%86%99)
            *   [说说基本类型的 final 域重排序规则?](#%E8%AF%B4%E8%AF%B4%E5%9F%BA%E6%9C%AC%E7%B1%BB%E5%9E%8B%E7%9A%84final%E5%9F%9F%E9%87%8D%E6%8E%92%E5%BA%8F%E8%A7%84%E5%88%99)
            *   [说说 final 的原理?](#%E8%AF%B4%E8%AF%B4final%E7%9A%84%E5%8E%9F%E7%90%86)
        *   [3.3 JUC 全局观](#33-juc%E5%85%A8%E5%B1%80%E8%A7%82)
            *   [JUC 框架包含几个部分?](#juc%E6%A1%86%E6%9E%B6%E5%8C%85%E5%90%AB%E5%87%A0%E4%B8%AA%E9%83%A8%E5%88%86)
            *   [Lock 框架和 Tools 哪些核心的类?](#lock%E6%A1%86%E6%9E%B6%E5%92%8Ctools%E5%93%AA%E4%BA%9B%E6%A0%B8%E5%BF%83%E7%9A%84%E7%B1%BB)
            *   [JUC 并发集合哪些核心的类?](#juc%E5%B9%B6%E5%8F%91%E9%9B%86%E5%90%88%E5%93%AA%E4%BA%9B%E6%A0%B8%E5%BF%83%E7%9A%84%E7%B1%BB)
            *   [JUC 原子类哪些核心的类?](#juc%E5%8E%9F%E5%AD%90%E7%B1%BB%E5%93%AA%E4%BA%9B%E6%A0%B8%E5%BF%83%E7%9A%84%E7%B1%BB)
            *   [JUC 线程池哪些核心的类?](#juc%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%93%AA%E4%BA%9B%E6%A0%B8%E5%BF%83%E7%9A%84%E7%B1%BB)
        *   [3.4 JUC 原子类](#34-juc%E5%8E%9F%E5%AD%90%E7%B1%BB)
            *   [线程安全的实现方法有哪些?](#%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84%E5%AE%9E%E7%8E%B0%E6%96%B9%E6%B3%95%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [什么是 CAS?](#%E4%BB%80%E4%B9%88%E6%98%AFcas)
            *   [CAS 使用示例，结合 AtomicInteger 给出示例?](#cas%E4%BD%BF%E7%94%A8%E7%A4%BA%E4%BE%8B%E7%BB%93%E5%90%88atomicinteger%E7%BB%99%E5%87%BA%E7%A4%BA%E4%BE%8B)
            *   [CAS 会有哪些问题?](#cas%E4%BC%9A%E6%9C%89%E5%93%AA%E4%BA%9B%E9%97%AE%E9%A2%98)
            *   [AtomicInteger 底层实现?](#atomicinteger%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0)
            *   [请阐述你对 Unsafe 类的理解?](#%E8%AF%B7%E9%98%90%E8%BF%B0%E4%BD%A0%E5%AF%B9unsafe%E7%B1%BB%E7%9A%84%E7%90%86%E8%A7%A3)
            *   [说说你对 Java 原子类的理解?](#%E8%AF%B4%E8%AF%B4%E4%BD%A0%E5%AF%B9java%E5%8E%9F%E5%AD%90%E7%B1%BB%E7%9A%84%E7%90%86%E8%A7%A3)
            *   [AtomicStampedReference 是怎么解决 ABA 的?](#atomicstampedreference%E6%98%AF%E6%80%8E%E4%B9%88%E8%A7%A3%E5%86%B3aba%E7%9A%84)
        *   [3.5 JUC 锁](#35-juc%E9%94%81)
            *   [为什么 LockSupport 也是核心基础类?](#%E4%B8%BA%E4%BB%80%E4%B9%88locksupport%E4%B9%9F%E6%98%AF%E6%A0%B8%E5%BF%83%E5%9F%BA%E7%A1%80%E7%B1%BB)
            *   [通过 wait/notify 实现同步?](#%E9%80%9A%E8%BF%87waitnotify%E5%AE%9E%E7%8E%B0%E5%90%8C%E6%AD%A5)
            *   [通过 LockSupport 的 park/unpark 实现同步？](#%E9%80%9A%E8%BF%87locksupport%E7%9A%84parkunpark%E5%AE%9E%E7%8E%B0%E5%90%8C%E6%AD%A5)
            *   [Thread.sleep()、Object.wait()、Condition.await()、LockSupport.park() 的区别? 重点](#threadsleepobjectwaitconditionawaitlocksupportpark%E7%9A%84%E5%8C%BA%E5%88%AB-%E9%87%8D%E7%82%B9)
            *   [如果在 wait() 之前执行了 notify() 会怎样?](#%E5%A6%82%E6%9E%9C%E5%9C%A8wait%E4%B9%8B%E5%89%8D%E6%89%A7%E8%A1%8C%E4%BA%86notify%E4%BC%9A%E6%80%8E%E6%A0%B7)
            *   [如果在 park() 之前执行了 unpark() 会怎样?](#%E5%A6%82%E6%9E%9C%E5%9C%A8park%E4%B9%8B%E5%89%8D%E6%89%A7%E8%A1%8C%E4%BA%86unpark%E4%BC%9A%E6%80%8E%E6%A0%B7)
            *   [什么是 AQS? 为什么它是核心?](#%E4%BB%80%E4%B9%88%E6%98%AFaqs-%E4%B8%BA%E4%BB%80%E4%B9%88%E5%AE%83%E6%98%AF%E6%A0%B8%E5%BF%83)
            *   [AQS 的核心思想是什么?](#aqs%E7%9A%84%E6%A0%B8%E5%BF%83%E6%80%9D%E6%83%B3%E6%98%AF%E4%BB%80%E4%B9%88)
            *   [AQS 有哪些核心的方法?](#aqs%E6%9C%89%E5%93%AA%E4%BA%9B%E6%A0%B8%E5%BF%83%E7%9A%84%E6%96%B9%E6%B3%95)
            *   [AQS 定义什么样的资源获取方式?](#aqs%E5%AE%9A%E4%B9%89%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E8%B5%84%E6%BA%90%E8%8E%B7%E5%8F%96%E6%96%B9%E5%BC%8F)
            *   [AQS 底层使用了什么样的设计模式?](#aqs%E5%BA%95%E5%B1%82%E4%BD%BF%E7%94%A8%E4%BA%86%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F)
            *   [什么是可重入，什么是可重入锁? 它用来解决什么问题?](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%8F%AF%E9%87%8D%E5%85%A5%E4%BB%80%E4%B9%88%E6%98%AF%E5%8F%AF%E9%87%8D%E5%85%A5%E9%94%81-%E5%AE%83%E7%94%A8%E6%9D%A5%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98)
            *   [ReentrantLock 的核心是 AQS，那么它怎么来实现的，继承吗?](#reentrantlock%E7%9A%84%E6%A0%B8%E5%BF%83%E6%98%AFaqs%E9%82%A3%E4%B9%88%E5%AE%83%E6%80%8E%E4%B9%88%E6%9D%A5%E5%AE%9E%E7%8E%B0%E7%9A%84%E7%BB%A7%E6%89%BF%E5%90%97)
            *   [ReentrantLock 是如何实现公平锁的?](#reentrantlock%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E5%85%AC%E5%B9%B3%E9%94%81%E7%9A%84)
            *   [ReentrantLock 是如何实现非公平锁的?](#reentrantlock%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E9%9D%9E%E5%85%AC%E5%B9%B3%E9%94%81%E7%9A%84)
            *   [ReentrantLock 默认实现的是公平还是非公平锁?](#reentrantlock%E9%BB%98%E8%AE%A4%E5%AE%9E%E7%8E%B0%E7%9A%84%E6%98%AF%E5%85%AC%E5%B9%B3%E8%BF%98%E6%98%AF%E9%9D%9E%E5%85%AC%E5%B9%B3%E9%94%81)
            *   [为了有了 ReentrantLock 还需要 ReentrantReadWriteLock?](#%E4%B8%BA%E4%BA%86%E6%9C%89%E4%BA%86reentrantlock%E8%BF%98%E9%9C%80%E8%A6%81reentrantreadwritelock)
            *   [ReentrantReadWriteLock 底层实现原理?](#reentrantreadwritelock%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86)
            *   [ReentrantReadWriteLock 底层读写状态如何设计的?](#reentrantreadwritelock%E5%BA%95%E5%B1%82%E8%AF%BB%E5%86%99%E7%8A%B6%E6%80%81%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%E7%9A%84)
            *   [读锁和写锁的最大数量是多少?](#%E8%AF%BB%E9%94%81%E5%92%8C%E5%86%99%E9%94%81%E7%9A%84%E6%9C%80%E5%A4%A7%E6%95%B0%E9%87%8F%E6%98%AF%E5%A4%9A%E5%B0%91)
            *   [本地线程计数器 ThreadLocalHoldCounter 是用来做什么的?](#%E6%9C%AC%E5%9C%B0%E7%BA%BF%E7%A8%8B%E8%AE%A1%E6%95%B0%E5%99%A8threadlocalholdcounter%E6%98%AF%E7%94%A8%E6%9D%A5%E5%81%9A%E4%BB%80%E4%B9%88%E7%9A%84)
            *   [写锁的获取与释放是怎么实现的?](#%E5%86%99%E9%94%81%E7%9A%84%E8%8E%B7%E5%8F%96%E4%B8%8E%E9%87%8A%E6%94%BE%E6%98%AF%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0%E7%9A%84)
            *   [读锁的获取与释放是怎么实现的?](#%E8%AF%BB%E9%94%81%E7%9A%84%E8%8E%B7%E5%8F%96%E4%B8%8E%E9%87%8A%E6%94%BE%E6%98%AF%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0%E7%9A%84)
            *   [什么是锁的升降级?](#%E4%BB%80%E4%B9%88%E6%98%AF%E9%94%81%E7%9A%84%E5%8D%87%E9%99%8D%E7%BA%A7)
        *   [3.6 JUC 集合类](#36-juc%E9%9B%86%E5%90%88%E7%B1%BB)
            *   [为什么 HashTable 慢? 它的并发度是什么? 那么 ConcurrentHashMap 并发度是什么?](#%E4%B8%BA%E4%BB%80%E4%B9%88hashtable%E6%85%A2-%E5%AE%83%E7%9A%84%E5%B9%B6%E5%8F%91%E5%BA%A6%E6%98%AF%E4%BB%80%E4%B9%88-%E9%82%A3%E4%B9%88concurrenthashmap%E5%B9%B6%E5%8F%91%E5%BA%A6%E6%98%AF%E4%BB%80%E4%B9%88)
            *   [ConcurrentHashMap 在 JDK1.7 和 JDK1.8 中实现有什么差别? JDK1.8 解決了 JDK1.7 中什么问题](#concurrenthashmap%E5%9C%A8jdk17%E5%92%8Cjdk18%E4%B8%AD%E5%AE%9E%E7%8E%B0%E6%9C%89%E4%BB%80%E4%B9%88%E5%B7%AE%E5%88%AB-jdk18%E8%A7%A3%E6%B1%BA%E4%BA%86jdk17%E4%B8%AD%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98)
            *   [ConcurrentHashMap JDK1.7 实现的原理是什么?](#concurrenthashmap-jdk17%E5%AE%9E%E7%8E%B0%E7%9A%84%E5%8E%9F%E7%90%86%E6%98%AF%E4%BB%80%E4%B9%88)
            *   [ConcurrentHashMap JDK1.7 中 Segment 数 (concurrencyLevel) 默认值是多少? 为何一旦初始化就不可再扩容?](#concurrenthashmap-jdk17%E4%B8%ADsegment%E6%95%B0concurrencylevel%E9%BB%98%E8%AE%A4%E5%80%BC%E6%98%AF%E5%A4%9A%E5%B0%91-%E4%B8%BA%E4%BD%95%E4%B8%80%E6%97%A6%E5%88%9D%E5%A7%8B%E5%8C%96%E5%B0%B1%E4%B8%8D%E5%8F%AF%E5%86%8D%E6%89%A9%E5%AE%B9)
            *   [ConcurrentHashMap JDK1.7 说说其 put 的机制?](#concurrenthashmap-jdk17%E8%AF%B4%E8%AF%B4%E5%85%B6put%E7%9A%84%E6%9C%BA%E5%88%B6)
            *   [ConcurrentHashMap JDK1.7 是如何扩容的?](#concurrenthashmap-jdk17%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%A9%E5%AE%B9%E7%9A%84)
            *   [ConcurrentHashMap JDK1.8 实现的原理是什么?](#concurrenthashmap-jdk18%E5%AE%9E%E7%8E%B0%E7%9A%84%E5%8E%9F%E7%90%86%E6%98%AF%E4%BB%80%E4%B9%88)
            *   [ConcurrentHashMap JDK1.8 是如何扩容的?](#concurrenthashmap-jdk18%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%A9%E5%AE%B9%E7%9A%84)
            *   [ConcurrentHashMap JDK1.8 链表转红黑树的时机是什么? 临界值为什么是 8?](#concurrenthashmap-jdk18%E9%93%BE%E8%A1%A8%E8%BD%AC%E7%BA%A2%E9%BB%91%E6%A0%91%E7%9A%84%E6%97%B6%E6%9C%BA%E6%98%AF%E4%BB%80%E4%B9%88-%E4%B8%B4%E7%95%8C%E5%80%BC%E4%B8%BA%E4%BB%80%E4%B9%88%E6%98%AF8)
            *   [ConcurrentHashMap JDK1.8 是如何进行数据迁移的?](#concurrenthashmap-jdk18%E6%98%AF%E5%A6%82%E4%BD%95%E8%BF%9B%E8%A1%8C%E6%95%B0%E6%8D%AE%E8%BF%81%E7%A7%BB%E7%9A%84)
            *   [先说说非并发集合中 Fail-fast 机制?](#%E5%85%88%E8%AF%B4%E8%AF%B4%E9%9D%9E%E5%B9%B6%E5%8F%91%E9%9B%86%E5%90%88%E4%B8%ADfail-fast%E6%9C%BA%E5%88%B6)
            *   [CopyOnWriteArrayList 的实现原理?](#copyonwritearraylist%E7%9A%84%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86)
            *   [弱一致性的迭代器原理是怎么样的?](#%E5%BC%B1%E4%B8%80%E8%87%B4%E6%80%A7%E7%9A%84%E8%BF%AD%E4%BB%A3%E5%99%A8%E5%8E%9F%E7%90%86%E6%98%AF%E6%80%8E%E4%B9%88%E6%A0%B7%E7%9A%84)
            *   [CopyOnWriteArrayList 为什么并发安全且性能比 Vector 好?](#copyonwritearraylist%E4%B8%BA%E4%BB%80%E4%B9%88%E5%B9%B6%E5%8F%91%E5%AE%89%E5%85%A8%E4%B8%94%E6%80%A7%E8%83%BD%E6%AF%94vector%E5%A5%BD)
            *   [CopyOnWriteArrayList 有何缺陷，说说其应用场景?](#copyonwritearraylist%E6%9C%89%E4%BD%95%E7%BC%BA%E9%99%B7%E8%AF%B4%E8%AF%B4%E5%85%B6%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF)
            *   [要想用线程安全的队列有哪些选择?](#%E8%A6%81%E6%83%B3%E7%94%A8%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84%E9%98%9F%E5%88%97%E6%9C%89%E5%93%AA%E4%BA%9B%E9%80%89%E6%8B%A9)
            *   [ConcurrentLinkedQueue 实现的数据结构?](#concurrentlinkedqueue%E5%AE%9E%E7%8E%B0%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
            *   [ConcurrentLinkedQueue 底层原理?](#concurrentlinkedqueue%E5%BA%95%E5%B1%82%E5%8E%9F%E7%90%86)
            *   [ConcurrentLinkedQueue 的核心方法有哪些?](#concurrentlinkedqueue%E7%9A%84%E6%A0%B8%E5%BF%83%E6%96%B9%E6%B3%95%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [说说 ConcurrentLinkedQueue 的 HOPS(延迟更新的策略) 的设计?](#%E8%AF%B4%E8%AF%B4concurrentlinkedqueue%E7%9A%84hops%E5%BB%B6%E8%BF%9F%E6%9B%B4%E6%96%B0%E7%9A%84%E7%AD%96%E7%95%A5%E7%9A%84%E8%AE%BE%E8%AE%A1)
            *   [ConcurrentLinkedQueue 适合什么样的使用场景?](#concurrentlinkedqueue%E9%80%82%E5%90%88%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF)
            *   [什么是 BlockingDeque? 适合用在什么样的场景?](#%E4%BB%80%E4%B9%88%E6%98%AFblockingdeque-%E9%80%82%E5%90%88%E7%94%A8%E5%9C%A8%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E5%9C%BA%E6%99%AF)
            *   [BlockingQueue 大家族有哪些?](#blockingqueue%E5%A4%A7%E5%AE%B6%E6%97%8F%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [BlockingQueue 常用的方法?](#blockingqueue%E5%B8%B8%E7%94%A8%E7%9A%84%E6%96%B9%E6%B3%95)
            *   [BlockingQueue 实现例子?](#blockingqueue-%E5%AE%9E%E7%8E%B0%E4%BE%8B%E5%AD%90)
            *   [什么是 BlockingDeque? 适合用在什么样的场景?](#%E4%BB%80%E4%B9%88%E6%98%AFblockingdeque-%E9%80%82%E5%90%88%E7%94%A8%E5%9C%A8%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E5%9C%BA%E6%99%AF-1)
            *   [BlockingDeque 与 BlockingQueue 有何关系，请对比下它们的方法?](#blockingdeque-%E4%B8%8Eblockingqueue%E6%9C%89%E4%BD%95%E5%85%B3%E7%B3%BB%E8%AF%B7%E5%AF%B9%E6%AF%94%E4%B8%8B%E5%AE%83%E4%BB%AC%E7%9A%84%E6%96%B9%E6%B3%95)
            *   [BlockingDeque 大家族有哪些?](#blockingdeque%E5%A4%A7%E5%AE%B6%E6%97%8F%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [BlockingDeque 实现例子?](#blockingdeque-%E5%AE%9E%E7%8E%B0%E4%BE%8B%E5%AD%90)
        *   [3.7 JUC 线程池](#37-juc%E7%BA%BF%E7%A8%8B%E6%B1%A0)
            *   [FutureTask 用来解决什么问题的? 为什么会出现?](#futuretask%E7%94%A8%E6%9D%A5%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98%E7%9A%84-%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BC%9A%E5%87%BA%E7%8E%B0)
            *   [FutureTask 类结构关系怎么样的?](#futuretask%E7%B1%BB%E7%BB%93%E6%9E%84%E5%85%B3%E7%B3%BB%E6%80%8E%E4%B9%88%E6%A0%B7%E7%9A%84)
            *   [FutureTask 的线程安全是由什么保证的?](#futuretask%E7%9A%84%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E6%98%AF%E7%94%B1%E4%BB%80%E4%B9%88%E4%BF%9D%E8%AF%81%E7%9A%84)
            *   [FutureTask 通常会怎么用? 举例说明。](#futuretask%E9%80%9A%E5%B8%B8%E4%BC%9A%E6%80%8E%E4%B9%88%E7%94%A8-%E4%B8%BE%E4%BE%8B%E8%AF%B4%E6%98%8E)
            *   [为什么要有线程池?](#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%9C%89%E7%BA%BF%E7%A8%8B%E6%B1%A0)
            *   [Java 是实现和管理线程池有哪些方式? 请简单举例如何使用。](#java%E6%98%AF%E5%AE%9E%E7%8E%B0%E5%92%8C%E7%AE%A1%E7%90%86%E7%BA%BF%E7%A8%8B%E6%B1%A0%E6%9C%89%E5%93%AA%E4%BA%9B%E6%96%B9%E5%BC%8F--%E8%AF%B7%E7%AE%80%E5%8D%95%E4%B8%BE%E4%BE%8B%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8)
            *   [ThreadPoolExecutor 的原理?](#threadpoolexecutor%E7%9A%84%E5%8E%9F%E7%90%86)
            *   [ThreadPoolExecutor 有哪些核心的配置参数? 请简要说明](#threadpoolexecutor%E6%9C%89%E5%93%AA%E4%BA%9B%E6%A0%B8%E5%BF%83%E7%9A%84%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0-%E8%AF%B7%E7%AE%80%E8%A6%81%E8%AF%B4%E6%98%8E)
            *   [ThreadPoolExecutor 可以创建哪是哪三种线程池呢?](#threadpoolexecutor%E5%8F%AF%E4%BB%A5%E5%88%9B%E5%BB%BA%E5%93%AA%E6%98%AF%E5%93%AA%E4%B8%89%E7%A7%8D%E7%BA%BF%E7%A8%8B%E6%B1%A0%E5%91%A2)
            *   [当队列满了并且 worker 的数量达到 maxSize 的时候，会怎么样?](#%E5%BD%93%E9%98%9F%E5%88%97%E6%BB%A1%E4%BA%86%E5%B9%B6%E4%B8%94worker%E7%9A%84%E6%95%B0%E9%87%8F%E8%BE%BE%E5%88%B0maxsize%E7%9A%84%E6%97%B6%E5%80%99%E4%BC%9A%E6%80%8E%E4%B9%88%E6%A0%B7)
            *   [说说 ThreadPoolExecutor 有哪些 RejectedExecutionHandler 策略? 默认是什么策略?](#%E8%AF%B4%E8%AF%B4threadpoolexecutor%E6%9C%89%E5%93%AA%E4%BA%9Brejectedexecutionhandler%E7%AD%96%E7%95%A5-%E9%BB%98%E8%AE%A4%E6%98%AF%E4%BB%80%E4%B9%88%E7%AD%96%E7%95%A5)
            *   [简要说下线程池的任务执行机制?](#%E7%AE%80%E8%A6%81%E8%AF%B4%E4%B8%8B%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%9A%84%E4%BB%BB%E5%8A%A1%E6%89%A7%E8%A1%8C%E6%9C%BA%E5%88%B6)
            *   [线程池中任务是如何提交的?](#%E7%BA%BF%E7%A8%8B%E6%B1%A0%E4%B8%AD%E4%BB%BB%E5%8A%A1%E6%98%AF%E5%A6%82%E4%BD%95%E6%8F%90%E4%BA%A4%E7%9A%84)
            *   [线程池中任务是如何关闭的?](#%E7%BA%BF%E7%A8%8B%E6%B1%A0%E4%B8%AD%E4%BB%BB%E5%8A%A1%E6%98%AF%E5%A6%82%E4%BD%95%E5%85%B3%E9%97%AD%E7%9A%84)
            *   [在配置线程池的时候需要考虑哪些配置因素?](#%E5%9C%A8%E9%85%8D%E7%BD%AE%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%9A%84%E6%97%B6%E5%80%99%E9%9C%80%E8%A6%81%E8%80%83%E8%99%91%E5%93%AA%E4%BA%9B%E9%85%8D%E7%BD%AE%E5%9B%A0%E7%B4%A0)
            *   [如何监控线程池的状态?](#%E5%A6%82%E4%BD%95%E7%9B%91%E6%8E%A7%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%9A%84%E7%8A%B6%E6%80%81)
            *   [为什么很多公司不允许使用 Executors 去创建线程池? 那么推荐怎么使用呢?](#%E4%B8%BA%E4%BB%80%E4%B9%88%E5%BE%88%E5%A4%9A%E5%85%AC%E5%8F%B8%E4%B8%8D%E5%85%81%E8%AE%B8%E4%BD%BF%E7%94%A8executors%E5%8E%BB%E5%88%9B%E5%BB%BA%E7%BA%BF%E7%A8%8B%E6%B1%A0-%E9%82%A3%E4%B9%88%E6%8E%A8%E8%8D%90%E6%80%8E%E4%B9%88%E4%BD%BF%E7%94%A8%E5%91%A2)
            *   [ScheduledThreadPoolExecutor 要解决什么样的问题?](#scheduledthreadpoolexecutor%E8%A6%81%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E9%97%AE%E9%A2%98)
            *   [ScheduledThreadPoolExecutor 相比 ThreadPoolExecutor 有哪些特性?](#scheduledthreadpoolexecutor%E7%9B%B8%E6%AF%94threadpoolexecutor%E6%9C%89%E5%93%AA%E4%BA%9B%E7%89%B9%E6%80%A7)
            *   [ScheduledThreadPoolExecutor 有什么样的数据结构，核心内部类和抽象类?](#scheduledthreadpoolexecutor%E6%9C%89%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E6%A0%B8%E5%BF%83%E5%86%85%E9%83%A8%E7%B1%BB%E5%92%8C%E6%8A%BD%E8%B1%A1%E7%B1%BB)
            *   [ScheduledThreadPoolExecutor 有哪两个关闭策略? 区别是什么?](#scheduledthreadpoolexecutor%E6%9C%89%E5%93%AA%E4%B8%A4%E4%B8%AA%E5%85%B3%E9%97%AD%E7%AD%96%E7%95%A5-%E5%8C%BA%E5%88%AB%E6%98%AF%E4%BB%80%E4%B9%88)
            *   [ScheduledThreadPoolExecutor 中 scheduleAtFixedRate 和 scheduleWithFixedDelay 区别是什么?](#scheduledthreadpoolexecutor%E4%B8%ADscheduleatfixedrate-%E5%92%8C-schedulewithfixeddelay%E5%8C%BA%E5%88%AB%E6%98%AF%E4%BB%80%E4%B9%88)
            *   [为什么 ThreadPoolExecutor 的调整策略却不适用于 ScheduledThreadPoolExecutor?](#%E4%B8%BA%E4%BB%80%E4%B9%88threadpoolexecutor-%E7%9A%84%E8%B0%83%E6%95%B4%E7%AD%96%E7%95%A5%E5%8D%B4%E4%B8%8D%E9%80%82%E7%94%A8%E4%BA%8E-scheduledthreadpoolexecutor)
            *   [Executors 提供了几种方法来构造 ScheduledThreadPoolExecutor?](#executors-%E6%8F%90%E4%BE%9B%E4%BA%86%E5%87%A0%E7%A7%8D%E6%96%B9%E6%B3%95%E6%9D%A5%E6%9E%84%E9%80%A0-scheduledthreadpoolexecutor)
            *   [Fork/Join 主要用来解决什么样的问题?](#forkjoin%E4%B8%BB%E8%A6%81%E7%94%A8%E6%9D%A5%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E9%97%AE%E9%A2%98)
            *   [Fork/Join 框架是在哪个 JDK 版本中引入的?](#forkjoin%E6%A1%86%E6%9E%B6%E6%98%AF%E5%9C%A8%E5%93%AA%E4%B8%AAjdk%E7%89%88%E6%9C%AC%E4%B8%AD%E5%BC%95%E5%85%A5%E7%9A%84)
            *   [Fork/Join 框架主要包含哪三个模块? 模块之间的关系是怎么样的?](#forkjoin%E6%A1%86%E6%9E%B6%E4%B8%BB%E8%A6%81%E5%8C%85%E5%90%AB%E5%93%AA%E4%B8%89%E4%B8%AA%E6%A8%A1%E5%9D%97-%E6%A8%A1%E5%9D%97%E4%B9%8B%E9%97%B4%E7%9A%84%E5%85%B3%E7%B3%BB%E6%98%AF%E6%80%8E%E4%B9%88%E6%A0%B7%E7%9A%84)
            *   [ForkJoinPool 类继承关系?](#forkjoinpool%E7%B1%BB%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB)
            *   [ForkJoinTask 抽象类继承关系?](#forkjointask%E6%8A%BD%E8%B1%A1%E7%B1%BB%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB)
            *   [整个 Fork/Join 框架的执行流程 / 运行机制是怎么样的?](#%E6%95%B4%E4%B8%AAforkjoin-%E6%A1%86%E6%9E%B6%E7%9A%84%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B%E8%BF%90%E8%A1%8C%E6%9C%BA%E5%88%B6%E6%98%AF%E6%80%8E%E4%B9%88%E6%A0%B7%E7%9A%84)
            *   [具体阐述 Fork/Join 的分治思想和 work-stealing 实现方式?](#%E5%85%B7%E4%BD%93%E9%98%90%E8%BF%B0forkjoin%E7%9A%84%E5%88%86%E6%B2%BB%E6%80%9D%E6%83%B3%E5%92%8Cwork-stealing-%E5%AE%9E%E7%8E%B0%E6%96%B9%E5%BC%8F)
            *   [有哪些 JDK 源码中使用了 Fork/Join 思想?](#%E6%9C%89%E5%93%AA%E4%BA%9Bjdk%E6%BA%90%E7%A0%81%E4%B8%AD%E4%BD%BF%E7%94%A8%E4%BA%86forkjoin%E6%80%9D%E6%83%B3)
            *   [如何使用 Executors 工具类创建 ForkJoinPool?](#%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8executors%E5%B7%A5%E5%85%B7%E7%B1%BB%E5%88%9B%E5%BB%BAforkjoinpool)
            *   [写一个例子: 用 ForkJoin 方式实现 1+2+3+...+100000?](#%E5%86%99%E4%B8%80%E4%B8%AA%E4%BE%8B%E5%AD%90-%E7%94%A8forkjoin%E6%96%B9%E5%BC%8F%E5%AE%9E%E7%8E%B0123100000)
            *   [Fork/Join 在使用时有哪些注意事项? 结合 JDK 中的斐波那契数列实例具体说明。](#forkjoin%E5%9C%A8%E4%BD%BF%E7%94%A8%E6%97%B6%E6%9C%89%E5%93%AA%E4%BA%9B%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9-%E7%BB%93%E5%90%88jdk%E4%B8%AD%E7%9A%84%E6%96%90%E6%B3%A2%E9%82%A3%E5%A5%91%E6%95%B0%E5%88%97%E5%AE%9E%E4%BE%8B%E5%85%B7%E4%BD%93%E8%AF%B4%E6%98%8E)
        *   [3.8 JUC 工具类](#38-juc%E5%B7%A5%E5%85%B7%E7%B1%BB)
            *   [什么是 CountDownLatch?](#%E4%BB%80%E4%B9%88%E6%98%AFcountdownlatch)
            *   [CountDownLatch 底层实现原理?](#countdownlatch%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86)
            *   [CountDownLatch 一次可以唤醒几个任务?](#countdownlatch%E4%B8%80%E6%AC%A1%E5%8F%AF%E4%BB%A5%E5%94%A4%E9%86%92%E5%87%A0%E4%B8%AA%E4%BB%BB%E5%8A%A1)
            *   [CountDownLatch 有哪些主要方法?](#countdownlatch%E6%9C%89%E5%93%AA%E4%BA%9B%E4%B8%BB%E8%A6%81%E6%96%B9%E6%B3%95)
            *   [写道题：实现一个容器，提供两个方法，add，size 写两个线程，线程 1 添加 10 个元素到容器中，线程 2 实现监控元素的个数，当个数到 5 个时，线程 2 给出提示并结束?](#%E5%86%99%E9%81%93%E9%A2%98%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E5%AE%B9%E5%99%A8%E6%8F%90%E4%BE%9B%E4%B8%A4%E4%B8%AA%E6%96%B9%E6%B3%95addsize-%E5%86%99%E4%B8%A4%E4%B8%AA%E7%BA%BF%E7%A8%8B%E7%BA%BF%E7%A8%8B1%E6%B7%BB%E5%8A%A010%E4%B8%AA%E5%85%83%E7%B4%A0%E5%88%B0%E5%AE%B9%E5%99%A8%E4%B8%AD%E7%BA%BF%E7%A8%8B2%E5%AE%9E%E7%8E%B0%E7%9B%91%E6%8E%A7%E5%85%83%E7%B4%A0%E7%9A%84%E4%B8%AA%E6%95%B0%E5%BD%93%E4%B8%AA%E6%95%B0%E5%88%B05%E4%B8%AA%E6%97%B6%E7%BA%BF%E7%A8%8B2%E7%BB%99%E5%87%BA%E6%8F%90%E7%A4%BA%E5%B9%B6%E7%BB%93%E6%9D%9F)
            *   [什么是 CyclicBarrier?](#%E4%BB%80%E4%B9%88%E6%98%AFcyclicbarrier)
            *   [CountDownLatch 和 CyclicBarrier 对比?](#countdownlatch%E5%92%8Ccyclicbarrier%E5%AF%B9%E6%AF%94)
            *   [什么是 Semaphore?](#%E4%BB%80%E4%B9%88%E6%98%AFsemaphore)
            *   [Semaphore 内部原理?](#semaphore%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86)
            *   [Semaphore 常用方法有哪些? 如何实现线程同步和互斥的?](#semaphore%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95%E6%9C%89%E5%93%AA%E4%BA%9B-%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%AD%A5%E5%92%8C%E4%BA%92%E6%96%A5%E7%9A%84)
            *   [单独使用 Semaphore 是不会使用到 AQS 的条件队列?](#%E5%8D%95%E7%8B%AC%E4%BD%BF%E7%94%A8semaphore%E6%98%AF%E4%B8%8D%E4%BC%9A%E4%BD%BF%E7%94%A8%E5%88%B0aqs%E7%9A%84%E6%9D%A1%E4%BB%B6%E9%98%9F%E5%88%97)
            *   [Semaphore 初始化有 10 个令牌，11 个线程同时各调用 1 次 acquire 方法，会发生什么?](#semaphore%E5%88%9D%E5%A7%8B%E5%8C%96%E6%9C%8910%E4%B8%AA%E4%BB%A4%E7%89%8C11%E4%B8%AA%E7%BA%BF%E7%A8%8B%E5%90%8C%E6%97%B6%E5%90%84%E8%B0%83%E7%94%A81%E6%AC%A1acquire%E6%96%B9%E6%B3%95%E4%BC%9A%E5%8F%91%E7%94%9F%E4%BB%80%E4%B9%88)
            *   [Semaphore 初始化有 10 个令牌，一个线程重复调用 11 次 acquire 方法，会发生什么?](#semaphore%E5%88%9D%E5%A7%8B%E5%8C%96%E6%9C%8910%E4%B8%AA%E4%BB%A4%E7%89%8C%E4%B8%80%E4%B8%AA%E7%BA%BF%E7%A8%8B%E9%87%8D%E5%A4%8D%E8%B0%83%E7%94%A811%E6%AC%A1acquire%E6%96%B9%E6%B3%95%E4%BC%9A%E5%8F%91%E7%94%9F%E4%BB%80%E4%B9%88)
            *   [Semaphore 初始化有 1 个令牌，1 个线程调用一次 acquire 方法，然后调用两次 release 方法，之后另外一个线程调用 acquire(2) 方法，此线程能够获取到足够的令牌并继续运行吗?](#semaphore%E5%88%9D%E5%A7%8B%E5%8C%96%E6%9C%891%E4%B8%AA%E4%BB%A4%E7%89%8C1%E4%B8%AA%E7%BA%BF%E7%A8%8B%E8%B0%83%E7%94%A8%E4%B8%80%E6%AC%A1acquire%E6%96%B9%E6%B3%95%E7%84%B6%E5%90%8E%E8%B0%83%E7%94%A8%E4%B8%A4%E6%AC%A1release%E6%96%B9%E6%B3%95%E4%B9%8B%E5%90%8E%E5%8F%A6%E5%A4%96%E4%B8%80%E4%B8%AA%E7%BA%BF%E7%A8%8B%E8%B0%83%E7%94%A8acquire2%E6%96%B9%E6%B3%95%E6%AD%A4%E7%BA%BF%E7%A8%8B%E8%83%BD%E5%A4%9F%E8%8E%B7%E5%8F%96%E5%88%B0%E8%B6%B3%E5%A4%9F%E7%9A%84%E4%BB%A4%E7%89%8C%E5%B9%B6%E7%BB%A7%E7%BB%AD%E8%BF%90%E8%A1%8C%E5%90%97)
            *   [Semaphore 初始化有 2 个令牌，一个线程调用 1 次 release 方法，然后一次性获取 3 个令牌，会获取到吗?](#semaphore%E5%88%9D%E5%A7%8B%E5%8C%96%E6%9C%892%E4%B8%AA%E4%BB%A4%E7%89%8C%E4%B8%80%E4%B8%AA%E7%BA%BF%E7%A8%8B%E8%B0%83%E7%94%A81%E6%AC%A1release%E6%96%B9%E6%B3%95%E7%84%B6%E5%90%8E%E4%B8%80%E6%AC%A1%E6%80%A7%E8%8E%B7%E5%8F%963%E4%B8%AA%E4%BB%A4%E7%89%8C%E4%BC%9A%E8%8E%B7%E5%8F%96%E5%88%B0%E5%90%97)
            *   [Phaser 主要用来解决什么问题?](#phaser%E4%B8%BB%E8%A6%81%E7%94%A8%E6%9D%A5%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98)
            *   [Phaser 与 CyclicBarrier 和 CountDownLatch 的区别是什么?](#phaser%E4%B8%8Ecyclicbarrier%E5%92%8Ccountdownlatch%E7%9A%84%E5%8C%BA%E5%88%AB%E6%98%AF%E4%BB%80%E4%B9%88)
            *   [Phaser 运行机制是什么样的?](#phaser%E8%BF%90%E8%A1%8C%E6%9C%BA%E5%88%B6%E6%98%AF%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84)
            *   [给一个 Phaser 使用的示例?](#%E7%BB%99%E4%B8%80%E4%B8%AAphaser%E4%BD%BF%E7%94%A8%E7%9A%84%E7%A4%BA%E4%BE%8B)
            *   [Exchanger 主要解决什么问题?](#exchanger%E4%B8%BB%E8%A6%81%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98)
            *   [对比 SynchronousQueue，为什么说 Exchanger 可被视为 SynchronousQueue 的双向形式?](#%E5%AF%B9%E6%AF%94synchronousqueue%E4%B8%BA%E4%BB%80%E4%B9%88%E8%AF%B4exchanger%E5%8F%AF%E8%A2%AB%E8%A7%86%E4%B8%BA-synchronousqueue-%E7%9A%84%E5%8F%8C%E5%90%91%E5%BD%A2%E5%BC%8F)
            *   [Exchanger 在不同的 JDK 版本中实现有什么差别?](#exchanger%E5%9C%A8%E4%B8%8D%E5%90%8C%E7%9A%84jdk%E7%89%88%E6%9C%AC%E4%B8%AD%E5%AE%9E%E7%8E%B0%E6%9C%89%E4%BB%80%E4%B9%88%E5%B7%AE%E5%88%AB)
            *   [Exchanger 实现举例](#exchanger%E5%AE%9E%E7%8E%B0%E4%B8%BE%E4%BE%8B)
            *   [什么是 ThreadLocal? 用来解决什么问题的?](#%E4%BB%80%E4%B9%88%E6%98%AFthreadlocal-%E7%94%A8%E6%9D%A5%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98%E7%9A%84)
            *   [说说你对 ThreadLocal 的理解](#%E8%AF%B4%E8%AF%B4%E4%BD%A0%E5%AF%B9threadlocal%E7%9A%84%E7%90%86%E8%A7%A3)
            *   [ThreadLocal 是如何实现线程隔离的?](#threadlocal%E6%98%AF%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%BA%BF%E7%A8%8B%E9%9A%94%E7%A6%BB%E7%9A%84)
            *   [为什么 ThreadLocal 会造成内存泄露? 如何解决](#%E4%B8%BA%E4%BB%80%E4%B9%88threadlocal%E4%BC%9A%E9%80%A0%E6%88%90%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2-%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3)
            *   [还有哪些使用 ThreadLocal 的应用场景?](#%E8%BF%98%E6%9C%89%E5%93%AA%E4%BA%9B%E4%BD%BF%E7%94%A8threadlocal%E7%9A%84%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF)
    *   [4 Java IO](#4-java-io)
        *   [4.1 基础 IO](#41-%E5%9F%BA%E7%A1%80io)
            *   [如何从数据传输方式理解 IO 流？](#%E5%A6%82%E4%BD%95%E4%BB%8E%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%93%E6%96%B9%E5%BC%8F%E7%90%86%E8%A7%A3io%E6%B5%81)
            *   [如何从数据操作上理解 IO 流？](#%E5%A6%82%E4%BD%95%E4%BB%8E%E6%95%B0%E6%8D%AE%E6%93%8D%E4%BD%9C%E4%B8%8A%E7%90%86%E8%A7%A3io%E6%B5%81)
            *   [Java IO 设计上使用了什么设计模式？](#java-io%E8%AE%BE%E8%AE%A1%E4%B8%8A%E4%BD%BF%E7%94%A8%E4%BA%86%E4%BB%80%E4%B9%88%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F)
        *   [4.2 5 种 IO 模型](#42-5%E7%A7%8Dio%E6%A8%A1%E5%9E%8B)
            *   [什么是阻塞？什么是同步？](#%E4%BB%80%E4%B9%88%E6%98%AF%E9%98%BB%E5%A1%9E%E4%BB%80%E4%B9%88%E6%98%AF%E5%90%8C%E6%AD%A5)
            *   [什么是 Linux 的 IO 模型？](#%E4%BB%80%E4%B9%88%E6%98%AFlinux%E7%9A%84io%E6%A8%A1%E5%9E%8B)
            *   [什么是同步阻塞 IO？](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%90%8C%E6%AD%A5%E9%98%BB%E5%A1%9Eio)
            *   [什么是同步非阻塞 IO？](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%90%8C%E6%AD%A5%E9%9D%9E%E9%98%BB%E5%A1%9Eio)
            *   [什么是多路复用 IO？](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8io)
            *   [有哪些多路复用 IO？](#%E6%9C%89%E5%93%AA%E4%BA%9B%E5%A4%9A%E8%B7%AF%E5%A4%8D%E7%94%A8io)
            *   [什么是信号驱动 IO？](#%E4%BB%80%E4%B9%88%E6%98%AF%E4%BF%A1%E5%8F%B7%E9%A9%B1%E5%8A%A8io)
            *   [什么是异步 IO？](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%BC%82%E6%AD%A5io)
            *   [什么是 Reactor 模型？](#%E4%BB%80%E4%B9%88%E6%98%AFreactor%E6%A8%A1%E5%9E%8B)
            *   [什么是 Java NIO？](#%E4%BB%80%E4%B9%88%E6%98%AFjava-nio)
        *   [4.3 零拷贝](#43-%E9%9B%B6%E6%8B%B7%E8%B4%9D)
            *   [传统的 IO 存在什么问题？为什么引入零拷贝的？](#%E4%BC%A0%E7%BB%9F%E7%9A%84io%E5%AD%98%E5%9C%A8%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98%E4%B8%BA%E4%BB%80%E4%B9%88%E5%BC%95%E5%85%A5%E9%9B%B6%E6%8B%B7%E8%B4%9D%E7%9A%84)
            *   [mmap + write 怎么实现的零拷贝？](#mmap--write%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0%E7%9A%84%E9%9B%B6%E6%8B%B7%E8%B4%9D)
            *   [sendfile 怎么实现的零拷贝？](#sendfile%E6%80%8E%E4%B9%88%E5%AE%9E%E7%8E%B0%E7%9A%84%E9%9B%B6%E6%8B%B7%E8%B4%9D)
    *   [5 JVM 和调优](#5-jvm%E5%92%8C%E8%B0%83%E4%BC%98)
        *   [5.1 类加载机制](#51-%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6)
            *   [类加载的生命周期？](#%E7%B1%BB%E5%8A%A0%E8%BD%BD%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)
            *   [类加载器的层次?](#%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8%E7%9A%84%E5%B1%82%E6%AC%A1)
            *   [Class.forName() 和 ClassLoader.loadClass() 区别?](#classforname%E5%92%8Cclassloaderloadclass%E5%8C%BA%E5%88%AB)
            *   [JVM 有哪些类加载机制？](#jvm%E6%9C%89%E5%93%AA%E4%BA%9B%E7%B1%BB%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6)
        *   [5.2 内存结构](#52-%E5%86%85%E5%AD%98%E7%BB%93%E6%9E%84)
            *   [说说 JVM 内存整体的结构？线程私有还是共享的？](#%E8%AF%B4%E8%AF%B4jvm%E5%86%85%E5%AD%98%E6%95%B4%E4%BD%93%E7%9A%84%E7%BB%93%E6%9E%84%E7%BA%BF%E7%A8%8B%E7%A7%81%E6%9C%89%E8%BF%98%E6%98%AF%E5%85%B1%E4%BA%AB%E7%9A%84)
            *   [什么是程序计数器（线程私有）？](#%E4%BB%80%E4%B9%88%E6%98%AF%E7%A8%8B%E5%BA%8F%E8%AE%A1%E6%95%B0%E5%99%A8%E7%BA%BF%E7%A8%8B%E7%A7%81%E6%9C%89)
            *   [什么是虚拟机栈（线程私有）？](#%E4%BB%80%E4%B9%88%E6%98%AF%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A0%88%E7%BA%BF%E7%A8%8B%E7%A7%81%E6%9C%89)
            *   [Java 虚拟机栈如何进行方法计算的？](#java%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A0%88%E5%A6%82%E4%BD%95%E8%BF%9B%E8%A1%8C%E6%96%B9%E6%B3%95%E8%AE%A1%E7%AE%97%E7%9A%84)
            *   [什么是本地方法栈（线程私有）？](#%E4%BB%80%E4%B9%88%E6%98%AF%E6%9C%AC%E5%9C%B0%E6%96%B9%E6%B3%95%E6%A0%88%E7%BA%BF%E7%A8%8B%E7%A7%81%E6%9C%89)
            *   [什么是方法区（线程共享）？](#%E4%BB%80%E4%B9%88%E6%98%AF%E6%96%B9%E6%B3%95%E5%8C%BA%E7%BA%BF%E7%A8%8B%E5%85%B1%E4%BA%AB)
            *   [永久代和元空间内存使用上的差异?](#%E6%B0%B8%E4%B9%85%E4%BB%A3%E5%92%8C%E5%85%83%E7%A9%BA%E9%97%B4%E5%86%85%E5%AD%98%E4%BD%BF%E7%94%A8%E4%B8%8A%E7%9A%84%E5%B7%AE%E5%BC%82)
            *   [堆区内存是怎么细分的？](#%E5%A0%86%E5%8C%BA%E5%86%85%E5%AD%98%E6%98%AF%E6%80%8E%E4%B9%88%E7%BB%86%E5%88%86%E7%9A%84)
            *   [JVM 中对象在堆中的生命周期?](#jvm%E4%B8%AD%E5%AF%B9%E8%B1%A1%E5%9C%A8%E5%A0%86%E4%B8%AD%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F)
            *   [JVM 中对象的分配过程?](#jvm%E4%B8%AD%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%88%86%E9%85%8D%E8%BF%87%E7%A8%8B)
            *   [什么是 TLAB （Thread Local Allocation Buffer）?](#%E4%BB%80%E4%B9%88%E6%98%AF-tlab-thread-local-allocation-buffer)
            *   [为什么要有 TLAB ?](#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%9C%89-tlab-)
        *   [5.3 GC 垃圾回收](#53-gc%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6)
            *   [如何判断一个对象是否可以回收？](#%E5%A6%82%E4%BD%95%E5%88%A4%E6%96%AD%E4%B8%80%E4%B8%AA%E5%AF%B9%E8%B1%A1%E6%98%AF%E5%90%A6%E5%8F%AF%E4%BB%A5%E5%9B%9E%E6%94%B6)
            *   [对象有哪些引用类型？](#%E5%AF%B9%E8%B1%A1%E6%9C%89%E5%93%AA%E4%BA%9B%E5%BC%95%E7%94%A8%E7%B1%BB%E5%9E%8B)
            *   [有哪些基本的垃圾回收算法？](#%E6%9C%89%E5%93%AA%E4%BA%9B%E5%9F%BA%E6%9C%AC%E7%9A%84%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E7%AE%97%E6%B3%95)
            *   [分代收集算法和分区收集算法区别？](#%E5%88%86%E4%BB%A3%E6%94%B6%E9%9B%86%E7%AE%97%E6%B3%95%E5%92%8C%E5%88%86%E5%8C%BA%E6%94%B6%E9%9B%86%E7%AE%97%E6%B3%95%E5%8C%BA%E5%88%AB)
            *   [什么是 Minor GC、Major GC、Full GC?](#%E4%BB%80%E4%B9%88%E6%98%AFminor-gcmajor-gcfull-gc)
            *   [说说 JVM 内存分配策略？](#%E8%AF%B4%E8%AF%B4jvm%E5%86%85%E5%AD%98%E5%88%86%E9%85%8D%E7%AD%96%E7%95%A5)
            *   [什么情况下会触发 Full GC？](#%E4%BB%80%E4%B9%88%E6%83%85%E5%86%B5%E4%B8%8B%E4%BC%9A%E8%A7%A6%E5%8F%91full-gc)
            *   [Hotspot 中有哪些垃圾回收器？](#hotspot%E4%B8%AD%E6%9C%89%E5%93%AA%E4%BA%9B%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8)
        *   [5.4 问题排查](#54-%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5)
            *   [常见的 Linux 定位问题的工具？](#%E5%B8%B8%E8%A7%81%E7%9A%84linux%E5%AE%9A%E4%BD%8D%E9%97%AE%E9%A2%98%E7%9A%84%E5%B7%A5%E5%85%B7)
            *   [JDK 自带的定位问题的工具？](#jdk%E8%87%AA%E5%B8%A6%E7%9A%84%E5%AE%9A%E4%BD%8D%E9%97%AE%E9%A2%98%E7%9A%84%E5%B7%A5%E5%85%B7)
            *   [如何使用在线调试工具 Arthas？](#%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8%E5%9C%A8%E7%BA%BF%E8%B0%83%E8%AF%95%E5%B7%A5%E5%85%B7arthas)
            *   [如何使用 Idea 的远程调试？](#%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8idea%E7%9A%84%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95)
            *   [复杂综合类型问题的定位思路？](#%E5%A4%8D%E6%9D%82%E7%BB%BC%E5%90%88%E7%B1%BB%E5%9E%8B%E9%97%AE%E9%A2%98%E7%9A%84%E5%AE%9A%E4%BD%8D%E6%80%9D%E8%B7%AF)
    *   [6 Java 新版本](#6-java-%E6%96%B0%E7%89%88%E6%9C%AC)
        *   [6.1 Java 8 特性](#61-java-8-%E7%89%B9%E6%80%A7)
            *   [什么是函数式编程？Lambda 表达式？](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8Blambda%E8%A1%A8%E8%BE%BE%E5%BC%8F)
            *   [Stream 中常用方法？](#stream%E4%B8%AD%E5%B8%B8%E7%94%A8%E6%96%B9%E6%B3%95)
            *   [什么是 FunctionalInterface？](#%E4%BB%80%E4%B9%88%E6%98%AFfunctionalinterface)
            *   [如何自定义函数接口？](#%E5%A6%82%E4%BD%95%E8%87%AA%E5%AE%9A%E4%B9%89%E5%87%BD%E6%95%B0%E6%8E%A5%E5%8F%A3)
            *   [内置的四大函数接口及使用？](#%E5%86%85%E7%BD%AE%E7%9A%84%E5%9B%9B%E5%A4%A7%E5%87%BD%E6%95%B0%E6%8E%A5%E5%8F%A3%E5%8F%8A%E4%BD%BF%E7%94%A8)
            *   [Optional 要解决什么问题？](#optional%E8%A6%81%E8%A7%A3%E5%86%B3%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98)
            *   [如何使用 Optional 来解决嵌套对象的判空问题？](#%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8optional%E6%9D%A5%E8%A7%A3%E5%86%B3%E5%B5%8C%E5%A5%97%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%88%A4%E7%A9%BA%E9%97%AE%E9%A2%98)
            *   [什么是默认方法，为什么要有默认方法？](#%E4%BB%80%E4%B9%88%E6%98%AF%E9%BB%98%E8%AE%A4%E6%96%B9%E6%B3%95%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%9C%89%E9%BB%98%E8%AE%A4%E6%96%B9%E6%B3%95)
            *   [什么是类型注解？](#%E4%BB%80%E4%B9%88%E6%98%AF%E7%B1%BB%E5%9E%8B%E6%B3%A8%E8%A7%A3)
            *   [什么是重复注解？](#%E4%BB%80%E4%B9%88%E6%98%AF%E9%87%8D%E5%A4%8D%E6%B3%A8%E8%A7%A3)
        *   [6.2 Java 9+ 特性](#62-java-9-%E7%89%B9%E6%80%A7)
            *   [Java 9 后续版本发布是按照什么样的发布策略呢？](#java-9%E5%90%8E%E7%BB%AD%E7%89%88%E6%9C%AC%E5%8F%91%E5%B8%83%E6%98%AF%E6%8C%89%E7%85%A7%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84%E5%8F%91%E5%B8%83%E7%AD%96%E7%95%A5%E5%91%A2)
            *   [Java 9 后续新版本中你知道哪些？](#java-9%E5%90%8E%E7%BB%AD%E6%96%B0%E7%89%88%E6%9C%AC%E4%B8%AD%E4%BD%A0%E7%9F%A5%E9%81%93%E5%93%AA%E4%BA%9B)
    *   [7 数据结构和算法](#7-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%92%8C%E7%AE%97%E6%B3%95)
        *   [7.1 数据结构基础](#71-%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E5%9F%BA%E7%A1%80)
            *   [如何理解基础的数据结构？](#%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3%E5%9F%BA%E7%A1%80%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
        *   [7.2 算法思想](#72-%E7%AE%97%E6%B3%95%E6%80%9D%E6%83%B3)
            *   [有哪些常见的算法思想？](#%E6%9C%89%E5%93%AA%E4%BA%9B%E5%B8%B8%E8%A7%81%E7%9A%84%E7%AE%97%E6%B3%95%E6%80%9D%E6%83%B3)
        *   [7.3 常见排序算法](#73-%E5%B8%B8%E8%A7%81%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)
            *   [有哪些常见的排序算法？](#%E6%9C%89%E5%93%AA%E4%BA%9B%E5%B8%B8%E8%A7%81%E7%9A%84%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)
        *   [7.4 大数据处理算法](#74-%E5%A4%A7%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E7%AE%97%E6%B3%95)
            *   [何谓海量数据处理? 解决的思路？](#%E4%BD%95%E8%B0%93%E6%B5%B7%E9%87%8F%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86-%E8%A7%A3%E5%86%B3%E7%9A%84%E6%80%9D%E8%B7%AF)
            *   [大数据处理之分治思想？](#%E5%A4%A7%E6%95%B0%E6%8D%AE%E5%A4%84%E7%90%86%E4%B9%8B%E5%88%86%E6%B2%BB%E6%80%9D%E6%83%B3)
            *   [海量日志数据，提取出某日访问百度次数最多的那个 IP?](#%E6%B5%B7%E9%87%8F%E6%97%A5%E5%BF%97%E6%95%B0%E6%8D%AE%E6%8F%90%E5%8F%96%E5%87%BA%E6%9F%90%E6%97%A5%E8%AE%BF%E9%97%AE%E7%99%BE%E5%BA%A6%E6%AC%A1%E6%95%B0%E6%9C%80%E5%A4%9A%E7%9A%84%E9%82%A3%E4%B8%AAip)
            *   [寻找热门查询，300 万个查询字符串中统计最热门的 10 个查询?](#%E5%AF%BB%E6%89%BE%E7%83%AD%E9%97%A8%E6%9F%A5%E8%AF%A2300%E4%B8%87%E4%B8%AA%E6%9F%A5%E8%AF%A2%E5%AD%97%E7%AC%A6%E4%B8%B2%E4%B8%AD%E7%BB%9F%E8%AE%A1%E6%9C%80%E7%83%AD%E9%97%A8%E7%9A%8410%E4%B8%AA%E6%9F%A5%E8%AF%A2)
            *   [有一个 1G 大小的一个文件，里面每一行是一个词，词的大小不超过 16 字节，内存限制大小是 1M。返回频数最高的 100 个词?](#%E6%9C%89%E4%B8%80%E4%B8%AA1g%E5%A4%A7%E5%B0%8F%E7%9A%84%E4%B8%80%E4%B8%AA%E6%96%87%E4%BB%B6%E9%87%8C%E9%9D%A2%E6%AF%8F%E4%B8%80%E8%A1%8C%E6%98%AF%E4%B8%80%E4%B8%AA%E8%AF%8D%E8%AF%8D%E7%9A%84%E5%A4%A7%E5%B0%8F%E4%B8%8D%E8%B6%85%E8%BF%8716%E5%AD%97%E8%8A%82%E5%86%85%E5%AD%98%E9%99%90%E5%88%B6%E5%A4%A7%E5%B0%8F%E6%98%AF1m%E8%BF%94%E5%9B%9E%E9%A2%91%E6%95%B0%E6%9C%80%E9%AB%98%E7%9A%84100%E4%B8%AA%E8%AF%8D)
            *   [海量数据分布在 100 台电脑中，想个办法高效统计出这批数据的 TOP10?](#%E6%B5%B7%E9%87%8F%E6%95%B0%E6%8D%AE%E5%88%86%E5%B8%83%E5%9C%A8100%E5%8F%B0%E7%94%B5%E8%84%91%E4%B8%AD%E6%83%B3%E4%B8%AA%E5%8A%9E%E6%B3%95%E9%AB%98%E6%95%88%E7%BB%9F%E8%AE%A1%E5%87%BA%E8%BF%99%E6%89%B9%E6%95%B0%E6%8D%AE%E7%9A%84top10)
            *   [有 10 个文件，每个文件 1G，每个文件的每一行存放的都是用户的 query，每个文件的 query 都可能重复。要求你按照 query 的频度排序?](#%E6%9C%8910%E4%B8%AA%E6%96%87%E4%BB%B6%E6%AF%8F%E4%B8%AA%E6%96%87%E4%BB%B61g%E6%AF%8F%E4%B8%AA%E6%96%87%E4%BB%B6%E7%9A%84%E6%AF%8F%E4%B8%80%E8%A1%8C%E5%AD%98%E6%94%BE%E7%9A%84%E9%83%BD%E6%98%AF%E7%94%A8%E6%88%B7%E7%9A%84query%E6%AF%8F%E4%B8%AA%E6%96%87%E4%BB%B6%E7%9A%84query%E9%83%BD%E5%8F%AF%E8%83%BD%E9%87%8D%E5%A4%8D%E8%A6%81%E6%B1%82%E4%BD%A0%E6%8C%89%E7%85%A7query%E7%9A%84%E9%A2%91%E5%BA%A6%E6%8E%92%E5%BA%8F)
            *   [给定 a、b 两个文件，各存放 50 亿个 url，每个 url 各占 64 字节，内存限制是 4G，让你找出 a、b 文件共同的 url?](#%E7%BB%99%E5%AE%9Aab%E4%B8%A4%E4%B8%AA%E6%96%87%E4%BB%B6%E5%90%84%E5%AD%98%E6%94%BE50%E4%BA%BF%E4%B8%AAurl%E6%AF%8F%E4%B8%AAurl%E5%90%84%E5%8D%A064%E5%AD%97%E8%8A%82%E5%86%85%E5%AD%98%E9%99%90%E5%88%B6%E6%98%AF4g%E8%AE%A9%E4%BD%A0%E6%89%BE%E5%87%BAab%E6%96%87%E4%BB%B6%E5%85%B1%E5%90%8C%E7%9A%84url)
            *   [怎么在海量数据中找出重复次数最多的一个?](#%E6%80%8E%E4%B9%88%E5%9C%A8%E6%B5%B7%E9%87%8F%E6%95%B0%E6%8D%AE%E4%B8%AD%E6%89%BE%E5%87%BA%E9%87%8D%E5%A4%8D%E6%AC%A1%E6%95%B0%E6%9C%80%E5%A4%9A%E7%9A%84%E4%B8%80%E4%B8%AA)
            *   [上千万或上亿数据 (有重复)，统计其中出现次数最多的前 N 个数据?](#%E4%B8%8A%E5%8D%83%E4%B8%87%E6%88%96%E4%B8%8A%E4%BA%BF%E6%95%B0%E6%8D%AE%E6%9C%89%E9%87%8D%E5%A4%8D%E7%BB%9F%E8%AE%A1%E5%85%B6%E4%B8%AD%E5%87%BA%E7%8E%B0%E6%AC%A1%E6%95%B0%E6%9C%80%E5%A4%9A%E7%9A%84%E5%89%8Dn%E4%B8%AA%E6%95%B0%E6%8D%AE)
            *   [一个文本文件，大约有一万行，每行一个词，要求统计出其中最频繁出现的前 10 个词，请给出思想，给出时间复杂度分析?](#%E4%B8%80%E4%B8%AA%E6%96%87%E6%9C%AC%E6%96%87%E4%BB%B6%E5%A4%A7%E7%BA%A6%E6%9C%89%E4%B8%80%E4%B8%87%E8%A1%8C%E6%AF%8F%E8%A1%8C%E4%B8%80%E4%B8%AA%E8%AF%8D%E8%A6%81%E6%B1%82%E7%BB%9F%E8%AE%A1%E5%87%BA%E5%85%B6%E4%B8%AD%E6%9C%80%E9%A2%91%E7%B9%81%E5%87%BA%E7%8E%B0%E7%9A%84%E5%89%8D10%E4%B8%AA%E8%AF%8D%E8%AF%B7%E7%BB%99%E5%87%BA%E6%80%9D%E6%83%B3%E7%BB%99%E5%87%BA%E6%97%B6%E9%97%B4%E5%A4%8D%E6%9D%82%E5%BA%A6%E5%88%86%E6%9E%90)
            *   [一个文本文件，找出前 10 个经常出现的词，但这次文件比较长，说是上亿行或十亿行，总之无法一次读入内存，问最优解?](#%E4%B8%80%E4%B8%AA%E6%96%87%E6%9C%AC%E6%96%87%E4%BB%B6%E6%89%BE%E5%87%BA%E5%89%8D10%E4%B8%AA%E7%BB%8F%E5%B8%B8%E5%87%BA%E7%8E%B0%E7%9A%84%E8%AF%8D%E4%BD%86%E8%BF%99%E6%AC%A1%E6%96%87%E4%BB%B6%E6%AF%94%E8%BE%83%E9%95%BF%E8%AF%B4%E6%98%AF%E4%B8%8A%E4%BA%BF%E8%A1%8C%E6%88%96%E5%8D%81%E4%BA%BF%E8%A1%8C%E6%80%BB%E4%B9%8B%E6%97%A0%E6%B3%95%E4%B8%80%E6%AC%A1%E8%AF%BB%E5%85%A5%E5%86%85%E5%AD%98%E9%97%AE%E6%9C%80%E4%BC%98%E8%A7%A3)
            *   [100w 个数中找出最大的 100 个数?](#100w%E4%B8%AA%E6%95%B0%E4%B8%AD%E6%89%BE%E5%87%BA%E6%9C%80%E5%A4%A7%E7%9A%84100%E4%B8%AA%E6%95%B0)
            *   [5 亿个 int 找它们的中位数?](#5%E4%BA%BF%E4%B8%AAint%E6%89%BE%E5%AE%83%E4%BB%AC%E7%9A%84%E4%B8%AD%E4%BD%8D%E6%95%B0)
            *   [在 2.5 亿个整数中找出不重复的整数，注，内存不足以容纳这 2.5 亿个整数。](#%E5%9C%A825%E4%BA%BF%E4%B8%AA%E6%95%B4%E6%95%B0%E4%B8%AD%E6%89%BE%E5%87%BA%E4%B8%8D%E9%87%8D%E5%A4%8D%E7%9A%84%E6%95%B4%E6%95%B0%E6%B3%A8%E5%86%85%E5%AD%98%E4%B8%8D%E8%B6%B3%E4%BB%A5%E5%AE%B9%E7%BA%B3%E8%BF%9925%E4%BA%BF%E4%B8%AA%E6%95%B4%E6%95%B0)
            *   [给 40 亿个不重复的 unsigned int 的整数，没排过序的，然后再给一个数，如何快速判断这个数是否在那 40 亿个数当中?](#%E7%BB%9940%E4%BA%BF%E4%B8%AA%E4%B8%8D%E9%87%8D%E5%A4%8D%E7%9A%84unsigned-int%E7%9A%84%E6%95%B4%E6%95%B0%E6%B2%A1%E6%8E%92%E8%BF%87%E5%BA%8F%E7%9A%84%E7%84%B6%E5%90%8E%E5%86%8D%E7%BB%99%E4%B8%80%E4%B8%AA%E6%95%B0%E5%A6%82%E4%BD%95%E5%BF%AB%E9%80%9F%E5%88%A4%E6%96%AD%E8%BF%99%E4%B8%AA%E6%95%B0%E6%98%AF%E5%90%A6%E5%9C%A8%E9%82%A340%E4%BA%BF%E4%B8%AA%E6%95%B0%E5%BD%93%E4%B8%AD)
        *   [7.5 加密算法](#75-%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95)
            *   [什么是摘要算法？有哪些?](#%E4%BB%80%E4%B9%88%E6%98%AF%E6%91%98%E8%A6%81%E7%AE%97%E6%B3%95%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [什么是加密算法？有哪些？](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%8A%A0%E5%AF%86%E7%AE%97%E6%B3%95%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [什么是国密算法？有哪些？](#%E4%BB%80%E4%B9%88%E6%98%AF%E5%9B%BD%E5%AF%86%E7%AE%97%E6%B3%95%E6%9C%89%E5%93%AA%E4%BA%9B)
    *   [8 数据库](#8-%E6%95%B0%E6%8D%AE%E5%BA%93)
        *   [8.1 原理和 SQL](#81-%E5%8E%9F%E7%90%86%E5%92%8Csql)
            *   [什么是事务？事务基本特性 ACID？](#%E4%BB%80%E4%B9%88%E6%98%AF%E4%BA%8B%E5%8A%A1%E4%BA%8B%E5%8A%A1%E5%9F%BA%E6%9C%AC%E7%89%B9%E6%80%A7acid)
            *   [数据库中并发一致性问题？](#%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%AD%E5%B9%B6%E5%8F%91%E4%B8%80%E8%87%B4%E6%80%A7%E9%97%AE%E9%A2%98)
            *   [事务的隔离等级？](#%E4%BA%8B%E5%8A%A1%E7%9A%84%E9%9A%94%E7%A6%BB%E7%AD%89%E7%BA%A7)
            *   [ACID 靠什么保证的呢？](#acid%E9%9D%A0%E4%BB%80%E4%B9%88%E4%BF%9D%E8%AF%81%E7%9A%84%E5%91%A2)
            *   [SQL 优化的实践经验？](#sql-%E4%BC%98%E5%8C%96%E7%9A%84%E5%AE%9E%E8%B7%B5%E7%BB%8F%E9%AA%8C)
            *   [Buffer Pool、Redo Log Buffer 和 undo log、redo log、bin log 概念以及关系？](#buffer-poolredo-log-buffer-%E5%92%8Cundo-logredo-logbin-log-%E6%A6%82%E5%BF%B5%E4%BB%A5%E5%8F%8A%E5%85%B3%E7%B3%BB)
            *   [从准备更新一条数据到事务的提交的流程描述？](#%E4%BB%8E%E5%87%86%E5%A4%87%E6%9B%B4%E6%96%B0%E4%B8%80%E6%9D%A1%E6%95%B0%E6%8D%AE%E5%88%B0%E4%BA%8B%E5%8A%A1%E7%9A%84%E6%8F%90%E4%BA%A4%E7%9A%84%E6%B5%81%E7%A8%8B%E6%8F%8F%E8%BF%B0)
        *   [8.2 MySQL](#82-mysql)
            *   [能说下 myisam 和 innodb 的区别吗？](#%E8%83%BD%E8%AF%B4%E4%B8%8Bmyisam-%E5%92%8C-innodb%E7%9A%84%E5%8C%BA%E5%88%AB%E5%90%97)
            *   [说下 MySQL 的索引有哪些吧？](#%E8%AF%B4%E4%B8%8Bmysql%E7%9A%84%E7%B4%A2%E5%BC%95%E6%9C%89%E5%93%AA%E4%BA%9B%E5%90%A7)
            *   [什么是 B + 树？为什么 B + 树成为主要的 SQL 数据库的索引实现？](#%E4%BB%80%E4%B9%88%E6%98%AFb%E6%A0%91%E4%B8%BA%E4%BB%80%E4%B9%88b%E6%A0%91%E6%88%90%E4%B8%BA%E4%B8%BB%E8%A6%81%E7%9A%84sql%E6%95%B0%E6%8D%AE%E5%BA%93%E7%9A%84%E7%B4%A2%E5%BC%95%E5%AE%9E%E7%8E%B0)
            *   [那你知道什么是覆盖索引和回表吗？](#%E9%82%A3%E4%BD%A0%E7%9F%A5%E9%81%93%E4%BB%80%E4%B9%88%E6%98%AF%E8%A6%86%E7%9B%96%E7%B4%A2%E5%BC%95%E5%92%8C%E5%9B%9E%E8%A1%A8%E5%90%97)
            *   [什么是 MVCC？ 说说 MySQL 实现 MVCC 的原理？](#%E4%BB%80%E4%B9%88%E6%98%AFmvcc-%E8%AF%B4%E8%AF%B4mysql%E5%AE%9E%E7%8E%B0mvcc%E7%9A%84%E5%8E%9F%E7%90%86)
            *   [MySQL 锁的类型有哪些呢？](#mysql-%E9%94%81%E7%9A%84%E7%B1%BB%E5%9E%8B%E6%9C%89%E5%93%AA%E4%BA%9B%E5%91%A2)
            *   [你们数据量级多大？分库分表怎么做的？](#%E4%BD%A0%E4%BB%AC%E6%95%B0%E6%8D%AE%E9%87%8F%E7%BA%A7%E5%A4%9A%E5%A4%A7%E5%88%86%E5%BA%93%E5%88%86%E8%A1%A8%E6%80%8E%E4%B9%88%E5%81%9A%E7%9A%84)
            *   [那分表后的 ID 怎么保证唯一性的呢？](#%E9%82%A3%E5%88%86%E8%A1%A8%E5%90%8E%E7%9A%84id%E6%80%8E%E4%B9%88%E4%BF%9D%E8%AF%81%E5%94%AF%E4%B8%80%E6%80%A7%E7%9A%84%E5%91%A2)
            *   [分表后非 sharding_key 的查询怎么处理呢？](#%E5%88%86%E8%A1%A8%E5%90%8E%E9%9D%9Esharding_key%E7%9A%84%E6%9F%A5%E8%AF%A2%E6%80%8E%E4%B9%88%E5%A4%84%E7%90%86%E5%91%A2)
            *   [MySQL 主从复制？](#mysql%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6)
            *   [MySQL 主从的延迟怎么解决呢？](#mysql%E4%B8%BB%E4%BB%8E%E7%9A%84%E5%BB%B6%E8%BF%9F%E6%80%8E%E4%B9%88%E8%A7%A3%E5%86%B3%E5%91%A2)
            *   [MySQL 读写分离方案?](#mysql%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB%E6%96%B9%E6%A1%88)
        *   [8.3 Redis](#83-redis)
            *   [什么是 Redis，为什么用 Redis？](#%E4%BB%80%E4%B9%88%E6%98%AFredis%E4%B8%BA%E4%BB%80%E4%B9%88%E7%94%A8redis)
            *   [为什么 Redis 是单线程的以及为什么这么快？](#%E4%B8%BA%E4%BB%80%E4%B9%88redis-%E6%98%AF%E5%8D%95%E7%BA%BF%E7%A8%8B%E7%9A%84%E4%BB%A5%E5%8F%8A%E4%B8%BA%E4%BB%80%E4%B9%88%E8%BF%99%E4%B9%88%E5%BF%AB)
            *   [Redis 一般有哪些使用场景？](#redis-%E4%B8%80%E8%88%AC%E6%9C%89%E5%93%AA%E4%BA%9B%E4%BD%BF%E7%94%A8%E5%9C%BA%E6%99%AF)
            *   [Redis 有哪些数据类型？](#redis-%E6%9C%89%E5%93%AA%E4%BA%9B%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)
            *   [谈谈 Redis 的对象机制（redisObject)？](#%E8%B0%88%E8%B0%88redis-%E7%9A%84%E5%AF%B9%E8%B1%A1%E6%9C%BA%E5%88%B6redisobject)
            *   [Redis 数据类型有哪些底层数据结构？](#redis-%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B%E6%9C%89%E5%93%AA%E4%BA%9B%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84)
            *   [为什么要设计 sds？](#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E8%AE%BE%E8%AE%A1sds)
            *   [Redis 一个字符串类型的值能存储最大容量是多少？](#redis-%E4%B8%80%E4%B8%AA%E5%AD%97%E7%AC%A6%E4%B8%B2%E7%B1%BB%E5%9E%8B%E7%9A%84%E5%80%BC%E8%83%BD%E5%AD%98%E5%82%A8%E6%9C%80%E5%A4%A7%E5%AE%B9%E9%87%8F%E6%98%AF%E5%A4%9A%E5%B0%91)
            *   [为什么会设计 Stream？](#%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BC%9A%E8%AE%BE%E8%AE%A1stream)
            *   [Redis Stream 用在什么样场景？](#redis-stream%E7%94%A8%E5%9C%A8%E4%BB%80%E4%B9%88%E6%A0%B7%E5%9C%BA%E6%99%AF)
            *   [Redis Stream 消息 ID 的设计是否考虑了时间回拨的问题？](#redis-stream%E6%B6%88%E6%81%AFid%E7%9A%84%E8%AE%BE%E8%AE%A1%E6%98%AF%E5%90%A6%E8%80%83%E8%99%91%E4%BA%86%E6%97%B6%E9%97%B4%E5%9B%9E%E6%8B%A8%E7%9A%84%E9%97%AE%E9%A2%98)
            *   [Redis Stream 消费者崩溃带来的会不会消息丢失问题?](#redis-stream%E6%B6%88%E8%B4%B9%E8%80%85%E5%B4%A9%E6%BA%83%E5%B8%A6%E6%9D%A5%E7%9A%84%E4%BC%9A%E4%B8%8D%E4%BC%9A%E6%B6%88%E6%81%AF%E4%B8%A2%E5%A4%B1%E9%97%AE%E9%A2%98)
            *   [Redis Steam 坏消息问题，死信问题?](#redis-steam-%E5%9D%8F%E6%B6%88%E6%81%AF%E9%97%AE%E9%A2%98%E6%AD%BB%E4%BF%A1%E9%97%AE%E9%A2%98)
            *   [Redis 的持久化机制是什么？各自的优缺点？一般怎么用？](#redis-%E7%9A%84%E6%8C%81%E4%B9%85%E5%8C%96%E6%9C%BA%E5%88%B6%E6%98%AF%E4%BB%80%E4%B9%88%E5%90%84%E8%87%AA%E7%9A%84%E4%BC%98%E7%BC%BA%E7%82%B9%E4%B8%80%E8%88%AC%E6%80%8E%E4%B9%88%E7%94%A8)
            *   [RDB 触发方式?](#rdb-%E8%A7%A6%E5%8F%91%E6%96%B9%E5%BC%8F)
            *   [RDB 由于生产环境中我们为 Redis 开辟的内存区域都比较大（例如 6GB），那么将内存中的数据同步到硬盘的过程可能就会持续比较长的时间，而实际情况是这段时间 Redis 服务一般都会收到数据写操作请求。那么如何保证数据一致性呢？](#rdb%E7%94%B1%E4%BA%8E%E7%94%9F%E4%BA%A7%E7%8E%AF%E5%A2%83%E4%B8%AD%E6%88%91%E4%BB%AC%E4%B8%BAredis%E5%BC%80%E8%BE%9F%E7%9A%84%E5%86%85%E5%AD%98%E5%8C%BA%E5%9F%9F%E9%83%BD%E6%AF%94%E8%BE%83%E5%A4%A7%E4%BE%8B%E5%A6%826gb%E9%82%A3%E4%B9%88%E5%B0%86%E5%86%85%E5%AD%98%E4%B8%AD%E7%9A%84%E6%95%B0%E6%8D%AE%E5%90%8C%E6%AD%A5%E5%88%B0%E7%A1%AC%E7%9B%98%E7%9A%84%E8%BF%87%E7%A8%8B%E5%8F%AF%E8%83%BD%E5%B0%B1%E4%BC%9A%E6%8C%81%E7%BB%AD%E6%AF%94%E8%BE%83%E9%95%BF%E7%9A%84%E6%97%B6%E9%97%B4%E8%80%8C%E5%AE%9E%E9%99%85%E6%83%85%E5%86%B5%E6%98%AF%E8%BF%99%E6%AE%B5%E6%97%B6%E9%97%B4redis%E6%9C%8D%E5%8A%A1%E4%B8%80%E8%88%AC%E9%83%BD%E4%BC%9A%E6%94%B6%E5%88%B0%E6%95%B0%E6%8D%AE%E5%86%99%E6%93%8D%E4%BD%9C%E8%AF%B7%E6%B1%82%E9%82%A3%E4%B9%88%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E6%95%B0%E6%8D%AE%E4%B8%80%E8%87%B4%E6%80%A7%E5%91%A2)
            *   [在进行 RDB 快照操作的这段时间，如果发生服务崩溃怎么办？](#%E5%9C%A8%E8%BF%9B%E8%A1%8Crdb%E5%BF%AB%E7%85%A7%E6%93%8D%E4%BD%9C%E7%9A%84%E8%BF%99%E6%AE%B5%E6%97%B6%E9%97%B4%E5%A6%82%E6%9E%9C%E5%8F%91%E7%94%9F%E6%9C%8D%E5%8A%A1%E5%B4%A9%E6%BA%83%E6%80%8E%E4%B9%88%E5%8A%9E)
            *   [可以每秒做一次 RDB 快照吗？](#%E5%8F%AF%E4%BB%A5%E6%AF%8F%E7%A7%92%E5%81%9A%E4%B8%80%E6%AC%A1rdb%E5%BF%AB%E7%85%A7%E5%90%97)
            *   [AOF 是写前日志还是写后日志？](#aof%E6%98%AF%E5%86%99%E5%89%8D%E6%97%A5%E5%BF%97%E8%BF%98%E6%98%AF%E5%86%99%E5%90%8E%E6%97%A5%E5%BF%97)
            *   [如何实现 AOF 的？](#%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0aof%E7%9A%84)
            *   [什么是 AOF 重写？](#%E4%BB%80%E4%B9%88%E6%98%AFaof%E9%87%8D%E5%86%99)
            *   [AOF 重写会阻塞吗？](#aof%E9%87%8D%E5%86%99%E4%BC%9A%E9%98%BB%E5%A1%9E%E5%90%97)
            *   [AOF 日志何时会重写？](#aof%E6%97%A5%E5%BF%97%E4%BD%95%E6%97%B6%E4%BC%9A%E9%87%8D%E5%86%99)
            *   [AOF 重写日志时，有新数据写入咋整？](#aof%E9%87%8D%E5%86%99%E6%97%A5%E5%BF%97%E6%97%B6%E6%9C%89%E6%96%B0%E6%95%B0%E6%8D%AE%E5%86%99%E5%85%A5%E5%92%8B%E6%95%B4)
            *   [主线程 fork 出子进程的是如何复制内存数据的？](#%E4%B8%BB%E7%BA%BF%E7%A8%8Bfork%E5%87%BA%E5%AD%90%E8%BF%9B%E7%A8%8B%E7%9A%84%E6%98%AF%E5%A6%82%E4%BD%95%E5%A4%8D%E5%88%B6%E5%86%85%E5%AD%98%E6%95%B0%E6%8D%AE%E7%9A%84)
            *   [在重写日志整个过程时，主线程有哪些地方会被阻塞？](#%E5%9C%A8%E9%87%8D%E5%86%99%E6%97%A5%E5%BF%97%E6%95%B4%E4%B8%AA%E8%BF%87%E7%A8%8B%E6%97%B6%E4%B8%BB%E7%BA%BF%E7%A8%8B%E6%9C%89%E5%93%AA%E4%BA%9B%E5%9C%B0%E6%96%B9%E4%BC%9A%E8%A2%AB%E9%98%BB%E5%A1%9E)
            *   [为什么 AOF 重写不复用原 AOF 日志？](#%E4%B8%BA%E4%BB%80%E4%B9%88aof%E9%87%8D%E5%86%99%E4%B8%8D%E5%A4%8D%E7%94%A8%E5%8E%9Faof%E6%97%A5%E5%BF%97)
            *   [Redis 过期键的删除策略有哪些?](#redis-%E8%BF%87%E6%9C%9F%E9%94%AE%E7%9A%84%E5%88%A0%E9%99%A4%E7%AD%96%E7%95%A5%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [Redis 内存淘汰算法有哪些?](#redis-%E5%86%85%E5%AD%98%E6%B7%98%E6%B1%B0%E7%AE%97%E6%B3%95%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [Redis 的内存用完了会发生什么？](#redis%E7%9A%84%E5%86%85%E5%AD%98%E7%94%A8%E5%AE%8C%E4%BA%86%E4%BC%9A%E5%8F%91%E7%94%9F%E4%BB%80%E4%B9%88)
            *   [Redis 如何做内存优化？](#redis%E5%A6%82%E4%BD%95%E5%81%9A%E5%86%85%E5%AD%98%E4%BC%98%E5%8C%96)
            *   [Redis key 的过期时间和永久有效分别怎么设置？](#redis-key-%E7%9A%84%E8%BF%87%E6%9C%9F%E6%97%B6%E9%97%B4%E5%92%8C%E6%B0%B8%E4%B9%85%E6%9C%89%E6%95%88%E5%88%86%E5%88%AB%E6%80%8E%E4%B9%88%E8%AE%BE%E7%BD%AE)
            *   [Redis 中的管道有什么用？](#redis-%E4%B8%AD%E7%9A%84%E7%AE%A1%E9%81%93%E6%9C%89%E4%BB%80%E4%B9%88%E7%94%A8)
            *   [什么是 redis 事务？](#%E4%BB%80%E4%B9%88%E6%98%AFredis%E4%BA%8B%E5%8A%A1)
            *   [Redis 事务相关命令？](#redis%E4%BA%8B%E5%8A%A1%E7%9B%B8%E5%85%B3%E5%91%BD%E4%BB%A4)
            *   [Redis 事务的三个阶段？](#redis%E4%BA%8B%E5%8A%A1%E7%9A%84%E4%B8%89%E4%B8%AA%E9%98%B6%E6%AE%B5)
            *   [Redis 事务其它实现？](#redis%E4%BA%8B%E5%8A%A1%E5%85%B6%E5%AE%83%E5%AE%9E%E7%8E%B0)
            *   [Redis 事务中出现错误的处理？](#redis%E4%BA%8B%E5%8A%A1%E4%B8%AD%E5%87%BA%E7%8E%B0%E9%94%99%E8%AF%AF%E7%9A%84%E5%A4%84%E7%90%86)
            *   [Redis 事务中 watch 是如何监视实现的呢？](#redis%E4%BA%8B%E5%8A%A1%E4%B8%ADwatch%E6%98%AF%E5%A6%82%E4%BD%95%E7%9B%91%E8%A7%86%E5%AE%9E%E7%8E%B0%E7%9A%84%E5%91%A2)
            *   [为什么 Redis 不支持回滚？](#%E4%B8%BA%E4%BB%80%E4%B9%88-redis-%E4%B8%8D%E6%94%AF%E6%8C%81%E5%9B%9E%E6%BB%9A)
            *   [Redis 对 ACID 的支持性理解？](#redis-%E5%AF%B9-acid%E7%9A%84%E6%94%AF%E6%8C%81%E6%80%A7%E7%90%86%E8%A7%A3)
            *   [Redis 事务其他实现？](#redis%E4%BA%8B%E5%8A%A1%E5%85%B6%E4%BB%96%E5%AE%9E%E7%8E%B0)
            *   [Redis 集群的主从复制模型是怎样的？](#redis%E9%9B%86%E7%BE%A4%E7%9A%84%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6%E6%A8%A1%E5%9E%8B%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84)
            *   [Redis 全量复制的三个阶段？](#redis-%E5%85%A8%E9%87%8F%E5%A4%8D%E5%88%B6%E7%9A%84%E4%B8%89%E4%B8%AA%E9%98%B6%E6%AE%B5)
            *   [Redis 为什么会设计增量复制？](#redis-%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BC%9A%E8%AE%BE%E8%AE%A1%E5%A2%9E%E9%87%8F%E5%A4%8D%E5%88%B6)
            *   [Redis 增量复制的流程？](#redis-%E5%A2%9E%E9%87%8F%E5%A4%8D%E5%88%B6%E7%9A%84%E6%B5%81%E7%A8%8B)
            *   [增量复制如果在网络断开期间，repl_backlog_size 环形缓冲区写满之后，从库是会丢失掉那部分被覆盖掉的数据，还是直接进行全量复制呢？](#%E5%A2%9E%E9%87%8F%E5%A4%8D%E5%88%B6%E5%A6%82%E6%9E%9C%E5%9C%A8%E7%BD%91%E7%BB%9C%E6%96%AD%E5%BC%80%E6%9C%9F%E9%97%B4repl_backlog_size%E7%8E%AF%E5%BD%A2%E7%BC%93%E5%86%B2%E5%8C%BA%E5%86%99%E6%BB%A1%E4%B9%8B%E5%90%8E%E4%BB%8E%E5%BA%93%E6%98%AF%E4%BC%9A%E4%B8%A2%E5%A4%B1%E6%8E%89%E9%82%A3%E9%83%A8%E5%88%86%E8%A2%AB%E8%A6%86%E7%9B%96%E6%8E%89%E7%9A%84%E6%95%B0%E6%8D%AE%E8%BF%98%E6%98%AF%E7%9B%B4%E6%8E%A5%E8%BF%9B%E8%A1%8C%E5%85%A8%E9%87%8F%E5%A4%8D%E5%88%B6%E5%91%A2)
            *   [Redis 为什么不持久化的主服务器自动重启非常危险呢?](#redis-%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%8D%E6%8C%81%E4%B9%85%E5%8C%96%E7%9A%84%E4%B8%BB%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%87%AA%E5%8A%A8%E9%87%8D%E5%90%AF%E9%9D%9E%E5%B8%B8%E5%8D%B1%E9%99%A9%E5%91%A2)
            *   [Redis 为什么主从全量复制使用 RDB 而不使用 AOF？](#redis-%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%BB%E4%BB%8E%E5%85%A8%E9%87%8F%E5%A4%8D%E5%88%B6%E4%BD%BF%E7%94%A8rdb%E8%80%8C%E4%B8%8D%E4%BD%BF%E7%94%A8aof)
            *   [Redis 为什么还有无磁盘复制模式？](#redis-%E4%B8%BA%E4%BB%80%E4%B9%88%E8%BF%98%E6%9C%89%E6%97%A0%E7%A3%81%E7%9B%98%E5%A4%8D%E5%88%B6%E6%A8%A1%E5%BC%8F)
            *   [Redis 为什么还会有从库的从库的设计？](#redis-%E4%B8%BA%E4%BB%80%E4%B9%88%E8%BF%98%E4%BC%9A%E6%9C%89%E4%BB%8E%E5%BA%93%E7%9A%84%E4%BB%8E%E5%BA%93%E7%9A%84%E8%AE%BE%E8%AE%A1)
            *   [Redis 哨兵机制？哨兵实现了什么功能呢?](#redis%E5%93%A8%E5%85%B5%E6%9C%BA%E5%88%B6%E5%93%A8%E5%85%B5%E5%AE%9E%E7%8E%B0%E4%BA%86%E4%BB%80%E4%B9%88%E5%8A%9F%E8%83%BD%E5%91%A2)
            *   [Redis 哨兵集群是通过什么方式组建的？](#redis-%E5%93%A8%E5%85%B5%E9%9B%86%E7%BE%A4%E6%98%AF%E9%80%9A%E8%BF%87%E4%BB%80%E4%B9%88%E6%96%B9%E5%BC%8F%E7%BB%84%E5%BB%BA%E7%9A%84)
            *   [Redis 哨兵是如何监控 Redis 集群的？](#redis-%E5%93%A8%E5%85%B5%E6%98%AF%E5%A6%82%E4%BD%95%E7%9B%91%E6%8E%A7redis%E9%9B%86%E7%BE%A4%E7%9A%84)
            *   [Redis 哨兵如何判断主库已经下线了呢？](#redis-%E5%93%A8%E5%85%B5%E5%A6%82%E4%BD%95%E5%88%A4%E6%96%AD%E4%B8%BB%E5%BA%93%E5%B7%B2%E7%BB%8F%E4%B8%8B%E7%BA%BF%E4%BA%86%E5%91%A2)
            *   [Redis 哨兵的选举机制是什么样的？](#redis-%E5%93%A8%E5%85%B5%E7%9A%84%E9%80%89%E4%B8%BE%E6%9C%BA%E5%88%B6%E6%98%AF%E4%BB%80%E4%B9%88%E6%A0%B7%E7%9A%84)
            *   [Redis 1 主 4 从，5 个哨兵，哨兵配置 quorum 为 2，如果 3 个哨兵故障，当主库宕机时，哨兵能否判断主库 “客观下线”？能否自动切换？](#redis-1%E4%B8%BB4%E4%BB%8E5%E4%B8%AA%E5%93%A8%E5%85%B5%E5%93%A8%E5%85%B5%E9%85%8D%E7%BD%AEquorum%E4%B8%BA2%E5%A6%82%E6%9E%9C3%E4%B8%AA%E5%93%A8%E5%85%B5%E6%95%85%E9%9A%9C%E5%BD%93%E4%B8%BB%E5%BA%93%E5%AE%95%E6%9C%BA%E6%97%B6%E5%93%A8%E5%85%B5%E8%83%BD%E5%90%A6%E5%88%A4%E6%96%AD%E4%B8%BB%E5%BA%93%E5%AE%A2%E8%A7%82%E4%B8%8B%E7%BA%BF%E8%83%BD%E5%90%A6%E8%87%AA%E5%8A%A8%E5%88%87%E6%8D%A2)
            *   [主库判定客观下线了，那么如何从剩余的从库中选择一个新的主库呢？](#%E4%B8%BB%E5%BA%93%E5%88%A4%E5%AE%9A%E5%AE%A2%E8%A7%82%E4%B8%8B%E7%BA%BF%E4%BA%86%E9%82%A3%E4%B9%88%E5%A6%82%E4%BD%95%E4%BB%8E%E5%89%A9%E4%BD%99%E7%9A%84%E4%BB%8E%E5%BA%93%E4%B8%AD%E9%80%89%E6%8B%A9%E4%B8%80%E4%B8%AA%E6%96%B0%E7%9A%84%E4%B8%BB%E5%BA%93%E5%91%A2)
            *   [新的主库选择出来后，如何进行故障的转移？](#%E6%96%B0%E7%9A%84%E4%B8%BB%E5%BA%93%E9%80%89%E6%8B%A9%E5%87%BA%E6%9D%A5%E5%90%8E%E5%A6%82%E4%BD%95%E8%BF%9B%E8%A1%8C%E6%95%85%E9%9A%9C%E7%9A%84%E8%BD%AC%E7%A7%BB)
            *   [Redis 事件机制?](#redis%E4%BA%8B%E4%BB%B6%E6%9C%BA%E5%88%B6)
            *   [Redis 文件事件的模型？](#redis%E6%96%87%E4%BB%B6%E4%BA%8B%E4%BB%B6%E7%9A%84%E6%A8%A1%E5%9E%8B)
            *   [什么是 Redis 发布订阅？](#%E4%BB%80%E4%B9%88%E6%98%AFredis%E5%8F%91%E5%B8%83%E8%AE%A2%E9%98%85)
            *   [Redis 发布订阅有哪两种方式？](#redis%E5%8F%91%E5%B8%83%E8%AE%A2%E9%98%85%E6%9C%89%E5%93%AA%E4%B8%A4%E7%A7%8D%E6%96%B9%E5%BC%8F)
            *   [什么是 Redis Cluster？](#%E4%BB%80%E4%B9%88%E6%98%AFredis-cluster)
            *   [说说 Redis 哈希槽的概念？为什么是 16384 个？](#%E8%AF%B4%E8%AF%B4redis%E5%93%88%E5%B8%8C%E6%A7%BD%E7%9A%84%E6%A6%82%E5%BF%B5%E4%B8%BA%E4%BB%80%E4%B9%88%E6%98%AF16384%E4%B8%AA)
            *   [Redis 集群会有写操作丢失吗？为什么？](#redis%E9%9B%86%E7%BE%A4%E4%BC%9A%E6%9C%89%E5%86%99%E6%93%8D%E4%BD%9C%E4%B8%A2%E5%A4%B1%E5%90%97%E4%B8%BA%E4%BB%80%E4%B9%88)
            *   [Redis 客户端有哪些？](#redis-%E5%AE%A2%E6%88%B7%E7%AB%AF%E6%9C%89%E5%93%AA%E4%BA%9B)
            *   [Redis 如何做大量数据插入？](#redis%E5%A6%82%E4%BD%95%E5%81%9A%E5%A4%A7%E9%87%8F%E6%95%B0%E6%8D%AE%E6%8F%92%E5%85%A5)
            *   [Redis 实现分布式锁实现? 什么是 RedLock?](#redis%E5%AE%9E%E7%8E%B0%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E5%AE%9E%E7%8E%B0-%E4%BB%80%E4%B9%88%E6%98%AF-redlock)
            *   [Redis 缓存有哪些问题，如何解决？](#redis%E7%BC%93%E5%AD%98%E6%9C%89%E5%93%AA%E4%BA%9B%E9%97%AE%E9%A2%98%E5%A6%82%E4%BD%95%E8%A7%A3%E5%86%B3)
            *   [Redis 性能问题有哪些，如何分析定位解决?](#redis%E6%80%A7%E8%83%BD%E9%97%AE%E9%A2%98%E6%9C%89%E5%93%AA%E4%BA%9B%E5%A6%82%E4%BD%95%E5%88%86%E6%9E%90%E5%AE%9A%E4%BD%8D%E8%A7%A3%E5%86%B3)
            *   [Redis 单线程模型？ 在 6.0 之前如何提高多核 CPU 的利用率？](#redis%E5%8D%95%E7%BA%BF%E7%A8%8B%E6%A8%A1%E5%9E%8B-%E5%9C%A860%E4%B9%8B%E5%89%8D%E5%A6%82%E4%BD%95%E6%8F%90%E9%AB%98%E5%A4%9A%E6%A0%B8cpu%E7%9A%84%E5%88%A9%E7%94%A8%E7%8E%87)
            *   [Redis6.0 之前的版本真的是单线程的吗?](#redis60%E4%B9%8B%E5%89%8D%E7%9A%84%E7%89%88%E6%9C%AC%E7%9C%9F%E7%9A%84%E6%98%AF%E5%8D%95%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%90%97)
            *   [Redis6.0 之前为什么一致不用多线程?](#redis60%E4%B9%8B%E5%89%8D%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%80%E8%87%B4%E4%B8%8D%E7%94%A8%E5%A4%9A%E7%BA%BF%E7%A8%8B)
            *   [Redis6.0 为什么要引入多线程呢？](#redis60%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E5%BC%95%E5%85%A5%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%91%A2)
            *   [Redis6.0 默认是否开启了多线程？](#redis60%E9%BB%98%E8%AE%A4%E6%98%AF%E5%90%A6%E5%BC%80%E5%90%AF%E4%BA%86%E5%A4%9A%E7%BA%BF%E7%A8%8B)
            *   [Redis6.0 多线程开启时，线程数如何设置？](#redis60%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%BC%80%E5%90%AF%E6%97%B6%E7%BA%BF%E7%A8%8B%E6%95%B0%E5%A6%82%E4%BD%95%E8%AE%BE%E7%BD%AE)
            *   [Redis6.0 多线程的实现机制？](#redis60%E5%A4%9A%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%AE%9E%E7%8E%B0%E6%9C%BA%E5%88%B6)
            *   [开启多线程后，是否会存在线程并发安全问题？](#%E5%BC%80%E5%90%AF%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%90%8E%E6%98%AF%E5%90%A6%E4%BC%9A%E5%AD%98%E5%9C%A8%E7%BA%BF%E7%A8%8B%E5%B9%B6%E5%8F%91%E5%AE%89%E5%85%A8%E9%97%AE%E9%A2%98)
        *   [8.4 MongoDB](#84-mongodb)
            *   [什么是 MongoDB？为什么使用 MongoDB？](#%E4%BB%80%E4%B9%88%E6%98%AFmongodb%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%BF%E7%94%A8mongodb)
            *   [MongoDB 与 RDBMS 区别？有哪些术语？](#mongodb%E4%B8%8Erdbms%E5%8C%BA%E5%88%AB%E6%9C%89%E5%93%AA%E4%BA%9B%E6%9C%AF%E8%AF%AD)
            *   [MongoDB 聚合的管道方式？](#mongodb%E8%81%9A%E5%90%88%E7%9A%84%E7%AE%A1%E9%81%93%E6%96%B9%E5%BC%8F)
            *   [MongoDB 聚合的 Map Reduce 方式？](#mongodb%E8%81%9A%E5%90%88%E7%9A%84map-reduce%E6%96%B9%E5%BC%8F)
            *   [Spring Data 和 MongoDB 集成？](#spring-data-%E5%92%8Cmongodb%E9%9B%86%E6%88%90)
            *   [MongoDB 有哪几种存储引擎？](#mongodb-%E6%9C%89%E5%93%AA%E5%87%A0%E7%A7%8D%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E)
            *   [谈谈你对 MongoDB WT 存储引擎的理解？](#%E8%B0%88%E8%B0%88%E4%BD%A0%E5%AF%B9mongodb-wt%E5%AD%98%E5%82%A8%E5%BC%95%E6%93%8E%E7%9A%84%E7%90%86%E8%A7%A3)
            *   [MongoDB 为什么要引入复制集？有哪些成员？](#mongodb%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E5%BC%95%E5%85%A5%E5%A4%8D%E5%88%B6%E9%9B%86%E6%9C%89%E5%93%AA%E4%BA%9B%E6%88%90%E5%91%98)
            *   [MongoDB 复制集常见部署架构？](#mongodb%E5%A4%8D%E5%88%B6%E9%9B%86%E5%B8%B8%E8%A7%81%E9%83%A8%E7%BD%B2%E6%9E%B6%E6%9E%84)
            *   [MongoDB 复制集是如何保证数据高可用的？](#mongodb%E5%A4%8D%E5%88%B6%E9%9B%86%E6%98%AF%E5%A6%82%E4%BD%95%E4%BF%9D%E8%AF%81%E6%95%B0%E6%8D%AE%E9%AB%98%E5%8F%AF%E7%94%A8%E7%9A%84)
            *   [MongoDB 复制集如何同步数据？](#mongodb%E5%A4%8D%E5%88%B6%E9%9B%86%E5%A6%82%E4%BD%95%E5%90%8C%E6%AD%A5%E6%95%B0%E6%8D%AE)
            *   [MongoDB 为什么要引入分片？](#mongodb%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E5%BC%95%E5%85%A5%E5%88%86%E7%89%87)
            *   [MongoDB 分片集群的结构？](#mongodb%E5%88%86%E7%89%87%E9%9B%86%E7%BE%A4%E7%9A%84%E7%BB%93%E6%9E%84)
            *   [MongoDB 分片的内部是如何管理数据的呢？](#mongodb%E5%88%86%E7%89%87%E7%9A%84%E5%86%85%E9%83%A8%E6%98%AF%E5%A6%82%E4%BD%95%E7%AE%A1%E7%90%86%E6%95%B0%E6%8D%AE%E7%9A%84%E5%91%A2)
            *   [MongoDB 中 Collection 的数据是根据什么进行分片的呢？](#mongodb-%E4%B8%ADcollection%E7%9A%84%E6%95%B0%E6%8D%AE%E6%98%AF%E6%A0%B9%E6%8D%AE%E4%BB%80%E4%B9%88%E8%BF%9B%E8%A1%8C%E5%88%86%E7%89%87%E7%9A%84%E5%91%A2)
            *   [MongoDB 如何做备份恢复？](#mongodb%E5%A6%82%E4%BD%95%E5%81%9A%E5%A4%87%E4%BB%BD%E6%81%A2%E5%A4%8D)
            *   [MongoDB 如何设计文档模型？](#mongodb%E5%A6%82%E4%BD%95%E8%AE%BE%E8%AE%A1%E6%96%87%E6%A1%A3%E6%A8%A1%E5%9E%8B)
            *   [MongoDB 如何进行性能优化？](#mongodb%E5%A6%82%E4%BD%95%E8%BF%9B%E8%A1%8C%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96)
        *   [8.5 ElasticSearch](#85-elasticsearch)
            *   [ElasticSearch 是什么？基于 Lucene 的，那么为什么不是直接使用 Lucene 呢？](#elasticsearch%E6%98%AF%E4%BB%80%E4%B9%88%E5%9F%BA%E4%BA%8Elucene%E7%9A%84%E9%82%A3%E4%B9%88%E4%B8%BA%E4%BB%80%E4%B9%88%E4%B8%8D%E6%98%AF%E7%9B%B4%E6%8E%A5%E4%BD%BF%E7%94%A8lucene%E5%91%A2)
            *   [ELK 技术栈的常见应用场景？](#elk-%E6%8A%80%E6%9C%AF%E6%A0%88%E7%9A%84%E5%B8%B8%E8%A7%81%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF)
            *   [ES 中索引模板是什么？](#es%E4%B8%AD%E7%B4%A2%E5%BC%95%E6%A8%A1%E6%9D%BF%E6%98%AF%E4%BB%80%E4%B9%88)
            *   [ES 中索引的生命周期管理？](#es%E4%B8%AD%E7%B4%A2%E5%BC%95%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E7%AE%A1%E7%90%86)
            *   [ES 查询和聚合的有哪些方式？](#es%E6%9F%A5%E8%AF%A2%E5%92%8C%E8%81%9A%E5%90%88%E7%9A%84%E6%9C%89%E5%93%AA%E4%BA%9B%E6%96%B9%E5%BC%8F)
            *   [ES 查询中 query 和 filter 的区别？](#es%E6%9F%A5%E8%AF%A2%E4%B8%ADquery%E5%92%8Cfilter%E7%9A%84%E5%8C%BA%E5%88%AB)
            *   [ES 查询中 match 和 term 的区别？](#es%E6%9F%A5%E8%AF%A2%E4%B8%ADmatch%E5%92%8Cterm%E7%9A%84%E5%8C%BA%E5%88%AB)
            *   [ES 查询中 should 和 must 的区别？](#es%E6%9F%A5%E8%AF%A2%E4%B8%ADshould%E5%92%8Cmust%E7%9A%84%E5%8C%BA%E5%88%AB)
            *   [ES 查询中 match，match_phrase 和 match_phrase_prefix 有什么区别？](#es%E6%9F%A5%E8%AF%A2%E4%B8%ADmatchmatch_phrase%E5%92%8Cmatch_phrase_prefix%E6%9C%89%E4%BB%80%E4%B9%88%E5%8C%BA%E5%88%AB)
            *   [ES 查询中什么是复合查询？有哪些复合查询方式？](#es%E6%9F%A5%E8%AF%A2%E4%B8%AD%E4%BB%80%E4%B9%88%E6%98%AF%E5%A4%8D%E5%90%88%E6%9F%A5%E8%AF%A2%E6%9C%89%E5%93%AA%E4%BA%9B%E5%A4%8D%E5%90%88%E6%9F%A5%E8%AF%A2%E6%96%B9%E5%BC%8F)
            *   [ES 聚合中的 Bucket 聚合有哪些？如何理解？](#es%E8%81%9A%E5%90%88%E4%B8%AD%E7%9A%84bucket%E8%81%9A%E5%90%88%E6%9C%89%E5%93%AA%E4%BA%9B%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3)
            *   [ES 聚合中的 Metric 聚合有哪些？如何理解？](#es%E8%81%9A%E5%90%88%E4%B8%AD%E7%9A%84metric%E8%81%9A%E5%90%88%E6%9C%89%E5%93%AA%E4%BA%9B%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3)
            *   [ES 聚合中的管道聚合有哪些？如何理解？](#es%E8%81%9A%E5%90%88%E4%B8%AD%E7%9A%84%E7%AE%A1%E9%81%93%E8%81%9A%E5%90%88%E6%9C%89%E5%93%AA%E4%BA%9B%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3)
            *   [如何理解 ES 的结构和底层实现？](#%E5%A6%82%E4%BD%95%E7%90%86%E8%A7%A3es%E7%9A%84%E7%BB%93%E6%9E%84%E5%92%8C%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0)
            *   [ES 内部读取文档是怎样的？如何实现的？](#es%E5%86%85%E9%83%A8%E8%AF%BB%E5%8F%96%E6%96%87%E6%A1%A3%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%9A%84)
            *   [ES 内部索引文档是怎样的？如何实现的？](#es%E5%86%85%E9%83%A8%E7%B4%A2%E5%BC%95%E6%96%87%E6%A1%A3%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84%E5%A6%82%E4%BD%95%E5%AE%9E%E7%8E%B0%E7%9A%84)
            *   [ES 底层数据持久化的过程？](#es%E5%BA%95%E5%B1%82%E6%95%B0%E6%8D%AE%E6%8C%81%E4%B9%85%E5%8C%96%E7%9A%84%E8%BF%87%E7%A8%8B)
            *   [ES 遇到什么性能问题，如何优化的？](#es%E9%81%87%E5%88%B0%E4%BB%80%E4%B9%88%E6%80%A7%E8%83%BD%E9%97%AE%E9%A2%98%E5%A6%82%E4%BD%95%E4%BC%98%E5%8C%96%E7%9A%84)

## 1. [#](#_1-java-基础) 1 Java 基础

Java 基础部分，包括语法基础，泛型，注解，异常，反射和其它（如 SPI 机制等）。

### 1.1. [#](#_1-1-语法基础) 1.1 语法基础

#### 1.1.1. [#](#面向对象特性) 面向对象特性？

*   **封装**

利用抽象数据类型将数据和基于数据的操作封装在一起，使其构成一个不可分割的独立实体。数据被保护在抽象数据类型的内部，尽可能地隐藏内部的细节，只保留一些对外接口使之与外部发生联系。用户无需知道对象内部的细节，但可以通过对象对外提供的接口来访问该对象。

优点:

*   减少耦合: 可以独立地开发、测试、优化、使用、理解和修改
*   减轻维护的负担: 可以更容易被程序员理解，并且在调试的时候可以不影响其他模块
*   有效地调节性能: 可以通过剖析确定哪些模块影响了系统的性能
*   提高软件的可重用性
*   降低了构建大型系统的风险: 即使整个系统不可用，但是这些独立的模块却有可能是可用的

以下 Person 类封装 name、gender、age 等属性，外界只能通过 get() 方法获取一个 Person 对象的 name 属性和 gender 属性，而无法获取 age 属性，但是 age 属性可以供 work() 方法使用。

注意到 gender 属性使用 int 数据类型进行存储，封装使得用户注意不到这种实现细节。并且在需要修改 gender 属性使用的数据类型时，也可以在不影响客户端代码的情况下进行。

```
public class Person {

    private String name;
    private int gender;
    private int age;

    public String getName() {
        return name;
    }

    public String getGender() {
        return gender == 0 ? "man" : "woman";
    }

    public void work() {
        if (18 <= age && age <= 50) {
            System.out.println(name + " is working very hard!");
        } else {
            System.out.println(name + " can't work any more!");
        }
    }
}
```

*   **继承**

继承实现了 **IS-A** 关系，例如 Cat 和 Animal 就是一种 IS-A 关系，因此 Cat 可以继承自 Animal，从而获得 Animal 非 private 的属性和方法。

继承应该遵循里氏替换原则，子类对象必须能够替换掉所有父类对象。

Cat 可以当做 Animal 来使用，也就是说可以使用 Animal 引用 Cat 对象。父类引用指向子类对象称为 **向上转型** 。

```
Animal animal = new Cat();
```

*   **多态**

多态分为编译时多态和运行时多态:

*   编译时多态主要指方法的重载
*   运行时多态指程序中定义的对象引用所指向的具体类型在运行期间才确定

运行时多态有三个条件:

*   继承
*   覆盖 (重写)
*   向上转型

下面的代码中，乐器类 (Instrument) 有两个子类: Wind 和 Percussion，它们都覆盖了父类的 play() 方法，并且在 main() 方法中使用父类 Instrument 来引用 Wind 和 Percussion 对象。在 Instrument 引用调用 play() 方法时，会执行实际引用对象所在类的 play() 方法，而不是 Instrument 类的方法。

```
public class Instrument {
    public void play() {
        System.out.println("Instrument is playing...");
    }
}

public class Wind extends Instrument {
    public void play() {
        System.out.println("Wind is playing...");
    }
}

public class Percussion extends Instrument {
    public void play() {
        System.out.println("Percussion is playing...");
    }
}

public class Music {
    public static void main(String[] args) {
        List<Instrument> instruments = new ArrayList<>();
        instruments.add(new Wind());
        instruments.add(new Percussion());
        for(Instrument instrument : instruments) {
            instrument.play();
        }
    }
}
```

#### 1.1.2. [#](#a-a-b-与-a-b-的区别) a = a + b 与 a += b 的区别

+= 隐式的将加操作的结果类型强制转换为持有结果的类型。如果两个整型相加，如 byte、short 或者 int，首先会将它们提升到 int 类型，然后在执行加法操作。

```
byte a = 127;
byte b = 127;
b = a + b; // error : cannot convert from int to byte
b += a; // ok
```

(因为 a+b 操作会将 a、b 提升为 int 类型，所以将 int 类型赋值给 byte 就会编译出错)

#### 1.1.3. [#](#_3-0-1-0-3-将会返回什么-true-还是-false) 3*0.1 == 0.3 将会返回什么? true 还是 false?

false，因为有些浮点数不能完全精确的表示出来。

#### 1.1.4. [#](#能在-switch-中使用-string-吗) 能在 Switch 中使用 String 吗?

从 Java 7 开始，我们可以在 switch case 中使用字符串，但这仅仅是一个语法糖。内部实现在 switch 中使用字符串的 hash code。

#### 1.1.5. [#](#对equals-和hashcode-的理解) 对 equals() 和 hashCode() 的理解?

*   **为什么在重写 equals 方法的时候需要重写 hashCode 方法**?

因为有强制的规范指定需要同时重写 hashcode 与 equals 是方法，许多容器类，如 HashMap、HashSet 都依赖于 hashcode 与 equals 的规定。

*   **有没有可能两个不相等的对象有相同的 hashcode**?

有可能，两个不相等的对象可能会有相同的 hashcode 值，这就是为什么在 hashmap 中会有冲突。相等 hashcode 值的规定只是说如果两个对象相等，必须有相同的 hashcode 值，但是没有关于不相等对象的任何规定。

*   **两个相同的对象会有不同的 hash code 吗**?

不能，根据 hash code 的规定，这是不可能的。

#### 1.1.6. [#](#final、finalize-和-finally-的不同之处) final、finalize 和 finally 的不同之处?

*   final 是一个修饰符，可以修饰变量、方法和类。如果 final 修饰变量，意味着该变量的值在初始化后不能被改变。
*   Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的，但是什么时候调用 finalize 没有保证。
*   finally 是一个关键字，与 try 和 catch 一起用于异常的处理。finally 块一定会被执行，无论在 try 块中是否有发生异常。

#### 1.1.7. [#](#string、stringbuffer与stringbuilder的区别) String、StringBuffer 与 StringBuilder 的区别？

第一点: 可变和适用范围。String 对象是不可变的，而 StringBuffer 和 StringBuilder 是可变字符序列。每次对 String 的操作相当于生成一个新的 String 对象，而对 StringBuffer 和 StringBuilder 的操作是对对象本身的操作，而不会生成新的对象，所以对于频繁改变内容的字符串避免使用 String，因为频繁的生成对象将会对系统性能产生影响。

第二点: 线程安全。String 由于有 final 修饰，是 immutable 的，安全性是简单而纯粹的。StringBuilder 和 StringBuffer 的区别在于 StringBuilder 不保证同步，也就是说如果需要线程安全需要使用 StringBuffer，不需要同步的 StringBuilder 效率更高。

#### 1.1.8. [#](#接口与抽象类的区别) 接口与抽象类的区别？

*   一个子类只能继承一个抽象类, 但能实现多个接口
*   抽象类可以有构造方法, 接口没有构造方法
*   抽象类可以有普通成员变量, 接口没有普通成员变量
*   抽象类和接口都可有静态成员变量, 抽象类中静态成员变量访问类型任意，接口只能 public static final(默认)
*   抽象类可以没有抽象方法, 抽象类可以有普通方法；接口在 JDK8 之前都是抽象方法，在 JDK8 可以有 default 方法，在 JDK9 中允许有私有普通方法
*   抽象类可以有静态方法；接口在 JDK8 之前不能有静态方法，在 JDK8 中可以有静态方法，且只能被接口类直接调用（不能被实现类的对象调用）
*   抽象类中的方法可以是 public、protected; 接口方法在 JDK8 之前只有 public abstract，在 JDK8 可以有 default 方法，在 JDK9 中允许有 private 方法

#### 1.1.9. [#](#this-super-在构造方法中的区别) this() & super() 在构造方法中的区别？

*   调用 super() 必须写在子类构造方法的第一行, 否则编译不通过
*   super 从子类调用父类构造, this 在同一类中调用其他构造均需要放在第一行
*   尽管可以用 this 调用一个构造器, 却不能调用 2 个
*   this 和 super 不能出现在同一个构造器中, 否则编译不通过
*   this()、super() 都指的对象, 不可以在 static 环境中使用
*   本质 this 指向本对象的指针。super 是一个关键字

#### 1.1.10. [#](#java移位运算符) Java 移位运算符？

java 中有三种移位运算符

*   `<<` : 左移运算符,`x << 1`, 相当于 x 乘以 2(不溢出的情况下), 低位补 0
*   `>>` : 带符号右移,`x >> 1`, 相当于 x 除以 2, 正数高位补 0, 负数高位补 1
*   `>>>` : 无符号右移, 忽略符号位, 空位都以 0 补齐

### 1.2. [#](#_1-2-泛型) 1.2 泛型

#### 1.2.1. [#](#为什么需要泛型) 为什么需要泛型？

1.  **适用于多种数据类型执行相同的代码**

```
private static int add(int a, int b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}

private static float add(float a, float b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}

private static double add(double a, double b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}
```

如果没有泛型，要实现不同类型的加法，每种类型都需要重载一个 add 方法；通过泛型，我们可以复用为一个方法：

```
private static <T extends Number> double add(T a, T b) {
    System.out.println(a + "+" + b + "=" + (a.doubleValue() + b.doubleValue()));
    return a.doubleValue() + b.doubleValue();
}
```

*   **泛型中的类型在使用时指定，不需要强制类型转换**（**类型安全**，编译器会**检查类型**）

看下这个例子：

```
List list = new ArrayList();
list.add("xxString");
list.add(100d);
list.add(new Person());
```

我们在使用上述 list 中，list 中的元素都是 Object 类型（无法约束其中的类型），所以在取出集合元素时需要人为的强制类型转化到具体的目标类型，且很容易出现`java.lang.ClassCastException`异常。

引入泛型，它将提供类型的约束，提供编译前的检查：

```
List<String> list = new ArrayList<String>();

// list中只能放String, 不能放其它类型的元素
```

#### 1.2.2. [#](#泛型类如何定义使用) 泛型类如何定义使用？

*   从一个简单的泛型类看起：

```
class Point<T>{         // 此处可以随便写标识符号，T是type的简称 
    private T var ;     // var的类型由T指定，即：由外部指定 
    public T getVar(){  // 返回值的类型由外部决定 
        return var ;  
    }  
    public void setVar(T var){  // 设置的类型也由外部决定 
        this.var = var ;  
    }  
}  
public class GenericsDemo06{  
    public static void main(String args[]){  
        Point<String> p = new Point<String>() ;     // 里面的var类型为String类型 
        p.setVar("it") ;                            // 设置字符串 
        System.out.println(p.getVar().length()) ;   // 取得字符串的长度 
    }  
}
```

*   多元泛型

```
class Notepad<K,V>{       // 此处指定了两个泛型类型 
    private K key ;     // 此变量的类型由外部决定 
    private V value ;   // 此变量的类型由外部决定 
    public K getKey(){  
        return this.key ;  
    }  
    public V getValue(){  
        return this.value ;  
    }  
    public void setKey(K key){  
        this.key = key ;  
    }  
    public void setValue(V value){  
        this.value = value ;  
    }  
} 
public class GenericsDemo09{  
    public static void main(String args[]){  
        Notepad<String,Integer> t = null ;        // 定义两个泛型类型的对象 
        t = new Notepad<String,Integer>() ;       // 里面的key为String，value为Integer 
        t.setKey("汤姆") ;        // 设置第一个内容 
        t.setValue(20) ;            // 设置第二个内容 
        System.out.print("姓名；" + t.getKey()) ;      // 取得信息 
        System.out.print("，年龄；" + t.getValue()) ;       // 取得信息 
  
    }  
}
```

#### 1.2.3. [#](#泛型接口如何定义使用) 泛型接口如何定义使用？

*   简单的泛型接口

```
interface Info<T>{        // 在接口上定义泛型 
    public T getVar() ; // 定义抽象方法，抽象方法的返回值就是泛型类型 
}  
class InfoImpl<T> implements Info<T>{   // 定义泛型接口的子类 
    private T var ;             // 定义属性 
    public InfoImpl(T var){     // 通过构造方法设置属性内容 
        this.setVar(var) ;    
    }  
    public void setVar(T var){  
        this.var = var ;  
    }  
    public T getVar(){  
        return this.var ;  
    }  
} 
public class GenericsDemo24{  
    public static void main(String arsg[]){  
        Info<String> i = null;        // 声明接口对象 
        i = new InfoImpl<String>("汤姆") ;  // 通过子类实例化对象 
        System.out.println("内容：" + i.getVar()) ;  
    }  
}
```

#### 1.2.4. [#](#泛型方法如何定义使用) 泛型方法如何定义使用？

泛型方法，是在调用方法的时候指明泛型的具体类型。

*   定义泛型方法语法格式

![](https://pdai.tech/images/java/java-basic-generic-4.png)

*   调用泛型方法语法格式

![](https://pdai.tech/images/java/java-basic-generic-5.png)

说明一下，定义泛型方法时，必须在返回值前边加一个`<T>`，来声明这是一个泛型方法，持有一个泛型`T`，然后才可以用泛型 T 作为方法的返回值。

`Class<T>`的作用就是指明泛型的具体类型，而`Class<T>`类型的变量 c，可以用来创建泛型类的对象。

为什么要用变量 c 来创建对象呢？既然是泛型方法，就代表着我们不知道具体的类型是什么，也不知道构造方法如何，因此没有办法去 new 一个对象，但可以利用变量 c 的 newInstance 方法去创建对象，也就是利用反射创建对象。

泛型方法要求的参数是`Class<T>`类型，而`Class.forName()`方法的返回值也是`Class<T>`，因此可以用`Class.forName()`作为参数。其中，`forName()`方法中的参数是何种类型，返回的`Class<T>`就是何种类型。在本例中，`forName()`方法中传入的是 User 类的完整路径，因此返回的是`Class<User>`类型的对象，因此调用泛型方法时，变量 c 的类型就是`Class<User>`，因此泛型方法中的泛型 T 就被指明为 User，因此变量 obj 的类型为 User。

当然，泛型方法不是仅仅可以有一个参数`Class<T>`，可以根据需要添加其他参数。

**为什么要使用泛型方法呢**？因为泛型类要在实例化的时候就指明类型，如果想换一种类型，不得不重新 new 一次，可能不够灵活；而泛型方法可以在调用的时候指明类型，更加灵活。

#### 1.2.5. [#](#泛型的上限和下限) 泛型的上限和下限？

在使用泛型的时候，我们可以为传入的泛型类型实参进行上下边界的限制，如：类型实参只准传入某种类型的父类或某种类型的子类。

上限

```
class Info<T extends Number>{    // 此处泛型只能是数字类型
    private T var ;        // 定义泛型变量
    public void setVar(T var){
        this.var = var ;
    }
    public T getVar(){
        return this.var ;
    }
    public String toString(){    // 直接打印
        return this.var.toString() ;
    }
}
public class demo1{
    public static void main(String args[]){
        Info<Integer> i1 = new Info<Integer>() ;        // 声明Integer的泛型对象
    }
}
```

下限

```
class Info<T>{
    private T var ;        // 定义泛型变量
    public void setVar(T var){
        this.var = var ;
    }
    public T getVar(){
        return this.var ;
    }
    public String toString(){    // 直接打印
        return this.var.toString() ;
    }
}
public class GenericsDemo21{
    public static void main(String args[]){
        Info<String> i1 = new Info<String>() ;        // 声明String的泛型对象
        Info<Object> i2 = new Info<Object>() ;        // 声明Object的泛型对象
        i1.setVar("hello") ;
        i2.setVar(new Object()) ;
        fun(i1) ;
        fun(i2) ;
    }
    public static void fun(Info<? super String> temp){    // 只能接收String或Object类型的泛型，String类的父类只有Object类
        System.out.print(temp + ", ") ;
    }
}
```

#### 1.2.6. [#](#如何理解java中的泛型是伪泛型) 如何理解 Java 中的泛型是伪泛型？

泛型中类型擦除 Java 泛型这个特性是从 JDK 1.5 才开始加入的，因此为了兼容之前的版本，Java 泛型的实现采取了 “伪泛型” 的策略，即 Java 在语法上支持泛型，但是在编译阶段会进行所谓的“类型擦除”（Type Erasure），将所有的泛型表示（尖括号中的内容）都替换为具体的类型（其对应的原生态类型），就像完全没有泛型一样。

### 1.3. [#](#_1-3-注解) 1.3 注解

#### 1.3.1. [#](#注解的作用) 注解的作用？

注解是 JDK1.5 版本开始引入的一个特性，用于对代码进行说明，可以对包、类、接口、字段、方法参数、局部变量等进行注解。它主要的作用有以下四方面：

*   生成文档，通过代码里标识的元数据生成 javadoc 文档。
*   编译检查，通过代码里标识的元数据让编译器在编译期间进行检查验证。
*   编译时动态处理，编译时通过代码里标识的元数据动态处理，例如动态生成代码。
*   运行时动态处理，运行时通过代码里标识的元数据动态处理，例如使用反射注入实例。

#### 1.3.2. [#](#注解的常见分类) 注解的常见分类？

*   **Java 自带的标准注解**，包括`@Override`、`@Deprecated`和`@SuppressWarnings`，分别用于标明重写某个方法、标明某个类或方法过时、标明要忽略的警告，用这些注解标明后编译器就会进行检查。
*   **元注解**，元注解是用于定义注解的注解，包括`@Retention`、`@Target`、`@Inherited`、`@Documented`
    *   `@Retention`用于标明注解被保留的阶段
    *   `@Target`用于标明注解使用的范围
    *   `@Inherited`用于标明注解可继承
    *   `@Documented`用于标明是否生成 javadoc 文档
*   **自定义注解**，可以根据自己的需求定义注解，并可用元注解对自定义注解进行注解。

### 1.4. [#](#_1-4-异常) 1.4 异常

#### 1.4.1. [#](#java异常类层次结构) Java 异常类层次结构?

*   **Throwable** 是 Java 语言中所有错误与异常的超类。
    *   **Error** 类及其子类：程序中无法处理的错误，表示运行应用程序中出现了严重的错误。
    *   **Exception** 程序本身可以捕获并且可以处理的异常。Exception 这种异常又分为两类：运行时异常和编译时异常。

![](https://pdai.tech/images/java/java-basic-exception-1.png)

*   **运行时异常**

都是 RuntimeException 类及其子类异常，如 NullPointerException(空指针异常)、IndexOutOfBoundsException(下标越界异常) 等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。

运行时异常的特点是 Java 编译器不会检查它，也就是说，当程序中可能出现这类异常，即使没有用 try-catch 语句捕获它，也没有用 throws 子句声明抛出它，也会编译通过。

*   **非运行时异常** （编译异常）

是 RuntimeException 以外的异常，类型上都属于 Exception 类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如 IOException、SQLException 等以及用户自定义的 Exception 异常，一般情况下不自定义检查异常。

#### 1.4.2. [#](#可查的异常-checked-exceptions-和不可查的异常-unchecked-exceptions-区别) 可查的异常（checked exceptions）和不可查的异常（unchecked exceptions）区别？

*   **可查异常**（编译器要求必须处置的异常）：

正确的程序在运行中，很容易出现的、情理可容的异常状况。可查异常虽然是异常状况，但在一定程度上它的发生是可以预计的，而且一旦发生这种异常状况，就必须采取某种方式进行处理。

除了 RuntimeException 及其子类以外，其他的 Exception 类及其子类都属于可查异常。这种异常的特点是 Java 编译器会检查它，也就是说，当程序中可能出现这类异常，要么用 try-catch 语句捕获它，要么用 throws 子句声明抛出它，否则编译不会通过。

*   **不可查异常** (编译器不要求强制处置的异常)

包括运行时异常（RuntimeException 与其子类）和错误（Error）。

#### 1.4.3. [#](#throw和throws的区别) throw 和 throws 的区别？

*   **异常的申明 (throws)**

在 Java 中，当前执行的语句必属于某个方法，Java 解释器调用 main 方法执行开始执行程序。若方法中存在检查异常，如果不对其捕获，那必须在方法头中显式声明该异常，以便于告知方法调用者此方法有异常，需要进行处理。 在方法中声明一个异常，方法头中使用关键字 throws，后面接上要声明的异常。若声明多个异常，则使用逗号分割。如下所示：

```
public static void method() throws IOException, FileNotFoundException{
    //something statements
}
```

*   **异常的抛出 (throw)**

如果代码可能会引发某种错误，可以创建一个合适的异常类实例并抛出它，这就是抛出异常。如下所示：

```
public static double method(int value) {
    if(value == 0) {
        throw new ArithmeticException("参数不能为0"); //抛出一个运行时异常
    }
    return 5.0 / value;
}
```

#### 1.4.4. [#](#java-7-的-try-with-resource) Java 7 的 try-with-resource?

如果你的资源实现了 AutoCloseable 接口，你可以使用这个语法。大多数的 Java 标准资源都继承了这个接口。当你在 try 子句中打开资源，资源会在 try 代码块执行后或异常处理后自动关闭。

```
public void automaticallyCloseResource() {
    File file = new File("./tmp.txt");
    try (FileInputStream inputStream = new FileInputStream(file);) {
        // use the inputStream to read a file
    } catch (FileNotFoundException e) {
        log.error(e);
    } catch (IOException e) {
        log.error(e);
    }
}
```

#### 1.4.5. [#](#异常的底层) 异常的底层？

提到 JVM 处理异常的机制，就需要提及 Exception Table，以下称为异常表。我们暂且不急于介绍异常表，先看一个简单的 Java 处理异常的小例子。

```
public static void simpleTryCatch() {
   try {
       testNPE();
   } catch (Exception e) {
       e.printStackTrace();
   }
}
```

使用 javap 来分析这段代码（需要先使用 javac 编译）

```
//javap -c Main
 public static void simpleTryCatch();
    Code:
       0: invokestatic  #3                  // Method testNPE:()V
       3: goto          11
       6: astore_0
       7: aload_0
       8: invokevirtual #5                  // Method java/lang/Exception.printStackTrace:()V
      11: return
    Exception table:
       from    to  target type
           0     3     6   Class java/lang/Exception
```

看到上面的代码，应该会有会心一笑，因为终于看到了 Exception table，也就是我们要研究的异常表。

异常表中包含了一个或多个异常处理者 (Exception Handler) 的信息，这些信息包含如下

*   **from** 可能发生异常的起始点
*   **to** 可能发生异常的结束点
*   **target** 上述 from 和 to 之前发生异常后的异常处理者的位置
*   **type** 异常处理者处理的异常的类信息

### 1.5. [#](#_1-5-反射) 1.5 反射

#### 1.5.1. [#](#什么是反射) 什么是反射？

JAVA 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为 java 语言的反射机制。

![](https://pdai.tech/images/java/java-basic-reflection-3.png)

#### 1.5.2. [#](#反射的使用) 反射的使用？

在 Java 中，Class 类与 java.lang.reflect 类库一起对反射技术进行了全力的支持。在反射包中，我们常用的类主要有 Constructor 类表示的是 Class 对象所表示的类的构造方法，利用它可以在运行时动态创建对象、Field 表示 Class 对象所表示的类的成员变量，通过它可以在运行时动态修改成员变量的属性值 (包含 private)、Method 表示 Class 对象所表示的类的成员方法，通过它可以动态调用对象的方法 (包含 private)

*   Class 类对象的获取

```
@Test
    public void classTest() throws Exception {
        // 获取Class对象的三种方式
        logger.info("根据类名:  \t" + User.class);
        logger.info("根据对象:  \t" + new User().getClass());
        logger.info("根据全限定类名:\t" + Class.forName("com.test.User"));
        // 常用的方法
        logger.info("获取全限定类名:\t" + userClass.getName());
        logger.info("获取类名:\t" + userClass.getSimpleName());
        logger.info("实例化:\t" + userClass.newInstance());
    }
```

*   Constructor 类及其用法
*   Field 类及其用法
*   Method 类及其用法

#### 1.5.3. [#](#getname、getcanonicalname与getsimplename的区别) getName、getCanonicalName 与 getSimpleName 的区别?

*   getSimpleName：只获取类名
*   getName：类的全限定名，jvm 中 Class 的表示，可以用于动态加载 Class 对象，例如 Class.forName。
*   getCanonicalName：返回更容易理解的表示，主要用于输出（toString）或 log 打印，大多数情况下和 getName 一样，但是在内部类、数组等类型的表示形式就不同了。

### 1.6. [#](#_1-6-spi机制) 1.6 SPI 机制

#### 1.6.1. [#](#什么是spi机制) 什么是 SPI 机制？

SPI（Service Provider Interface），是 JDK 内置的一种 服务提供发现机制，可以用来启用框架扩展和替换组件，主要是被框架的开发人员使用，比如 java.sql.Driver 接口，其他不同厂商可以针对同一接口做出不同的实现，MySQL 和 PostgreSQL 都有不同的实现提供给用户，而 Java 的 SPI 机制可以为某个接口寻找服务实现。Java 中 SPI 机制主要思想是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要，其核心思想就是 **解耦**。

SPI 整体机制图如下：

![](https://pdai.tech/images/java/java-advanced-spi-8.jpg)

当服务的提供者提供了一种接口的实现之后，需要在 classpath 下的`META-INF/services/`目录里创建一个以服务接口命名的文件，这个文件里的内容就是这个接口的具体的实现类。当其他的程序需要这个服务的时候，就可以通过查找这个 jar 包（一般都是以 jar 包做依赖）的`META-INF/services/`中的配置文件，配置文件中有接口的具体实现类名，可以根据这个类名进行加载实例化，就可以使用该服务了。JDK 中查找服务的实现的工具类是：`java.util.ServiceLoader`。

#### 1.6.2. [#](#spi机制的应用) SPI 机制的应用？

*   SPI 机制 - JDBC DriverManager

在 JDBC4.0 之前，我们开发有连接数据库的时候，通常会用 Class.forName("com.mysql.jdbc.Driver") 这句先加载数据库相关的驱动，然后再进行获取连接等的操作。**而 JDBC4.0 之后不需要用 Class.forName("com.mysql.jdbc.Driver") 来加载驱动，直接获取连接就可以了，现在这种方式就是使用了 Java 的 SPI 扩展机制来实现**。

*   JDBC 接口定义

首先在 java 中定义了接口`java.sql.Driver`，并没有具体的实现，具体的实现都是由不同厂商来提供的。

*   mysql 实现

在 mysql 的 jar 包`mysql-connector-java-6.0.6.jar`中，可以找到`META-INF/services`目录，该目录下会有一个名字为`java.sql.Driver`的文件，文件内容是`com.mysql.cj.jdbc.Driver`，这里面的内容就是针对 Java 中定义的接口的实现。

*   postgresql 实现

同样在 postgresql 的 jar 包`postgresql-42.0.0.jar`中，也可以找到同样的配置文件，文件内容是`org.postgresql.Driver`，这是 postgresql 对 Java 的`java.sql.Driver`的实现。

*   使用方法

上面说了，现在使用 SPI 扩展来加载具体的驱动，我们在 Java 中写连接数据库的代码的时候，不需要再使用`Class.forName("com.mysql.jdbc.Driver")`来加载驱动了，而是直接使用如下代码：

```
String url = "jdbc:xxxx://xxxx:xxxx/xxxx";
Connection conn = DriverManager.getConnection(url,username,password);
.....
```

#### 1.6.3. [#](#spi机制的简单示例) SPI 机制的简单示例？

我们现在需要使用一个内容搜索接口，搜索的实现可能是基于文件系统的搜索，也可能是基于数据库的搜索。

*   先定义好接口

```
public interface Search {
    public List<String> searchDoc(String keyword);   
}
```

*   文件搜索实现

```
public class FileSearch implements Search{
    @Override
    public List<String> searchDoc(String keyword) {
        System.out.println("文件搜索 "+keyword);
        return null;
    }
}
```

*   数据库搜索实现

```
public class DatabaseSearch implements Search{
    @Override
    public List<String> searchDoc(String keyword) {
        System.out.println("数据搜索 "+keyword);
        return null;
    }
}
```

*   resources 接下来可以在 resources 下新建 META-INF/services / 目录，然后新建接口全限定名的文件：`com.cainiao.ys.spi.learn.Search`，里面加上我们需要用到的实现类

```
com.cainiao.ys.spi.learn.FileSearch
```

*   测试方法

```
public class TestCase {
    public static void main(String[] args) {
        ServiceLoader<Search> s = ServiceLoader.load(Search.class);
        Iterator<Search> iterator = s.iterator();
        while (iterator.hasNext()) {
           Search search =  iterator.next();
           search.searchDoc("hello world");
        }
    }
}
```

可以看到输出结果：文件搜索 hello world

如果在`com.cainiao.ys.spi.learn.Search`文件里写上两个实现类，那最后的输出结果就是两行了。

这就是因为`ServiceLoader.load(Search.class)`在加载某接口时，会去`META-INF/services`下找接口的全限定名文件，再根据里面的内容加载相应的实现类。

这就是 spi 的思想，接口的实现由 provider 实现，provider 只用在提交的 jar 包里的`META-INF/services`下根据平台定义的接口新建文件，并添加进相应的实现类内容就好。

## 2. [#](#_2-java-集合) 2 Java 集合

容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对 (两个对象) 的映射表。

### 2.1. [#](#_2-1-collection) 2.1 Collection

#### 2.1.1. [#](#集合有哪些类) 集合有哪些类？

*   Set
    *   TreeSet 基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。
    *   HashSet 基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。
    *   LinkedHashSet 具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序。
*   List
    *   ArrayList 基于动态数组实现，支持随机访问。
    *   Vector 和 ArrayList 类似，但它是线程安全的。
    *   LinkedList 基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。
*   Queue
    *   LinkedList 可以用它来实现双向队列。
    *   PriorityQueue 基于堆结构实现，可以用它来实现优先队列。

#### 2.1.2. [#](#arraylist的底层) ArrayList 的底层？

_ArrayList_ 实现了 _List_ 接口，是顺序容器，即元素存放的数据与放进去的顺序相同，允许放入`null`元素，底层通过**数组实现**。除该类未实现同步外，其余跟 _Vector_ 大致相同。每个 _ArrayList_ 都有一个容量 (capacity)，表示底层数组的实际大小，容器内存储元素的个数不能多于当前容量。当向容器中添加元素时，如果容量不足，容器会自动增大底层数组的大小。前面已经提过，Java 泛型只是编译器提供的语法糖，所以这里的数组是一个 Object 数组，以便能够容纳任何类型的对象。

![](https://pdai.tech/images/collection/ArrayList_base.png)

#### 2.1.3. [#](#arraylist自动扩容) ArrayList 自动扩容？

每当向数组中添加元素时，都要去检查添加后元素的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。数组扩容通过 ensureCapacity(int minCapacity) 方法来实现。在实际添加大量元素前，我也可以使用 ensureCapacity 来手动增加 ArrayList 实例的容量，以减少递增式再分配的数量。

数组进行扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量的增长大约是其原容量的 1.5 倍。这种操作的代价是很高的，因此在实际使用时，我们应该尽量避免数组容量的扩张。当我们可预知要保存的元素的多少时，要在构造 ArrayList 实例时，就指定其容量，以避免数组扩容的发生。或者根据实际需求，通过调用 ensureCapacity 方法来手动增加 ArrayList 实例的容量。

![](https://pdai.tech/images/collection/ArrayList_add.png)

#### 2.1.4. [#](#arraylist的fail-fast机制) ArrayList 的 Fail-Fast 机制？

ArrayList 也采用了快速失败的机制，通过记录 modCount 参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

### 2.2. [#](#_2-2-map) 2.2 Map

#### 2.2.1. [#](#map有哪些类) Map 有哪些类？

*   `TreeMap` 基于红黑树实现。
*   `HashMap` 1.7 基于哈希表实现，1.8 基于数组 + 链表 + 红黑树。
*   `HashTable` 和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高 (1.7 ConcurrentHashMap 引入了分段锁, 1.8 引入了红黑树)。
*   `LinkedHashMap` 使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用 (LRU) 顺序。

#### 2.2.2. [#](#jdk7-hashmap如何实现) JDK7 HashMap 如何实现？

哈希表有两种实现方式，一种开放地址方式 (Open addressing)，另一种是冲突链表方式 (Separate chaining with linked lists)。**Java7 _HashMap_ 采用的是冲突链表方式**。

![](https://pdai.tech/images/collection/HashMap_base.png)

从上图容易看出，如果选择合适的哈希函数，`put()`和`get()`方法可以在常数时间内完成。但在对 _HashMap_ 进行迭代时，需要遍历整个 table 以及后面跟的冲突链表。因此对于迭代比较频繁的场景，不宜将 _HashMap_ 的初始大小设的过大。

有两个参数可以影响 _HashMap_ 的性能: 初始容量 (inital capacity) 和负载系数(load factor)。初始容量指定了初始`table`的大小，负载系数用来指定自动扩容的临界值。当`entry`的数量超过`capacity*load_factor`时，容器将自动扩容并重新哈希。对于插入元素较多的场景，将初始容量设大可以减少重新哈希的次数。

#### 2.2.3. [#](#jdk8-hashmap如何实现) JDK8 HashMap 如何实现？

根据 Java7 HashMap 的介绍，我们知道，查找的时候，根据 hash 值我们能够快速定位到数组的具体下标，但是之后的话，需要顺着链表一个个比较下去才能找到我们需要的，时间复杂度取决于链表的长度，为 O(n)。

为了降低这部分的开销，在 Java8 中，当链表中的元素达到了 8 个时，会将链表转换为红黑树，在这些位置进行查找的时候可以降低时间复杂度为 O(logN)。

![](https://pdai.tech/images/java/java-collection-hashmap8.png)

#### 2.2.4. [#](#hashset是如何实现的) HashSet 是如何实现的？

_HashSet_ 是对 _HashMap_ 的简单包装，对 _HashSet_ 的函数调用都会转换成合适的 _HashMap_ 方法

```
//HashSet是对HashMap的简单包装
public class HashSet<E>
{
	......
	private transient HashMap<E,Object> map;//HashSet里面有一个HashMap
    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
    public HashSet() {
        map = new HashMap<>();
    }
    ......
    public boolean add(E e) {//简单的方法转换
        return map.put(e, PRESENT)==null;
    }
    ......
}
```

#### 2.2.5. [#](#什么是weakhashmap) 什么是 WeakHashMap?

我们都知道 Java 中内存是通过 GC 自动管理的，GC 会在程序运行过程中自动判断哪些对象是可以被回收的，并在合适的时机进行内存释放。GC 判断某个对象是否可被回收的依据是，**是否有有效的引用指向该对象**。如果没有有效引用指向该对象 (基本意味着不存在访问该对象的方式)，那么该对象就是可回收的。这里的**有效引用** 并不包括**弱引用**。也就是说，**虽然弱引用可以用来访问对象，但进行垃圾回收时弱引用并不会被考虑在内，仅有弱引用指向的对象仍然会被 GC 回收**。

WeakHashMap 内部是通过弱引用来管理 entry 的，弱引用的特性对应到 WeakHashMap 上意味着什么呢？

_WeakHashMap_ 里的`entry`可能会被 GC 自动删除，即使程序员没有调用`remove()`或者`clear()`方法。

**_WeakHashMap_ 的这个特点特别适用于需要缓存的场景**。在缓存场景下，由于内存是有限的，不能缓存所有对象；对象缓存命中可以提高系统效率，但缓存 MISS 也不会造成错误，因为可以通过计算重新得到。

## 3. [#](#_3-java-并发) 3 Java 并发

并发和多线程

### 3.1. [#](#_3-1-并发基础) 3.1 并发基础

*   [Java 并发 - 理论基础](https://pdai.tech/md/java/thread/java-thread-x-theorty.html)
*   [Java 并发 - 线程基础](https://pdai.tech/md/java/thread/java-thread-x-thread-basic.html)

#### 3.1.1. [#](#多线程的出现是要解决什么问题的-本质什么) 多线程的出现是要解决什么问题的? 本质什么?

CPU、内存、I/O 设备的速度是有极大差异的，为了合理利用 CPU 的高性能，平衡这三者的速度差异，计算机体系结构、操作系统、编译程序都做出了贡献，主要体现为:

*   CPU 增加了缓存，以均衡与内存的速度差异；// 导致可见性问题
*   操作系统增加了进程、线程，以分时复用 CPU，进而均衡 CPU 与 I/O 设备的速度差异；// 导致原子性问题
*   编译程序优化指令执行次序，使得缓存能够得到更加合理地利用。// 导致有序性问题

#### 3.1.2. [#](#java是怎么解决并发问题的) Java 是怎么解决并发问题的?

Java 内存模型是个很复杂的规范，具体看 [Java 内存模型详解](https://pdai.tech/md/java/jvm/java-jvm-jmm.html)。

**理解的第一个维度：核心知识点**

JMM 本质上可以理解为，Java 内存模型规范了 JVM 如何提供按需禁用缓存和编译优化的方法。具体来说，这些方法包括：

*   volatile、synchronized 和 final 三个关键字
*   Happens-Before 规则

**理解的第二个维度：可见性，有序性，原子性**

*   **原子性**

在 Java 中，对基本数据类型的变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行。 请分析以下哪些操作是原子性操作：

```
x = 10;        //语句1: 直接将数值10赋值给x，也就是说线程执行这个语句的会直接将数值10写入到工作内存中
y = x;         //语句2: 包含2个操作，它先要去读取x的值，再将x的值写入工作内存，虽然读取x的值以及 将x的值写入工作内存 这2个操作都是原子性操作，但是合起来就不是原子性操作了。
x++;           //语句3： x++包括3个操作：读取x的值，进行加1操作，写入新的值。
x = x + 1;     //语句4： 同语句3
```

上面 4 个语句只有语句 1 的操作具备原子性。

也就是说，只有简单的读取、赋值（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）才是原子操作。

从上面可以看出，Java 内存模型只保证了基本读取和赋值是原子性操作，如果要实现更大范围操作的原子性，可以通过 synchronized 和 Lock 来实现。由于 synchronized 和 Lock 能够保证任一时刻只有一个线程执行该代码块，那么自然就不存在原子性问题了，从而保证了原子性。

*   **可见性**

Java 提供了 volatile 关键字来保证可见性。

当一个共享变量被 volatile 修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。

而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。

另外，通过 synchronized 和 Lock 也能够保证可见性，synchronized 和 Lock 能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性。

*   **有序性**

在 Java 里面，可以通过 volatile 关键字来保证一定的 “有序性”。另外可以通过 synchronized 和 Lock 来保证有序性，很显然，synchronized 和 Lock 保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。当然 JMM 是通过 Happens-Before 规则来保证有序性的。

#### 3.1.3. [#](#线程安全有哪些实现思路) 线程安全有哪些实现思路?

1.  **互斥同步**

synchronized 和 ReentrantLock。

2.  **非阻塞同步**

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁 (这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁)、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

*   CAS

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略: 先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施 (不断地重试，直到成功为止)。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是: 比较并交换 (Compare-and-Swap，CAS)。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

*   AtomicInteger

J.U.C 包里面的整数原子类 AtomicInteger，其中的 compareAndSet() 和 getAndIncrement() 等方法都使用了 Unsafe 类的 CAS 操作。

3.  **无同步方案**

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

*   栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

*   线程本地存储 (Thread Local Storage)

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

#### 3.1.4. [#](#如何理解并发和并行的区别) 如何理解并发和并行的区别?

并发是指一个处理器同时处理多个任务。

![](https://pdai.tech/images/java/java-bingfa-2.png)

并行是指多个处理器或者是多核的处理器同时处理多个不同的任务。

![](https://pdai.tech/images/java/java-bingfa-1.png)

#### 3.1.5. [#](#线程有哪几种状态-分别说明从一种状态到另一种状态转变有哪些方式) 线程有哪几种状态? 分别说明从一种状态到另一种状态转变有哪些方式?

![](https://pdai.tech/images/pics/ace830df-9919-48ca-91b5-60b193f593d2.png)

*   新建 (New)

创建后尚未启动。

*   可运行 (Runnable)

可能正在运行，也可能正在等待 CPU 时间片。

包含了操作系统线程状态中的 Running 和 Ready。

*   阻塞 (Blocking)

等待获取一个排它锁，如果其线程释放了锁就会结束此状态。

*   无限期等待 (Waiting)

等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。

<table><thead><tr><th>进入方法</th><th>退出方法</th></tr></thead><tbody><tr><td>没有设置 Timeout 参数的 Object.wait() 方法</td><td>Object.notify() / Object.notifyAll()</td></tr><tr><td>没有设置 Timeout 参数的 Thread.join() 方法</td><td>被调用的线程执行完毕</td></tr><tr><td>LockSupport.park() 方法</td><td>-</td></tr></tbody></table>

*   限期等待 (Timed Waiting)

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用 “使一个线程睡眠” 进行描述。

调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用 “挂起一个线程” 进行描述。

睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

<table><thead><tr><th>进入方法</th><th>退出方法</th></tr></thead><tbody><tr><td>Thread.sleep() 方法</td><td>时间结束</td></tr><tr><td>设置了 Timeout 参数的 Object.wait() 方法</td><td>时间结束 / Object.notify() / Object.notifyAll()</td></tr><tr><td>设置了 Timeout 参数的 Thread.join() 方法</td><td>时间结束 / 被调用的线程执行完毕</td></tr><tr><td>LockSupport.parkNanos() 方法</td><td>-</td></tr><tr><td>LockSupport.parkUntil() 方法</td><td>-</td></tr></tbody></table>

*   死亡 (Terminated)

可以是线程结束任务之后自己结束，或者产生了异常而结束。

#### 3.1.6. [#](#通常线程有哪几种使用方式) 通常线程有哪几种使用方式?

有三种使用线程的方法:

*   实现 Runnable 接口；
*   实现 Callable 接口；
*   继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

#### 3.1.7. [#](#基础线程机制有哪些) 基础线程机制有哪些?

*   **Executor**

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor:

1.  CachedThreadPool: 一个任务创建一个线程；
2.  FixedThreadPool: 所有任务只能使用固定大小的线程；
3.  SingleThreadExecutor: 相当于大小为 1 的 FixedThreadPool。

*   **Daemon**

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于非守护线程。使用 setDaemon() 方法将一个线程设置为守护线程。

*   **sleep()**

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

*   **yield()**

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

#### 3.1.8. [#](#线程的中断方式有哪些) 线程的中断方式有哪些?

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

*   **InterruptedException**

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

```
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new MyThread1();
        thread1.start();
        thread1.interrupt();
        System.out.println("Main run");
    }
}
```

```
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

*   **interrupted()**

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

*   **Executor 的中断操作**

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

#### 3.1.9. [#](#线程的互斥同步方式有哪些-如何比较和选择) 线程的互斥同步方式有哪些? 如何比较和选择?

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

**1. 锁的实现**

synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

**2. 性能**

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

**3. 等待可中断**

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

**4. 公平锁**

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

**5. 锁绑定多个条件**

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

#### 3.1.10. [#](#线程之间有哪些协作方式) 线程之间有哪些协作方式?

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

*   **join()**

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。

```
public class JoinExample {

    private class A extends Thread {
        @Override
        public void run() {
            System.out.println("A");
        }
    }

    private class B extends Thread {

        private A a;

        B(A a) {
            this.a = a;
        }

        @Override
        public void run() {
            try {
                a.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("B");
        }
    }

    public void test() {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }
}
```

```
public static void main(String[] args) {
    JoinExample example = new JoinExample();
    example.test();
}
```

*   **wait() notify() notifyAll()**

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateExeception。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

**wait() 和 sleep() 的区别**

*   wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
*   wait() 会释放锁，sleep() 不会。
*   **await() signal() signalAll()**

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

### 3.2. [#](#_3-2-并发关键字) 3.2 并发关键字

*   [关键字: synchronized 详解](https://pdai.tech/md/java/thread/java-thread-x-key-synchronized.html)
*   [关键字: volatile 详解](https://pdai.tech/md/java/thread/java-thread-x-key-volatile.html)
*   [关键字: final 详解](https://pdai.tech/md/java/thread/java-thread-x-key-final.html)

#### 3.2.1. [#](#synchronized可以作用在哪里) Synchronized 可以作用在哪里?

*   对象锁
*   方法锁
*   类锁

#### 3.2.2. [#](#synchronized本质上是通过什么保证线程安全的) Synchronized 本质上是通过什么保证线程安全的?

*   **加锁和释放锁的原理**

深入 JVM 看字节码，创建如下的代码：

```
A
B
```

使用 javac 命令进行编译生成. class 文件

```
public class SynchronizedDemo2 {
    Object object = new Object();
    public void method1() {
        synchronized (object) {

        }
    }
}
```

使用 javap 命令反编译查看. class 文件的信息

```
>javac SynchronizedDemo2.java
```

得到如下的信息：

![](https://pdai.tech/images/thread/java-thread-x-key-schronized-1.png)

关注红色方框里的`monitorenter`和`monitorexit`即可。

`Monitorenter`和`Monitorexit`指令，会让对象在执行，使其锁计数器加 1 或者减 1。每一个对象在同一时间只与一个 monitor(锁) 相关联，而一个 monitor 在同一时间只能被一个线程获得，一个对象在尝试获得与这个对象相关联的 Monitor 锁的所有权的时候，monitorenter 指令会发生如下 3 中情况之一：

*   monitor 计数器为 0，意味着目前还没有被获得，那这个线程就会立刻获得然后把锁计数器 + 1，一旦 + 1，别的线程再想获取，就需要等待
*   如果这个 monitor 已经拿到了这个锁的所有权，又重入了这把锁，那锁计数器就会累加，变成 2，并且随着重入的次数，会一直累加
*   这把锁已经被别的线程获取了，等待锁释放

`monitorexit指令`：释放对于 monitor 的所有权，释放过程很简单，就是讲 monitor 的计数器减 1，如果减完以后，计数器不是 0，则代表刚才是重入进来的，当前线程还继续持有这把锁的所有权，如果计数器变成 0，则代表当前线程不再拥有该 monitor 的所有权，即释放锁。

下图表现了对象，对象监视器，同步队列以及执行线程状态之间的关系：

![](https://pdai.tech/images/thread/java-thread-x-key-schronized-2.png)

该图可以看出，任意线程对 Object 的访问，首先要获得 Object 的监视器，如果获取失败，该线程就进入同步状态，线程状态变为 BLOCKED，当 Object 的监视器占有者释放后，在同步队列中得线程就会有机会重新获取该监视器。

*   **可重入原理：加锁次数计数器**

看如下的例子：

```
>javap -verbose SynchronizedDemo2.class
```

对应的字节码

```
public class SynchronizedDemo {

    public static void main(String[] args) {
        synchronized (SynchronizedDemo.class) {

        }
        method2();
    }

    private synchronized static void method2() {

    }
}
```

上面的 SynchronizedDemo 中在执行完同步代码块之后紧接着再会去执行一个静态同步方法，而这个方法锁的对象依然就这个类对象，那么这个正在执行的线程还需要获取该锁吗? 答案是不必的，从上图中就可以看出来，执行静态同步方法的时候就只有一条 monitorexit 指令，并没有 monitorenter 获取锁的指令。这就是锁的重入性，即在同一锁程中，线程不需要再次获取同一把锁。

Synchronized 先天具有重入性。每个对象拥有一个计数器，当线程获取该对象锁后，计数器就会加一，释放锁后就会将计数器减一。

*   **保证可见性的原理：内存模型和 happens-before 规则**

Synchronized 的 happens-before 规则，即监视器锁规则：对同一个监视器的解锁，happens-before 于对该监视器的加锁。继续来看代码：

```
public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #2                  // class tech/pdai/test/synchronized/SynchronizedDemo
         2: dup
         3: astore_1
         4: monitorenter
         5: aload_1
         6: monitorexit
         7: goto          15
        10: astore_2
        11: aload_1
        12: monitorexit
        13: aload_2
        15: invokestatic  #3                  // Method method2:()V
      Exception table:
         from    to  target type
             5     7    10   any
            10    13    10   any
```

该代码的 happens-before 关系如图所示：

![](https://pdai.tech/images/thread/java-thread-x-key-schronized-3.png)

在图中每一个箭头连接的两个节点就代表之间的 happens-before 关系，黑色的是通过程序顺序规则推导出来，红色的为监视器锁规则推导而出：线程 A 释放锁 happens-before 线程 B 加锁，蓝色的则是通过程序顺序规则和监视器锁规则推测出来 happens-befor 关系，通过传递性规则进一步推导的 happens-before 关系。现在我们来重点关注 2 happens-before 5，通过这个关系我们可以得出什么?

根据 happens-before 的定义中的一条: 如果 A happens-before B，则 A 的执行结果对 B 可见，并且 A 的执行顺序先于 B。线程 A 先对共享变量 A 进行加一，由 2 happens-before 5 关系可知线程 A 的执行结果对线程 B 可见即线程 B 所读取到的 a 的值为 1。

#### 3.2.3. [#](#synchronized使得同时只有一个线程可以执行-性能比较差-有什么提升的方法) Synchronized 使得同时只有一个线程可以执行，性能比较差，有什么提升的方法?

简单来说在 JVM 中 monitorenter 和 monitorexit 字节码依赖于底层的操作系统的 Mutex Lock 来实现的，但是由于使用 Mutex Lock 需要将当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的；然而在现实中的大部分情况下，同步方法是运行在单线程环境 (无锁竞争环境) 如果每次都调用 Mutex Lock 那么将严重的影响程序的性能。**不过在 jdk1.6 中对锁的实现引入了大量的优化，如锁粗化 (Lock Coarsening)、锁消除(Lock Elimination)、轻量级锁(Lightweight Locking)、偏向锁(Biased Locking)、适应性自旋(Adaptive Spinning) 等技术来减少锁操作的开销**。

*   **锁粗化 (Lock Coarsening)**：也就是减少不必要的紧连在一起的 unlock，lock 操作，将多个连续的锁扩展成一个范围更大的锁。
    
*   **锁消除 (Lock Elimination)**：通过运行时 JIT 编译器的逃逸分析来消除一些没有在当前同步块以外被其他线程共享的数据的锁保护，通过逃逸分析也可以在线程本地 Stack 上进行对象空间的分配 (同时还可以减少 Heap 上的垃圾收集开销)。
    
*   **轻量级锁 (Lightweight Locking)**：这种锁实现的背后基于这样一种假设，即在真实的情况下我们程序中的大部分同步代码一般都处于无锁竞争状态 (即单线程执行环境)，在无锁竞争的情况下完全可以避免调用操作系统层面的重量级互斥锁，取而代之的是在 monitorenter 和 monitorexit 中只需要依靠一条 CAS 原子指令就可以完成锁的获取及释放。当存在锁竞争的情况下，执行 CAS 指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒。
    
*   **偏向锁 (Biased Locking)**：是为了在无锁竞争的情况下避免在锁获取过程中执行不必要的 CAS 原子指令，因为 CAS 原子指令虽然相对于重量级锁来说开销比较小但还是存在非常可观的本地延迟。
    
*   **适应性自旋 (Adaptive Spinning)**：当线程在获取轻量级锁的过程中执行 CAS 操作失败时，在进入与 monitor 相关联的操作系统重量级锁 (mutex semaphore) 前会进入忙等待 (Spinning) 然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该 monitor 关联的 semaphore(即互斥锁)进入到阻塞状态。

#### 3.2.4. [#](#synchronized由什么样的缺陷-java-lock是怎么弥补这些缺陷的) Synchronized 由什么样的缺陷? Java Lock 是怎么弥补这些缺陷的?

*   **synchronized 的缺陷**

1.  **效率低**：锁的释放情况少，只有代码执行完毕或者异常结束才会释放锁；试图获取锁的时候不能设定超时，不能中断一个正在使用锁的线程，相对而言，Lock 可以中断和设置超时
2.  **不够灵活**：加锁和释放的时机单一，每个锁仅有一个单一的条件 (某个对象)，相对而言，读写锁更加灵活
3.  **无法知道是否成功获得锁**，相对而言，Lock 可以拿到状态

*   **Lock 解决相应问题**

Lock 类这里不做过多解释，主要看里面的 4 个方法:

1.  `lock()`: 加锁
2.  `unlock()`: 解锁
3.  `tryLock()`: 尝试获取锁，返回一个 boolean 值
4.  `tryLock(long,TimeUtil)`: 尝试获取锁，可以设置超时

Synchronized 只有锁只与一个条件 (是否获取锁) 相关联，不灵活，后来 **Condition 与 Lock 的结合**解决了这个问题。

多线程竞争一个锁时，其余未得到锁的线程只能不停的尝试获得锁，而不能中断。高并发的情况下会导致性能下降。ReentrantLock 的 lockInterruptibly() 方法可以优先考虑响应中断。 一个线程等待时间过长，它可以中断自己，然后 ReentrantLock 响应这个中断，不再让这个线程继续等待。有了这个机制，使用 ReentrantLock 时就不会像 synchronized 那样产生死锁了。

#### 3.2.5. [#](#synchronized和lock的对比-和选择) Synchronized 和 Lock 的对比，和选择?

*   **存在层次上**

synchronized: Java 的关键字，在 jvm 层面上

Lock: 是一个接口

*   **锁的释放**

synchronized: 1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm 会让线程释放锁

Lock: 在 finally 中必须释放锁，不然容易造成线程死锁

*   **锁的获取**

synchronized: 假设 A 线程获得锁，B 线程等待。如果 A 线程阻塞，B 线程会一直等待

Lock: 分情况而定，Lock 有多个锁获取的方式，大致就是可以尝试获得锁，线程可以不用一直等待 (可以通过 tryLock 判断有没有锁)

*   **锁的释放（死锁产生）**

synchronized: 在发生异常时候会自动释放占有的锁，因此不会出现死锁

Lock: 发生异常时候，不会主动释放占有的锁，必须手动 unlock 来释放锁，可能引起死锁的发生

*   **锁的状态**

synchronized: 无法判断

Lock: 可以判断

*   **锁的类型**

synchronized: 可重入 不可中断 非公平

Lock: 可重入 可判断 可公平（两者皆可）

*   **性能**

synchronized: 少量同步

Lock: 大量同步

Lock 可以提高多个线程进行读操作的效率。（可以通过 readwritelock 实现读写分离） 在资源竞争不是很激烈的情况下，Synchronized 的性能要优于 ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized 的性能会下降几十倍，但是 ReetrantLock 的性能能维持常态；

ReentrantLock 提供了多样化的同步，比如有时间限制的同步，可以被 Interrupt 的同步（synchronized 的同步是不能 Interrupt 的）等。在资源竞争不激烈的情形下，性能稍微比 synchronized 差点点。但是当同步非常激烈的时候，synchronized 的性能一下子能下降好几十倍。而 ReentrantLock 确还能维持常态。

*   **调度**

synchronized: 使用 Object 对象本身的 wait 、notify、notifyAll 调度机制

Lock: 可以使用 Condition 进行线程之间的调度

*   **用法**

synchronized: 在需要同步的对象中加入此控制，synchronized 可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。

Lock: 一般使用 ReentrantLock 类做为锁。在加锁和解锁处需要通过 lock() 和 unlock() 显示指出。所以一般会在 finally 块中写 unlock() 以防死锁。

*   **底层实现**

synchronized: 底层使用指令码方式来控制锁的，映射成字节码指令就是增加来两个指令：monitorenter 和 monitorexit。当线程执行遇到 monitorenter 指令时会尝试获取内置锁，如果获取锁则锁计数器 + 1，如果没有获取锁则阻塞；当遇到 monitorexit 指令时锁计数器 - 1，如果计数器为 0 则释放锁。

Lock: 底层是 CAS 乐观锁，依赖 AbstractQueuedSynchronizer 类，把所有的请求线程构成一个 CLH 队列。而对该队列的操作均通过 Lock-Free（CAS）操作。

#### 3.2.6. [#](#synchronized在使用时有何注意事项) Synchronized 在使用时有何注意事项?

*   锁对象不能为空，因为锁的信息都保存在对象头里
*   作用域不宜过大，影响程序执行的速度，控制范围过大，编写代码也容易出错
*   避免死锁
*   在能选择的情况下，既不要用 Lock 也不要用 synchronized 关键字，用 java.util.concurrent 包中的各种各样的类，如果不用该包下的类，在满足业务的情况下，可以使用 synchronized 关键，因为代码量少，避免出错

#### 3.2.7. [#](#synchronized修饰的方法在抛出异常时-会释放锁吗) Synchronized 修饰的方法在抛出异常时, 会释放锁吗?

会

#### 3.2.8. [#](#多个线程等待同一个synchronized锁的时候-jvm如何选择下一个获取锁的线程) 多个线程等待同一个 Synchronized 锁的时候，JVM 如何选择下一个获取锁的线程?

非公平锁，即抢占式。

#### 3.2.9. [#](#synchronized是公平锁吗) synchronized 是公平锁吗？

synchronized 实际上是非公平的，新来的线程有可能立即获得监视器，而在等待区中等候已久的线程可能再次等待，这样有利于提高性能，但是也可能会导致饥饿现象。

#### 3.2.10. [#](#volatile关键字的作用是什么) volatile 关键字的作用是什么?

*   **防重排序** 我们从一个最经典的例子来分析重排序问题。大家应该都很熟悉单例模式的实现，而在并发环境下的单例实现方式，我们通常可以采用双重检查加锁 (DCL) 的方式来实现。其源码如下：

```
public class MonitorDemo {
    private int a = 0;

    public synchronized void writer() {     // 1
        a++;                                // 2
    }                                       // 3

    public synchronized void reader() {    // 4
        int i = a;                         // 5
    }                                      // 6
}
```

现在我们分析一下为什么要在变量 singleton 之间加上 volatile 关键字。要理解这个问题，先要了解对象的构造过程，实例化一个对象其实可以分为三个步骤：

1.  分配内存空间。
2.  初始化对象。
3.  将内存空间的地址赋值给对应的引用。

但是由于操作系统可以**对指令进行重排序**，所以上面的过程也可能会变成如下过程：

1.  分配内存空间。
2.  将内存空间的地址赋值给对应的引用。
3.  初始化对象

如果是这个流程，多线程环境下就可能将一个未初始化的对象引用暴露出来，从而导致不可预料的结果。因此，为了防止这个过程的重排序，我们需要将变量设置为 volatile 类型的变量。

*   **实现可见性**

可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——线程工作内存。volatile 关键字能有效的解决这个问题，我们看下下面的例子，就可以知道其作用：

```
public class Singleton {
    public static volatile Singleton singleton;
    /**
     * 构造函数私有，禁止外部实例化
     */
    private Singleton() {};
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

执行输出如下

```
public class TestVolatile {
    private static boolean stop = false;

    public static void main(String[] args) {
        // Thread-A
        new Thread("Thread A") {
            @Override
            public void run() {
                while (!stop) {
                }
                System.out.println(Thread.currentThread() + " stopped");
            }
        }.start();

        // Thread-main
        try {
            TimeUnit.SECONDS.sleep(1);
            System.out.println(Thread.currentThread() + " after 1 seconds");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        stop = true;
    }
}
```

可以看到 Thread-main 休眠 1 秒之后，设置 stop = ture，但是 Thread A 根本没停下来，这就是可见性问题。如果通过在 stop 变量前面加上 volatile 关键字则会真正 stop:

```
Thread[main,5,main] after 1 seconds

// Thread A一直在loop, 因为Thread A 由于可见性原因看不到Thread Main 已经修改stop的值
```

*   **保证原子性: 单次读 / 写**

volatile 不能保证完全的原子性，只能保证单次的读 / 写操作具有原子性。

#### 3.2.11. [#](#volatile能保证原子性吗) volatile 能保证原子性吗?

不能完全保证，只能保证单次的读 / 写操作具有原子性。

#### 3.2.12. [#](#_32位机器上共享的long和double变量的为什么要用volatile) 32 位机器上共享的 long 和 double 变量的为什么要用 volatile?

因为 long 和 double 两种数据类型的操作可分为高 32 位和低 32 位两部分，因此普通的 long 或 double 类型读 / 写可能不是原子的。因此，鼓励大家将共享的 long 和 double 变量设置为 volatile 类型，这样能保证任何情况下对 long 和 double 的单次读 / 写操作都具有原子性。

如下是 JLS 中的解释：

17.7 Non-Atomic Treatment of double and long

*   For the purposes of the Java programming language memory model, a single write to a non-volatile long or double value is treated as two separate writes: one to each 32-bit half. This can result in a situation where a thread sees the first 32 bits of a 64-bit value from one write, and the second 32 bits from another write.
*   Writes and reads of volatile long and double values are always atomic.
*   Writes to and reads of references are always atomic, regardless of whether they are implemented as 32-bit or 64-bit values.
*   Some implementations may find it convenient to divide a single write action on a 64-bit long or double value into two write actions on adjacent 32-bit values. For efficiency’s sake, this behavior is implementation-specific; an implementation of the Java Virtual Machine is free to perform writes to long and double values atomically or in two parts.
*   Implementations of the Java Virtual Machine are encouraged to avoid splitting 64-bit values where possible. Programmers are encouraged to declare shared 64-bit values as volatile or synchronize their programs correctly to avoid possible complications.

目前各种平台下的商用虚拟机都选择把 64 位数据的读写操作作为原子操作来对待，因此我们在编写代码时一般不把 long 和 double 变量专门声明为 volatile 多数情况下也是不会错的。

#### 3.2.13. [#](#volatile是如何实现可见性的) volatile 是如何实现可见性的?

内存屏障。

#### 3.2.14. [#](#volatile是如何实现有序性的) volatile 是如何实现有序性的?

happens-before 等

#### 3.2.15. [#](#说下volatile的应用场景) 说下 volatile 的应用场景?

使用 volatile 必须具备的条件

1.  对变量的写操作不依赖于当前值。
2.  该变量没有包含在具有其他变量的不变式中。
3.  只有在状态真正独立于程序内其他内容时才能使用 volatile。

*   **例子 1： 单例模式**

单例模式的一种实现方式，但很多人会忽略 volatile 关键字，因为没有该关键字，程序也可以很好的运行，只不过代码的稳定性总不是 100%，说不定在未来的某个时刻，隐藏的 bug 就出来了。

```
Thread[main,5,main] after 1 seconds
Thread[Thread A,5,main] stopped

Process finished with exit code 0
```

*   **例子 2： volatile bean**

在 volatile bean 模式中，JavaBean 的所有数据成员都是 volatile 类型的，并且 getter 和 setter 方法必须非常普通 —— 除了获取或设置相应的属性外，不能包含任何逻辑。此外，对于对象引用的数据成员，引用的对象必须是有效不可变的。(这将禁止具有数组值的属性，因为当数组引用被声明为 volatile 时，只有引用而不是数组本身具有 volatile 语义)。对于任何 volatile 变量，不变式或约束都不能包含 JavaBean 属性。

```
class Singleton {
    private volatile static Singleton instance;
    private Singleton() {
    }
    public static Singleton getInstance() {
        if (instance == null) {
            syschronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    } 
}
```

#### 3.2.16. [#](#所有的final修饰的字段都是编译期常量吗) 所有的 final 修饰的字段都是编译期常量吗?

不是

#### 3.2.17. [#](#如何理解private所修饰的方法是隐式的final) 如何理解 private 所修饰的方法是隐式的 final?

类中所有 private 方法都隐式地指定为 final 的，由于无法取用 private 方法，所以也就不能覆盖它。可以对 private 方法增添 final 关键字，但这样做并没有什么好处。看下下面的例子：

```
@ThreadSafe
public class Person {
    private volatile String firstName;
    private volatile String lastName;
    private volatile int age;
 
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public int getAge() { return age; }
 
    public void setFirstName(String firstName) { 
        this.firstName = firstName;
    }
 
    public void setLastName(String lastName) { 
        this.lastName = lastName;
    }
 
    public void setAge(int age) { 
        this.age = age;
    }
}
```

Base 和 Son 都有方法 test(), 但是这并不是一种覆盖，因为 private 所修饰的方法是隐式的 final，也就是无法被继承，所以更不用说是覆盖了，在 Son 中的 test() 方法不过是属于 Son 的新成员罢了，Son 进行向上转型得到 father，但是 father.test() 是不可执行的，因为 Base 中的 test 方法是 private 的，无法被访问到。

#### 3.2.18. [#](#说说final类型的类如何拓展) 说说 final 类型的类如何拓展?

比如 String 是 final 类型，我们想写个 MyString 复用所有 String 中方法，同时增加一个新的 toMyString() 的方法，应该如何做?

外观模式：

```
public class Base {
    private void test() {
    }
}

public class Son extends Base{
    public void test() {
    }
    public static void main(String[] args) {
        Son son = new Son();
        Base father = son;
        //father.test();
    }
}
```

#### 3.2.19. [#](#final方法可以被重载吗) final 方法可以被重载吗?

我们知道父类的 final 方法是不能够被子类重写的，那么 final 方法可以被重载吗? 答案是可以的，下面代码是正确的。

```
/**
* @pdai
*/
class MyString{

    private String innerString;

    // ...init & other methods

    // 支持老的方法
    public int length(){
        return innerString.length(); // 通过innerString调用老的方法
    }

    // 添加新方法
    public String toMyString(){
        //...
    }
}
```

#### 3.2.20. [#](#父类的final方法能不能够被子类重写) 父类的 final 方法能不能够被子类重写?

不可以

#### 3.2.21. [#](#说说基本类型的final域重排序规则) 说说基本类型的 final 域重排序规则?

先看一段示例性的代码：

```
public class FinalExampleParent {
    public final void test() {
    }

    public final void test(String str) {
    }
}
```

假设线程 A 在执行 writer() 方法，线程 B 执行 reader() 方法。

*   **写 final 域重排序规则**

写 final 域的重排序规则禁止对 final 域的写重排序到构造函数之外，这个规则的实现主要包含了两个方面：

*   JMM 禁止编译器把 final 域的写重排序到构造函数之外；
*   编译器会在 final 域写之后，构造函数 return 之前，插入一个 storestore 屏障。这个屏障可以禁止处理器把 final 域的写重排序到构造函数之外。

我们再来分析 writer 方法，虽然只有一行代码，但实际上做了两件事情：

*   构造了一个 FinalDemo 对象；
*   把这个对象赋值给成员变量 finalDemo。

我们来画下存在的一种可能执行时序图，如下：

![](https://pdai.tech/images/thread/java-thread-x-key-final-1.png)

由于 a,b 之间没有数据依赖性，普通域 (普通变量)a 可能会被重排序到构造函数之外，线程 B 就有可能读到的是普通变量 a 初始化之前的值 (零值)，这样就可能出现错误。而 final 域变量 b，根据重排序规则，会禁止 final 修饰的变量 b 重排序到构造函数之外，从而 b 能够正确赋值，线程 B 就能够读到 final 变量初始化后的值。

因此，写 final 域的重排序规则可以确保：在对象引用为任意线程可见之前，对象的 final 域已经被正确初始化过了，而普通域就不具有这个保障。比如在上例，线程 B 有可能就是一个未正确初始化的对象 finalDemo。

*   **读 final 域重排序规则**

读 final 域重排序规则为：在一个线程中，初次读对象引用和初次读该对象包含的 final 域，JMM 会禁止这两个操作的重排序。(注意，这个规则仅仅是针对处理器)，处理器会在读 final 域操作的前面插入一个 LoadLoad 屏障。实际上，读对象的引用和读该对象的 final 域存在间接依赖性，一般处理器不会重排序这两个操作。但是有一些处理器会重排序，因此，这条禁止重排序规则就是针对这些处理器而设定的。

read() 方法主要包含了三个操作：

*   初次读引用变量 finalDemo;
*   初次读引用变量 finalDemo 的普通域 a;
*   初次读引用变量 finalDemo 的 final 域 b;

假设线程 A 写过程没有重排序，那么线程 A 和线程 B 有一种的可能执行时序为下图：

![](https://pdai.tech/images/thread/java-thread-x-key-final-2.png)

读对象的普通域被重排序到了读对象引用的前面就会出现线程 B 还未读到对象引用就在读取该对象的普通域变量，这显然是错误的操作。而 final 域的读操作就 “限定” 了在读 final 域变量前已经读到了该对象的引用，从而就可以避免这种情况。

读 final 域的重排序规则可以确保：在读一个对象的 final 域之前，一定会先读这个包含这个 final 域的对象的引用。

#### 3.2.22. [#](#说说final的原理) 说说 final 的原理?

*   写 final 域会要求编译器在 final 域写之后，构造函数返回前插入一个 StoreStore 屏障。
*   读 final 域的重排序规则会要求编译器在读 final 域的操作前插入一个 LoadLoad 屏障。

PS: 很有意思的是，如果以 X86 处理为例，X86 不会对写 - 写重排序，所以 StoreStore 屏障可以省略。由于不会对有间接依赖性的操作重排序，所以在 X86 处理器中，读 final 域需要的 LoadLoad 屏障也会被省略掉。也就是说，以 X86 为例的话，对 final 域的读 / 写的内存屏障都会被省略！具体是否插入还是得看是什么处理器。

### 3.3. [#](#_3-3-juc全局观) 3.3 JUC 全局观

*   [JUC - 类汇总和学习指南](https://pdai.tech/md/java/thread/java-thread-x-juc-overview.html)

#### 3.3.1. [#](#juc框架包含几个部分) JUC 框架包含几个部分?

五个部分：

![](https://pdai.tech/images/thread/java-thread-x-juc-overview-1-u.png)

主要包含: (注意: 上图是网上找的图，无法表述一些继承关系，同时少了部分类；但是主体上可以看出其分类关系也够了)

*   Lock 框架和 Tools 类 (把图中这两个放到一起理解)
*   Collections: 并发集合
*   Atomic: 原子类
*   Executors: 线程池

#### 3.3.2. [#](#lock框架和tools哪些核心的类) Lock 框架和 Tools 哪些核心的类?

![](https://pdai.tech/images/thread/java-thread-x-juc-overview-lock.png)

*   **接口: Condition**， Condition 为接口类型，它将 Object 监视器方法 (wait、notify 和 notifyAll) 分解成截然不同的对象，以便通过将这些对象与任意 Lock 实现组合使用，为每个对象提供多个等待 set (wait-set)。其中，Lock 替代了 synchronized 方法和语句的使用，Condition 替代了 Object 监视器方法的使用。可以通过 await(),signal()来休眠 / 唤醒线程。
*   **接口: Lock**，Lock 为接口类型，Lock 实现提供了比使用 synchronized 方法和语句可获得的更广泛的锁定操作。此实现允许更灵活的结构，可以具有差别很大的属性，可以支持多个相关的 Condition 对象。
*   **接口 ReadWriteLock** ReadWriteLock 为接口类型， 维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。
*   **抽象类: AbstractOwnableSynchonizer** AbstractOwnableSynchonizer 为抽象类，可以由线程以独占方式拥有的同步器。此类为创建锁和相关同步器 (伴随着所有权的概念) 提供了基础。AbstractOwnableSynchronizer 类本身不管理或使用此信息。但是，子类和工具可以使用适当维护的值帮助控制和监视访问以及提供诊断。
*   **抽象类 (long): AbstractQueuedLongSynchronizer** AbstractQueuedLongSynchronizer 为抽象类，以 long 形式维护同步状态的一个 AbstractQueuedSynchronizer 版本。此类具有的结构、属性和方法与 AbstractQueuedSynchronizer 完全相同，但所有与状态相关的参数和结果都定义为 long 而不是 int。当创建需要 64 位状态的多级别锁和屏障等同步器时，此类很有用。
*   **核心抽象类 (int): AbstractQueuedSynchronizer** AbstractQueuedSynchronizer 为抽象类，其为实现依赖于先进先出 (FIFO) 等待队列的阻塞锁和相关同步器 (信号量、事件，等等) 提供一个框架。此类的设计目标是成为依靠单个原子 int 值来表示状态的大多数同步器的一个有用基础。
*   **锁常用类: LockSupport** LockSupport 为常用类，用来创建锁和其他同步类的基本线程阻塞原语。LockSupport 的功能和 "Thread 中的 Thread.suspend()和 Thread.resume()有点类似"，LockSupport 中的 park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程。但是 park()和 unpark()不会遇到 “Thread.suspend 和 Thread.resume 所可能引发的死锁” 问题。
*   **锁常用类: ReentrantLock** ReentrantLock 为常用类，它是一个可重入的互斥锁 Lock，它具有与使用 synchronized 方法和语句所访问的隐式监视器锁相同的一些基本行为和语义，但功能更强大。
*   **锁常用类: ReentrantReadWriteLock** ReentrantReadWriteLock 是读写锁接口 ReadWriteLock 的实现类，它包括 Lock 子类 ReadLock 和 WriteLock。ReadLock 是共享锁，WriteLock 是独占锁。
*   **锁常用类: StampedLock** 它是 java8 在 java.util.concurrent.locks 新增的一个 API。StampedLock 控制锁有三种模式 (写，读，乐观读)，一个 StampedLock 状态是由版本和模式两个部分组成，锁获取方法返回一个数字作为票据 stamp，它用相应的锁状态表示并控制访问，数字 0 表示没有写锁被授权访问。在读锁上分为悲观锁和乐观锁。
*   **工具常用类: CountDownLatch** CountDownLatch 为常用类，它是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
*   **工具常用类: CyclicBarrier** CyclicBarrier 为常用类，其是一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。
*   **工具常用类: Phaser** Phaser 是 JDK 7 新增的一个同步辅助类，它可以实现 CyclicBarrier 和 CountDownLatch 类似的功能，而且它支持对任务的动态调整，并支持分层结构来达到更高的吞吐量。
*   **工具常用类: Semaphore** Semaphore 为常用类，其是一个计数信号量，从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。通常用于限制可以访问某些资源 (物理或逻辑的) 的线程数目。
*   **工具常用类: Exchanger** Exchanger 是用于线程协作的工具类, 主要用于两个线程之间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过 exchange() 方法交换数据，当一个线程先执行 exchange() 方法后，它会一直等待第二个线程也执行 exchange() 方法，当这两个线程到达同步点时，这两个线程就可以交换数据了。

#### 3.3.3. [#](#juc并发集合哪些核心的类) JUC 并发集合哪些核心的类?

![](https://pdai.tech/images/thread/java-thread-x-juc-overview-2.png)

*   **Queue: ArrayBlockingQueue** 一个由数组支持的有界阻塞队列。此队列按 FIFO(先进先出) 原则对元素进行排序。队列的头部 是在队列中存在时间最长的元素。队列的尾部 是在队列中存在时间最短的元素。新元素插入到队列的尾部，队列获取操作则是从队列头部开始获得元素。
*   **Queue: LinkedBlockingQueue** 一个基于已链接节点的、范围任意的 blocking queue。此队列按 FIFO(先进先出) 排序元素。队列的头部 是在队列中时间最长的元素。队列的尾部 是在队列中时间最短的元素。新元素插入到队列的尾部，并且队列获取操作会获得位于队列头部的元素。链接队列的吞吐量通常要高于基于数组的队列，但是在大多数并发应用程序中，其可预知的性能要低。
*   **Queue: LinkedBlockingDeque** 一个基于已链接节点的、任选范围的阻塞双端队列。
*   **Queue: ConcurrentLinkedQueue** 一个基于链接节点的无界线程安全队列。此队列按照 FIFO(先进先出) 原则对元素进行排序。队列的头部 是队列中时间最长的元素。队列的尾部 是队列中时间最短的元素。新的元素插入到队列的尾部，队列获取操作从队列头部获得元素。当多个线程共享访问一个公共 collection 时，ConcurrentLinkedQueue 是一个恰当的选择。此队列不允许使用 null 元素。
*   **Queue: ConcurrentLinkedDeque** 是双向链表实现的无界队列，该队列同时支持 FIFO 和 FILO 两种操作方式。
*   **Queue: DelayQueue** 延时无界阻塞队列，使用 Lock 机制实现并发访问。队列里只允许放可以 “延期” 的元素，队列中的 head 是最先 “到期” 的元素。如果队里中没有元素到“到期”，那么就算队列中有元素也不能获取到。
*   **Queue: PriorityBlockingQueue** 无界优先级阻塞队列，使用 Lock 机制实现并发访问。priorityQueue 的线程安全版，不允许存放 null 值，依赖于 comparable 的排序，不允许存放不可比较的对象类型。
*   **Queue: SynchronousQueue** 没有容量的同步队列，通过 CAS 实现并发访问，支持 FIFO 和 FILO。
*   **Queue: LinkedTransferQueue** JDK 7 新增，单向链表实现的无界阻塞队列，通过 CAS 实现并发访问，队列元素使用 FIFO(先进先出) 方式。LinkedTransferQueue 可以说是 ConcurrentLinkedQueue、SynchronousQueue(公平模式) 和 LinkedBlockingQueue 的超集, 它不仅仅综合了这几个类的功能，同时也提供了更高效的实现。
*   **List: CopyOnWriteArrayList** ArrayList 的一个线程安全的变体，其中所有可变操作 (add、set 等等) 都是通过对底层数组进行一次新的复制来实现的。这一般需要很大的开销，但是当遍历操作的数量大大超过可变操作的数量时，这种方法可能比其他替代方法更 有效。在不能或不想进行同步遍历，但又需要从并发线程中排除冲突时，它也很有用。
*   **Set: CopyOnWriteArraySet** 对其所有操作使用内部 CopyOnWriteArrayList 的 Set。即将所有操作转发至 CopyOnWriteArayList 来进行操作，能够保证线程安全。在 add 时，会调用 addIfAbsent，由于每次 add 时都要进行数组遍历，因此性能会略低于 CopyOnWriteArrayList。
*   **Set: ConcurrentSkipListSet** 一个基于 ConcurrentSkipListMap 的可缩放并发 NavigableSet 实现。set 的元素可以根据它们的自然顺序进行排序，也可以根据创建 set 时所提供的 Comparator 进行排序，具体取决于使用的构造方法。
*   **Map: ConcurrentHashMap** 是线程安全 HashMap 的。ConcurrentHashMap 在 JDK 7 之前是通过 Lock 和 segment(分段锁) 实现，JDK 8 之后改为 CAS+synchronized 来保证并发安全。
*   **Map: ConcurrentSkipListMap** 线程安全的有序的哈希表 (相当于线程安全的 TreeMap); 映射可以根据键的自然顺序进行排序，也可以根据创建映射时所提供的 Comparator 进行排序，具体取决于使用的构造方法。

#### 3.3.4. [#](#juc原子类哪些核心的类) JUC 原子类哪些核心的类?

其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由 JVM 从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。实际上是借助硬件的相关指令来实现的，不会阻塞线程 (或者说只是在硬件级别上阻塞了)。

*   **原子更新基本类型**
    *   AtomicBoolean: 原子更新布尔类型。
    *   AtomicInteger: 原子更新整型。
    *   AtomicLong: 原子更新长整型。
*   **原子更新数组**
    *   AtomicIntegerArray: 原子更新整型数组里的元素。
    *   AtomicLongArray: 原子更新长整型数组里的元素。
    *   AtomicReferenceArray: 原子更新引用类型数组里的元素。
*   **原子更新引用类型**
    *   AtomicIntegerFieldUpdater: 原子更新整型的字段的更新器。
    *   AtomicLongFieldUpdater: 原子更新长整型字段的更新器。
    *   AtomicStampedFieldUpdater: 原子更新带有版本号的引用类型。
    *   AtomicReferenceFieldUpdater: 上面已经说过此处不在赘述
*   **原子更新字段类**
    *   AtomicReference: 原子更新引用类型。
    *   AtomicStampedReference: 原子更新引用类型, 内部使用 Pair 来存储元素值及其版本号。
    *   AtomicMarkableReferce: 原子更新带有标记位的引用类型。

#### 3.3.5. [#](#juc线程池哪些核心的类) JUC 线程池哪些核心的类?

![](https://pdai.tech/images/thread/java-thread-x-juc-executors-1.png)

*   **接口: Executor** Executor 接口提供一种将任务提交与每个任务将如何运行的机制 (包括线程使用的细节、调度等) 分离开来的方法。通常使用 Executor 而不是显式地创建线程。
*   **ExecutorService** ExecutorService 继承自 Executor 接口，ExecutorService 提供了管理终止的方法，以及可为跟踪一个或多个异步任务执行状况而生成 Future 的方法。 可以关闭 ExecutorService，这将导致其停止接受新任务。关闭后，执行程序将最后终止，这时没有任务在执行，也没有任务在等待执行，并且无法提交新任务。
*   **ScheduledExecutorService** ScheduledExecutorService 继承自 ExecutorService 接口，可安排在给定的延迟后运行或定期执行的命令。
*   **AbstractExecutorService** AbstractExecutorService 继承自 ExecutorService 接口，其提供 ExecutorService 执行方法的默认实现。此类使用 newTaskFor 返回的 RunnableFuture 实现 submit、invokeAny 和 invokeAll 方法，默认情况下，RunnableFuture 是此包中提供的 FutureTask 类。
*   **FutureTask** FutureTask 为 Future 提供了基础实现，如获取任务执行结果 (get) 和取消任务 (cancel) 等。如果任务尚未完成，获取任务执行结果时将会阻塞。一旦执行结束，任务就不能被重启或取消(除非使用 runAndReset 执行计算)。FutureTask 常用来封装 Callable 和 Runnable，也可以作为一个任务提交到线程池中执行。除了作为一个独立的类之外，此类也提供了一些功能性函数供我们创建自定义 task 类使用。FutureTask 的线程安全由 CAS 来保证。
*   **核心: ThreadPoolExecutor** ThreadPoolExecutor 实现了 AbstractExecutorService 接口，也是一个 ExecutorService，它使用可能的几个池线程之一执行每个提交的任务，通常使用 Executors 工厂方法配置。 线程池可以解决两个不同问题: 由于减少了每个任务调用的开销，它们通常可以在执行大量异步任务时提供增强的性能，并且还可以提供绑定和管理资源 (包括执行任务集时使用的线程) 的方法。每个 ThreadPoolExecutor 还维护着一些基本的统计数据，如完成的任务数。
*   **核心: ScheduledThreadExecutor** ScheduledThreadPoolExecutor 实现 ScheduledExecutorService 接口，可安排在给定的延迟后运行命令，或者定期执行命令。需要多个辅助线程时，或者要求 ThreadPoolExecutor 具有额外的灵活性或功能时，此类要优于 Timer。
*   **核心: Fork/Join 框架** ForkJoinPool 是 JDK 7 加入的一个线程池类。Fork/Join 技术是分治算法 (Divide-and-Conquer) 的并行实现，它是一项可以获得良好的并行性能的简单且高效的设计技术。目的是为了帮助我们更好地利用多处理器带来的好处，使用所有可用的运算能力来提升应用的性能。
*   **工具类: Executors** Executors 是一个工具类，用其可以创建 ExecutorService、ScheduledExecutorService、ThreadFactory、Callable 等对象。它的使用融入到了 ThreadPoolExecutor, ScheduledThreadExecutor 和 ForkJoinPool 中。

### 3.4. [#](#_3-4-juc原子类) 3.4 JUC 原子类

*   [JUC 原子类: CAS, Unsafe 和原子类详解](https://pdai.tech/md/java/thread/java-thread-x-juc-AtomicInteger.html)

#### 3.4.1. [#](#线程安全的实现方法有哪些) 线程安全的实现方法有哪些?

线程安全的实现方法包含:

*   互斥同步: synchronized 和 ReentrantLock
*   非阻塞同步: CAS, AtomicXXXX
*   无同步方案: 栈封闭，Thread Local，可重入代码

#### 3.4.2. [#](#什么是cas) 什么是 CAS?

CAS 的全称为 Compare-And-Swap，直译就是对比交换。是一条 CPU 的原子指令，其作用是让 CPU 先进行比较两个值是否相等，然后原子地更新某个位置的值，经过调查发现，其实现方式是基于硬件平台的汇编指令，就是说 CAS 是靠硬件实现的，JVM 只是封装了汇编调用，那些 AtomicInteger 类便是使用了这些封装后的接口。   简单解释：CAS 操作需要输入两个数值，一个旧值 (期望操作前的值) 和一个新值，在操作期间先比较下在旧值有没有发生变化，如果没有发生变化，才交换成新值，发生了变化则不交换。

CAS 操作是原子性的，所以多线程并发使用 CAS 更新数据时，可以不使用锁。JDK 中大量使用了 CAS 来更新数据而防止加锁 (synchronized 重量级锁) 来保持原子更新。

相信 sql 大家都熟悉，类似 sql 中的条件更新一样：update set id=3 from table where id=2。因为单条 sql 执行具有原子性，如果有多个线程同时执行此 sql 语句，只有一条能更新成功。

#### 3.4.3. [#](#cas使用示例-结合atomicinteger给出示例) CAS 使用示例，结合 AtomicInteger 给出示例?

如果不使用 CAS，在高并发下，多线程同时修改一个变量的值我们需要 synchronized 加锁 (可能有人说可以用 Lock 加锁，Lock 底层的 AQS 也是基于 CAS 进行获取锁的)。

```
public class FinalDemo {
    private int a;  //普通域
    private final int b; //final域
    private static FinalDemo finalDemo;

    public FinalDemo() {
        a = 1; // 1. 写普通域
        b = 2; // 2. 写final域
    }

    public static void writer() {
        finalDemo = new FinalDemo();
    }

    public static void reader() {
        FinalDemo demo = finalDemo; // 3.读对象引用
        int a = demo.a;    //4.读普通域
        int b = demo.b;    //5.读final域
    }
}
```

java 中为我们提供了 AtomicInteger 原子类 (底层基于 CAS 进行更新数据的)，不需要加锁就在多线程并发场景下实现数据的一致性。

```
public class Test {
    private int i=0;
    public synchronized int add(){
        return i++;
    }
}
```

#### 3.4.4. [#](#cas会有哪些问题) CAS 会有哪些问题?

CAS 方式为乐观锁，synchronized 为悲观锁。因此使用 CAS 解决并发问题通常情况下性能更优。

但使用 CAS 方式也会有几个问题：

*   ABA 问题

因为 CAS 需要在操作值的时候，检查值有没有发生变化，比如没有发生变化则更新，但是如果一个值原来是 A，变成了 B，又变成了 A，那么使用 CAS 进行检查时则会发现它的值没有发生变化，但是实际上却变化了。

ABA 问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加 1，那么 A->B->A 就会变成 1A->2B->3A。

从 Java 1.5 开始，JDK 的 Atomic 包里提供了一个类 AtomicStampedReference 来解决 ABA 问题。这个类的 compareAndSet 方法的作用是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

*   循环时间长开销大

自旋 CAS 如果长时间不成功，会给 CPU 带来非常大的执行开销。如果 JVM 能支持处理器提供的 pause 指令，那么效率会有一定的提升。pause 指令有两个作用：第一，它可以延迟流水线执行命令 (de-pipeline)，使 CPU 不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零；第二，它可以避免在退出循环的时候因内存顺序冲突(Memory Order Violation) 而引起 CPU 流水线被清空(CPU Pipeline Flush)，从而提高 CPU 的执行效率。

*   只能保证一个共享变量的原子操作

当对一个共享变量执行操作时，我们可以使用循环 CAS 的方式来保证原子操作，但是对多个共享变量操作时，循环 CAS 就无法保证操作的原子性，这个时候就可以用锁。

还有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如，有两个共享变量 i = 2，j = a，合并一下 ij = 2a，然后用 CAS 来操作 ij。

从 Java 1.5 开始，JDK 提供了 AtomicReference 类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行 CAS 操作。

#### 3.4.5. [#](#atomicinteger底层实现) AtomicInteger 底层实现?

*   CAS+volatile
*   volatile 保证线程的可见性，多线程并发时，一个线程修改数据，可以保证其它线程立马看到修改后的值 CAS 保证数据更新的原子性。

#### 3.4.6. [#](#请阐述你对unsafe类的理解) 请阐述你对 Unsafe 类的理解?

UnSafe 类总体功能：

![](https://pdai.tech/images/thread/java-thread-x-atomicinteger-unsafe.png)

如上图所示，Unsafe 提供的 API 大致可分为内存操作、CAS、Class 相关、对象操作、线程调度、系统信息获取、内存屏障、数组操作等几类，下面将对其相关方法和应用场景进行详细介绍。

#### 3.4.7. [#](#说说你对java原子类的理解) 说说你对 Java 原子类的理解?

包含 13 个，4 组分类，说说作用和使用场景。

*   原子更新基本类型
    *   AtomicBoolean: 原子更新布尔类型。
    *   AtomicInteger: 原子更新整型。
    *   AtomicLong: 原子更新长整型。
*   原子更新数组
    *   AtomicIntegerArray: 原子更新整型数组里的元素。
    *   AtomicLongArray: 原子更新长整型数组里的元素。
    *   AtomicReferenceArray: 原子更新引用类型数组里的元素。
*   原子更新引用类型
    *   AtomicIntegerFieldUpdater: 原子更新整型的字段的更新器。
    *   AtomicLongFieldUpdater: 原子更新长整型字段的更新器。
    *   AtomicStampedFieldUpdater: 原子更新带有版本号的引用类型。
    *   AtomicReferenceFieldUpdater: 上面已经说过此处不在赘述
*   原子更新字段类
    *   AtomicReference: 原子更新引用类型。
    *   AtomicStampedReference: 原子更新引用类型, 内部使用 Pair 来存储元素值及其版本号。
    *   AtomicMarkableReferce: 原子更新带有标记位的引用类型。

#### 3.4.8. [#](#atomicstampedreference是怎么解决aba的) AtomicStampedReference 是怎么解决 ABA 的?

AtomicStampedReference 主要维护包含一个对象引用以及一个可以自动更新的整数 "stamp" 的 pair 对象来解决 ABA 问题。

### 3.5. [#](#_3-5-juc锁) 3.5 JUC 锁

*   [JUC 锁: LockSupport 详解](https://pdai.tech/md/java/thread/java-thread-x-lock-LockSupport.html)
*   [JUC 锁: 锁核心类 AQS 详解](https://pdai.tech/md/java/thread/java-thread-x-lock-AbstractQueuedSynchronizer.html)
*   [JUC 锁: ReentrantReadWriteLock 详解](https://pdai.tech/md/java/thread/java-thread-x-lock-ReentrantReadWriteLock.html)

#### 3.5.1. [#](#为什么locksupport也是核心基础类) 为什么 LockSupport 也是核心基础类?

AQS 框架借助于两个类：Unsafe(提供 CAS 操作) 和 LockSupport(提供 park/unpark 操作)

#### 3.5.2. [#](#通过wait-notify实现同步) 通过 wait/notify 实现同步?

```
public class Test {
    private  AtomicInteger i = new AtomicInteger(0);
    public int add(){
        return i.addAndGet(1);
    }
}
```

运行结果

```
class MyThread extends Thread {
    
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}

public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();            
        synchronized (myThread) {
            try {        
                myThread.start();
                // 主线程睡眠3s
                Thread.sleep(3000);
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
```

说明: 具体的流程图如下

![](https://pdai.tech/images/thread/java-thread-x-locksupport-1.png)

使用 wait/notify 实现同步时，必须先调用 wait，后调用 notify，如果先调用 notify，再调用 wait，将起不了作用。具体代码如下

```
before wait
before notify
after notify
after wait
```

运行结果:

```
class MyThread extends Thread {
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}

public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();        
        myThread.start();
        // 主线程睡眠3s
        Thread.sleep(3000);
        synchronized (myThread) {
            try {        
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
```

说明: 由于先调用了 notify，再调用的 wait，此时主线程还是会一直阻塞。

#### 3.5.3. [#](#通过locksupport的park-unpark实现同步) 通过 LockSupport 的 park/unpark 实现同步？

```
before notify
after notify
before wait
```

运行结果:

```
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before unpark");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));
        // 释放许可
        LockSupport.unpark((Thread) object);
        // 休眠500ms，保证先执行park中的setBlocker(t, null);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 再次获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));

        System.out.println("after unpark");
    }
}

public class test {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
```

说明: 本程序先执行 park，然后在执行 unpark，进行同步，并且在 unpark 的前后都调用了 getBlocker，可以看到两次的结果不一样，并且第二次调用的结果为 null，这是因为在调用 unpark 之后，执行了 Lock.park(Object blocker) 函数中的 setBlocker(t, null) 函数，所以第二次调用 getBlocker 时为 null。

上例是先调用 park，然后调用 unpark，现在修改程序，先调用 unpark，然后调用 park，看能不能正确同步。具体代码如下

```
before park
before unpark
Blocker info ParkAndUnparkDemo
after park
Blocker info null
after unpark
```

运行结果:

```
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before unpark");        
        // 释放许可
        LockSupport.unpark((Thread) object);
        System.out.println("after unpark");
    }
}

public class ParkAndUnparkDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        try {
            // 主线程睡眠3s
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
```

说明: 可以看到，在先调用 unpark，再调用 park 时，仍能够正确实现同步，不会造成由 wait/notify 调用顺序不当所引起的阻塞。因此 park/unpark 相比 wait/notify 更加的灵活。

#### 3.5.4. [#](#thread-sleep-、object-wait-、condition-await-、locksupport-park-的区别-重点) Thread.sleep()、Object.wait()、Condition.await()、LockSupport.park() 的区别? 重点

*   **Thread.sleep() 和 Object.wait() 的区别**

首先，我们先来看看 Thread.sleep() 和 Object.wait() 的区别，这是一个烂大街的题目了，大家应该都能说上来两点。

1.  Thread.sleep() 不会释放占有的锁，Object.wait() 会释放占有的锁；
2.  Thread.sleep() 必须传入时间，Object.wait() 可传可不传，不传表示一直阻塞下去；
3.  Thread.sleep() 到时间了会自动唤醒，然后继续执行；
4.  Object.wait() 不带时间的，需要另一个线程使用 Object.notify() 唤醒；
5.  Object.wait() 带时间的，假如没有被 notify，到时间了会自动唤醒，这时又分好两种情况，一是立即获取到了锁，线程自然会继续执行；二是没有立即获取锁，线程进入同步队列等待获取锁；

其实，他们俩最大的区别就是 Thread.sleep() 不会释放锁资源，Object.wait() 会释放锁资源。

*   **Object.wait() 和 Condition.await() 的区别**

Object.wait() 和 Condition.await() 的原理是基本一致的，不同的是 Condition.await() 底层是调用 LockSupport.park() 来实现阻塞当前线程的。

实际上，它在阻塞当前线程之前还干了两件事，一是把当前线程添加到条件队列中，二是 “完全” 释放锁，也就是让 state 状态变量变为 0，然后才是调用 LockSupport.park()阻塞当前线程。

*   **Thread.sleep() 和 LockSupport.park() 的区别** LockSupport.park() 还有几个兄弟方法——parkNanos()、parkUtil() 等，我们这里说的 park() 方法统称这一类方法。

1.  从功能上来说，Thread.sleep() 和 LockSupport.park() 方法类似，都是阻塞当前线程的执行，且都不会释放当前线程占有的锁资源；
2.  Thread.sleep() 没法从外部唤醒，只能自己醒过来；
3.  LockSupport.park() 方法可以被另一个线程调用 LockSupport.unpark() 方法唤醒；
4.  Thread.sleep() 方法声明上抛出了 InterruptedException 中断异常，所以调用者需要捕获这个异常或者再抛出；
5.  LockSupport.park() 方法不需要捕获中断异常；
6.  Thread.sleep() 本身就是一个 native 方法；
7.  LockSupport.park() 底层是调用的 Unsafe 的 native 方法；

*   **Object.wait() 和 LockSupport.park() 的区别**

二者都会阻塞当前线程的运行，他们有什么区别呢? 经过上面的分析相信你一定很清楚了，真的吗? 往下看！

1.  Object.wait() 方法需要在 synchronized 块中执行；
2.  LockSupport.park() 可以在任意地方执行；
3.  Object.wait() 方法声明抛出了中断异常，调用者需要捕获或者再抛出；
4.  LockSupport.park() 不需要捕获中断异常；
5.  Object.wait() 不带超时的，需要另一个线程执行 notify() 来唤醒，但不一定继续执行后续内容；
6.  LockSupport.park() 不带超时的，需要另一个线程执行 unpark() 来唤醒，一定会继续执行后续内容；

park()/unpark()底层的原理是 “二元信号量”，你可以把它相像成只有一个许可证的 Semaphore，只不过这个信号量在重复执行 unpark() 的时候也不会再增加许可证，最多只有一个许可证。

#### 3.5.5. [#](#如果在wait-之前执行了notify-会怎样) 如果在 wait() 之前执行了 notify() 会怎样?

如果当前的线程不是此对象锁的所有者，却调用该对象的 notify() 或 wait() 方法时抛出 IllegalMonitorStateException 异常；

如果当前线程是此对象锁的所有者，wait() 将一直阻塞，因为后续将没有其它 notify() 唤醒它。

#### 3.5.6. [#](#如果在park-之前执行了unpark-会怎样) 如果在 park() 之前执行了 unpark() 会怎样?

线程不会被阻塞，直接跳过 park()，继续执行后续内容

#### 3.5.7. [#](#什么是aqs-为什么它是核心) 什么是 AQS? 为什么它是核心?

AQS 是一个用来构建锁和同步器的框架，使用 AQS 能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的 ReentrantLock，Semaphore，其他的诸如 ReentrantReadWriteLock，SynchronousQueue，FutureTask 等等皆是基于 AQS 的。

AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

AbstractQueuedSynchronizer 类底层的数据结构是使用 **CLH(Craig,Landin,and Hagersten) 队列**是一个虚拟的双向队列 (虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)。AQS 是将每条请求共享资源的线程封装成一个 CLH 锁队列的一个结点(Node) 来实现锁的分配。其中 Sync queue，即同步队列，是双向链表，包括 head 结点和 tail 结点，head 结点主要用作后续的调度。而 Condition queue 不是必须的，其是一个单向链表，只有当使用 Condition 时，才会存在此单向链表。并且可能会有多个 Condition queue。

![](https://pdai.tech/images/thread/java-thread-x-juc-aqs-1.png)

#### 3.5.8. [#](#aqs的核心思想是什么) AQS 的核心思想是什么?

底层数据结构: AQS 核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 AQS 是用 CLH 队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

#### 3.5.9. [#](#aqs有哪些核心的方法) AQS 有哪些核心的方法?

```
before unpark
after unpark
before park
after park
```

#### 3.5.10. [#](#aqs定义什么样的资源获取方式) AQS 定义什么样的资源获取方式?

AQS 定义了两种资源获取方式：

*   **独占** (只有一个线程能访问执行，又根据是否按队列的顺序分为**公平锁**和**非公平锁**，如`ReentrantLock`)
*   **共享** (多个线程可同时访问执行，如`Semaphore`、`CountDownLatch`、 `CyclicBarrier` )。`ReentrantReadWriteLock`可以看成是组合式，允许多个线程同时对某一资源进行读。

#### 3.5.11. [#](#aqs底层使用了什么样的设计模式) AQS 底层使用了什么样的设计模式?

模板， 共享锁和独占锁在一个接口类中。

*   [JUC 锁: ReentrantLock 详解](https://pdai.tech/md/java/thread/java-thread-x-lock-ReentrantLock.html)

#### 3.5.12. [#](#什么是可重入-什么是可重入锁-它用来解决什么问题) 什么是可重入，什么是可重入锁? 它用来解决什么问题?

**可重入**：（来源于维基百科）若一个程序或子程序可以 “在任意时刻被中断然后操作系统调度执行另外一段代码，这段代码又调用了该子程序不会出错”，则称其为可重入（reentrant 或 re-entrant）的。即当该子程序正在运行时，执行线程可以再次进入并执行它，仍然获得符合设计时预期的结果。与多线程并发执行的线程安全不同，可重入强调对单个线程执行时重新进入同一个子程序仍然是安全的。

**可重入锁**：又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者 class），不会因为之前已经获取过还没释放而阻塞。

#### 3.5.13. [#](#reentrantlock的核心是aqs-那么它怎么来实现的-继承吗) ReentrantLock 的核心是 AQS，那么它怎么来实现的，继承吗?

ReentrantLock 总共有三个内部类，并且三个内部类是紧密相关的，下面先看三个类的关系。

![](https://pdai.tech/images/thread/java-thread-x-juc-reentrantlock-1.png)

说明: ReentrantLock 类内部总共存在 Sync、NonfairSync、FairSync 三个类，NonfairSync 与 FairSync 类继承自 Sync 类，Sync 类继承自 AbstractQueuedSynchronizer 抽象类。下面逐个进行分析。

#### 3.5.14. [#](#reentrantlock是如何实现公平锁的) ReentrantLock 是如何实现公平锁的?

FairSync

#### 3.5.15. [#](#reentrantlock是如何实现非公平锁的) ReentrantLock 是如何实现非公平锁的?

UnFairSync

#### 3.5.16. [#](#reentrantlock默认实现的是公平还是非公平锁) ReentrantLock 默认实现的是公平还是非公平锁?

非公平锁

#### 3.5.17. [#](#为了有了reentrantlock还需要reentrantreadwritelock) 为了有了 ReentrantLock 还需要 ReentrantReadWriteLock?

读锁和写锁分离：ReentrantReadWriteLock 表示可重入读写锁，ReentrantReadWriteLock 中包含了两种锁，读锁 ReadLock 和写锁 WriteLock，可以通过这两种锁实现线程间的同步。

#### 3.5.18. [#](#reentrantreadwritelock底层实现原理) ReentrantReadWriteLock 底层实现原理?

ReentrantReadWriteLock 有五个内部类，五个内部类之间也是相互关联的。内部类的关系如下图所示。

![](https://pdai.tech/images/thread/java-thread-x-readwritelock-1.png)

说明: 如上图所示，Sync 继承自 AQS、NonfairSync 继承自 Sync 类、FairSync 继承自 Sync 类；ReadLock 实现了 Lock 接口、WriteLock 也实现了 Lock 接口。

#### 3.5.19. [#](#reentrantreadwritelock底层读写状态如何设计的) ReentrantReadWriteLock 底层读写状态如何设计的?

高 16 位为读锁，低 16 位为写锁

#### 3.5.20. [#](#读锁和写锁的最大数量是多少) 读锁和写锁的最大数量是多少?

2 的 16 次方 - 1

#### 3.5.21. [#](#本地线程计数器threadlocalholdcounter是用来做什么的) 本地线程计数器 ThreadLocalHoldCounter 是用来做什么的?

本地线程计数器，与对象绑定（线程 -》线程重入的次数）

#### 3.5.22. [#](#写锁的获取与释放是怎么实现的) 写锁的获取与释放是怎么实现的?

tryAcquire/tryRelease

#### 3.5.23. [#](#读锁的获取与释放是怎么实现的) 读锁的获取与释放是怎么实现的?

tryAcquireShared/tryReleaseShared

#### 3.5.24. [#](#什么是锁的升降级) 什么是锁的升降级?

RentrantReadWriteLock 为什么不支持锁升级? RentrantReadWriteLock 不支持锁升级 (把持读锁、获取写锁，最后释放读锁的过程)。目的也是保证数据可见性，如果读锁已被多个线程获取，其中任意线程成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见的。

### 3.6. [#](#_3-6-juc集合类) 3.6 JUC 集合类

*   [JUC 集合: ConcurrentHashMap 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-collection-ConcurrentHashMap.html)
*   [JUC 集合: CopyOnWriteArrayList 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-collection-CopyOnWriteArrayList.html)
*   [JUC 集合: ConcurrentLinkedQueue 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-collection-ConcurrentLinkedQueue.html)
*   [JUC 集合: BlockingQueue 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-collection-BlockingQueue.html)

#### 3.6.1. [#](#为什么hashtable慢-它的并发度是什么-那么concurrenthashmap并发度是什么) 为什么 HashTable 慢? 它的并发度是什么? 那么 ConcurrentHashMap 并发度是什么?

Hashtable 之所以效率低下主要是因为其实现使用了 synchronized 关键字对 put 等操作进行加锁，而 synchronized 关键字加锁是对整个对象进行加锁，也就是说在进行 put 等修改 Hash 表的操作时，锁住了整个 Hash 表，从而使得其表现的效率低下。

#### 3.6.2. [#](#concurrenthashmap在jdk1-7和jdk1-8中实现有什么差别-jdk1-8解決了jdk1-7中什么问题) ConcurrentHashMap 在 JDK1.7 和 JDK1.8 中实现有什么差别? JDK1.8 解決了 JDK1.7 中什么问题

*   `HashTable` : 使用了 synchronized 关键字对 put 等操作进行加锁;
*   `ConcurrentHashMap JDK1.7`: 使用分段锁机制实现;
*   `ConcurrentHashMap JDK1.8`: 则使用数组 + 链表 + 红黑树数据结构和 CAS 原子操作实现;

#### 3.6.3. [#](#concurrenthashmap-jdk1-7实现的原理是什么) ConcurrentHashMap JDK1.7 实现的原理是什么?

在 JDK1.5~1.7 版本，Java 使用了分段锁机制实现 ConcurrentHashMap.

简而言之，ConcurrentHashMap 在对象中保存了一个 Segment 数组，即将整个 Hash 表划分为多个分段；而每个 Segment 元素，它通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全；这样，在执行 put 操作时首先根据 hash 算法定位到元素属于哪个 Segment，然后对该 Segment 加锁即可。因此，ConcurrentHashMap 在多线程并发编程中可是实现多线程 put 操作。

![](https://pdai.tech/images/thread/java-thread-x-concurrent-hashmap-1.png)

`concurrencyLevel`: Segment 数（并行级别、并发数）。默认是 16，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。

#### 3.6.4. [#](#concurrenthashmap-jdk1-7中segment数-concurrencylevel-默认值是多少-为何一旦初始化就不可再扩容) ConcurrentHashMap JDK1.7 中 Segment 数 (concurrencyLevel) 默认值是多少? 为何一旦初始化就不可再扩容?

默认是 16

#### 3.6.5. [#](#concurrenthashmap-jdk1-7说说其put的机制) ConcurrentHashMap JDK1.7 说说其 put 的机制?

整体流程还是比较简单的，由于有独占锁的保护，所以 segment 内部的操作并不复杂

1.  计算 key 的 hash 值
2.  根据 hash 值找到 Segment 数组中的位置 j； ensureSegment(j) 对 segment[j] 进行初始化（Segment 内部是由 **数组 + 链表** 组成的）
3.  插入新值到 槽 s 中

#### 3.6.6. [#](#concurrenthashmap-jdk1-7是如何扩容的) ConcurrentHashMap JDK1.7 是如何扩容的?

rehash(注：segment 数组不能扩容，扩容是 segment 数组某个位置内部的数组 HashEntry<K,V>[] 进行扩容)

#### 3.6.7. [#](#concurrenthashmap-jdk1-8实现的原理是什么) ConcurrentHashMap JDK1.8 实现的原理是什么?

在 JDK1.7 之前，ConcurrentHashMap 是通过分段锁机制来实现的，所以其最大并发度受 Segment 的个数限制。因此，在 JDK1.8 中，ConcurrentHashMap 的实现原理摒弃了这种设计，而是选择了与 HashMap 类似的数组 + 链表 + 红黑树的方式实现，而加锁则采用 CAS 和 synchronized 实现。

简而言之：数组 + 链表 + 红黑树，CAS

#### 3.6.8. [#](#concurrenthashmap-jdk1-8是如何扩容的) ConcurrentHashMap JDK1.8 是如何扩容的?

tryPresize, 扩容也是做翻倍扩容的，扩容后数组容量为原来的 2 倍

#### 3.6.9. [#](#concurrenthashmap-jdk1-8链表转红黑树的时机是什么-临界值为什么是8) ConcurrentHashMap JDK1.8 链表转红黑树的时机是什么? 临界值为什么是 8?

size = 8, log(N)

#### 3.6.10. [#](#concurrenthashmap-jdk1-8是如何进行数据迁移的) ConcurrentHashMap JDK1.8 是如何进行数据迁移的?

transfer, 将原来的 tab 数组的元素迁移到新的 nextTab 数组中

#### 3.6.11. [#](#先说说非并发集合中fail-fast机制) 先说说非并发集合中 Fail-fast 机制?

快速失败

#### 3.6.12. [#](#copyonwritearraylist的实现原理) CopyOnWriteArrayList 的实现原理?

COW 基于拷贝

```
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

属性中有一个可重入锁，用来保证线程安全访问，还有一个 Object 类型的数组，用来存放具体的元素。当然，也使用到了反射机制和 CAS 来保证原子性的修改 lock 域。

```
// 将toCopyIn转化为Object[]类型数组，然后设置当前数组
  setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
```

#### 3.6.13. [#](#弱一致性的迭代器原理是怎么样的) 弱一致性的迭代器原理是怎么样的?

`COWIterator<E>`

COWIterator 表示迭代器，其也有一个 Object 类型的数组作为 CopyOnWriteArrayList 数组的快照，这种快照风格的迭代器方法在创建迭代器时使用了对当时数组状态的引用。此数组在迭代器的生存期内不会更改，因此不可能发生冲突，并且迭代器保证不会抛出 ConcurrentModificationException。创建迭代器以后，迭代器就不会反映列表的添加、移除或者更改。在迭代器上进行的元素更改操作 (remove、set 和 add) 不受支持。这些方法将抛出 UnsupportedOperationException。

#### 3.6.14. [#](#copyonwritearraylist为什么并发安全且性能比vector好) CopyOnWriteArrayList 为什么并发安全且性能比 Vector 好?

Vector 对单独的 add，remove 等方法都是在方法上加了 synchronized; 并且如果一个线程 A 调用 size 时，另一个线程 B 执行了 remove，然后 size 的值就不是最新的，然后线程 A 调用 remove 就会越界 (这时就需要再加一个 Synchronized)。这样就导致有了双重锁，效率大大降低，何必呢。于是 vector 废弃了，要用就用 CopyOnWriteArrayList 吧。

#### 3.6.15. [#](#copyonwritearraylist有何缺陷-说说其应用场景) CopyOnWriteArrayList 有何缺陷，说说其应用场景?

CopyOnWriteArrayList 有几个缺点：

*   由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致 young gc 或者 full gc
    
*   不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个 set 操作后，读取到数据可能还是旧的, 虽然 CopyOnWriteArrayList 能做到最终一致性, 但是还是没法满足实时性要求；

**CopyOnWriteArrayList 合适读多写少的场景，不过这类慎用**

因为谁也没法保证 CopyOnWriteArrayList 到底要放置多少数据，万一数据稍微有点多，每次 add/set 都要重新复制数组，这个代价实在太高昂了。在高性能的互联网应用中，这种操作分分钟引起故障。

#### 3.6.16. [#](#要想用线程安全的队列有哪些选择) 要想用线程安全的队列有哪些选择?

Vector，`Collections.synchronizedList( List<T> list)`, ConcurrentLinkedQueue 等

#### 3.6.17. [#](#concurrentlinkedqueue实现的数据结构) ConcurrentLinkedQueue 实现的数据结构?

ConcurrentLinkedQueue 的数据结构与 LinkedBlockingQueue 的数据结构相同，都是使用的链表结构。ConcurrentLinkedQueue 的数据结构如下:

![](https://pdai.tech/images/thread/java-thread-x-juc-concurrentlinkedqueue-1.png)

说明: ConcurrentLinkedQueue 采用的链表结构，并且包含有一个头节点和一个尾结点。

#### 3.6.18. [#](#concurrentlinkedqueue底层原理) ConcurrentLinkedQueue 底层原理?

```
// 可重入锁
final transient ReentrantLock lock = new ReentrantLock();
// 对象数组，用于存放元素
private transient volatile Object[] array;
// 反射机制
private static final sun.misc.Unsafe UNSAFE;
// lock域的内存偏移量
private static final long lockOffset;
```

说明: 属性中包含了 head 域和 tail 域，表示链表的头节点和尾结点，同时，ConcurrentLinkedQueue 也使用了反射机制和 CAS 机制来更新头节点和尾结点，保证原子性。

#### 3.6.19. [#](#concurrentlinkedqueue的核心方法有哪些) ConcurrentLinkedQueue 的核心方法有哪些?

offer()，poll()，peek()，isEmpty() 等队列常用方法

#### 3.6.20. [#](#说说concurrentlinkedqueue的hops-延迟更新的策略-的设计) 说说 ConcurrentLinkedQueue 的 HOPS(延迟更新的策略) 的设计?

通过上面对 offer 和 poll 方法的分析，我们发现 tail 和 head 是延迟更新的，两者更新触发时机为：

*   **tail 更新触发时机**：当 tail 指向的节点的下一个节点不为 null 的时候，会执行定位队列真正的队尾节点的操作，找到队尾节点后完成插入之后才会通过 casTail 进行 tail 更新；当 tail 指向的节点的下一个节点为 null 的时候，只插入节点不更新 tail。
    
*   **head 更新触发时机**：当 head 指向的节点的 item 域为 null 的时候，会执行定位队列真正的队头节点的操作，找到队头节点后完成删除之后才会通过 updateHead 进行 head 更新；当 head 指向的节点的 item 域不为 null 的时候，只删除节点不更新 head。

并且在更新操作时，源码中会有注释为：`hop two nodes at a time`。所以这种延迟更新的策略就被叫做 HOPS 的大概原因是这个 (猜的 😃)，从上面更新时的状态图可以看出，head 和 tail 的更新是“跳着的” 即中间总是间隔了一个。那么这样设计的意图是什么呢?

如果让 tail 永远作为队列的队尾节点，实现的代码量会更少，而且逻辑更易懂。但是，这样做有一个缺点，如果大量的入队操作，每次都要执行 CAS 进行 tail 的更新，汇总起来对性能也会是大大的损耗。如果能减少 CAS 更新的操作，无疑可以大大提升入队的操作效率，所以 doug lea 大师每间隔 1 次 (tail 和队尾节点的距离为 1) 进行才利用 CAS 更新 tail。对 head 的更新也是同样的道理，虽然，这样设计会多出在循环中定位队尾节点，但总体来说读的操作效率要远远高于写的性能，因此，多出来的在循环中定位尾节点的操作的性能损耗相对而言是很小的。

#### 3.6.21. [#](#concurrentlinkedqueue适合什么样的使用场景) ConcurrentLinkedQueue 适合什么样的使用场景?

ConcurrentLinkedQueue 通过无锁来做到了更高的并发量，是个高性能的队列，但是使用场景相对不如阻塞队列常见，毕竟取数据也要不停的去循环，不如阻塞的逻辑好设计，但是在并发量特别大的情况下，是个不错的选择，性能上好很多，而且这个队列的设计也是特别费力，尤其的使用的改良算法和对哨兵的处理。整体的思路都是比较严谨的，这个也是使用了无锁造成的，我们自己使用无锁的条件的话，这个队列是个不错的参考。

#### 3.6.22. [#](#什么是blockingdeque-适合用在什么样的场景) 什么是 BlockingDeque? 适合用在什么样的场景?

BlockingQueue 通常用于一个线程生产对象，而另外一个线程消费这些对象的场景。下图是对这个原理的阐述:

![](https://pdai.tech/images/thread/java-thread-x-blocking-queue-1.png)

一个线程往里边放，另外一个线程从里边取的一个 BlockingQueue。

一个线程将会持续生产新对象并将其插入到队列之中，直到队列达到它所能容纳的临界点。也就是说，它是有限的。如果该阻塞队列到达了其临界点，负责生产的线程将会在往里边插入新对象时发生阻塞。它会一直处于阻塞之中，直到负责消费的线程从队列中拿走一个对象。 负责消费的线程将会一直从该阻塞队列中拿出对象。如果消费线程尝试去从一个空的队列中提取对象的话，这个消费线程将会处于阻塞之中，直到一个生产线程把一个对象丢进队列。

#### 3.6.23. [#](#blockingqueue大家族有哪些) BlockingQueue 大家族有哪些?

ArrayBlockingQueue, DelayQueue, LinkedBlockingQueue, SynchronousQueue...

#### 3.6.24. [#](#blockingqueue常用的方法) BlockingQueue 常用的方法?

BlockingQueue 具有 4 组不同的方法用于插入、移除以及对队列中的元素进行检查。如果请求的操作不能得到立即执行的话，每个方法的表现也不同。这些方法如下:

<table><thead><tr><th></th><th>抛异常</th><th>特定值</th><th>阻塞</th><th>超时</th></tr></thead><tbody><tr><td>插入</td><td>add(o)</td><td>offer(o)</td><td>put(o)</td><td>offer(o, timeout, timeunit)</td></tr><tr><td>移除</td><td>remove()</td><td>poll()</td><td>take()</td><td>poll(timeout, timeunit)</td></tr><tr><td>检查</td><td>element()</td><td>peek()</td><td></td><td></td></tr></tbody></table>

四组不同的行为方式解释:

*   抛异常: 如果试图的操作无法立即执行，抛一个异常。
*   特定值: 如果试图的操作无法立即执行，返回一个特定的值 (常常是 true / false)。
*   阻塞: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。
*   超时: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功 (典型的是 true / false)。

#### 3.6.25. [#](#blockingqueue-实现例子) BlockingQueue 实现例子?

这里是一个 Java 中使用 BlockingQueue 的示例。本示例使用的是 BlockingQueue 接口的 ArrayBlockingQueue 实现。 首先，BlockingQueueExample 类分别在两个独立的线程中启动了一个 Producer 和 一个 Consumer。Producer 向一个共享的 BlockingQueue 中注入字符串，而 Consumer 则会从中把它们拿出来。

```
// 反射机制
private static final sun.misc.Unsafe UNSAFE;
// head域的偏移量
private static final long headOffset;
// tail域的偏移量
private static final long tailOffset;
```

以下是 Producer 类。注意它在每次 put() 调用时是如何休眠一秒钟的。这将导致 Consumer 在等待队列中对象的时候发生阻塞。

```
public class BlockingQueueExample {
 
    public static void main(String[] args) throws Exception {
 
        BlockingQueue queue = new ArrayBlockingQueue(1024);
 
        Producer producer = new Producer(queue);
        Consumer consumer = new Consumer(queue);
 
        new Thread(producer).start();
        new Thread(consumer).start();
 
        Thread.sleep(4000);
    }
}
```

以下是 Consumer 类。它只是把对象从队列中抽取出来，然后将它们打印到 System.out。

```
public class Producer implements Runnable{
 
    protected BlockingQueue queue = null;
 
    public Producer(BlockingQueue queue) {
        this.queue = queue;
    }
 
    public void run() {
        try {
            queue.put("1");
            Thread.sleep(1000);
            queue.put("2");
            Thread.sleep(1000);
            queue.put("3");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

#### 3.6.26. [#](#什么是blockingdeque-适合用在什么样的场景-1) 什么是 BlockingDeque? 适合用在什么样的场景?

java.util.concurrent 包里的 BlockingDeque 接口表示一个线程安放入和提取实例的双端队列。

BlockingDeque 类是一个双端队列，在不能够插入元素时，它将阻塞住试图插入元素的线程；在不能够抽取元素时，它将阻塞住试图抽取的线程。 deque(双端队列) 是 "Double Ended Queue" 的缩写。因此，双端队列是一个你可以从任意一端插入或者抽取元素的队列。

在线程既是一个队列的生产者又是这个队列的消费者的时候可以使用到 BlockingDeque。如果生产者线程需要在队列的两端都可以插入数据，消费者线程需要在队列的两端都可以移除数据，这个时候也可以使用 BlockingDeque。BlockingDeque 图解:

![](https://pdai.tech/images/thread/java-thread-x-blocking-deque-1.png)

#### 3.6.27. [#](#blockingdeque-与blockingqueue有何关系-请对比下它们的方法) BlockingDeque 与 BlockingQueue 有何关系，请对比下它们的方法?

BlockingDeque 接口继承自 BlockingQueue 接口。这就意味着你可以像使用一个 BlockingQueue 那样使用 BlockingDeque。如果你这么干的话，各种插入方法将会把新元素添加到双端队列的尾端，而移除方法将会把双端队列的首端的元素移除。正如 BlockingQueue 接口的插入和移除方法一样。

以下是 BlockingDeque 对 BlockingQueue 接口的方法的具体内部实现:

<table><thead><tr><th>BlockingQueue</th><th>BlockingDeque</th></tr></thead><tbody><tr><td>add()</td><td>addLast()</td></tr><tr><td>offer() x 2</td><td>offerLast() x 2</td></tr><tr><td>put()</td><td>putLast()</td></tr><tr><td>remove()</td><td>removeFirst()</td></tr><tr><td>poll() x 2</td><td>pollFirst()</td></tr><tr><td>take()</td><td>takeFirst()</td></tr><tr><td>element()</td><td>getFirst()</td></tr><tr><td>peek()</td><td>peekFirst()</td></tr></tbody></table>

#### 3.6.28. [#](#blockingdeque大家族有哪些) BlockingDeque 大家族有哪些?

LinkedBlockingDeque 是一个双端队列，在它为空的时候，一个试图从中抽取数据的线程将会阻塞，无论该线程是试图从哪一端抽取数据。

#### 3.6.29. [#](#blockingdeque-实现例子) BlockingDeque 实现例子?

既然 BlockingDeque 是一个接口，那么你想要使用它的话就得使用它的众多的实现类的其中一个。java.util.concurrent 包提供了以下 BlockingDeque 接口的实现类: LinkedBlockingDeque。

以下是如何使用 BlockingDeque 方法的一个简短代码示例:

```
public class Consumer implements Runnable{
 
    protected BlockingQueue queue = null;
 
    public Consumer(BlockingQueue queue) {
        this.queue = queue;
    }
 
    public void run() {
        try {
            System.out.println(queue.take());
            System.out.println(queue.take());
            System.out.println(queue.take());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 3.7. [#](#_3-7-juc线程池) 3.7 JUC 线程池

*   [JUC 线程池: FutureTask 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-executor-FutureTask.html)
*   [JUC 线程池: ThreadPoolExecutor 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-executor-ThreadPoolExecutor.html)
*   [JUC 线程池: ScheduledThreadPool 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-executor-ScheduledThreadPoolExecutor.html)
*   [JUC 线程池: Fork/Join 框架详解](https://pdai.tech/md/java/thread/java-thread-x-juc-executor-ForkJoinPool.html)

#### 3.7.1. [#](#futuretask用来解决什么问题的-为什么会出现) FutureTask 用来解决什么问题的? 为什么会出现?

FutureTask 为 Future 提供了基础实现，如获取任务执行结果 (get) 和取消任务 (cancel) 等。如果任务尚未完成，获取任务执行结果时将会阻塞。一旦执行结束，任务就不能被重启或取消(除非使用 runAndReset 执行计算)。FutureTask 常用来封装 Callable 和 Runnable，也可以作为一个任务提交到线程池中执行。除了作为一个独立的类之外，此类也提供了一些功能性函数供我们创建自定义 task 类使用。FutureTask 的线程安全由 CAS 来保证。

#### 3.7.2. [#](#futuretask类结构关系怎么样的) FutureTask 类结构关系怎么样的?

![](https://pdai.tech/images/thread/java-thread-x-juc-futuretask-1.png)

可以看到, FutureTask 实现了 RunnableFuture 接口，则 RunnableFuture 接口继承了 Runnable 接口和 Future 接口，所以 FutureTask 既能当做一个 Runnable 直接被 Thread 执行，也能作为 Future 用来得到 Callable 的计算结果。

#### 3.7.3. [#](#futuretask的线程安全是由什么保证的) FutureTask 的线程安全是由什么保证的?

FutureTask 的线程安全由 CAS 来保证。

#### 3.7.4. [#](#futuretask通常会怎么用-举例说明。) FutureTask 通常会怎么用? 举例说明。

```
BlockingDeque<String> deque = new LinkedBlockingDeque<String>();
deque.addFirst("1");
deque.addLast("2");
 
String two = deque.takeLast();
String one = deque.takeFirst();
```

#### 3.7.5. [#](#为什么要有线程池) 为什么要有线程池?

线程池能够对线程进行统一分配，调优和监控:

*   降低资源消耗 (线程无限制地创建，然后使用完毕后销毁)
*   提高响应速度 (无须创建线程)
*   提高线程的可管理性

#### 3.7.6. [#](#java是实现和管理线程池有哪些方式-请简单举例如何使用。) Java 是实现和管理线程池有哪些方式? 请简单举例如何使用。

从 JDK 5 开始，把工作单元与执行机制分离开来，工作单元包括 Runnable 和 Callable，而执行机制由 Executor 框架提供。

*   WorkerThread

```
import java.util.concurrent.*;
 
public class CallDemo {
 
    public static void main(String[] args) throws ExecutionException, InterruptedException {
 
        /**
         * 第一种方式:Future + ExecutorService
         * Task task = new Task();
         * ExecutorService service = Executors.newCachedThreadPool();
         * Future<Integer> future = service.submit(task1);
         * service.shutdown();
         */
 
 
        /**
         * 第二种方式: FutureTask + ExecutorService
         * ExecutorService executor = Executors.newCachedThreadPool();
         * Task task = new Task();
         * FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
         * executor.submit(futureTask);
         * executor.shutdown();
         */
 
        /**
         * 第三种方式:FutureTask + Thread
         */
 
        // 2. 新建FutureTask,需要一个实现了Callable接口的类的实例作为构造函数参数
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Task());
        // 3. 新建Thread对象并启动
        Thread thread = new Thread(futureTask);
        thread.setName("Task thread");
        thread.start();
 
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
 
        System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
 
        // 4. 调用isDone()判断任务是否结束
        if(!futureTask.isDone()) {
            System.out.println("Task is not done");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        int result = 0;
        try {
            // 5. 调用get()方法获取任务结果,如果任务没有执行完成则阻塞等待
            result = futureTask.get();
        } catch (Exception e) {
            e.printStackTrace();
        }
 
        System.out.println("result is " + result);
 
    }
 
    // 1. 继承Callable接口,实现call()方法,泛型参数为要返回的类型
    static class Task  implements Callable<Integer> {
 
        @Override
        public Integer call() throws Exception {
            System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
            int result = 0;
            for(int i = 0; i < 100;++i) {
                result += i;
            }
 
            Thread.sleep(3000);
            return result;
        }
    }
}
```

*   SimpleThreadPool

```
public class WorkerThread implements Runnable {
     
    private String command;
     
    public WorkerThread(String s){
        this.command=s;
    }
 
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+" Start. Command = "+command);
        processCommand();
        System.out.println(Thread.currentThread().getName()+" End.");
    }
 
    private void processCommand() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
    @Override
    public String toString(){
        return this.command;
    }
}
```

程序中我们创建了固定大小为五个工作线程的线程池。然后分配给线程池十个工作，因为线程池大小为五，它将启动五个工作线程先处理五个工作，其他的工作则处于等待状态，一旦有工作完成，空闲下来工作线程就会捡取等待队列里的其他工作进行执行。

这里是以上程序的输出。

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
public class SimpleThreadPool {
 
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++) {
            Runnable worker = new WorkerThread("" + i);
            executor.execute(worker);
          }
        executor.shutdown(); // This will make the executor accept no new threads and finish all existing threads in the queue
        while (!executor.isTerminated()) { // Wait until all threads are finish,and also you can use "executor.awaitTermination();" to wait
        }
        System.out.println("Finished all threads");
    }

}
```

输出表明线程池中至始至终只有五个名为 "pool-1-thread-1" 到 "pool-1-thread-5" 的五个线程，这五个线程不随着工作的完成而消亡，会一直存在，并负责执行分配给线程池的任务，直到线程池消亡。

Executors 类提供了使用了 ThreadPoolExecutor 的简单的 ExecutorService 实现，但是 ThreadPoolExecutor 提供的功能远不止于此。我们可以在创建 ThreadPoolExecutor 实例时指定活动线程的数量，我们也可以限制线程池的大小并且创建我们自己的 RejectedExecutionHandler 实现来处理不能适应工作队列的工作。

这里是我们自定义的 RejectedExecutionHandler 接口的实现。

*   RejectedExecutionHandlerImpl.java

```
pool-1-thread-2 Start. Command = 1
pool-1-thread-4 Start. Command = 3
pool-1-thread-1 Start. Command = 0
pool-1-thread-3 Start. Command = 2
pool-1-thread-5 Start. Command = 4
pool-1-thread-4 End.
pool-1-thread-5 End.
pool-1-thread-1 End.
pool-1-thread-3 End.
pool-1-thread-3 Start. Command = 8
pool-1-thread-2 End.
pool-1-thread-2 Start. Command = 9
pool-1-thread-1 Start. Command = 7
pool-1-thread-5 Start. Command = 6
pool-1-thread-4 Start. Command = 5
pool-1-thread-2 End.
pool-1-thread-4 End.
pool-1-thread-3 End.
pool-1-thread-5 End.
pool-1-thread-1 End.
Finished all threads
```

ThreadPoolExecutor 提供了一些方法，我们可以使用这些方法来查询 executor 的当前状态，线程池大小，活动线程数量以及任务数量。因此我是用来一个监控线程在特定的时间间隔内打印 executor 信息。

*   MyMonitorThread.java

```
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadPoolExecutor;
 
public class RejectedExecutionHandlerImpl implements RejectedExecutionHandler {
 
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println(r.toString() + " is rejected");
    }
 
}
```

这里是使用 ThreadPoolExecutor 的线程池实现例子。

*   WorkerPool.java

```
import java.util.concurrent.ThreadPoolExecutor;
 
public class MyMonitorThread implements Runnable {
    private ThreadPoolExecutor executor;
     
    private int seconds;
     
    private boolean run=true;
 
    public MyMonitorThread(ThreadPoolExecutor executor, int delay) {
        this.executor = executor;
        this.seconds=delay;
    }
     
    public void shutdown(){
        this.run=false;
    }
 
    @Override
    public void run() {
        while(run){
                System.out.println(
                    String.format("[monitor] [%d/%d] Active: %d, Completed: %d, Task: %d, isShutdown: %s, isTerminated: %s",
                        this.executor.getPoolSize(),
                        this.executor.getCorePoolSize(),
                        this.executor.getActiveCount(),
                        this.executor.getCompletedTaskCount(),
                        this.executor.getTaskCount(),
                        this.executor.isShutdown(),
                        this.executor.isTerminated()));
                try {
                    Thread.sleep(seconds*1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        }
             
    }
}
```

注意在初始化 ThreadPoolExecutor 时，我们保持初始池大小为 2，最大池大小为 4 而工作队列大小为 2。因此如果已经有四个正在执行的任务而此时分配来更多任务的话，工作队列将仅仅保留他们 (新任务) 中的两个，其他的将会被 RejectedExecutionHandlerImpl 处理。

上面程序的输出可以证实以上观点。

```
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
 
public class WorkerPool {
 
    public static void main(String args[]) throws InterruptedException{
        //RejectedExecutionHandler implementation
        RejectedExecutionHandlerImpl rejectionHandler = new RejectedExecutionHandlerImpl();
        //Get the ThreadFactory implementation to use
        ThreadFactory threadFactory = Executors.defaultThreadFactory();
        //creating the ThreadPoolExecutor
        ThreadPoolExecutor executorPool = new ThreadPoolExecutor(2, 4, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2), threadFactory, rejectionHandler);
        //start the monitoring thread
        MyMonitorThread monitor = new MyMonitorThread(executorPool, 3);
        Thread monitorThread = new Thread(monitor);
        monitorThread.start();
        //submit work to the thread pool
        for(int i=0; i<10; i++){
            executorPool.execute(new WorkerThread("cmd"+i));
        }
         
        Thread.sleep(30000);
        //shut down the pool
        executorPool.shutdown();
        //shut down the monitor thread
        Thread.sleep(5000);
        monitor.shutdown();
         
    }
}
```

注意 executor 的活动任务、完成任务以及所有完成任务，这些数量上的变化。我们可以调用 shutdown() 方法来结束所有提交的任务并终止线程池。

#### 3.7.7. [#](#threadpoolexecutor的原理) ThreadPoolExecutor 的原理?

其实 java 线程池的实现原理很简单，说白了就是一个线程集合 workerSet 和一个阻塞队列 workQueue。当用户向线程池提交一个任务 (也就是线程) 时，线程池会先将任务放入 workQueue 中。workerSet 中的线程会不断的从 workQueue 中获取线程然后执行。当 workQueue 中没有任务的时候，worker 就会阻塞，直到队列中有任务了就取出来继续执行。

![](https://pdai.tech/images/thread/java-thread-x-executors-1.png)

当一个任务提交至线程池之后:

1.  线程池首先当前运行的线程数量是否少于 corePoolSize。如果是，则创建一个新的工作线程来执行任务。如果都在执行任务，则进入 2.
2.  判断 BlockingQueue 是否已经满了，倘若还没有满，则将线程放入 BlockingQueue。否则进入 3.
3.  如果创建一个新的工作线程将使当前运行的线程数量超过 maximumPoolSize，则交给 RejectedExecutionHandler 来处理任务。

当 ThreadPoolExecutor 创建新线程时，通过 CAS 来更新线程池的状态 ctl.

#### 3.7.8. [#](#threadpoolexecutor有哪些核心的配置参数-请简要说明) ThreadPoolExecutor 有哪些核心的配置参数? 请简要说明

```
pool-1-thread-1 Start. Command = cmd0
pool-1-thread-4 Start. Command = cmd5
cmd6 is rejected
pool-1-thread-3 Start. Command = cmd4
pool-1-thread-2 Start. Command = cmd1
cmd7 is rejected
cmd8 is rejected
cmd9 is rejected
[monitor] [0/2] Active: 4, Completed: 0, Task: 6, isShutdown: false, isTerminated: false
[monitor] [4/2] Active: 4, Completed: 0, Task: 6, isShutdown: false, isTerminated: false
pool-1-thread-4 End.
pool-1-thread-1 End.
pool-1-thread-2 End.
pool-1-thread-3 End.
pool-1-thread-1 Start. Command = cmd3
pool-1-thread-4 Start. Command = cmd2
[monitor] [4/2] Active: 2, Completed: 4, Task: 6, isShutdown: false, isTerminated: false
[monitor] [4/2] Active: 2, Completed: 4, Task: 6, isShutdown: false, isTerminated: false
pool-1-thread-1 End.
pool-1-thread-4 End.
[monitor] [4/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [0/2] Active: 0, Completed: 6, Task: 6, isShutdown: true, isTerminated: true
[monitor] [0/2] Active: 0, Completed: 6, Task: 6, isShutdown: true, isTerminated: true
```

*   `corePoolSize` 线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于 corePoolSize, 即使有其他空闲线程能够执行新来的任务, 也会继续创建线程；如果当前线程数为 corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的 prestartAllCoreThreads() 方法，线程池会提前创建并启动所有核心线程。
    
*   `workQueue` 用来保存等待被执行的任务的阻塞队列. 在 JDK 中提供了如下阻塞队列: 具体可以参考 [JUC 集合: BlockQueue 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-collection-BlockingQueue.html)
    
    *   `ArrayBlockingQueue`: 基于数组结构的有界阻塞队列，按 FIFO 排序任务；
    *   `LinkedBlockingQueue`: 基于链表结构的阻塞队列，按 FIFO 排序任务，吞吐量通常要高于 ArrayBlockingQueue；
    *   `SynchronousQueue`: 一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于 LinkedBlockingQueue；
    *   `PriorityBlockingQueue`: 具有优先级的无界阻塞队列；

`LinkedBlockingQueue`比`ArrayBlockingQueue`在插入删除节点性能方面更优，但是二者在`put()`, `take()`任务的时均需要加锁，`SynchronousQueue`使用无锁算法，根据节点的状态判断执行，而不需要用到锁，其核心是`Transfer.transfer()`.

*   `maximumPoolSize` 线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于 maximumPoolSize；当阻塞队列是无界队列, 则 maximumPoolSize 则不起作用, 因为无法提交至核心线程池的线程会一直持续地放入 workQueue.
    
*   `keepAliveTime` 线程空闲时的存活时间，即当线程没有任务执行时，该线程继续存活的时间；默认情况下，该参数只在线程数大于 corePoolSize 时才有用, 超过这个时间的空闲线程将被终止；
    
*   `unit` keepAliveTime 的单位
    
*   `threadFactory` 创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。默认为 DefaultThreadFactory
    
*   `handler` 线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了 4 种策略:
    
    *   `AbortPolicy`: 直接抛出异常，默认策略；
    *   `CallerRunsPolicy`: 用调用者所在的线程来执行任务；
    *   `DiscardOldestPolicy`: 丢弃阻塞队列中靠最前的任务，并执行当前任务；
    *   `DiscardPolicy`: 直接丢弃任务；

当然也可以根据应用场景实现 RejectedExecutionHandler 接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。

#### 3.7.9. [#](#threadpoolexecutor可以创建哪是哪三种线程池呢) ThreadPoolExecutor 可以创建哪是哪三种线程池呢?

*   newFixedThreadPool

```
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler)
```

线程池的线程数量达 corePoolSize 后，即使线程池没有可执行任务时，也不会释放线程。

FixedThreadPool 的工作队列为无界队列 LinkedBlockingQueue(队列容量为 Integer.MAX_VALUE), 这会导致以下问题:

*   线程池里的线程数量不超过 corePoolSize, 这导致了 maximumPoolSize 和 keepAliveTime 将会是个无用参数
*   由于使用了无界队列, 所以 FixedThreadPool 永远不会拒绝, 即饱和策略失效
*   newSingleThreadExecutor

```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>());
}
```

初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行.

由于使用了无界队列, 所以 SingleThreadPool 永远不会拒绝, 即饱和策略失效

*   newCachedThreadPool

```
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

线程池的线程数可达到 Integer.MAX_VALUE，即 2147483647，内部使用 SynchronousQueue 作为阻塞队列； 和 newFixedThreadPool 创建的线程池不同，newCachedThreadPool 在没有任务执行时，当线程的空闲时间超过 keepAliveTime，会自动释放线程资源，当提交新任务时，如果没有空闲线程，则创建新线程执行任务，会导致一定的系统开销； 执行过程与前两种稍微不同:

*   主线程调用 SynchronousQueue 的 offer() 方法放入 task, 倘若此时线程池中有空闲的线程尝试读取 SynchronousQueue 的 task, 即调用了 SynchronousQueue 的 poll(), 那么主线程将该 task 交给空闲线程. 否则执行 (2)
*   当线程池为空或者没有空闲的线程, 则创建新的线程执行任务.
*   执行完任务的线程倘若在 60s 内仍空闲, 则会被终止. 因此长时间空闲的 CachedThreadPool 不会持有任何线程资源.

#### 3.7.10. [#](#当队列满了并且worker的数量达到maxsize的时候-会怎么样) 当队列满了并且 worker 的数量达到 maxSize 的时候，会怎么样?

当队列满了并且 worker 的数量达到 maxSize 的时候, 执行具体的拒绝策略

```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
}
```

#### 3.7.11. [#](#说说threadpoolexecutor有哪些rejectedexecutionhandler策略-默认是什么策略) 说说 ThreadPoolExecutor 有哪些 RejectedExecutionHandler 策略? 默认是什么策略?

*   AbortPolicy, 默认

该策略是线程池的默认策略。使用该策略时，如果线程池队列满了丢掉这个任务并且抛出 RejectedExecutionException 异常。 源码如下：

```
private volatile RejectedExecutionHandler handler;
```

*   DiscardPolicy

这个策略和 AbortPolicy 的 slient 版本，如果线程池队列满了，会直接丢掉这个任务并且不会有任何异常。 源码如下：

```
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
  //不做任何处理，直接抛出异常
  throw new RejectedExecutionException("xxx");
}
```

*   DiscardOldestPolicy

这个策略从字面上也很好理解，丢弃最老的。也就是说如果队列满了，会将最早进入队列的任务删掉腾出空间，再尝试加入队列。 因为队列是队尾进，队头出，所以队头元素是最老的，因此每次都是移除对头元素后再尝试入队。 源码如下：

```
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    //就是一个空的方法
}
```

*   CallerRunsPolicy

使用此策略，如果添加到线程池失败，那么主线程会自己去执行该任务，不会等待线程池中的线程去执行。就像是个急脾气的人，我等不到别人来做这件事就干脆自己干。 源码如下：

```
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    if (!e.isShutdown()) {
        //移除队头元素
        e.getQueue().poll();
        //再尝试入队
        e.execute(r);
    }
}
```

#### 3.7.12. [#](#简要说下线程池的任务执行机制) 简要说下线程池的任务执行机制?

execute –> addWorker –>runworker (getTask)

1.  线程池的工作线程通过 Woker 类实现，在 ReentrantLock 锁的保证下，把 Woker 实例插入到 HashSet 后，并启动 Woker 中的线程。
2.  从 Woker 类的构造方法实现可以发现: 线程工厂在创建线程 thread 时，将 Woker 实例本身 this 作为参数传入，当执行 start 方法启动线程 thread 时，本质是执行了 Worker 的 runWorker 方法。
3.  firstTask 执行完成之后，通过 getTask 方法从阻塞队列中获取等待的任务，如果队列中没有任务，getTask 方法会被阻塞并挂起，不会占用 cpu 资源；

#### 3.7.13. [#](#线程池中任务是如何提交的) 线程池中任务是如何提交的?

![](https://pdai.tech/images/thread/java-thread-x-executors-3.png)

1.  submit 任务，等待线程池 execute
2.  执行 FutureTask 类的 get 方法时，会把主线程封装成 WaitNode 节点并保存在 waiters 链表中， 并阻塞等待运行结果；
3.  FutureTask 任务执行完成后，通过 UNSAFE 设置 waiters 相应的 waitNode 为 null，并通过 LockSupport 类 unpark 方法唤醒主线程；

```
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    if (!e.isShutdown()) {
        //直接执行run方法
        r.run();
    }
}
```

在实际业务场景中，Future 和 Callable 基本是成对出现的，Callable 负责产生结果，Future 负责获取结果。

1.  Callable 接口类似于 Runnable，只是 Runnable 没有返回值。
2.  Callable 任务除了返回正常结果之外，如果发生异常，该异常也会被返回，即 Future 可以拿到异步执行任务各种结果；
3.  Future.get 方法会导致主线程阻塞，直到 Callable 任务执行完成；

#### 3.7.14. [#](#线程池中任务是如何关闭的) 线程池中任务是如何关闭的?

*   shutdown

将线程池里的线程状态设置成 SHUTDOWN 状态, 然后中断所有没有正在执行任务的线程.

*   shutdownNow

将线程池里的线程状态设置成 STOP 状态, 然后停止所有正在执行或暂停任务的线程. 只要调用这两个关闭方法中的任意一个, isShutDown() 返回 true. 当所有任务都成功关闭了, isTerminated() 返回 true.

#### 3.7.15. [#](#在配置线程池的时候需要考虑哪些配置因素) 在配置线程池的时候需要考虑哪些配置因素?

从任务的优先级，任务的执行时间长短，任务的性质 (CPU 密集 / IO 密集)，任务的依赖关系这四个角度来分析。并且近可能地使用有界的工作队列。

性质不同的任务可用使用不同规模的线程池分开处理:

*   CPU 密集型: 尽可能少的线程，Ncpu+1
*   IO 密集型: 尽可能多的线程, Ncpu*2，比如数据库连接池
*   混合型: CPU 密集型的任务与 IO 密集型任务的执行时间差别较小，拆分为两个线程池；否则没有必要拆分。

#### 3.7.16. [#](#如何监控线程池的状态) 如何监控线程池的状态?

可以使用 ThreadPoolExecutor 以下方法:

*   `getTaskCount()` Returns the approximate total number of tasks that have ever been scheduled for execution.
*   `getCompletedTaskCount()` Returns the approximate total number of tasks that have completed execution. 返回结果少于 getTaskCount()。
*   `getLargestPoolSize()` Returns the largest number of threads that have ever simultaneously been in the pool. 返回结果小于等于 maximumPoolSize
*   `getPoolSize()` Returns the current number of threads in the pool.
*   `getActiveCount()` Returns the approximate number of threads that are actively executing tasks.

#### 3.7.17. [#](#为什么很多公司不允许使用executors去创建线程池-那么推荐怎么使用呢) 为什么很多公司不允许使用 Executors 去创建线程池? 那么推荐怎么使用呢?

线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：Executors 各个方法的弊端：

*   newFixedThreadPool 和 newSingleThreadExecutor:   主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至 OOM。
    
*   newCachedThreadPool 和 newScheduledThreadPool:   主要问题是线程数最大数是 Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至 OOM。
    
*   推荐方式 1 首先引入：commons-lang3 包

```
public class Test{
    public static void main(String[] args) {

        ExecutorService es = Executors.newCachedThreadPool();
        Future<String> future = es.submit(new Callable<String>() {
            @Override
            public String call() throws Exception {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return "future result";
            }
        });
        try {
            String result = future.get();
            System.out.println(result);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

*   推荐方式 2 首先引入：com.google.guava 包

```
ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1,
        new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build());
```

*   推荐方式 3 spring 配置线程池方式：自定义线程工厂 bean 需要实现 ThreadFactory，可参考该接口的其它默认实现类，使用方式直接注入 bean 调用 execute(Runnable task) 方法即可

```
ThreadFactory namedThreadFactory = new ThreadFactoryBuilder().setNameFormat("demo-pool-%d").build();

//Common Thread Pool
ExecutorService pool = new ThreadPoolExecutor(5, 200, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());

// excute
pool.execute(()-> System.out.println(Thread.currentThread().getName()));

 //gracefully shutdown
pool.shutdown();
```

#### 3.7.18. [#](#scheduledthreadpoolexecutor要解决什么样的问题) ScheduledThreadPoolExecutor 要解决什么样的问题?

在很多业务场景中，我们可能需要周期性的运行某项任务来获取结果，比如周期数据统计，定时发送数据等。在并发包出现之前，Java 早在 1.3 就提供了 Timer 类 (只需要了解，目前已渐渐被 ScheduledThreadPoolExecutor 代替) 来适应这些业务场景。随着业务量的不断增大，我们可能需要多个工作线程运行任务来尽可能的增加产品性能，或者是需要更高的灵活性来控制和监控这些周期业务。这些都是 ScheduledThreadPoolExecutor 诞生的必然性。

#### 3.7.19. [#](#scheduledthreadpoolexecutor相比threadpoolexecutor有哪些特性) ScheduledThreadPoolExecutor 相比 ThreadPoolExecutor 有哪些特性?

ScheduledThreadPoolExecutor 继承自 ThreadPoolExecutor，为任务提供延迟或周期执行，属于线程池的一种。和 ThreadPoolExecutor 相比，它还具有以下几种特性:

*   使用专门的任务类型—ScheduledFutureTask 来执行周期任务，也可以接收不需要时间调度的任务 (这些任务通过 ExecutorService 来执行)。
*   使用专门的存储队列—DelayedWorkQueue 来存储任务，DelayedWorkQueue 是无界延迟队列 DelayQueue 的一种。相比 ThreadPoolExecutor 也简化了执行机制 (delayedExecute 方法，后面单独分析)。
*   支持可选的 run-after-shutdown 参数，在池被关闭 (shutdown) 之后支持可选的逻辑来决定是否继续运行周期或延迟任务。并且当任务 (重新) 提交操作与 shutdown 操作重叠时，复查逻辑也不相同。

#### 3.7.20. [#](#scheduledthreadpoolexecutor有什么样的数据结构-核心内部类和抽象类) ScheduledThreadPoolExecutor 有什么样的数据结构，核心内部类和抽象类?

![](https://pdai.tech/images/thread/java-thread-x-stpe-1.png)

ScheduledThreadPoolExecutor 继承自 `ThreadPoolExecutor`:

*   详情请参考: [JUC 线程池: ThreadPoolExecutor 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-executor-ThreadPoolExecutor.html)

ScheduledThreadPoolExecutor 内部构造了两个内部类 `ScheduledFutureTask` 和 `DelayedWorkQueue`:

*   `ScheduledFutureTask`: 继承了 FutureTask，说明是一个异步运算任务；最上层分别实现了 Runnable、Future、Delayed 接口，说明它是一个可以延迟执行的异步运算任务。
    
*   `DelayedWorkQueue`: 这是 ScheduledThreadPoolExecutor 为存储周期或延迟任务专门定义的一个延迟队列，继承了 AbstractQueue，为了契合 ThreadPoolExecutor 也实现了 BlockingQueue 接口。它内部只允许存储 RunnableScheduledFuture 类型的任务。与 DelayQueue 的不同之处就是它只允许存放 RunnableScheduledFuture 对象，并且自己实现了二叉堆 (DelayQueue 是利用了 PriorityQueue 的二叉堆结构)。

#### 3.7.21. [#](#scheduledthreadpoolexecutor有哪两个关闭策略-区别是什么) ScheduledThreadPoolExecutor 有哪两个关闭策略? 区别是什么?

**shutdown**: 在 shutdown 方法中调用的关闭钩子 onShutdown 方法，它的主要作用是在关闭线程池后取消并清除由于关闭策略不应该运行的所有任务，这里主要是根据 run-after-shutdown 参数 (continueExistingPeriodicTasksAfterShutdown 和 executeExistingDelayedTasksAfterShutdown) 来决定线程池关闭后是否关闭已经存在的任务。

**showDownNow**: 立即关闭

#### 3.7.22. [#](#scheduledthreadpoolexecutor中scheduleatfixedrate-和-schedulewithfixeddelay区别是什么) ScheduledThreadPoolExecutor 中 scheduleAtFixedRate 和 scheduleWithFixedDelay 区别是什么?

**注意 scheduleAtFixedRate 和 scheduleWithFixedDelay 的区别**: 乍一看两个方法一模一样，其实，在 unit.toNanos 这一行代码中还是有区别的。没错，scheduleAtFixedRate 传的是正值，而 scheduleWithFixedDelay 传的则是负值，这个值就是 ScheduledFutureTask 的 period 属性。

#### 3.7.23. [#](#为什么threadpoolexecutor-的调整策略却不适用于-scheduledthreadpoolexecutor) 为什么 ThreadPoolExecutor 的调整策略却不适用于 ScheduledThreadPoolExecutor?

例如: 由于 ScheduledThreadPoolExecutor 是一个固定核心线程数大小的线程池，并且使用了一个无界队列，所以调整 maximumPoolSize 对其没有任何影响 (所以 ScheduledThreadPoolExecutor 没有提供可以调整最大线程数的构造函数，默认最大线程数固定为 Integer.MAX_VALUE)。此外，设置 corePoolSize 为 0 或者设置核心线程空闲后清除(allowCoreThreadTimeOut) 同样也不是一个好的策略，因为一旦周期任务到达某一次运行周期时，可能导致线程池内没有线程去处理这些任务。

#### 3.7.24. [#](#executors-提供了几种方法来构造-scheduledthreadpoolexecutor) Executors 提供了几种方法来构造 ScheduledThreadPoolExecutor?

*   newScheduledThreadPool: 可指定核心线程数的线程池。
*   newSingleThreadScheduledExecutor: 只有一个工作线程的线程池。如果内部工作线程由于执行周期任务异常而被终止，则会新建一个线程替代它的位置。

#### 3.7.25. [#](#fork-join主要用来解决什么样的问题) Fork/Join 主要用来解决什么样的问题?

ForkJoinPool 是 JDK 7 加入的一个线程池类。Fork/Join 技术是分治算法 (Divide-and-Conquer) 的并行实现，它是一项可以获得良好的并行性能的简单且高效的设计技术。目的是为了帮助我们更好地利用多处理器带来的好处，使用所有可用的运算能力来提升应用的性能。

#### 3.7.26. [#](#fork-join框架是在哪个jdk版本中引入的) Fork/Join 框架是在哪个 JDK 版本中引入的?

JDK 7

#### 3.7.27. [#](#fork-join框架主要包含哪三个模块-模块之间的关系是怎么样的) Fork/Join 框架主要包含哪三个模块? 模块之间的关系是怎么样的?

Fork/Join 框架主要包含三个模块:

*   任务对象: `ForkJoinTask` (包括`RecursiveTask`、`RecursiveAction` 和 `CountedCompleter`)
*   执行 Fork/Join 任务的线程: `ForkJoinWorkerThread`
*   线程池: `ForkJoinPool`

这三者的关系是: ForkJoinPool 可以通过池中的 ForkJoinWorkerThread 来处理 ForkJoinTask 任务。

#### 3.7.28. [#](#forkjoinpool类继承关系) ForkJoinPool 类继承关系?

![](https://pdai.tech/images/thread/java-thread-x-forkjoin-1.png)

内部类介绍:

*   ForkJoinWorkerThreadFactory: 内部线程工厂接口，用于创建工作线程 ForkJoinWorkerThread
    
*   DefaultForkJoinWorkerThreadFactory: ForkJoinWorkerThreadFactory 的默认实现类
    
*   InnocuousForkJoinWorkerThreadFactory: 实现了 ForkJoinWorkerThreadFactory，无许可线程工厂，当系统变量中有系统安全管理相关属性时，默认使用这个工厂创建工作线程。
    
*   EmptyTask: 内部占位类，用于替换队列中 join 的任务。
    
*   ManagedBlocker: 为 ForkJoinPool 中的任务提供扩展管理并行数的接口，一般用在可能会阻塞的任务 (如在 Phaser 中用于等待 phase 到下一个 generation)。
    
*   WorkQueue: ForkJoinPool 的核心数据结构，本质上是 work-stealing 模式的双端任务队列，内部存放 ForkJoinTask 对象任务，使用 @Contented 注解修饰防止伪共享。
    
    *   工作线程在运行中产生新的任务 (通常是因为调用了 fork()) 时，此时可以把 WorkQueue 的数据结构视为一个栈，新的任务会放入栈顶(top 位)；工作线程在处理自己工作队列的任务时，按照 LIFO 的顺序。
    *   工作线程在处理自己的工作队列同时，会尝试窃取一个任务 (可能是来自于刚刚提交到 pool 的任务，或是来自于其他工作线程的队列任务)，此时可以把 WorkQueue 的数据结构视为一个 FIFO 的队列，窃取的任务位于其他线程的工作队列的队首 (base 位)。
*   伪共享状态: 缓存系统中是以缓存行 (cache line) 为单位存储的。缓存行是 2 的整数幂个连续字节，一般为 32-256 个字节。最常见的缓存行大小是 64 个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。

#### 3.7.29. [#](#forkjointask抽象类继承关系) ForkJoinTask 抽象类继承关系?

![](https://pdai.tech/images/thread/java-thread-x-forkjoin-4.png)

ForkJoinTask 实现了 Future 接口，说明它也是一个可取消的异步运算任务，实际上 ForkJoinTask 是 Future 的轻量级实现，主要用在纯粹是计算的函数式任务或者操作完全独立的对象计算任务。fork 是主运行方法，用于异步执行；而 join 方法在任务结果计算完毕之后才会运行，用来合并或返回计算结果。 其内部类都比较简单，ExceptionNode 是用于存储任务执行期间的异常信息的单向链表；其余四个类是为 Runnable/Callable 任务提供的适配器类，用于把 Runnable/Callable 转化为 ForkJoinTask 类型的任务 (因为 ForkJoinPool 只可以运行 ForkJoinTask 类型的任务)。

#### 3.7.30. [#](#整个fork-join-框架的执行流程-运行机制是怎么样的) 整个 Fork/Join 框架的执行流程 / 运行机制是怎么样的?

*   首先介绍任务的提交流程 - 外部任务 (external/submissions task) 提交
*   然后介绍任务的提交流程 - 子任务 (Worker task) 提交
*   再分析任务的执行过程 (ForkJoinWorkerThread.run() 到 ForkJoinTask.doExec()这一部分)；
*   最后介绍任务的结果获取 (ForkJoinTask.join() 和 ForkJoinTask.invoke())

#### 3.7.31. [#](#具体阐述fork-join的分治思想和work-stealing-实现方式) 具体阐述 Fork/Join 的分治思想和 work-stealing 实现方式?

*   分治算法 (Divide-and-Conquer)

分治算法 (Divide-and-Conquer) 把任务递归的拆分为各个子任务，这样可以更好的利用系统资源，尽可能的使用所有可用的计算能力来提升应用性能。首先看一下 Fork/Join 框架的任务运行机制:

![](https://pdai.tech/images/thread/java-thread-x-forkjoin-2.png)

*   work-stealing(工作窃取) 算法

work-stealing(工作窃取)算法: 线程池内的所有工作线程都尝试找到并执行已经提交的任务，或者是被其他活动任务创建的子任务 (如果不存在就阻塞等待)。这种特性使得 ForkJoinPool 在运行多个可以产生子任务的任务，或者是提交的许多小任务时效率更高。尤其是构建异步模型的 ForkJoinPool 时，对不需要合并(join) 的事件类型任务也非常适用。

在 ForkJoinPool 中，线程池中每个工作线程 (ForkJoinWorkerThread) 都对应一个任务队列(WorkQueue)，工作线程优先处理来自自身队列的任务(LIFO 或 FIFO 顺序，参数 mode 决定)，然后以 FIFO 的顺序随机窃取其他队列中的任务。

具体思路如下:

*   每个线程都有自己的一个 WorkQueue，该工作队列是一个双端队列。
*   队列支持三个功能 push、pop、poll
*   push/pop 只能被队列的所有者线程调用，而 poll 可以被其他线程调用。
*   划分的子任务调用 fork 时，都会被 push 到自己的队列中。
*   默认情况下，工作线程从自己的双端队列获出任务并执行。
*   当自己的队列为空时，线程随机从另一个线程的队列末尾调用 poll 方法窃取任务。

![](https://pdai.tech/images/thread/java-thread-x-forkjoin-3.png)

#### 3.7.32. [#](#有哪些jdk源码中使用了fork-join思想) 有哪些 JDK 源码中使用了 Fork/Join 思想?

我们常用的数组工具类 Arrays 在 JDK 8 之后新增的并行排序方法 (parallelSort) 就运用了 ForkJoinPool 的特性，还有 ConcurrentHashMap 在 JDK 8 之后添加的函数式方法 (如 forEach 等) 也有运用。

#### 3.7.33. [#](#如何使用executors工具类创建forkjoinpool) 如何使用 Executors 工具类创建 ForkJoinPool?

Java8 在 Executors 工具类中新增了两个工厂方法:

```
<bean id="userThreadPool" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
        <property name="corePoolSize" value="10" />
        <property name="maxPoolSize" value="100" />
        <property name="queueCapacity" value="2000" />

    <property name="threadFactory" value= threadFactory />
        <property name="rejectedExecutionHandler">
            <ref local="rejectedExecutionHandler" />
        </property>
    </bean>
    
    //in code
    userThreadPool.execute(thread);
```

#### 3.7.34. [#](#写一个例子-用forkjoin方式实现1-2-3-100000) 写一个例子: 用 ForkJoin 方式实现 1+2+3+...+100000?

```
// parallelism定义并行级别
public static ExecutorService newWorkStealingPool(int parallelism);
// 默认并行级别为JVM可用的处理器个数
// Runtime.getRuntime().availableProcessors()
public static ExecutorService newWorkStealingPool();
```

*   执行结果

```
public class Test {
	static final class SumTask extends RecursiveTask<Integer> {
		private static final long serialVersionUID = 1L;
		
		final int start; //开始计算的数
		final int end; //最后计算的数
		
		SumTask(int start, int end) {
			this.start = start;
			this.end = end;
		}

		@Override
		protected Integer compute() {
			//如果计算量小于1000，那么分配一个线程执行if中的代码块，并返回执行结果
			if(end - start < 1000) {
				System.out.println(Thread.currentThread().getName() + " 开始执行: " + start + "-" + end);
				int sum = 0;
				for(int i = start; i <= end; i++)
					sum += i;
				return sum;
			}
			//如果计算量大于1000，那么拆分为两个任务
			SumTask task1 = new SumTask(start, (start + end) / 2);
			SumTask task2 = new SumTask((start + end) / 2 + 1, end);
			//执行任务
			task1.fork();
			task2.fork();
			//获取任务执行的结果
			return task1.join() + task2.join();
		}
	}
	
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		ForkJoinPool pool = new ForkJoinPool();
		ForkJoinTask<Integer> task = new SumTask(1, 10000);
		pool.submit(task);
		System.out.println(task.get());
	}
}
```

#### 3.7.35. [#](#fork-join在使用时有哪些注意事项-结合jdk中的斐波那契数列实例具体说明。) Fork/Join 在使用时有哪些注意事项? 结合 JDK 中的斐波那契数列实例具体说明。

斐波那契数列: 1、1、2、3、5、8、13、21、34、…… 公式 : F(1)=1，F(2)=1, F(n)=F(n-1)+F(n-2)(n>=3，n∈N*)

```
ForkJoinPool-1-worker-1 开始执行: 1-625
ForkJoinPool-1-worker-7 开始执行: 6251-6875
ForkJoinPool-1-worker-6 开始执行: 5626-6250
ForkJoinPool-1-worker-10 开始执行: 3751-4375
ForkJoinPool-1-worker-13 开始执行: 2501-3125
ForkJoinPool-1-worker-8 开始执行: 626-1250
ForkJoinPool-1-worker-11 开始执行: 5001-5625
ForkJoinPool-1-worker-3 开始执行: 7501-8125
ForkJoinPool-1-worker-14 开始执行: 1251-1875
ForkJoinPool-1-worker-4 开始执行: 9376-10000
ForkJoinPool-1-worker-8 开始执行: 8126-8750
ForkJoinPool-1-worker-0 开始执行: 1876-2500
ForkJoinPool-1-worker-12 开始执行: 4376-5000
ForkJoinPool-1-worker-5 开始执行: 8751-9375
ForkJoinPool-1-worker-7 开始执行: 6876-7500
ForkJoinPool-1-worker-1 开始执行: 3126-3750
```

当然你也可以两个任务都 fork，要注意的是两个任务都 fork 的情况，必须按照 f1.fork()，f2.fork()， f2.join()，f1.join() 这样的顺序，不然有性能问题，详见上面注意事项中的说明。

官方 API 文档是这样写到的，所以平日用 invokeAll 就好了。invokeAll 会把传入的任务的第一个交给当前线程来执行，其他的任务都 fork 加入工作队列，这样等于利用当前线程也执行任务了。

```
public static void main(String[] args) {
    ForkJoinPool forkJoinPool = new ForkJoinPool(4); // 最大并发数4
    Fibonacci fibonacci = new Fibonacci(20);
    long startTime = System.currentTimeMillis();
    Integer result = forkJoinPool.invoke(fibonacci);
    long endTime = System.currentTimeMillis();
    System.out.println("Fork/join sum: " + result + " in " + (endTime - startTime) + " ms.");
}
//以下为官方API文档示例
static  class Fibonacci extends RecursiveTask<Integer> {
    final int n;
    Fibonacci(int n) {
        this.n = n;
    }
    @Override
    protected Integer compute() {
        if (n <= 1) {
            return n;
        }
        Fibonacci f1 = new Fibonacci(n - 1);
        f1.fork(); 
        Fibonacci f2 = new Fibonacci(n - 2);
        return f2.compute() + f1.join(); 
    }
}
```

### 3.8. [#](#_3-8-juc工具类) 3.8 JUC 工具类

*   [JUC 工具类: CountDownLatch 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-tool-countdownlatch.html)
*   [JUC 工具类: CyclicBarrier 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-tool-cyclicbarrier.html)
*   [JUC 工具类: Semaphore 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-tool-semaphore.html)
*   [JUC 工具类: Phaser 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-tool-phaser.html)
*   [JUC 工具类: Exchanger 详解](https://pdai.tech/md/java/thread/java-thread-x-juc-tool-exchanger.html)
*   [Java 并发 - ThreadLocal 详解](https://pdai.tech/md/java/thread/java-thread-x-threadlocal.html)

#### 3.8.1. [#](#什么是countdownlatch) 什么是 CountDownLatch?

CountDownLatch 底层也是由 AQS，用来同步一个或多个任务的常用并发工具类，强制它们等待由其他任务执行的一组操作完成。

#### 3.8.2. [#](#countdownlatch底层实现原理) CountDownLatch 底层实现原理?

其底层是由 AQS 提供支持，所以其数据结构可以参考 AQS 的数据结构，而 AQS 的数据结构核心就是两个虚拟队列: 同步队列 sync queue 和条件队列 condition queue，不同的条件会有不同的条件队列。CountDownLatch 典型的用法是将一个程序分为 n 个互相独立的可解决任务，并创建值为 n 的 CountDownLatch。当每一个任务完成时，都会在这个锁存器上调用 countDown，等待问题被解决的任务调用这个锁存器的 await，将他们自己拦住，直至锁存器计数结束。

#### 3.8.3. [#](#countdownlatch一次可以唤醒几个任务) CountDownLatch 一次可以唤醒几个任务?

多个

#### 3.8.4. [#](#countdownlatch有哪些主要方法) CountDownLatch 有哪些主要方法?

await(), 此函数将会使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断。

countDown(), 此函数将递减锁存器的计数，如果计数到达零，则释放所有等待的线程

#### 3.8.5. [#](#写道题-实现一个容器-提供两个方法-add-size-写两个线程-线程1添加10个元素到容器中-线程2实现监控元素的个数-当个数到5个时-线程2给出提示并结束) 写道题：实现一个容器，提供两个方法，add，size 写两个线程，线程 1 添加 10 个元素到容器中，线程 2 实现监控元素的个数，当个数到 5 个时，线程 2 给出提示并结束?

说出使用 CountDownLatch 代替 wait notify 好处?

```
{
    // ...
    Fibonacci f1 = new Fibonacci(n - 1);
    Fibonacci f2 = new Fibonacci(n - 2);
    invokeAll(f1,f2);
    return f2.join() + f1.join();
}

public static void invokeAll(ForkJoinTask<?>... tasks) {
    Throwable ex = null;
    int last = tasks.length - 1;
    for (int i = last; i >= 0; --i) {
        ForkJoinTask<?> t = tasks[i];
        if (t == null) {
            if (ex == null)
                ex = new NullPointerException();
        }
        else if (i != 0)   //除了第一个都fork
            t.fork();
        else if (t.doInvoke() < NORMAL && ex == null)  //留一个自己执行
            ex = t.getException();
    }
    for (int i = 1; i <= last; ++i) {
        ForkJoinTask<?> t = tasks[i];
        if (t != null) {
            if (ex != null)
                t.cancel(false);
            else if (t.doJoin() < NORMAL)
                ex = t.getException();
        }
    }
    if (ex != null)
        rethrow(ex);
}
```

#### 3.8.6. [#](#什么是cyclicbarrier) 什么是 CyclicBarrier?

*   对于 CountDownLatch，其他线程为游戏玩家，比如英雄联盟，主线程为控制游戏开始的线程。在所有的玩家都准备好之前，主线程是处于等待状态的，也就是游戏不能开始。当所有的玩家准备好之后，下一步的动作实施者为主线程，即开始游戏。
    
*   对于 CyclicBarrier，假设有一家公司要全体员工进行团建活动，活动内容为翻越三个障碍物，每一个人翻越障碍物所用的时间是不一样的。但是公司要求所有人在翻越当前障碍物之后再开始翻越下一个障碍物，也就是所有人翻越第一个障碍物之后，才开始翻越第二个，以此类推。类比地，每一个员工都是一个 “其他线程”。当所有人都翻越的所有的障碍物之后，程序才结束。而主线程可能早就结束了，这里我们不用管主线程。

#### 3.8.7. [#](#countdownlatch和cyclicbarrier对比) CountDownLatch 和 CyclicBarrier 对比?

*   CountDownLatch 减计数，CyclicBarrier 加计数。
*   CountDownLatch 是一次性的，CyclicBarrier 可以重用。
*   CountDownLatch 和 CyclicBarrier 都有让多个线程等待同步然后再开始下一步动作的意思，但是 CountDownLatch 的下一步的动作实施者是主线程，具有不可重复性；而 CyclicBarrier 的下一步动作实施者还是 “其他线程” 本身，具有往复多次实施动作的特点。

#### 3.8.8. [#](#什么是semaphore) 什么是 Semaphore?

Semaphore 底层是基于 AbstractQueuedSynchronizer 来实现的。Semaphore 称为计数信号量，它允许 n 个任务同时访问某个资源，可以将信号量看做是在向外分发使用资源的许可证，只有成功获取许可证，才能使用资源

#### 3.8.9. [#](#semaphore内部原理) Semaphore 内部原理?

Semaphore 总共有三个内部类，并且三个内部类是紧密相关的，下面先看三个类的关系。

![](https://pdai.tech/images/thread/java-thread-x-semaphore-1.png)

说明: Semaphore 与 ReentrantLock 的内部类的结构相同，类内部总共存在 Sync、NonfairSync、FairSync 三个类，NonfairSync 与 FairSync 类继承自 Sync 类，Sync 类继承自 AbstractQueuedSynchronizer 抽象类。下面逐个进行分析。

#### 3.8.10. [#](#semaphore常用方法有哪些-如何实现线程同步和互斥的) Semaphore 常用方法有哪些? 如何实现线程同步和互斥的?

#### 3.8.11. [#](#单独使用semaphore是不会使用到aqs的条件队列) 单独使用 Semaphore 是不会使用到 AQS 的条件队列?

不同于 CyclicBarrier 和 ReentrantLock，单独使用 Semaphore 是不会使用到 AQS 的条件队列的，其实，只有进行 await 操作才会进入条件队列，其他的都是在同步队列中，只是当前线程会被 park。

#### 3.8.12. [#](#semaphore初始化有10个令牌-11个线程同时各调用1次acquire方法-会发生什么) Semaphore 初始化有 10 个令牌，11 个线程同时各调用 1 次 acquire 方法，会发生什么?

拿不到令牌的线程阻塞，不会继续往下运行。

#### 3.8.13. [#](#semaphore初始化有10个令牌-一个线程重复调用11次acquire方法-会发生什么) Semaphore 初始化有 10 个令牌，一个线程重复调用 11 次 acquire 方法，会发生什么?

线程阻塞，不会继续往下运行。可能你会考虑类似于锁的重入的问题，很好，但是，令牌没有重入的概念。你只要调用一次 acquire 方法，就需要有一个令牌才能继续运行。

#### 3.8.14. [#](#semaphore初始化有1个令牌-1个线程调用一次acquire方法-然后调用两次release方法-之后另外一个线程调用acquire-2-方法-此线程能够获取到足够的令牌并继续运行吗) Semaphore 初始化有 1 个令牌，1 个线程调用一次 acquire 方法，然后调用两次 release 方法，之后另外一个线程调用 acquire(2) 方法，此线程能够获取到足够的令牌并继续运行吗?

能，原因是 release 方法会添加令牌，并不会以初始化的大小为准。

#### 3.8.15. [#](#semaphore初始化有2个令牌-一个线程调用1次release方法-然后一次性获取3个令牌-会获取到吗) Semaphore 初始化有 2 个令牌，一个线程调用 1 次 release 方法，然后一次性获取 3 个令牌，会获取到吗?

能，原因是 release 会添加令牌，并不会以初始化的大小为准。Semaphore 中 release 方法的调用并没有限制要在 acquire 后调用。

具体示例如下，如果不相信的话，可以运行一下下面的 demo，在做实验之前，笔者也认为应该是不允许的。。

```
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;

/**
 * 使用CountDownLatch 代替wait notify 好处是通讯方式简单，不涉及锁定  Count 值为0时当前线程继续执行，
 */
public class T3 {

   volatile List list = new ArrayList();

    public void add(int i){
        list.add(i);
    }

    public int getSize(){
        return list.size();
    }


    public static void main(String[] args) {
        T3 t = new T3();
        CountDownLatch countDownLatch = new CountDownLatch(1);

        new Thread(() -> {
            System.out.println("t2 start");
           if(t.getSize() != 5){
               try {
                   countDownLatch.await();
                   System.out.println("t2 end");
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
        },"t2").start();

        new Thread(()->{
            System.out.println("t1 start");
           for (int i = 0;i<9;i++){
               t.add(i);
               System.out.println("add"+ i);
               if(t.getSize() == 5){
                   System.out.println("countdown is open");
                   countDownLatch.countDown();
               }
           }
            System.out.println("t1 end");
        },"t1").start();
    }

}
```

#### 3.8.16. [#](#phaser主要用来解决什么问题) Phaser 主要用来解决什么问题?

Phaser 是 JDK 7 新增的一个同步辅助类，它可以实现 CyclicBarrier 和 CountDownLatch 类似的功能，而且它支持对任务的动态调整，并支持分层结构来达到更高的吞吐量。

#### 3.8.17. [#](#phaser与cyclicbarrier和countdownlatch的区别是什么) Phaser 与 CyclicBarrier 和 CountDownLatch 的区别是什么?

Phaser 和 CountDownLatch、CyclicBarrier 都有很相似的地方。

Phaser 顾名思义，就是可以分阶段的进行线程同步。

*   CountDownLatch 只能在创建实例时，通过构造方法指定同步数量；
*   Phaser 支持线程动态地向它注册。

利用这个动态注册的特性，可以达到分阶段同步控制的目的：

注册一批操作，等待它们执行结束；再注册一批操作，等它们结束...

#### 3.8.18. [#](#phaser运行机制是什么样的) Phaser 运行机制是什么样的?

![](https://pdai.tech/images/thread/java-thread-x-juc-phaser-1.png)

*   **Registration(注册)**

跟其他 barrier 不同，在 phaser 上注册的 parties 会随着时间的变化而变化。任务可以随时注册 (使用方法 register,bulkRegister 注册，或者由构造器确定初始 parties)，并且在任何抵达点可以随意地撤销注册 (方法 arriveAndDeregister)。就像大多数基本的同步结构一样，注册和撤销只影响内部 count；不会创建更深的内部记录，所以任务不能查询他们是否已经注册。(不过，可以通过继承来实现类似的记录)

*   **Synchronization(同步机制)**

和 CyclicBarrier 一样，Phaser 也可以重复 await。方法 arriveAndAwaitAdvance 的效果类似 CyclicBarrier.await。phaser 的每一代都有一个相关的 phase number，初始值为 0，当所有注册的任务都到达 phaser 时 phase+1，到达最大值 (Integer.MAX_VALUE) 之后清零。使用 phase number 可以独立控制 到达 phaser 和 等待其他线程 的动作，通过下面两种类型的方法:

*   **Arrival(到达机制)** arrive 和 arriveAndDeregister 方法记录到达状态。这些方法不会阻塞，但是会返回一个相关的 arrival phase number；也就是说，phase number 用来确定到达状态。当所有任务都到达给定 phase 时，可以执行一个可选的函数，这个函数通过重写 onAdvance 方法实现，通常可以用来控制终止状态。重写此方法类似于为 CyclicBarrier 提供一个 barrierAction，但比它更灵活。
    
*   **Waiting(等待机制)** awaitAdvance 方法需要一个表示 arrival phase number 的参数，并且在 phaser 前进到与给定 phase 不同的 phase 时返回。和 CyclicBarrier 不同，即使等待线程已经被中断，awaitAdvance 方法也会一直等待。中断状态和超时时间同样可用，但是当任务等待中断或超时后未改变 phaser 的状态时会遭遇异常。如果有必要，在方法 forceTermination 之后可以执行这些异常的相关的 handler 进行恢复操作，Phaser 也可能被 ForkJoinPool 中的任务使用，这样在其他任务阻塞等待一个 phase 时可以保证足够的并行度来执行任务。
*   **Termination(终止机制)** :

可以用 isTerminated 方法检查 phaser 的终止状态。在终止时，所有同步方法立刻返回一个负值。在终止时尝试注册也没有效果。当调用 onAdvance 返回 true 时 Termination 被触发。当 deregistration 操作使已注册的 parties 变为 0 时，onAdvance 的默认实现就会返回 true。也可以重写 onAdvance 方法来定义终止动作。forceTermination 方法也可以释放等待线程并且允许它们终止。

*   **Tiering(分层结构)** :

Phaser 支持分层结构 (树状构造) 来减少竞争。注册了大量 parties 的 Phaser 可能会因为同步竞争消耗很高的成本， 因此可以设置一些子 Phaser 来共享一个通用的 parent。这样的话即使每个操作消耗了更多的开销，但是会提高整体吞吐量。 在一个分层结构的 phaser 里，子节点 phaser 的注册和取消注册都通过父节点管理。子节点 phaser 通过构造或方法 register、bulkRegister 进行首次注册时，在其父节点上注册。子节点 phaser 通过调用 arriveAndDeregister 进行最后一次取消注册时，也在其父节点上取消注册。

*   **Monitoring(状态监控)** :

由于同步方法可能只被已注册的 parties 调用，所以 phaser 的当前状态也可能被任何调用者监控。在任何时候，可以通过 getRegisteredParties 获取 parties 数，其中 getArrivedParties 方法返回已经到达当前 phase 的 parties 数。当剩余的 parties(通过方法 getUnarrivedParties 获取) 到达时，phase 进入下一代。这些方法返回的值可能只表示短暂的状态，所以一般来说在同步结构里并没有啥卵用。

#### 3.8.19. [#](#给一个phaser使用的示例) 给一个 Phaser 使用的示例?

模拟了 100 米赛跑，10 名选手，只等裁判一声令下。当所有人都到达终点时，比赛结束。

```
public class TestSemaphore2 {
    public static void main(String[] args) {
        int permitsNum = 2;
        final Semaphore semaphore = new Semaphore(permitsNum);
        try {
            System.out.println("availablePermits:"+semaphore.availablePermits()+",semaphore.tryAcquire(3,1, TimeUnit.SECONDS):"+semaphore.tryAcquire(3,1, TimeUnit.SECONDS));
            semaphore.release();
            System.out.println("availablePermits:"+semaphore.availablePermits()+",semaphore.tryAcquire(3,1, TimeUnit.SECONDS):"+semaphore.tryAcquire(3,1, TimeUnit.SECONDS));
        }catch (Exception e) {

        }
    }
}
```

#### 3.8.20. [#](#exchanger主要解决什么问题) Exchanger 主要解决什么问题?

Exchanger 用于进行两个线程之间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过 exchange() 方法交换数据，当一个线程先执行 exchange() 方法后，它会一直等待第二个线程也执行 exchange() 方法，当这两个线程到达同步点时，这两个线程就可以交换数据了。

#### 3.8.21. [#](#对比synchronousqueue-为什么说exchanger可被视为-synchronousqueue-的双向形式) 对比 SynchronousQueue，为什么说 Exchanger 可被视为 SynchronousQueue 的双向形式?

Exchanger 是一种线程间安全交换数据的机制。可以和之前分析过的 SynchronousQueue 对比一下：线程 A 通过 SynchronousQueue 将数据 a 交给线程 B；线程 A 通过 Exchanger 和线程 B 交换数据，线程 A 把数据 a 交给线程 B，同时线程 B 把数据 b 交给线程 A。可见，SynchronousQueue 是交给一个数据，Exchanger 是交换两个数据。

#### 3.8.22. [#](#exchanger在不同的jdk版本中实现有什么差别) Exchanger 在不同的 JDK 版本中实现有什么差别?

*   在 JDK5 中 Exchanger 被设计成一个容量为 1 的容器，存放一个等待线程，直到有另外线程到来就会发生数据交换，然后清空容器，等到下一个到来的线程。
*   从 JDK6 开始，Exchanger 用了类似 ConcurrentMap 的分段思想，提供了多个 slot，增加了并发执行时的吞吐量。

#### 3.8.23. [#](#exchanger实现举例) Exchanger 实现举例

来一个非常经典的并发问题：你有相同的数据 buffer，一个或多个数据生产者，和一个或多个数据消费者。只是 Exchange 类只能同步 2 个线程，所以你只能在你的生产者和消费者问题中只有一个生产者和一个消费者时使用这个类。

```
public class Match {

    // 模拟了100米赛跑，10名选手，只等裁判一声令下。当所有人都到达终点时，比赛结束。
    public static void main(String[] args) throws InterruptedException {

        final Phaser phaser=new Phaser(1) ;
        // 十名选手
        for (int index = 0; index < 10; index++) {
            phaser.register();
            new Thread(new player(phaser),"player"+index).start();
        }
        System.out.println("Game Start");
        //注销当前线程,比赛开始
        phaser.arriveAndDeregister();
        //是否非终止态一直等待
        while(!phaser.isTerminated()){
        }
        System.out.println("Game Over");
    }
}
class player implements Runnable{

    private  final Phaser phaser ;

    player(Phaser phaser){
        this.phaser=phaser;
    }
    @Override
    public void run() {
        try {
            // 第一阶段——等待创建好所有线程再开始
            phaser.arriveAndAwaitAdvance();

            // 第二阶段——等待所有选手准备好再开始
            Thread.sleep((long) (Math.random() * 10000));
            System.out.println(Thread.currentThread().getName() + " ready");
            phaser.arriveAndAwaitAdvance();

            // 第三阶段——等待所有选手准备好到达，到达后，该线程从phaser中注销，不在进行下面的阶段。
            Thread.sleep((long) (Math.random() * 10000));
            System.out.println(Thread.currentThread().getName() + " arrived");
            phaser.arriveAndDeregister();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

可以看到，其结果可能如下：

```
public class Test {
    static class Producer extends Thread {
        private Exchanger<Integer> exchanger;
        private static int data = 0;
        Producer(String name, Exchanger<Integer> exchanger) {
            super("Producer-" + name);
            this.exchanger = exchanger;
        }

        @Override
        public void run() {
            for (int i=1; i<5; i++) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                    data = i;
                    System.out.println(getName()+" 交换前:" + data);
                    data = exchanger.exchange(data);
                    System.out.println(getName()+" 交换后:" + data);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    static class Consumer extends Thread {
        private Exchanger<Integer> exchanger;
        private static int data = 0;
        Consumer(String name, Exchanger<Integer> exchanger) {
            super("Consumer-" + name);
            this.exchanger = exchanger;
        }

        @Override
        public void run() {
            while (true) {
                data = 0;
                System.out.println(getName()+" 交换前:" + data);
                try {
                    TimeUnit.SECONDS.sleep(1);
                    data = exchanger.exchange(data);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(getName()+" 交换后:" + data);
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Exchanger<Integer> exchanger = new Exchanger<Integer>();
        new Producer("", exchanger).start();
        new Consumer("", exchanger).start();
        TimeUnit.SECONDS.sleep(7);
        System.exit(-1);
    }
}
```

#### 3.8.24. [#](#什么是threadlocal-用来解决什么问题的) 什么是 ThreadLocal? 用来解决什么问题的?

我们在 [Java 并发 - 并发理论基础](https://pdai.tech/md/java/thread/java-thread-x-theorty.html#%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84%E5%AE%9E%E7%8E%B0%E6%96%B9%E6%B3%95)总结过线程安全 (是指广义上的共享资源访问安全性，因为线程隔离是通过副本保证本线程访问资源安全性，它不保证线程之间还存在共享关系的狭义上的安全性) 的解决思路：

*   互斥同步: synchronized 和 ReentrantLock
*   非阻塞同步: CAS, AtomicXXXX
*   无同步方案: 栈封闭，本地存储 (Thread Local)，可重入代码

ThreadLocal 是通过线程隔离的方式防止任务在共享资源上产生冲突, 线程本地存储是一种自动化机制，可以为使用相同变量的每个不同线程都创建不同的存储。

ThreadLocal 是一个将在多线程中为每一个线程创建单独的变量副本的类; 当使用 ThreadLocal 来维护变量时, ThreadLocal 会为每个线程创建单独的变量副本, 避免因多线程操作共享变量而导致的数据不一致的情况。

#### 3.8.25. [#](#说说你对threadlocal的理解) 说说你对 ThreadLocal 的理解

提到 ThreadLocal 被提到应用最多的是 session 管理和数据库链接管理，这里以数据访问为例帮助你理解 ThreadLocal：

*   如下数据库管理类在单线程使用是没有任何问题的

```
Consumer- 交换前:0
Producer- 交换前:1
Consumer- 交换后:1
Consumer- 交换前:0
Producer- 交换后:0
Producer- 交换前:2
Producer- 交换后:0
Consumer- 交换后:2
Consumer- 交换前:0
Producer- 交换前:3
Producer- 交换后:0
Consumer- 交换后:3
Consumer- 交换前:0
Producer- 交换前:4
Producer- 交换后:0
Consumer- 交换后:4
Consumer- 交换前:0
```

很显然，在多线程中使用会存在线程安全问题：第一，这里面的 2 个方法都没有进行同步，很可能在 openConnection 方法中会多次创建 connect；第二，由于 connect 是共享变量，那么必然在调用 connect 的地方需要使用到同步来保障线程安全，因为很可能一个线程在使用 connect 进行数据库操作，而另外一个线程调用 closeConnection 关闭链接。

*   为了解决上述线程安全的问题，第一考虑：互斥同步

你可能会说，将这段代码的两个方法进行同步处理，并且在调用 connect 的地方需要进行同步处理，比如用 Synchronized 或者 ReentrantLock 互斥锁。

*   这里再抛出一个问题：这地方到底需不需要将 connect 变量进行共享?

事实上，是不需要的。假如每个线程中都有一个 connect 变量，各个线程之间对 connect 变量的访问实际上是没有依赖关系的，即一个线程不需要关心其他线程是否对这个 connect 进行了修改的。即改后的代码可以这样：

```
class ConnectionManager {
    private static Connection connect = null;

    public static Connection openConnection() {
        if (connect == null) {
            connect = DriverManager.getConnection();
        }
        return connect;
    }

    public static void closeConnection() {
        if (connect != null)
            connect.close();
    }
}
```

这样处理确实也没有任何问题，由于每次都是在方法内部创建的连接，那么线程之间自然不存在线程安全问题。但是这样会有一个致命的影响：导致服务器压力非常大，并且严重影响程序执行性能。由于在方法中需要频繁地开启和关闭数据库连接，这样不仅严重影响程序执行效率，还可能导致服务器压力巨大。

*   这时候 ThreadLocal 登场了

那么这种情况下使用 ThreadLocal 是再适合不过的了，因为 ThreadLocal 在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间互不影响，这样一来就不存在线程安全问题，也不会严重影响程序执行性能。下面就是网上出现最多的例子：

```
class ConnectionManager {
    private Connection connect = null;

    public Connection openConnection() {
        if (connect == null) {
            connect = DriverManager.getConnection();
        }
        return connect;
    }

    public void closeConnection() {
        if (connect != null)
            connect.close();
    }
}

class Dao {
    public void insert() {
        ConnectionManager connectionManager = new ConnectionManager();
        Connection connection = connectionManager.openConnection();

        // 使用connection进行操作

        connectionManager.closeConnection();
    }
}
```

#### 3.8.26. [#](#threadlocal是如何实现线程隔离的) ThreadLocal 是如何实现线程隔离的?

ThreadLocalMap

#### 3.8.27. [#](#为什么threadlocal会造成内存泄露-如何解决) 为什么 ThreadLocal 会造成内存泄露? 如何解决

网上有这样一个例子：

```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectionManager {

    private static final ThreadLocal<Connection> dbConnectionLocal = new ThreadLocal<Connection>() {
        @Override
        protected Connection initialValue() {
            try {
                return DriverManager.getConnection("", "", "");
            } catch (SQLException e) {
                e.printStackTrace();
            }
            return null;
        }
    };

    public Connection getConnection() {
        return dbConnectionLocal.get();
    }
}
```

如果用线程池来操作 ThreadLocal 对象确实会造成内存泄露, 因为对于线程池里面不会销毁的线程, 里面总会存在着`<ThreadLocal, LocalVariable>`的强引用, 因为 final static 修饰的 ThreadLocal 并不会释放, 而 ThreadLocalMap 对于 Key 虽然是弱引用, 但是强引用不会释放, 弱引用当然也会一直有值, 同时创建的 LocalVariable 对象也不会释放, 就造成了内存泄露; 如果 LocalVariable 对象不是一个大对象的话, 其实泄露的并不严重, `泄露的内存 = 核心线程数 * LocalVariable`对象的大小;

所以, 为了避免出现内存泄露的情况, ThreadLocal 提供了一个清除线程中对象的方法, 即 remove, 其实内部实现就是调用 ThreadLocalMap 的 remove 方法:

```
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class ThreadLocalDemo {
    static class LocalVariable {
        private Long[] a = new Long[1024 * 1024];
    }

    // (1)
    final static ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(5, 5, 1, TimeUnit.MINUTES,
            new LinkedBlockingQueue<>());
    // (2)
    final static ThreadLocal<LocalVariable> localVariable = new ThreadLocal<LocalVariable>();

    public static void main(String[] args) throws InterruptedException {
        // (3)
        Thread.sleep(5000 * 4);
        for (int i = 0; i < 50; ++i) {
            poolExecutor.execute(new Runnable() {
                public void run() {
                    // (4)
                    localVariable.set(new LocalVariable());
                    // (5)
                    System.out.println("use local varaible" + localVariable.get());
                    localVariable.remove();
                }
            });
        }
        // (6)
        System.out.println("pool execute over");
    }
}
```

找到 Key 对应的 Entry, 并且清除 Entry 的 Key(ThreadLocal) 置空, 随后清除过期的 Entry 即可避免内存泄露。

#### 3.8.28. [#](#还有哪些使用threadlocal的应用场景) 还有哪些使用 ThreadLocal 的应用场景?

*   每个线程维护了一个 “序列号”

```
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            e.clear();
            expungeStaleEntry(i);
            return;
        }
    }
}
```

*   看看阿里巴巴 java 开发手册中推荐的 ThreadLocal 的用法:

```
public class SerialNum {
    // The next serial number to be assigned
    private static int nextSerialNum = 0;

    private static ThreadLocal serialNum = new ThreadLocal() {
        protected synchronized Object initialValue() {
            return new Integer(nextSerialNum++);
        }
    };

    public static int get() {
        return ((Integer) (serialNum.get())).intValue();
    }
}

+ 经典的另外一个例子：

```java
private static final ThreadLocal threadSession = new ThreadLocal();  
  
public static Session getSession() throws InfrastructureException {  
    Session s = (Session) threadSession.get();  
    try {  
        if (s == null) {  
            s = getSessionFactory().openSession();  
            threadSession.set(s);  
        }  
    } catch (HibernateException ex) {  
        throw new InfrastructureException(ex);  
    }  
    return s;  
}
```

然后我们再要用到 DateFormat 对象的地方，这样调用：

```
import java.text.DateFormat;
import java.text.SimpleDateFormat;
 
public class DateUtils {
    public static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>(){
        @Override
        protected DateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd");
        }
    };
}
```

## 4. [#](#_4-java-io) 4 Java IO

Java IO 相关

### 4.1. [#](#_4-1-基础io) 4.1 基础 IO

#### 4.1.1. [#](#如何从数据传输方式理解io流) 如何从数据传输方式理解 IO 流？

从数据传输方式或者说是运输方式角度看，可以将 IO 类分为:

1.  **字节流**, 字节流读取单个字节，字符流读取单个字符 (一个字符根据编码的不同，对应的字节也不同，如 UTF-8 编码中文汉字是 3 个字节，GBK 编码中文汉字是 2 个字节。)
2.  **字符流**, 字节流用来处理二进制文件 (图片、MP3、视频文件)，字符流用来处理文本文件 (可以看做是特殊的二进制文件，使用了某种编码，人可以阅读)。

**字节是给计算机看的，字符才是给人看的**

*   **字节流**

![](https://pdai.tech/images/io/java-io-category-1.png)

*   **字符流**

![](https://pdai.tech/images/io/java-io-category-2.png)

*   **字节转字符**？

![](https://pdai.tech/images/io/java-io-1.png)

#### 4.1.2. [#](#如何从数据操作上理解io流) 如何从数据操作上理解 IO 流？

从数据来源或者说是操作对象角度看，IO 类可以分为:

![](https://pdai.tech/images/io/java-io-category-3.png)

#### 4.1.3. [#](#java-io设计上使用了什么设计模式) Java IO 设计上使用了什么设计模式？

**装饰者模式**： 所谓装饰，就是把这个装饰者套在被装饰者之上，从而动态扩展被装饰者的功能。

*   **装饰者举例**

设计不同种类的饮料，饮料可以添加配料，比如可以添加牛奶，并且支持动态添加新配料。每增加一种配料，该饮料的价格就会增加，要求计算一种饮料的价格。

下图表示在 DarkRoast 饮料上新增新添加 Mocha 配料，之后又添加了 Whip 配料。DarkRoast 被 Mocha 包裹，Mocha 又被 Whip 包裹。它们都继承自相同父类，都有 cost() 方法，外层类的 cost() 方法调用了内层类的 cost() 方法。

![](https://pdai.tech/images/pics/c9cfd600-bc91-4f3a-9f99-b42f88a5bb24.jpg)

*   **以 InputStream 为例**
    *   InputStream 是抽象组件；
    *   FileInputStream 是 InputStream 的子类，属于具体组件，提供了字节流的输入操作；
    *   FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。

![](https://pdai.tech/images/pics/DP-Decorator-java.io.png)

实例化一个具有缓存功能的字节流对象时，只需要在 FileInputStream 对象上再套一层 BufferedInputStream 对象即可。

```
DateUtils.df.get().format(new Date());
```

DataInputStream 装饰者提供了对更多数据类型进行输入的操作，比如 int、double 等基本类型。

### 4.2. [#](#_4-2-5种io模型) 4.2 5 种 IO 模型

#### 4.2.1. [#](#什么是阻塞-什么是同步) 什么是阻塞？什么是同步？

*   **阻塞 IO 和 非阻塞 IO**

这两个概念是**程序级别**的。主要描述的是程序请求操作系统 IO 操作后，如果 IO 资源没有准备好，那么程序该如何处理的问题: 前者等待；后者继续执行 (并且使用线程一直轮询，直到有 IO 资源准备好了)

*   **同步 IO 和 非同步 IO**

这两个概念是**操作系统级别**的。主要描述的是操作系统在收到程序请求 IO 操作后，如果 IO 资源没有准备好，该如何响应程序的问题: 前者不响应，直到 IO 资源准备好以后；后者返回一个标记 (好让程序和自己知道以后的数据往哪里通知)，当 IO 资源准备好以后，再用事件机制返回给程序。

#### 4.2.2. [#](#什么是linux的io模型) 什么是 Linux 的 IO 模型？

网络 IO 的本质是 socket 的读取，socket 在 linux 系统被抽象为流，IO 可以理解为对流的操作。刚才说了，对于一次 IO 访问（以 read 举例），**数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间**。所以说，当一个 read 操作发生时，它会经历两个阶段：

*   第一阶段：等待数据准备 (Waiting for the data to be ready)。
*   第二阶段：将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)。

对于 socket 流而言，

*   第一步：通常涉及等待网络上的数据分组到达，然后被复制到内核的某个缓冲区。
*   第二步：把数据从内核缓冲区复制到应用进程缓冲区。

网络应用需要处理的无非就是两大类问题，网络 IO，数据计算。相对于后者，网络 IO 的延迟，给应用带来的性能瓶颈大于后者。网络 IO 的模型大致有如下几种：

1.  同步阻塞 IO（bloking IO）
2.  同步非阻塞 IO（non-blocking IO）
3.  多路复用 IO（multiplexing IO）
4.  信号驱动式 IO（signal-driven IO）
5.  异步 IO（asynchronous IO）

![](https://pdai.tech/images/io/java-io-compare.png)

PS: 这块略复杂，在后面的提供了问答，所以用了最简单的举例结合 Linux IO 图例帮你快速理解。@pdai

#### 4.2.3. [#](#什么是同步阻塞io) 什么是同步阻塞 IO？

应用进程被阻塞，直到数据复制到应用进程缓冲区中才返回。

*   **举例理解**

你早上去买有现炸油条，你点单，之后一直等店家做好，期间你啥其它事也做不了。（你就是应用级别，店家就是操作系统级别， 应用被阻塞了不能做其它事）

*   **Linux 中 IO 图例**

![](https://pdai.tech/images/io/java-io-model-0.png)

#### 4.2.4. [#](#什么是同步非阻塞io) 什么是同步非阻塞 IO？

应用进程执行系统调用之后，内核返回一个错误码。应用进程可以继续执行，但是需要不断的执行系统调用来获知 I/O 是否完成，这种方式称为轮询 (polling)。

*   **举例理解**

你早上去买现炸油条，你点单，点完后每隔一段时间询问店家有没有做好，期间你可以做点其它事情。（你就是应用级别，店家就是操作系统级别，应用可以做其它事情并通过轮询来看操作系统是否完成）

*   **Linux 中 IO 图例**

![](https://pdai.tech/images/io/java-io-model-1.png)

#### 4.2.5. [#](#什么是多路复用io) 什么是多路复用 IO？

系统调用可能是由多个任务组成的，所以可以拆成多个任务，这就是多路复用。

*   **举例理解**

你早上去买现炸油条，点单收钱和炸油条原来都是由一个人完成的，现在他成了瓶颈，所以专门找了个收银员下单收钱，他则专注在炸油条。（本质上炸油条是耗时的瓶颈，将他职责分离出不是瓶颈的部分，比如下单收银，对应到系统级别也时一样的意思）

*   **Linux 中 IO 图例**

使用 select 或者 poll 等待数据，并且可以等待多个套接字中的任何一个变为可读，这一过程会被阻塞，当某一个套接字可读时返回。之后再使用 recvfrom 把数据从内核复制到进程中。

它可以让单个进程具有处理多个 I/O 事件的能力。又被称为 Event Driven I/O，即事件驱动 I/O。

![](https://pdai.tech/images/io/java-io-model-2.png)

#### 4.2.6. [#](#有哪些多路复用io) 有哪些多路复用 IO？

目前流程的多路复用 IO 实现主要包括四种: `select`、`poll`、`epoll`、`kqueue`。下表是他们的一些重要特性的比较:

<table><thead><tr><th>IO 模型</th><th>相对性能</th><th>关键思路</th><th>操作系统</th><th>JAVA 支持情况</th></tr></thead><tbody><tr><td>select</td><td>较高</td><td>Reactor</td><td>windows/Linux</td><td>支持, Reactor 模式 (反应器设计模式)。Linux 操作系统的 kernels 2.4 内核版本之前，默认使用 select；而目前 windows 下对同步 IO 的支持，都是 select 模型</td></tr><tr><td>poll</td><td>较高</td><td>Reactor</td><td>Linux</td><td>Linux 下的 JAVA NIO 框架，Linux kernels 2.6 内核版本之前使用 poll 进行支持。也是使用的 Reactor 模式</td></tr><tr><td>epoll</td><td>高</td><td>Reactor/Proactor</td><td>Linux</td><td>Linux kernels 2.6 内核版本及以后使用 epoll 进行支持；Linux kernels 2.6 内核版本之前使用 poll 进行支持；另外一定注意，由于 Linux 下没有 Windows 下的 IOCP 技术提供真正的 异步 IO 支持，所以 Linux 下使用 epoll 模拟异步 IO</td></tr><tr><td>kqueue</td><td>高</td><td>Proactor</td><td>Linux</td><td>目前 JAVA 的版本不支持</td></tr></tbody></table>

多路复用 IO 技术最适用的是 “高并发” 场景，所谓高并发是指 1 毫秒内至少同时有上千个连接请求准备好。其他情况下多路复用 IO 技术发挥不出来它的优势。另一方面，使用 JAVA NIO 进行功能实现，相对于传统的 Socket 套接字实现要复杂一些，所以实际应用中，需要根据自己的业务需求进行技术选择。

#### 4.2.7. [#](#什么是信号驱动io) 什么是信号驱动 IO？

应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是非阻塞的。内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。

相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。

*   **举例理解**

你早上去买现炸油条，门口排队的人多，现在引入了一个叫号系统，点完单后你就可以做自己的事情了，然后等叫号就去拿就可以了。（所以不用再去自己频繁跑去问有没有做好了）

*   **Linux 中 IO 图例**

![](https://pdai.tech/images/io/java-io-model-3.png)

#### 4.2.8. [#](#什么是异步io) 什么是异步 IO？

相对于同步 IO，异步 IO 不是顺序执行。用户进程进行 aio_read 系统调用之后，无论内核数据是否准备好，都会直接返回给用户进程，然后用户态进程可以去做别的事情。等到 socket 数据准备好了，内核直接复制数据给进程，然后从内核向进程发送通知。IO 两个阶段，进程都是非阻塞的。

*   **举例理解**

你早上去买现炸油条， 不用去排队了，打开美团外卖下单，然后做其它事，一会外卖自己送上门。(你就是应用级别，店家就是操作系统级别, 应用无需阻塞，这就是非阻塞；系统还可能在处理中，但是立刻响应了应用，这就是异步)

*   **Linux 中 IO 图例**

（Linux 提供了 AIO 库函数实现异步，但是用的很少。目前有很多开源的异步 IO 库，例如 libevent、libev、libuv）

![](https://pdai.tech/images/io/java-io-model-4.png)

#### 4.2.9. [#](#什么是reactor模型) 什么是 Reactor 模型？

大多数网络框架都是基于 Reactor 模型进行设计和开发，Reactor 模型基于事件驱动，特别适合处理海量的 I/O 事件。

*   **传统的 IO 模型**？

这种模式是传统设计，每一个请求到来时，大致都会按照：请求读取 -> 请求解码 -> 服务执行 -> 编码响应 -> 发送答复 这个流程去处理。

![](https://pdai.tech/images/io/java-io-reactor-1.png)

服务器会分配一个线程去处理，如果请求暴涨起来，那么意味着需要更多的线程来处理该请求。若请求出现暴涨，线程池的工作线程数量满载那么其它请求就会出现等待或者被抛弃。若每个小任务都可以使用非阻塞的模式，然后基于异步回调模式。这样就大大提高系统的吞吐量，这便引入了 Reactor 模型。

*   **Reactor 模型中定义的三种角色**：

1.  **Reactor**：负责监听和分配事件，将 I/O 事件分派给对应的 Handler。新的事件包含连接建立就绪、读就绪、写就绪等。
2.  **Acceptor**：处理客户端新连接，并分派请求到处理器链中。
3.  **Handler**：将自身与事件绑定，执行非阻塞读 / 写任务，完成 channel 的读入，完成处理业务逻辑后，负责将结果写出 channel。可用资源池来管理。

*   **单 Reactor 单线程模型**

Reactor 线程负责多路分离套接字，accept 新连接，并分派请求到 handler。Redis 使用单 Reactor 单进程的模型。

![](https://pdai.tech/images/io/java-io-reactor-2.png)

消息处理流程：

1.  Reactor 对象通过 select 监控连接事件，收到事件后通过 dispatch 进行转发。
2.  如果是连接建立的事件，则由 acceptor 接受连接，并创建 handler 处理后续事件。
3.  如果不是建立连接事件，则 Reactor 会分发调用 Handler 来响应。
4.  handler 会完成 read-> 业务处理 ->send 的完整业务流程。

*   **单 Reactor 多线程模型**

将 handler 的处理池化。

![](https://pdai.tech/images/io/java-io-reactor-3.png)

*   **多 Reactor 多线程模型**

主从 Reactor 模型： 主 Reactor 用于响应连接请求，从 Reactor 用于处理 IO 操作请求，读写分离了。

![](https://pdai.tech/images/io/java-io-reactor-4.png)

#### 4.2.10. [#](#什么是java-nio) 什么是 Java NIO？

NIO 主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector。**传统 IO 基于字节流和字符流进行操作**，而 **NIO 基于 Channel 和 Buffer(缓冲区) 进行操作**，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区) 用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

NIO 和传统 IO（一下简称 IO）之间第一个最大的区别是，IO 是面向流的，NIO 是面向缓冲区的。

![](https://pdai.tech/images/io/java-io-nio-x.png)

### 4.3. [#](#_4-3-零拷贝) 4.3 零拷贝

#### 4.3.1. [#](#传统的io存在什么问题-为什么引入零拷贝的) 传统的 IO 存在什么问题？为什么引入零拷贝的？

如果服务端要提供文件传输的功能，我们能想到的最简单的方式是：将磁盘上的文件读取出来，然后通过网络协议发送给客户端。

传统 I/O 的工作方式是，数据读取和写入是从用户空间到内核空间来回复制，而内核空间的数据是通过操作系统层面的 I/O 接口从磁盘读取或写入。

代码通常如下，一般会需要两个系统调用：

```
FileInputStream fileInputStream = new FileInputStream(filePath);
BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
```

代码很简单，虽然就两行代码，但是这里面发生了不少的事情。

![](https://pdai.tech/images/io/java-io-copy-3.png)

首先，**期间共发生了 4 次用户态与内核态的上下文切换**，因为发生了两次系统调用，一次是 read() ，一次是 write()，每次系统调用都得先从用户态切换到内核态，等内核完成任务后，再从内核态切换回用户态。

上下文切换到成本并不小，一次切换需要耗时几十纳秒到几微秒，虽然时间看上去很短，但是在高并发的场景下，这类时间容易被累积和放大，从而影响系统的性能。

其次，还发生了 **4 次数据拷贝**，其中**两次是 DMA 的拷贝**，另外**两次则是通过 CPU 拷贝**的，下面说一下这个过程：

*   **第一次拷贝**，把磁盘上的数据拷贝到操作系统内核的缓冲区里，这个拷贝的过程是通过 DMA 搬运的。
*   **第二次拷贝**，把内核缓冲区的数据拷贝到用户的缓冲区里，于是我们应用程序就可以使用这部分数据了，这个拷贝到过程是由 CPU 完成的。
*   **第三次拷贝**，把刚才拷贝到用户的缓冲区里的数据，再拷贝到内核的 socket 的缓冲区里，这个过程依然还是由 CPU 搬运的。
*   **第四次拷贝**，把内核的 socket 缓冲区里的数据，拷贝到网卡的缓冲区里，这个过程又是由 DMA 搬运的。

我们回过头看这个文件传输的过程，我们只是搬运一份数据，结果却搬运了 4 次，过多的数据拷贝无疑会消耗 CPU 资源，大大降低了系统性能。

这种简单又传统的文件传输方式，存在冗余的上文切换和数据拷贝，在高并发系统里是非常糟糕的，多了很多不必要的开销，会严重影响系统性能。

所以，**要想提高文件传输的性能，就需要减少「用户态与内核态的上下文切换」和「内存拷贝」的次数**。

#### 4.3.2. [#](#mmap-write怎么实现的零拷贝) mmap + write 怎么实现的零拷贝？

在前面我们知道，read() 系统调用的过程中会把内核缓冲区的数据拷贝到用户的缓冲区里，于是为了减少这一步开销，我们可以用 mmap() 替换 read() 系统调用函数。

```
read(file, tmp_buf, len);
write(socket, tmp_buf, len);
```

mmap() 系统调用函数会直接把内核缓冲区里的数据「映射」到用户空间，这样，操作系统内核与用户空间就不需要再进行任何的数据拷贝操作。

![](https://pdai.tech/images/io/java-io-copy-4.png)

具体过程如下：

*   应用进程调用了 mmap() 后，DMA 会把磁盘的数据拷贝到内核的缓冲区里。接着，应用进程跟操作系统内核「共享」这个缓冲区；
*   应用进程再调用 write()，操作系统直接将内核缓冲区的数据拷贝到 socket 缓冲区中，这一切都发生在内核态，由 CPU 来搬运数据；
*   最后，把内核的 socket 缓冲区里的数据，拷贝到网卡的缓冲区里，这个过程是由 DMA 搬运的。

我们可以得知，通过使用 mmap() 来代替 read()， 可以减少一次数据拷贝的过程。

但这还不是最理想的零拷贝，因为仍然需要通过 CPU 把内核缓冲区的数据拷贝到 socket 缓冲区里，而且仍然需要 4 次上下文切换，因为系统调用还是 2 次。

#### 4.3.3. [#](#sendfile怎么实现的零拷贝) sendfile 怎么实现的零拷贝？

在 Linux 内核版本 2.1 中，提供了一个专门发送文件的系统调用函数 sendfile()，函数形式如下：

```
buf = mmap(file, len);
write(sockfd, buf, len);
```

它的前两个参数分别是目的端和源端的文件描述符，后面两个参数是源端的偏移量和复制数据的长度，返回值是实际复制数据的长度。

首先，它可以替代前面的 read() 和 write() 这两个系统调用，这样就可以减少一次系统调用，也就减少了 2 次上下文切换的开销。

其次，该系统调用，可以直接把内核缓冲区里的数据拷贝到 socket 缓冲区里，不再拷贝到用户态，这样就只有 2 次上下文切换，和 3 次数据拷贝。如下图：

![](https://pdai.tech/images/io/java-io-copy-5.png)

但是这还不是真正的零拷贝技术，如果网卡支持 SG-DMA（**The Scatter-Gather Direct Memory Access**）技术（和普通的 DMA 有所不同），我们可以进一步减少通过 CPU 把内核缓冲区里的数据拷贝到 socket 缓冲区的过程。

你可以在你的 Linux 系统通过下面这个命令，查看网卡是否支持 scatter-gather 特性：

```
#include <sys/socket.h>
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```

于是，从 Linux 内核 2.4 版本开始起，对于支持网卡支持 SG-DMA 技术的情况下， sendfile() 系统调用的过程发生了点变化，具体过程如下：

*   第一步，通过 DMA 将磁盘上的数据拷贝到内核缓冲区里；
*   第二步，缓冲区描述符和数据长度传到 socket 缓冲区，这样网卡的 SG-DMA 控制器就可以直接将内核缓存中的数据拷贝到网卡的缓冲区里，此过程不需要将数据从操作系统内核缓冲区拷贝到 socket 缓冲区中，这样就减少了一次数据拷贝；

所以，这个过程之中，只进行了 2 次数据拷贝，如下图：

![](https://pdai.tech/images/io/java-io-copy-6.png)

这就是所谓的**零拷贝（Zero-copy）技术，因为我们没有在内存层面去拷贝数据，也就是说全程没有通过 CPU 来搬运数据，所有的数据都是通过 DMA 来进行传输的**。

零拷贝技术的文件传输方式相比传统文件传输的方式，**减少了 2 次上下文切换和数据拷贝次数，只需要 2 次上下文切换和数据拷贝次数，就可以完成文件的传输，而且 2 次的数据拷贝过程，都不需要通过 CPU，2 次都是由 DMA 来搬运**。

## 5. [#](#_5-jvm和调优) 5 JVM 和调优

JVM 虚拟机和调优相关。

### 5.1. [#](#_5-1-类加载机制) 5.1 类加载机制

#### 5.1.1. [#](#类加载的生命周期) 类加载的生命周期？

其中类加载的过程包括了**加载、验证、准备、解析、初始化**五个阶段。在这五个阶段中，加载、验证、准备和初始化这四个阶段发生的顺序是确定的，而解析阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持 Java 语言的运行时绑定 (也成为动态绑定或晚期绑定)*。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段。

![](https://pdai.tech/images/jvm/java_jvm_classload_2.png)

*   类的加载: 查找并加载类的二进制数据
*   连接
    *   验证: 确保被加载的类的正确性
    *   准备: 为类的静态变量分配内存，并将其初始化为默认值
    *   解析: 把类中的符号引用转换为直接引用
*   初始化：为类的静态变量赋予正确的初始值，JVM 负责对类进行初始化，主要对类变量进行初始化。
*   使用： 类访问方法区内的数据结构的接口， 对象是 Heap 区的数据
*   卸载： 结束生命周期

#### 5.1.2. [#](#类加载器的层次) 类加载器的层次?

![](https://pdai.tech/images/jvm/java_jvm_classload_3.png)

*   **启动类加载器**: Bootstrap ClassLoader，负责加载存放在 JDK\jre\lib(JDK 代表 JDK 的安装目录，下同) 下，或被 - Xbootclasspath 参数指定的路径中的，并且能被虚拟机识别的类库 (如 rt.jar，所有的 java.* 开头的类均被 Bootstrap ClassLoader 加载)。启动类加载器是无法被 Java 程序直接引用的。
    
*   **扩展类加载器**: Extension ClassLoader，该加载器由`sun.misc.Launcher$ExtClassLoader`实现，它负责加载 JDK\jre\lib\ext 目录中，或者由 java.ext.dirs 系统变量指定的路径中的所有类库 (如 javax.* 开头的类)，开发者可以直接使用扩展类加载器。
    
*   **应用程序类加载器**: Application ClassLoader，该类加载器由`sun.misc.Launcher$AppClassLoader`来实现，它负责加载用户类路径 (ClassPath) 所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。
    
*   **自定义类加载器**: 因为 JVM 自带的 ClassLoader 只是懂得从本地文件系统加载标准的 java class 文件，因此如果编写了自己的 ClassLoader，便可以做到如下几点:
    
    *   在执行非置信代码之前，自动验证数字签名。
    *   动态地创建符合用户特定需要的定制化构建类。
    *   从特定的场所取得 java class，例如数据库中和网络中。

#### 5.1.3. [#](#class-forname-和classloader-loadclass-区别) Class.forName() 和 ClassLoader.loadClass() 区别?

*   `Class.forName()`: 将类的. class 文件加载到 jvm 中之外，还会对类进行解释，执行类中的 static 块；
*   `ClassLoader.loadClass()`: 只干一件事情，就是将. class 文件加载到 jvm 中，不会执行 static 中的内容, 只有在 newInstance 才会去执行 static 块。
*   `Class.forName(name, initialize, loader)`带参函数也可控制是否加载 static 块。并且只有调用了 newInstance() 方法采用调用构造函数，创建类的对象 。

#### 5.1.4. [#](#jvm有哪些类加载机制) JVM 有哪些类加载机制？

*   **JVM 类加载机制有哪些**？

1.  **全盘负责**，当一个类加载器负责加载某个 Class 时，该 Class 所依赖的和引用的其他 Class 也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入
2.  **父类委托**，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类
3.  **缓存机制**，缓存机制将会保证所有加载过的 Class 都会被缓存，当程序中需要使用某个 Class 时，类加载器先从缓存区寻找该 Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成 Class 对象，存入缓存区。这就是为什么修改了 Class 后，必须重启 JVM，程序的修改才会生效
4.  **双亲委派机制**, 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

*   **双亲委派机制过程？**

1.  当 AppClassLoader 加载一个 class 时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器 ExtClassLoader 去完成。
2.  当 ExtClassLoader 加载一个 class 时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给 BootStrapClassLoader 去完成。
3.  如果 BootStrapClassLoader 加载失败 (例如在 $JAVA_HOME/jre/lib 里未查找到该 class)，会使用 ExtClassLoader 来尝试加载；
4.  若 ExtClassLoader 也加载失败，则会使用 AppClassLoader 来加载，如果 AppClassLoader 也加载失败，则会报出异常 ClassNotFoundException。

### 5.2. [#](#_5-2-内存结构) 5.2 内存结构

#### 5.2.1. [#](#说说jvm内存整体的结构-线程私有还是共享的) 说说 JVM 内存整体的结构？线程私有还是共享的？

JVM 整体架构，中间部分就是 Java 虚拟机定义的各种运行时数据区域。

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

Java 虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁。另外一些则是与线程一一对应的，这些与线程一一对应的数据区域会随着线程开始和结束而创建和销毁。

*   **线程私有**：程序计数器、虚拟机栈、本地方法区
*   **线程共享**：堆、方法区, 堆外内存（Java7 的永久代或 JDK8 的元空间、代码缓存）

#### 5.2.2. [#](#什么是程序计数器-线程私有) 什么是程序计数器（线程私有）？

PC 寄存器用来存储指向下一条指令的地址，即将要执行的指令代码。由执行引擎读取下一条指令。

*   **PC 寄存器为什么会被设定为线程私有的？**

多线程在一个特定的时间段内只会执行其中某一个线程方法，CPU 会不停的做任务切换，这样必然会导致经常中断或恢复。为了能够准确的记录各个线程正在执行的当前字节码指令地址，所以为每个线程都分配了一个 PC 寄存器，每个线程都独立计算，不会互相影响。

#### 5.2.3. [#](#什么是虚拟机栈-线程私有) 什么是虚拟机栈（线程私有）？

主管 Java 程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的栈帧 (Stack Frame），对应着一次次 Java 方法调用，是线程私有的，生命周期和线程一致。

*   **特点？**

1.  栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器
2.  JVM 直接对虚拟机栈的操作只有两个：每个方法执行，伴随着**入栈**（进栈 / 压栈），方法执行结束**出栈**
3.  栈不存在垃圾回收问题
4.  可以通过参数`-Xss`来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度

*   **该区域有哪些异常**？

1.  如果采用固定大小的 Java 虚拟机栈，那每个线程的 Java 虚拟机栈容量可以在线程创建的时候独立选定。如果线程请求分配的栈容量超过 Java 虚拟机栈允许的最大容量，Java 虚拟机将会抛出一个 **StackOverflowError** 异常
2.  如果 Java 虚拟机栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，那 Java 虚拟机将会抛出一个 **OutOfMemoryError** 异常

*   **栈帧的内部结构？**

1.  局部变量表（Local Variables）
2.  操作数栈（Operand Stack）(或称为表达式栈)
3.  动态链接（Dynamic Linking）：指向运行时常量池的方法引用
4.  方法返回地址（Return Address）：方法正常退出或异常退出的地址
5.  一些附加信息

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

#### 5.2.4. [#](#java虚拟机栈如何进行方法计算的) Java 虚拟机栈如何进行方法计算的？

以如下代码为例：

```
$ ethtool -k eth0 | grep scatter-gather scatter-gather: on
```

可以通过 jsclass 等工具查看 bytecode

![](https://pdai.tech/images/jvm/java-jvm-stack-2.png)

压栈的步骤如下：

```
private static int add(int a, int b) {
    int c = 0;
    c = a + b;
    return c;
}
```

如果计算 100+98 的值，那么操作数栈的变化如下图

![](https://pdai.tech/images/jvm/java-jvm-stack-3.png)

#### 5.2.5. [#](#什么是本地方法栈-线程私有) 什么是本地方法栈（线程私有）？

*   **本地方法接口**

一个 Native Method 就是一个 Java 调用非 Java 代码的接口。我们知道的 Unsafe 类就有很多本地方法。

*   **本地方法栈 (Native Method Stack)**

Java 虚拟机栈用于管理 Java 方法的调用，而本地方法栈用于管理本地方法的调用

#### 5.2.6. [#](#什么是方法区-线程共享) 什么是方法区（线程共享）？

方法区（method area）只是 **JVM 规范**中定义的一个概念，用于存储类信息、常量池、静态变量、JIT 编译后的代码等数据，并没有规定如何去实现它，不同的厂商有不同的实现。而**永久代（PermGen）**是 **Hotspot** 虚拟机特有的概念， Java8 的时候又被**元空间**取代了，永久代和元空间都可以理解为方法区的落地实现。

JDK1.8 之前调节方法区大小：

```
0:   iconst_0 // 0压栈
1:   istore_2 // 弹出int，存放于局部变量2
2:   iload_0  // 把局部变量0压栈
3:   iload_1  // 局部变量1压栈
4:   iadd     //弹出2个变量，求和，结果压栈
5:   istore_2 //弹出结果，放于局部变量2
6:   iload_2  //局部变量2压栈
7:   ireturn  //返回
```

JDK1.8 开始方法区（HotSpot 的永久代）被彻底删除了，取而代之的是元空间，元空间直接使用的是本机内存。参数设置：

```
-XX:PermSize=N //方法区（永久代）初始大小
-XX:MaxPermSize=N //方法区（永久代）最大大小，超出这个值将会抛出OutOfMemoryError
```

**栈、堆、方法区的交互关系**

![](https://pdai.tech/images/jvm/java-jvm-stack-1.png)

#### 5.2.7. [#](#永久代和元空间内存使用上的差异) 永久代和元空间内存使用上的差异?

Java 虚拟机规范中只定义了方法区用于存储已被虚拟机加载的类信息、常量、静态变量和即时编译后的代码等数据

1.  jdk1.7 开始符号引用存储在 native heap 中，字符串常量和静态类型变量存储在普通的堆区中，但分离的并不彻底, 此时永久代中还保存另一些与类的元数据无关的杂项
2.  jdk8 后 HotSpot 原永久代中存储的类的**元数据将存储在 metaspace** 中，而**类的静态变量和字符串常量将放在 Java 堆中**，metaspace 是方法区的一种实现，只不过它使用的不是虚拟机内的内存，而是本地内存。在元空间中保存的数据比永久代中纯粹很多，就只是类的元数据，这些信息只对编译期或 JVM 的运行时有用。
3.  永久代有一个 JVM 本身设置固定大小上线，无法进行调整，而**元空间使用的是直接内存，受本机可用内存的限制，并且永远不会得到 java.lang.OutOfMemoryError**。
4.  **符号引用没有存在元空间中，而是存在 native heap 中**，这是两个方式和位置，不过都可以算作是本地内存，在虚拟机之外进行划分，没有设置限制参数时只受物理内存大小限制，即只有占满了操作系统可用内存后才 OOM。

#### 5.2.8. [#](#堆区内存是怎么细分的) 堆区内存是怎么细分的？

对于大多数应用，Java 堆是 Java 虚拟机管理的内存中最大的一块，被所有线程共享。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数据都在这里分配内存。

为了进行高效的垃圾回收，虚拟机把堆内存**逻辑上**划分成三块区域（分代的唯一理由就是优化 GC 性能）：

1.  新生带（年轻代）：新对象和没达到一定年龄的对象都在新生代
2.  老年代（养老区）：被长时间使用的对象，老年代的内存空间应该要比年轻代更大

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

Java 虚拟机规范规定，Java 堆可以是处于物理上不连续的内存空间中，只要逻辑上是连续的即可，像磁盘空间一样。实现时，既可以是固定大小，也可以是可扩展的，主流虚拟机都是可扩展的（通过 `-Xmx` 和 `-Xms` 控制），如果堆中没有完成实例分配，并且堆无法再扩展时，就会抛出 `OutOfMemoryError` 异常。

*   **年轻代 (Young Generation)**

年轻代是所有新对象创建的地方。当填充年轻代时，执行垃圾收集。这种垃圾收集称为 **Minor GC**。年轻一代被分为三个部分——伊甸园（**Eden Memory**）和两个幸存区（**Survivor Memory**，被称为 from/to 或 s0/s1），默认比例是`8:1:1`

1.  大多数新创建的对象都位于 Eden 内存空间中
2.  当 Eden 空间被对象填充时，执行 **Minor GC**，并将所有幸存者对象移动到一个幸存者空间中
3.  Minor GC 检查幸存者对象，并将它们移动到另一个幸存者空间。所以每次，一个幸存者空间总是空的
4.  经过多次 GC 循环后存活下来的对象被移动到老年代。通常，这是通过设置年轻一代对象的年龄阈值来实现的，然后他们才有资格提升到老一代

*   **老年代 (Old Generation)**

旧的一代内存包含那些经过许多轮小型 GC 后仍然存活的对象。通常，垃圾收集是在老年代内存满时执行的。老年代垃圾收集称为 主 GC（Major GC），通常需要更长的时间。

大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）。这样做的目的是避免在 Eden 区和两个 Survivor 区之间发生大量的内存拷贝

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

#### 5.2.9. [#](#jvm中对象在堆中的生命周期) JVM 中对象在堆中的生命周期?

1.  在 JVM 内存模型的堆中，堆被划分为新生代和老年代
    *   新生代又被进一步划分为 **Eden 区** 和 **Survivor 区**，Survivor 区由 **From Survivor** 和 **To Survivor** 组成
2.  当创建一个对象时，对象会被优先分配到新生代的 Eden 区
    *   此时 JVM 会给对象定义一个**对象年轻计数器**（`-XX:MaxTenuringThreshold`）
3.  当 Eden 空间不足时，JVM 将执行新生代的垃圾回收（Minor GC）
    *   JVM 会把存活的对象转移到 Survivor 中，并且对象年龄 +1
    *   对象在 Survivor 中同样也会经历 Minor GC，每经历一次 Minor GC，对象年龄都会 + 1
4.  如果分配的对象超过了`-XX:PetenureSizeThreshold`，对象会**直接被分配到老年代**

#### 5.2.10. [#](#jvm中对象的分配过程) JVM 中对象的分配过程?

为对象分配内存是一件非常严谨和复杂的任务，JVM 的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法和内存回收算法密切相关，所以还需要考虑 GC 执行完内存回收后是否会在内存空间中产生内存碎片。

1.  new 的对象先放在伊甸园区，此区有大小限制
2.  当伊甸园的空间填满时，程序又需要创建对象，JVM 的垃圾回收器将对伊甸园区进行垃圾回收（Minor GC），将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区
3.  然后将伊甸园中的剩余对象移动到幸存者 0 区
4.  如果再次触发垃圾回收，此时上次幸存下来的放到幸存者 0 区，如果没有回收，就会放到幸存者 1 区
5.  如果再次经历垃圾回收，此时会重新放回幸存者 0 区，接着再去幸存者 1 区
6.  什么时候才会去养老区呢？ 默认是 15 次回收标记
7.  在养老区，相对悠闲。当养老区内存不足时，再次触发 Major GC，进行养老区的内存清理
8.  若养老区执行了 Major GC 之后发现依然无法进行对象的保存，就会产生 OOM 异常

#### 5.2.11. [#](#什么是-tlab-thread-local-allocation-buffer) 什么是 TLAB （Thread Local Allocation Buffer）?

*   从内存模型而不是垃圾回收的角度，对 Eden 区域继续进行划分，JVM 为每个线程分配了一个私有缓存区域，它包含在 Eden 空间内
*   多线程同时分配内存时，使用 TLAB 可以避免一系列的非线程安全问题，同时还能提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为**快速分配策略**
*   OpenJDK 衍生出来的 JVM 大都提供了 TLAB 设计

#### 5.2.12. [#](#为什么要有-tlab) 为什么要有 TLAB ?

*   堆区是线程共享的，任何线程都可以访问到堆区中的共享数据
*   由于对象实例的创建在 JVM 中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
*   为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度

尽管不是所有的对象实例都能够在 TLAB 中成功分配内存，但 JVM 确实是将 TLAB 作为内存分配的首选。

在程序中，可以通过 `-XX:UseTLAB` 设置是否开启 TLAB 空间。

默认情况下，TLAB 空间的内存非常小，仅占有整个 Eden 空间的 1%，我们可以通过 `-XX:TLABWasteTargetPercent` 设置 TLAB 空间所占用 Eden 空间的百分比大小。

一旦对象在 TLAB 空间分配内存失败时，JVM 就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在 Eden 空间中分配内存。

### 5.3. [#](#_5-3-gc垃圾回收) 5.3 GC 垃圾回收

#### 5.3.1. [#](#如何判断一个对象是否可以回收) 如何判断一个对象是否可以回收？

*   **引用计数算法**

给对象添加一个引用计数器，当对象增加一个引用时计数器加 1，引用失效时计数器减 1。引用计数为 0 的对象可被回收。

两个对象出现循环引用的情况下，此时引用计数器永远不为 0，导致无法对它们进行回收。

正因为循环引用的存在，因此 Java 虚拟机不使用引用计数算法。

*   **可达性分析算法**

通过 GC Roots 作为起始点进行搜索，能够到达到的对象都是存活的，不可达的对象可被回收。

![](https://pdai.tech/images/pics/0635cbe8.png)

Java 虚拟机使用该算法来判断对象是否可被回收，在 Java 中 GC Roots 一般包含以下内容:

*   虚拟机栈中引用的对象
*   本地方法栈中引用的对象
*   方法区中类静态属性引用的对象
*   方法区中的常量引用的对象

#### 5.3.2. [#](#对象有哪些引用类型) 对象有哪些引用类型？

无论是通过引用计算算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，判定对象是否可被回收都与引用有关。

Java 具有四种强度不同的引用类型。

*   **强引用**

被强引用关联的对象不会被回收。

使用 new 一个新对象的方式来创建强引用。

```
-XX:MetaspaceSize=N //设置Metaspace的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置Metaspace的最大大小
```

*   **软引用**

被软引用关联的对象只有在内存不够的情况下才会被回收。

使用 SoftReference 类来创建软引用。

```
Object obj = new Object();
```

*   **弱引用**

被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。

使用 WeakReference 类来实现弱引用。

```
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联
```

*   **虚引用**

又称为幽灵引用或者幻影引用。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用取得一个对象。

为一个对象设置虚引用关联的唯一目的就是能在这个对象被回收时收到一个系统通知。

使用 PhantomReference 来实现虚引用。

```
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
```

#### 5.3.3. [#](#有哪些基本的垃圾回收算法) 有哪些基本的垃圾回收算法？

*   **标记 - 清除**

![](https://pdai.tech/images/pics/a4248c4b-6c1d-4fb8-a557-86da92d3a294.jpg)

将存活的对象进行标记，然后清理掉未被标记的对象。

不足:

*   标记和清除过程效率都不高；
*   会产生大量不连续的内存碎片，导致无法给大对象分配内存。
*   **标记 - 整理**

![](https://pdai.tech/images/pics/902b83ab-8054-4bd2-898f-9a4a0fe52830.jpg)

让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

*   **复制**

![](https://pdai.tech/images/pics/e6b733ad-606d-4028-b3e8-83c3a73a3797.jpg)

将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理。

主要不足是只使用了内存的一半。

现在的商业虚拟机都采用这种收集算法来回收新生代，但是并不是将新生代划分为大小相等的两块，而是分为一块较大的 Eden 空间和两块较小的 Survivor 空间，每次使用 Eden 空间和其中一块 Survivor。在回收时，将 Eden 和 Survivor 中还存活着的对象一次性复制到另一块 Survivor 空间上，最后清理 Eden 和使用过的那一块 Survivor。

HotSpot 虚拟机的 Eden 和 Survivor 的大小比例默认为 8:1，保证了内存的利用率达到 90%。如果每次回收有多于 10% 的对象存活，那么一块 Survivor 空间就不够用了，此时需要依赖于老年代进行分配担保，也就是借用老年代的空间存储放不下的对象。

*   **分代收集**

现在的商业虚拟机采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用适当的收集算法。

一般将堆分为新生代和老年代。

*   新生代使用: 复制算法
*   老年代使用: 标记 - 清除 或者 标记 - 整理 算法

#### 5.3.4. [#](#分代收集算法和分区收集算法区别) 分代收集算法和分区收集算法区别？

![](https://pdai.tech/images/jvm/cmsgc/cms-gc-5.jpg)

*   **分代收集算法**

当前主流 VM 垃圾收集都采用” 分代收集”(Generational Collection) 算法, 这种算法会根据 对象存活周期的不同将内存划分为几块, 如 JVM 中的 新生代、老年代、永久代，这样就可以根据 各年代特点分别采用最适当的 GC 算法

在新生代 - 复制算法：

每次垃圾收集都能发现大批对象已死, 只有少量存活. 因此选用复制算法, 只需要付出少量 存活对象的复制成本就可以完成收集

在老年代 - 标记整理算法：

因为对象存活率高、没有额外空间对它进行分配担保, 就必须采用 “标记—清理” 或“标 记—整理”算法来进行回收, 不必进行内存复制, 且直接腾出空闲内存.

1.  **ParNew**： 一款多线程的收集器，采用复制算法，主要工作在 Young 区，可以通过 `-XX:ParallelGCThreads` 参数来控制收集的线程数，整个过程都是 STW 的，常与 CMS 组合使用。
    
2.  **CMS**： 以获取最短回收停顿时间为目标，采用 “标记 - 清除” 算法，分 4 大步进行垃圾收集，其中初始标记和重新标记会 STW ，多数应用于互联网站或者 B/S 系统的服务器端上，JDK9 被标记弃用，JDK14 被删除。

*   **分区收集算法**

分区算法则将整个堆空间划分为连续的不同小区间, 每个小区间独立使用, 独立回收. 这样做的 好处是可以控制一次回收多少个小区间 , 根据目标停顿时间, 每次合理地回收若干个小区间 (而不是 整个堆), 从而减少一次 GC 所产生的停顿。

1.  **G1**： 一种服务器端的垃圾收集器，应用在多处理器和大容量内存环境中，在实现高吞吐量的同时，尽可能地满足垃圾收集暂停时间的要求。
    
2.  **ZGC**： JDK11 中推出的一款低延迟垃圾回收器，适用于大内存低延迟服务的内存管理和回收，SPECjbb 2015 基准测试，在 128G 的大堆下，最大停顿时间才 1.68 ms，停顿时间远胜于 G1 和 CMS。

#### 5.3.5. [#](#什么是minor-gc、major-gc、full-gc) 什么是 Minor GC、Major GC、Full GC?

JVM 在进行 GC 时，并非每次都对堆内存（新生代、老年代；方法区）区域一起回收的，大部分时候回收的都是指新生代。

针对 HotSpot VM 的实现，它里面的 GC 按照回收区域又分为两大类：部分收集（Partial GC），整堆收集（Full GC）

*   部分收集：不是完整收集整个 Java 堆的垃圾收集。其中又分为：
    *   新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集
    *   老年代收集（Major GC/Old GC）：只是老年代的垃圾收集
        *   目前，只有 CMS GC 会有单独收集老年代的行为
        *   很多时候 Major GC 会和 Full GC 混合使用，需要具体分辨是老年代回收还是整堆回收
    *   混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集
        *   目前只有 G1 GC 会有这种行为
*   整堆收集（Full GC）：收集整个 Java 堆和方法区的垃圾

#### 5.3.6. [#](#说说jvm内存分配策略) 说说 JVM 内存分配策略？

*   **对象优先在 Eden 分配**

大多数情况下，对象在新生代 Eden 区分配，当 Eden 区空间不够时，发起 Minor GC。

*   **大对象直接进入老年代**

大对象是指需要连续内存空间的对象，最典型的大对象是那种很长的字符串以及数组。

经常出现大对象会提前触发垃圾收集以获取足够的连续空间分配给大对象。

-XX:PretenureSizeThreshold，大于此值的对象直接在老年代分配，避免在 Eden 区和 Survivor 区之间的大量内存复制。

*   **长期存活的对象进入老年代**

为对象定义年龄计数器，对象在 Eden 出生并经过 Minor GC 依然存活，将移动到 Survivor 中，年龄就增加 1 岁，增加到一定年龄则移动到老年代中。

-XX:MaxTenuringThreshold 用来定义年龄的阈值。

*   **动态对象年龄判定**

虚拟机并不是永远地要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代，如果在 Survivor 中相同年龄所有对象大小的总和大于 Survivor 空间的一半，则年龄大于或等于该年龄的对象可以直接进入老年代，无需等到 MaxTenuringThreshold 中要求的年龄。

*   **空间分配担保**

在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果条件成立的话，那么 Minor GC 可以确认是安全的。

如果不成立的话虚拟机会查看 HandlePromotionFailure 设置值是否允许担保失败，如果允许那么就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次 Minor GC；如果小于，或者 HandlePromotionFailure 设置不允许冒险，那么就要进行一次 Full GC。

#### 5.3.7. [#](#什么情况下会触发full-gc) 什么情况下会触发 Full GC？

对于 Minor GC，其触发条件非常简单，当 Eden 空间满时，就将触发一次 Minor GC。而 Full GC 则相对复杂，有以下条件:

*   **调用 System.gc()**

只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存。

*   **老年代空间不足**

老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。

为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。

*   **空间分配担保失败**

使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。

*   **JDK 1.7 及以前的永久代空间不足**

在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据。

当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。

为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。

*   **Concurrent Mode Failure**

执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足 (可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足)，便会报 Concurrent Mode Failure 错误，并触发 Full GC。

#### 5.3.8. [#](#hotspot中有哪些垃圾回收器) Hotspot 中有哪些垃圾回收器？

![](https://pdai.tech/images/pics/c625baa0-dde6-449e-93df-c3a67f2f430f.jpg)

以上是 HotSpot 虚拟机中的 7 个垃圾收集器，连线表示垃圾收集器可以配合使用。

*   单线程与多线程: 单线程指的是垃圾收集器只使用一个线程进行收集，而多线程使用多个线程；
*   串行与并行: 串行指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；并形指的是垃圾收集器和用户程序同时执行。除了 CMS 和 G1 之外，其它垃圾收集器都是以串行的方式执行。

1.  **Serial 收集器**

![](https://pdai.tech/images/pics/22fda4ae-4dd5-489d-ab10-9ebfdad22ae0.jpg)

Serial 翻译为串行，也就是说它以串行的方式执行。

它是单线程的收集器，只会使用一个线程进行垃圾收集工作。

它的优点是简单高效，对于单个 CPU 环境来说，由于没有线程交互的开销，因此拥有最高的单线程收集效率。

它是 Client 模式下的默认新生代收集器，因为在用户的桌面应用场景下，分配给虚拟机管理的内存一般来说不会很大。Serial 收集器收集几十兆甚至一两百兆的新生代停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿是可以接受的。

2.  **ParNew 收集器**

![](https://pdai.tech/images/pics/81538cd5-1bcf-4e31-86e5-e198df1e013b.jpg)

它是 Serial 收集器的多线程版本。

是 Server 模式下的虚拟机首选新生代收集器，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合工作。

默认开启的线程数量与 CPU 数量相同，可以使用 -XX:ParallelGCThreads 参数来设置线程数。

3.  **Parallel Scavenge 收集器**

与 ParNew 一样是多线程收集器。

其它收集器关注点是尽可能缩短垃圾收集时用户线程的停顿时间，而它的目标是达到一个可控制的吞吐量，它被称为 “吞吐量优先” 收集器。这里的吞吐量指 CPU 用于运行用户代码的时间占总时间的比值。

停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验。而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

缩短停顿时间是以牺牲吞吐量和新生代空间来换取的: 新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

可以通过一个开关参数打开 GC 自适应的调节策略 (GC Ergonomics)，就不需要手动指定新生代的大小 (-Xmn)、Eden 和 Survivor 区的比例、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。

1.  **Serial Old 收集器**

![](https://pdai.tech/images/pics/08f32fd3-f736-4a67-81ca-295b2a7972f2.jpg)

是 Serial 收集器的老年代版本，也是给 Client 模式下的虚拟机使用。如果用在 Server 模式下，它有两大用途:

*   在 JDK 1.5 以及之前版本 (Parallel Old 诞生以前) 中与 Parallel Scavenge 收集器搭配使用。
*   作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用。

5.  **Parallel Old 收集器**

![](https://pdai.tech/images/pics/278fe431-af88-4a95-a895-9c3b80117de3.jpg)

是 Parallel Scavenge 收集器的老年代版本。

在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器。

6.  **CMS 收集器**

![](https://pdai.tech/images/pics/62e77997-6957-4b68-8d12-bfd609bb2c68.jpg)

CMS(Concurrent Mark Sweep)，Mark Sweep 指的是标记 - 清除算法。

分为以下四个流程:

*   初始标记: 仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿。
*   并发标记: 进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿。
*   重新标记: 为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
*   并发清除: 不需要停顿。

在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿。

具有以下缺点:

*   吞吐量低: 低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高。
*   无法处理浮动垃圾，可能出现 Concurrent Mode Failure。浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS。
*   标记 - 清除算法导致的空间碎片，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC。

7.  **G1 收集器**

G1(Garbage-First)，它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。

堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代，而 **G1 可以直接对新生代和老年代一起回收**。

![](https://pdai.tech/images/pics/4cf711a8-7ab2-4152-b85c-d5c226733807.png)

G1 把堆划分成多个大小相等的独立区域 (Region)，新生代和老年代不再物理隔离。

![](https://pdai.tech/images/pics/9bbddeeb-e939-41f0-8e8e-2b1a0aa7e0a7.png)

**通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收**。这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间 (这两个值是通过过去回收的经验获得)，并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region。

每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region。通过使用 Remembered Set，在做可达性分析的时候就可以避免全堆扫描。

![](https://pdai.tech/images/pics/f99ee771-c56f-47fb-9148-c0036695b5fe.jpg)

如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤:

*   初始标记
*   并发标记
*   最终标记: 为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段需要停顿线程，但是可并行执行。
*   筛选回收: 首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

具备如下特点:

*   空间整合: 整体来看是基于 “标记 - 整理” 算法实现的收集器，从局部 (两个 Region 之间) 上来看是基于 “复制” 算法实现的，这意味着运行期间不会产生内存空间碎片。
*   可预测的停顿: 能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒。

### 5.4. [#](#_5-4-问题排查) 5.4 问题排查

#### 5.4.1. [#](#常见的linux定位问题的工具) 常见的 Linux 定位问题的工具？

*   文本操作
    *   文本查找 - grep
    *   文本分析 - awk
    *   文本处理 - sed
*   文件操作
    *   文件监听 - tail
    *   文件查找 - find
*   网络和进程
    *   网络接口 - ifconfig
    *   防火墙 - iptables -L
    *   路由表 - route -n
    *   netstat
*   其它常用
    *   进程 ps -ef | grep java
    *   分区大小 df -h
    *   内存 free -m
    *   硬盘大小 fdisk -l |grep Disk
    *   top
    *   环境变量 env

#### 5.4.2. [#](#jdk自带的定位问题的工具) JDK 自带的定位问题的工具？

*   **jps** jps 是 jdk 提供的一个查看当前 java 进程的小工具， 可以看做是 JavaVirtual Machine Process Status Tool 的缩写。
*   **jstack** jstack 是 jdk 自带的线程堆栈分析工具，使用该命令可以查看或导出 Java 应用程序中线程堆栈信息。

```
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj);
obj = null;
```

*   **jinfo** jinfo 是 JDK 自带的命令，可以用来查看正在运行的 java 应用程序的扩展参数，包括 Java System 属性和 JVM 命令行参数；也可以动态的修改正在运行的 JVM 一些参数。当系统崩溃时，jinfo 可以从 core 文件里面知道崩溃的 Java 应用程序的配置信息
*   **jmap** 命令 jmap 是一个多功能的命令。它可以生成 java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列。

```
jps –l # 输出输出完全的包名，应用主类名，jar的完全路径名
```

*   **jstat** jstat 参数众多，但是使用一个就够了

#### 5.4.3. [#](#如何使用在线调试工具arthas) 如何使用在线调试工具 Arthas？

举几个例子

*   **查看最繁忙的线程，以及是否有阻塞情况发生**?

场景：我想看下查看最繁忙的线程，以及是否有阻塞情况发生? 常规查看线程，一般我们可以通过 top 等系统命令进行查看，但是那毕竟要很多个步骤，很麻烦。

```
# 基本
jstack 2815
jstack -m 2815 # java和native c/c++框架的所有栈信息
jstack -l 2815 # 额外的锁信息列表，查看是否死锁
```

*   **确认某个类是否已被系统加载**?

场景：我新写了一个类或者一个方法，我想知道新写的代码是否被部署了?

```
jinfo 2815 # 输出当前 jvm 进程的全部参数和系统属性
```

*   **如何查看一个 class 类的源码信息**?

场景：我新修改的内容在方法内部，而上一个步骤只能看到方法，这时候可以反编译看下源码

```
# 查看堆的情况
jmap -heap 2815

# dump
jmap -dump:live,format=b,file=/tmp/heap2.bin 2815
```

*   **如何跟踪某个方法的返回值、入参**?

场景：我想看下我新加的方法在线运行的参数和返回值?

```
jstat -gcutil 2815 1000
```

*   **如何看方法调用栈的信息**?

场景：我想看下某个方法的调用栈的信息?

```
thread -n 3 # 查看最繁忙的三个线程栈信息
thread  # 以直观的方式展现所有的线程情况
thread -b #找出当前阻塞其他线程的线程
```

运行此命令之后需要即时触发方法才会有响应的信息打印在控制台上

*   **找到最耗时的方法调用**?

场景：testMethod 这个方法入口响应很慢，如何找到最耗时的子调用?

```
# 即可以找到需要的类全路径，如果存在的话
sc *MyServlet

# 查看这个某个类所有的方法
sm pdai.tech.servlet.TestMyServlet *

# 查看某个方法的信息，如果存在的话
sm pdai.tech.servlet.TestMyServlet testMethod
```

运行此命令之后需要即时触发方法才会有响应的信息打印在控制台上，然后一层一层看子调用。

*   **如何临时更改代码运行**?

场景：我找到了问题所在，能否线上直接修改测试，而不需要在本地改了代码后，重新打包部署，然后重启观察效果?

```
# 直接反编译出java 源代码，包含一此额外信息的
jad pdai.tech.servlet.TestMyServlet
```

如上，是直接更改线上代码的方式，但是一般好像是编译不成功的。所以，最好是本地 ide 编译成 class 文件后，再上传替换为好！

总之，已经完全不用重启和发布了！这个功能真的很方便，比起重启带来的代价，真的是不可比的。比如，重启时可能导致负载重分配，选主等等问题，就不是你能控制的了。

*   **我如何测试某个方法的性能问题**?

```
# 同时监控入参，返回值，及异常
watch pdai.tech.servlet.TestMyServlet testMethod "{params, returnObj, throwExp}" -e -x 2
```

#### 5.4.4. [#](#如何使用idea的远程调试) 如何使用 Idea 的远程调试？

要让远程服务器运行的代码支持远程调试，则启动的时候必须加上特定的 JVM 参数，这些参数是：

```
stack pdai.tech.servlet.TestMyServlet testMethod
```

#### 5.4.5. [#](#复杂综合类型问题的定位思路) 复杂综合类型问题的定位思路？

![](https://pdai.tech/images/jvm/java-jvm-debug.png)

## 6. [#](#_6-java-新版本) 6 Java 新版本

Java 8 版本特性，及 Java8 + 版本特性。

### 6.1. [#](#_6-1-java-8-特性) 6.1 Java 8 特性

#### 6.1.1. [#](#什么是函数式编程-lambda表达式) 什么是函数式编程？Lambda 表达式？

*   **函数式编程**

面向对象编程是对数据进行抽象；函数式编程是对行为进行抽象。

核心思想: 使用不可变值和函数，函数对一个值进行处理，映射成另一个值。

*   **Lambda 表达式**

lambda 表达式仅能放入如下代码: 预定义使用了 `@Functional` 注释的函数式接口，自带一个抽象函数的方法，或者 SAM(Single Abstract Method 单个抽象方法) 类型。这些称为 lambda 表达式的目标类型，可以用作返回类型，或 lambda 目标代码的参数。例如，若一个方法接收 Runnable、Comparable 或者 Callable 接口，都有单个抽象方法，可以传入 lambda 表达式。类似的，如果一个方法接受声明于 java.util.function 包内的接口，例如 Predicate、Function、Consumer 或 Supplier，那么可以向其传 lambda 表达式

#### 6.1.2. [#](#stream中常用方法) Stream 中常用方法？

*   `stream()`, `parallelStream()`
*   `filter()`
*   `findAny()` `findFirst()`
*   `sort`
*   `forEach` void
*   `map(), reduce()`
*   `flatMap()` - 将多个 Stream 连接成一个 Stream
*   `collect(Collectors.toList())`
*   `distinct`, `limit`
*   `count`
*   `min`, `max`, `summaryStatistics`

#### 6.1.3. [#](#什么是functionalinterface) 什么是 FunctionalInterface？

```
# 执行的时候每个子调用的运行时长，可以找到最耗时的子调用。
stack pdai.tech.servlet.TestMyServlet testMethod
```

*   interface 做注解的注解类型，被定义成 java 语言规
*   一个被它注解的接口只能有一个抽象方法，有两种例外
*   第一是接口允许有实现的方法，这种实现的方法是用 default 关键字来标记的 (java 反射中 java.lang.reflect.Method#isDefault() 方法用来判断是否是 default 方法)
*   第二如果声明的方法和 java.lang.Object 中的某个方法一样，它可以不当做未实现的方法，不违背这个原则: 一个被它注解的接口只能有一个抽象方法, 比如: `java public interface Comparator<T> { int compare(T o1, T o2); boolean equals(Object obj); }`
*   如果一个类型被这个注解修饰，那么编译器会要求这个类型必须满足如下条件:
    *   这个类型必须是一个 interface，而不是其他的注解类型、枚举 enum 或者类 class
    *   这个类型必须满足 function interface 的所有要求，如你个包含两个抽象方法的接口增加这个注解，会有编译错误。
*   编译器会自动把满足 function interface 要求的接口自动识别为 function interface。

#### 6.1.4. [#](#如何自定义函数接口) 如何自定义函数接口？

```
# 先反编译出class源码
jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java 
 # 然后使用外部工具编辑内容
mc /tmp/UserController.java -d /tmp  # 再编译成class
 # 最后，重新载入定义的类，就可以实时验证你的猜测了
redefine /tmp/com/example/demo/arthas/user/UserController.class
```

#### 6.1.5. [#](#内置的四大函数接口及使用) 内置的四大函数接口及使用？

*   **消费型接口: Consumer<T> void accept(T t) 有参数，无返回值的抽象方法**；

```
monitor -c 5 demo.MathGame primeFactors
```

*   **供给型接口: Supplier <T> T get() 无参有返回值的抽象方法**；

以 stream().collect(Collector<? super T, A, R> collector) 为例:

比如:

```
-Xdebug -Xrunjdwp:transport=dt_socket,suspend=n,server=y,address=127.0.0.1:5555
```

*   **断定型接口: Predicate<T> boolean test(T t): 有参，但是返回值类型是固定的 boolean**

比如: steam().filter() 中参数就是 Predicate

```
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface{}
```

*   **函数型接口: Function<T,R> R apply(T t) 有参有返回值的抽象方法**；

比如: steam().map() 中参数就是 Function<? super T, ? extends R>；reduce() 中参数 BinaryOperator<T> (ps: BinaryOperator<T> extends BiFunction<T,T,T>)

```
@FunctionalInterface
public interface IMyInterface {
    void study();
}

public class TestIMyInterface {
    public static void main(String[] args) {
        IMyInterface iMyInterface = () -> System.out.println("I like study");
        iMyInterface.study();
    }
}
```

#### 6.1.6. [#](#optional要解决什么问题) Optional 要解决什么问题？

在调用一个方法得到了返回值却不能直接将返回值作为参数去调用别的方法，我们首先要判断这个返回值是否为 null，只有在非空的前提下才能将其作为其他方法的参数。Java 8 引入了一个新的 Optional 类：这是一个可以为 null 的容器对象，如果值存在则 isPresent() 方法会返回 true，调用 get() 方法会返回该对象。

#### 6.1.7. [#](#如何使用optional来解决嵌套对象的判空问题) 如何使用 Optional 来解决嵌套对象的判空问题？

假设我们有一个像这样的类层次结构:

```
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

解决这种结构的深层嵌套路径是有点麻烦的。我们必须编写一堆 null 检查来确保不会导致一个 NullPointerException:

```
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
```

我们可以通过利用 Java 8 的 Optional 类型来摆脱所有这些 null 检查。map 方法接收一个 Function 类型的 lambda 表达式，并自动将每个 function 的结果包装成一个 Optional 对象。这使我们能够在一行中进行多个 map 操作。Null 检查是在底层自动处理的。

```
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

还有一种实现相同作用的方式就是通过利用一个 supplier 函数来解决嵌套路径的问题:

```
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);

backToString.apply("123");     // "123"
```

#### 6.1.8. [#](#什么是默认方法-为什么要有默认方法) 什么是默认方法，为什么要有默认方法？

就是接口可以有实现方法，而且不需要实现类去实现其方法。只需在方法名前面加个 default 关键字即可。

```
class Outer {
    Nested nested;
    Nested getNested() {
        return nested;
    }
}
class Nested {
    Inner inner;
    Inner getInner() {
        return inner;
    }
}
class Inner {
    String foo;
    String getFoo() {
        return foo;
    }
}
```

*   **为什么出现默认方法**？

首先，之前的接口是个双刃剑，好处是面向抽象而不是面向具体编程，缺陷是，当需要修改接口时候，需要修改全部实现该接口的类，目前的 java 8 之前的集合框架没有 foreach 方法，通常能想到的解决办法是在 JDK 里给相关的接口添加新的方法及实现。然而，对于已经发布的版本，是没法在给接口添加新方法的同时不影响已有的实现。所以引进的默认方法。他们的目的是为了解决接口的修改与现有的实现不兼容的问题。

#### 6.1.9. [#](#什么是类型注解) 什么是类型注解？

类型注解**被用来支持在 Java 的程序中做强类型检查。配合插件式的 check framework，可以在编译的时候检测出 runtime error，以提高代码质量**。这就是类型注解的作用了。

1.  在 java 8 之前，注解只能是在声明的地方所使用，比如类，方法，属性；
    
2.  java 8 里面，注解可以应用在任何地方，比如:

创建类实例

```
Outer outer = new Outer();
if (outer != null && outer.nested != null && outer.nested.inner != null) {
    System.out.println(outer.nested.inner.foo);
}
```

类型映射

```
Optional.of(new Outer())
    .map(Outer::getNested)
    .map(Nested::getInner)
    .map(Inner::getFoo)
    .ifPresent(System.out::println);
```

implements 语句中

```
Outer obj = new Outer();
resolve(() -> obj.getNested().getInner().getFoo())
    .ifPresent(System.out::println);
```

throw exception 声明

```
public interface A {
    default void foo(){
       System.out.println("Calling A.foo()");
    }
}

public class Clazz implements A {
    public static void main(String[] args){
       Clazz clazz = new Clazz();
       clazz.foo();//调用A.foo()
    }
}
```

需要注意的是，**类型注解只是语法而不是语义，并不会影响 java 的编译时间，加载时间，以及运行时间，也就是说，编译成 class 文件的时候并不包含类型注解**。

#### 6.1.10. [#](#什么是重复注解) 什么是重复注解？

允许在同一申明类型 (类，属性，或方法) 的多次使用同一个注解

*   **JDK8 之前**

java 8 之前也有重复使用注解的解决方案，但可读性不是很好，比如下面的代码:

```
new @Interned MyObject();
```

由另一个注解来存储重复注解，在使用时候，用存储注解 Authorities 来扩展重复注解。

*   **Jdk8 重复注解**

我们再来看看 java 8 里面的做法:

```
myString = (@NonNull String) str;
```

不同的地方是，创建重复注解 Authority 时，加上 @Repeatable, 指向存储注解 Authorities，在使用时候，直接可以重复使用 Authority 注解。从上面例子看出，java 8 里面做法更适合常规的思维，可读性强一点

### 6.2. [#](#_6-2-java-9-特性) 6.2 Java 9+ 特性

#### 6.2.1. [#](#java-9后续版本发布是按照什么样的发布策略呢) Java 9 后续版本发布是按照什么样的发布策略呢？

Java 现在发布的版本很快，每年两个，但是真正会被大规模使用的是三年一个的 TLS 版本。@pdai

*   每 3 年发布一个 TLS，长期维护版本。意味着 Java 8 ，Java 11， Java 17 才可能被大规模使用。
*   每年发布两个正式版本，分别是 3 月份和 9 月份。

#### 6.2.2. [#](#java-9后续新版本中你知道哪些) Java 9 后续新版本中你知道哪些？

能够举几个即可：

*   **Java10 - 并行全垃圾回收器 G1**

大家如果接触过 Java 性能调优工作，应该会知道，调优的最终目标是通过参数设置来达到快速、低延时的内存垃圾回收以提高应用吞吐量，尽可能的避免因内存回收不及时而触发的完整 GC（Full GC 会带来应用出现卡顿）。

G1 垃圾回收器是 Java 9 中 Hotspot 的默认垃圾回收器，是以一种低延时的垃圾回收器来设计的，旨在避免进行 Full GC，但是当并发收集无法快速回收内存时，会触发垃圾回收器回退进行 Full GC。之前 Java 版本中的 G1 垃圾回收器执行 GC 时采用的是基于单线程标记扫描压缩算法（mark-sweep-compact）。为了最大限度地减少 Full GC 造成的应用停顿的影响，Java 10 中将为 G1 引入多线程并行 GC，同时会使用与年轻代回收和混合回收相同的并行工作线程数量，从而减少了 Full GC 的发生，以带来更好的性能提升、更大的吞吐量。

Java 10 中将采用并行化 mark-sweep-compact 算法，并使用与年轻代回收和混合回收相同数量的线程。具体并行 GC 线程数量可以通过： `-XX：ParallelGCThreads` 参数来调节，但这也会影响用于年轻代和混合收集的工作线程数。

*   **Java11 - ZGC：可伸缩低延迟垃圾收集器**

ZGC 即 Z Garbage Collector（垃圾收集器或垃圾回收器），这应该是 Java 11 中最为瞩目的特性，没有之一。ZGC 是一个可伸缩的、低延迟的垃圾收集器，主要为了满足如下目标进行设计：

1.  GC 停顿时间不超过 10ms
2.  即能处理几百 MB 的小堆，也能处理几个 TB 的大堆
3.  应用吞吐能力不会下降超过 15%（与 G1 回收算法相比）
4.  方便在此基础上引入新的 GC 特性和利用 colord
5.  针以及 Load barriers 优化奠定基础
6.  当前只支持 Linux/x64 位平台

停顿时间在 10ms 以下，10ms 其实是一个很保守的数据，即便是 10ms 这个数据，也是 GC 调优几乎达不到的极值。根据 SPECjbb 2015 的基准测试，128G 的大堆下最大停顿时间才 1.68ms，远低于 10ms，和 G1 算法相比，改进非常明显。

![](https://pdai.tech/images/java/java-11-1.png)

*   **Java 14 - Switch 表达式**（正式版）

switch 表达式在之前的 Java 12 和 Java 13 中都是处于预览阶段，而在这次更新的 Java 14 中，终于成为稳定版本，能够正式可用。

switch 表达式带来的不仅仅是编码上的简洁、流畅，也精简了 switch 语句的使用方式，同时也兼容之前的 switch 语句的使用；之前使用 switch 语句时，在每个分支结束之前，往往都需要加上 break 关键字进行分支跳出，以防 switch 语句一直往后执行到整个 switch 语句结束，由此造成一些意想不到的问题。switch 语句一般使用冒号 ：来作为语句分支代码的开始，而 switch 表达式则提供了新的分支切换方式，即 -> 符号右则表达式方法体在执行完分支方法之后，自动结束 switch 分支，同时 -> 右则方法块中可以是表达式、代码块或者是手动抛出的异常。以往的 switch 语句写法如下：

```
class UnmodifiableList<T> implements @Readonly List<@Readonly T> { … }
```

而现在 Java 14 可以使用 switch 表达式正式版之后，上面语句可以转换为下列写法：

```
void monitorTemperature() throws @Critical TemperatureException { … }
```

很明显，switch 表达式将之前 switch 语句从编码方式上简化了不少，但是还是需要注意下面几点：

1.  需要保持与之前 switch 语句同样的 case 分支情况。
2.  之前需要用变量来接收返回值，而现在直接使用 yield 关键字来返回 case 分支需要返回的结果。
3.  现在的 switch 表达式中不再需要显式地使用 return、break 或者 continue 来跳出当前分支。
4.  现在不需要像之前一样，在每个分支结束之前加上 break 关键字来结束当前分支，如果不加，则会默认往后执行，直到遇到 break 关键字或者整个 switch 语句结束，在 Java 14 表达式中，表达式默认执行完之后自动跳出，不会继续往后执行。
5.  对于多个相同的 case 方法块，可以将 case 条件并列，而不需要像之前一样，通过每个 case 后面故意不加 break 关键字来使用相同方法块。

使用 switch 表达式来替换之前的 switch 语句，确实精简了不少代码，提高了编码效率，同时也可以规避一些可能由于不太经意而出现的意想不到的情况，可见 Java 在提高使用者编码效率、编码体验和简化使用方面一直在不停的努力中，同时也期待未来有更多的类似 lambda、switch 表达式这样的新特性出来。

*   **Java 14 - Records**

在 Java 14 中引入了 Record 类型，其效果有些类似 Lombok 的 @Data 注解、Kotlin 中的 data class，但是又不尽完全相同，它们的共同点都是类的部分或者全部可以直接在类头中定义、描述，并且这个类只用于存储数据而已。对于 Record 类型，具体可以用下面代码来说明：

```
public @interface Authority {
     String role();
}

public @interface Authorities {
    Authority[] value();
}

public class RepeatAnnotationUseOldVersion {

    @Authorities({@Authority(role="Admin"),@Authority(role="Manager")})
    public void doSomeThing(){
    }
}
```

对上述代码进行编译，然后反编译之后可以看到如下结果：

```
@Repeatable(Authorities.class)
public @interface Authority {
     String role();
}

public @interface Authorities {
    Authority[] value();
}

public class RepeatAnnotationUseNewVersion {
    @Authority(role="Admin")
    @Authority(role="Manager")
    public void doSomeThing(){ }
}
```

## 7. [#](#_7-数据结构和算法) 7 数据结构和算法

数据结构和算法

### 7.1. [#](#_7-1-数据结构基础) 7.1 数据结构基础

#### 7.1.1. [#](#如何理解基础的数据结构) 如何理解基础的数据结构？

避免孤立的学习知识点，要关联学习。比如实际应用当中，我们经常使用的是**查找，排序以及增删改**，这在我们的各种管理系统、数据库系统、操作系统等当中，十分常用，我们通过这个线索将知识点串联起来：

*   **数组**的下标寻址十分迅速，但计算机的内存是有限的，故数组的长度也是有限的，实际应用当中的数据往往十分庞大；而且无序数组的查找最坏情况需要遍历整个数组；后来人们提出了二分查找，二分查找要求数组的构造一定有序，二分法查找解决了普通数组查找复杂度过高的问题。任何一种数组无法解决的问题就是插入、删除操作比较复杂，因此，在一个增删查改比较频繁的数据结构中，数组不会被优先考虑
*   **普通链表**由于它的结构特点被证明根本不适合进行查找
*   **哈希表**是数组和链表的折中，同时它的设计依赖散列函数的设计，数组不能无限长、链表也不适合查找，所以也不适合大规模的查找
*   **二叉查找树**因为可能退化成链表，同样不适合进行查找
*   **AVL 树**是为了解决二叉查找树可能退化成链表问题。**AVL 树是严格的平衡二叉树**，平衡条件必须满足（所有节点的左右子树高度差的绝对值不超过 1）。不管我们是执行插入还是删除操作，只要不满足上面的条件，就要通过旋转来保持平衡，而旋转是非常耗时的，由此我们可以知道 AVL 树适合用于插入与删除次数比较少，但查找多的情况。
*   **红黑树**是二叉查找树和 AVL 树的折中。它是一种弱平衡二叉树，但在每个节点增加一个存储位表示节点的颜色，可以是红或黑（非红即黑）。通过对任何一条从根到叶子的路径上各个节点着色的方式的限制，红黑树确保没有一条路径会比其它路径长出两倍，因此，**红黑树是一种弱平衡二叉树**（由于是弱平衡，可以看到，在相同的节点情况下，AVL 树的高度低于红黑树），相对于要求严格的 AVL 树来说，它的旋转次数少，所以对于搜索，插入，删除操作较多的情况下，我们就用红黑树。
*   **多路查找树** 是大规模数据存储中，实现索引查询这样一个实际背景下，树节点存储的元素数量是有限的 (如果元素数量非常多的话，查找就退化成节点内部的线性查找了)，这样导致二叉查找树结构由于树的深度过大而造成磁盘 I/O 读写过于频繁，进而导致查询效率低下。
*   **B 树**与自平衡二叉查找树不同，B 树适用于读写相对大的数据块的存储系统，例如磁盘。它的应用是文件系统及部分非关系型数据库索引。
*   **B + 树**在 B 树基础上，为叶子结点增加链表指针 (B 树 + 叶子有序链表)，所有关键字都在叶子结点 中出现，非叶子结点作为叶子结点的索引；B + 树总是到叶子结点才命中。通常用于关系型数据库(如 Mysql) 和操作系统的文件系统中。
*   **B * 树**是 B + 树的变体，在 B + 树的非根和非叶子结点再增加指向兄弟的指针, 在 B + 树基础上，为非叶子结点也增加链表指针，将结点的最低利用率从 1/2 提高到 2/3。
*   **R 树**是用来做空间数据存储的树状数据结构。例如给地理位置，矩形和多边形这类多维数据建立索引。
*   **Trie 树**是自然语言处理中最常用的数据结构，很多字符串处理任务都会用到。Trie 树本身是一种有限状态自动机，还有很多变体。什么模式匹配、正则表达式，都与这有关。

### 7.2. [#](#_7-2-算法思想) 7.2 算法思想

#### 7.2.1. [#](#有哪些常见的算法思想) 有哪些常见的算法思想？

*   **分治算法** ：分治算法的基本思想是将一个规模为 N 的问题分解为 K 个规模较小的子问题，这些子问题相互独立且与原问题性质相同。求出子问题的解，就可得到原问题的解
*   **动态规划算法**： 通常用于求解具有某种最优性质的问题。在这类问题中，可能会有许多可行解。每一个解都对应于一个值，我们希望找到具有最优值的解。动态规划算法与分治法类似，其基本思想也是将待求解问题分解成若干个子问题，先求解子问题，然后从这些子问题的解得到原问题的解。和分治算法最大的差别：适用于动态规划算法求解的问题经过分解后得到的子问题往往不是相互独立的，而是下一个子阶段的求解是建立在上一个子阶段的解的基础上的。
*   **贪心算法**: 保证每次操作都是局部最优的，并且最后得到的结果是全局最优的
*   **二分法**： 比如重要的二分法，比如二分查找；二分查找也称折半查找（Binary Search），它是一种效率较高的查找方法。但是，折半查找要求线性表必须采用顺序存储结构，而且表中元素按关键字有序排列。
*   **搜索算法**： 主要包含 BFS，DFS
*   **Backtracking(回溯)**： 属于 DFS, 回溯算法实际上一个类似枚举的搜索尝试过程，主要是在搜索尝试过程中寻找问题的解，当发现已不满足求解条件时，就 “回溯” 返回，尝试别的路径。回溯法是一种选优搜索法，按选优条件向前搜索，以达到目标。但当探索到某一步时，发现原先选择并不优或达不到目标，就退回一步重新选择，这种走不通就退回再走的技术为回溯法

### 7.3. [#](#_7-3-常见排序算法) 7.3 常见排序算法

#### 7.3.1. [#](#有哪些常见的排序算法) 有哪些常见的排序算法？

在综合复杂度及稳定性情况下，通常**希尔, 快排和 归并**需要重点掌握

![](https://pdai.tech/images/alg/alg-sort-overview-1.png)

*   **冒泡排序** (Bubble Sort)
    *   它是一种较简单的排序算法。它会遍历若干次要排序的数列，每次遍历时，它都会从前往后依次的比较相邻两个数的大小；如果前者比后者大，则交换它们的位置。这样，一次遍历之后，最大的元素就在数列的末尾！ 采用相同的方法再次遍历时，第二大的元素就被排列在最大元素之前。重复此操作，直到整个数列都有序为止
*   **快速排序** (Quick Sort)
    *   它的基本思想是: 选择一个基准数，通过一趟排序将要排序的数据分割成独立的两部分；其中一部分的所有数据都比另外一部分的所有数据都要小。然后，再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。
*   **插入排序** (Insertion Sort)
    *   直接插入排序 (Straight Insertion Sort) 的基本思想是: 把 n 个待排序的元素看成为一个有序表和一个无序表。开始时有序表中只包含 1 个元素，无序表中包含有 n-1 个元素，排序过程中每次从无序表中取出第一个元素，将它插入到有序表中的适当位置，使之成为新的有序表，重复 n-1 次可完成排序过程。
*   **Shell 排序** (Shell Sort)
    *   希尔排序实质上是一种分组插入方法。它的基本思想是: 对于 n 个待排序的数列，取一个小于 n 的整数 gap(gap 被称为步长) 将待排序元素分成若干个组子序列，所有距离为 gap 的倍数的记录放在同一个组中；然后，对各组内的元素进行直接插入排序。 这一趟排序完成之后，每一个组的元素都是有序的。然后减小 gap 的值，并重复执行上述的分组和排序。重复这样的操作，当 gap=1 时，整个数列就是有序的。
*   **选择排序** (Selection sort)
    *   它的基本思想是: 首先在未排序的数列中找到最小 (or 最大) 元素，然后将其存放到数列的起始位置；接着，再从剩余未排序的元素中继续寻找最小 (or 最大) 元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。
*   **堆排序** (Heap Sort)
    *   堆排序是指利用堆这种数据结构所设计的一种排序算法。堆是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。
*   **归并排序** (Merge Sort)
    *   将两个的有序数列合并成一个有序数列，我们称之为 "归并"。归并排序 (Merge Sort) 就是利用归并思想对数列进行排序。
*   **桶排序** (Bucket Sort)
    *   桶排序 (Bucket Sort) 的原理很简单，将数组分到有限数量的桶子里。每个桶子再个别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序）
*   **基数排序** (Radix Sort)
    *   它的基本思想是: 将整数按位数切割成不同的数字，然后按每个位数分别比较。具体做法是: 将所有待比较数值统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列

### 7.4. [#](#_7-4-大数据处理算法) 7.4 大数据处理算法

#### 7.4.1. [#](#何谓海量数据处理-解决的思路) 何谓海量数据处理? 解决的思路？

所谓海量数据处理，无非就是基于海量数据上的存储、处理、操作。何谓海量，就是数据量太大，所以导致要么是无法在较短时间内迅速解决，要么是数据太大，导致无法一次性装入内存。

那解决办法呢?

*   **针对时间**: 我们可以采用巧妙的算法搭配合适的数据结构，如 Bloom filter/Hash/bit-map / 堆 / 数据库或倒排索引 / trie 树；
*   **针对空间**: 无非就一个办法: 大而化小，分而治之 (hash 映射);
*   **集群 | 分布式**: 通俗点来讲，单机就是处理装载数据的机器有限 (只要考虑 cpu，内存，硬盘的数据交互); 而集群适合分布式处理，并行计算 (更多考虑节点和节点间的数据交互)。

#### 7.4.2. [#](#大数据处理之分治思想) 大数据处理之分治思想？

分而治之 / hash 映射 + hash 统计 + 堆 / 快速 / 归并排序，说白了，就是先映射，而后统计，最后排序:

*   **分而治之 / hash 映射**: 针对数据太大，内存受限，只能是: 把大文件化成 (取模映射) 小文件，即 16 字方针: 大而化小，各个击破，缩小规模，逐个解决
    
*   **hash_map 统计**: 当大文件转化了小文件，那么我们便可以采用常规的 hash_map(ip，value) 来进行频率统计。
    
*   **堆 / 快速排序**: 统计完了之后，便进行排序 (可采取堆排序)，得到次数最多的 IP。

#### 7.4.3. [#](#海量日志数据-提取出某日访问百度次数最多的那个ip) 海量日志数据，提取出某日访问百度次数最多的那个 IP?

分析: “首先是这一天，并且是访问百度的日志中的 IP 取出来，逐个写入到一个大文件中。注意到 IP 是 32 位的，最多有个 2^32 个 IP。同样可以采用映射的方法，比如 %1000，把整个大文件映射为 1000 个小文件，再找出每个小文中出现频率最大的 IP(可以采用 hash_map 对那 1000 个文件中的所有 IP 进行频率统计，然后依次找出各个文件中频率最大的那个 IP) 及相应的频率。然后再在这 1000 个最大的 IP 中，找出那个频率最大的 IP，即为所求。”

关于本题，还有几个问题，如下:

*   Hash 取模是一种等价映射，不会存在同一个元素分散到不同小文件中的情况，即这里采用的是 mod1000 算法，那么相同的 IP 在 hash 取模后，只可能落在同一个文件中，不可能被分散的。因为如果两个 IP 相等，那么经过 Hash(IP) 之后的哈希值是相同的，将此哈希值取模 (如模 1000)，必定仍然相等。
*   那到底什么是 hash 映射呢? 简单来说，就是为了便于计算机在有限的内存中处理 big 数据，从而通过一种映射散列的方式让数据均匀分布在对应的内存位置 (如大数据通过取余的方式映射成小树存放在内存中，或大文件映射成多个小文件)，而这个映射散列方式便是我们通常所说的 hash 函数，设计的好的 hash 函数能让数据均匀分布而减少冲突。尽管数据映射到了另外一些不同的位置，但数据还是原来的数据，只是代替和表示这些原始数据的形式发生了变化而已。

#### 7.4.4. [#](#寻找热门查询-300万个查询字符串中统计最热门的10个查询) 寻找热门查询，300 万个查询字符串中统计最热门的 10 个查询?

原题: 搜索引擎会通过日志文件把用户每次检索使用的所有检索串都记录下来，每个查询串的长度为 1-255 字节。假设目前有一千万个记录 (这些查询串的重复度比较高，虽然总数是 1 千万，但如果除去重复后，不超过 3 百万个。一个查询串的重复度越高，说明查询它的用户越多，也就是越热门)，请你统计最热门的 10 个查询串，要求使用的内存不能超过 1G。

解答: 由上面第 1 题，我们知道，数据大则划为小的，如如一亿个 Ip 求 Top 10，可先 %1000 将 ip 分到 1000 个小文件中去，并保证一种 ip 只出现在一个文件中，再对每个小文件中的 ip 进行 hashmap 计数统计并按数量排序，最后归并或者最小堆依次处理每个小文件的 top10 以得到最后的结。

但如果数据规模比较小，能一次性装入内存呢? 比如这第 2 题，虽然有一千万个 Query，但是由于重复度比较高，因此事实上只有 300 万的 Query，每个 Query255Byte，因此我们可以考虑把他们都放进内存中去 (300 万个字符串假设没有重复，都是最大长度，那么最多占用内存 3M*1K/4=0.75G。所以可以将所有字符串都存放在内存中进行处理)，而现在只是需要一个合适的数据结构，在这里，HashTable 绝对是我们优先的选择。

所以我们放弃分而治之 / hash 映射的步骤，直接上 hash 统计，然后排序。So，针对此类典型的 TOP K 问题，采取的对策往往是: hashmap + 堆。如下所示:

*   **hash_map 统计**: 先对这批海量数据预处理。具体方法是: 维护一个 Key 为 Query 字串，Value 为该 Query 出现次数的 HashTable，即 hash_map(Query，Value)，每次读取一个 Query，如果该字串不在 Table 中，那么加入该字串，并且将 Value 值设为 1；如果该字串在 Table 中，那么将该字串的计数加一即可。最终我们在 O(N) 的时间复杂度内用 Hash 表完成了统计； 堆排序: 第二步、借助堆这个数据结构，找出 Top K，时间复杂度为 N‘logK。即借助堆结构，我们可以在 log 量级的时间内查找和调整 / 移动。因此，维护一个 K(该题目中是 10) 大小的小根堆，然后遍历 300 万的 Query，分别和根元素进行对比。所以，我们最终的时间复杂度是: O(N) + N' * O(logK)，(N 为 1000 万，N’为 300 万)。

别忘了这篇文章中所述的堆排序思路: “维护 k 个元素的最小堆，即用容量为 k 的最小堆存储最先遍历到的 k 个数，并假设它们即是最大的 k 个数，建堆费时 O(k)，并调整堆 (费时 O(logk)) 后，有 k1>k2>...kmin(kmin 设为小顶堆中最小元素)。继续遍历数列，每次遍历一个元素 x，与堆顶元素比较，若 x>kmin，则更新堆(x 入堆，用时 logk)，否则不更新堆。这样下来，总费时 O(k*logk+(n-k)_logk)=O(n_logk)。此方法得益于在堆中，查找等各项操作时间复杂度均为 logk。”-- 第三章续、Top K 算法问题的实现。

当然，你也可以采用 trie 树，关键字域存该查询串出现的次数，没有出现为 0。最后用 10 个元素的最小推来对出现频率进行排序。

#### 7.4.5. [#](#有一个1g大小的一个文件-里面每一行是一个词-词的大小不超过16字节-内存限制大小是1m。返回频数最高的100个词) 有一个 1G 大小的一个文件，里面每一行是一个词，词的大小不超过 16 字节，内存限制大小是 1M。返回频数最高的 100 个词?

*   **分而治之 / hash 映射**: 顺序读文件中，对于每个词 x，取 hash(x)%5000，然后按照该值存到 5000 个小文件 (记为 x0,x1,...x4999) 中。这样每个文件大概是 200k 左右。如果其中的有的文件超过了 1M 大小，还可以按照类似的方法继续往下分，直到分解得到的小文件的大小都不超过 1M。
*   **hash_map 统计**: 对每个小文件，采用 trie 树 / hash_map 等统计每个文件中出现的词以及相应的频率。
*   **堆 / 归并排序**: 取出出现频率最大的 100 个词 (可以用含 100 个结点的最小堆) 后，再把 100 个词及相应的频率存入文件，这样又得到了 5000 个文件。最后就是把这 5000 个文件进行归并 (类似于归并排序) 的过程了。

#### 7.4.6. [#](#海量数据分布在100台电脑中-想个办法高效统计出这批数据的top10) 海量数据分布在 100 台电脑中，想个办法高效统计出这批数据的 TOP10?

如果每个数据元素只出现一次，而且只出现在某一台机器中，那么可以采取以下步骤统计出现次数 TOP10 的数据元素:

*   **堆排序**: 在每台电脑上求出 TOP10，可以采用包含 10 个元素的堆完成 (TOP10 小，用最大堆，TOP10 大，用最小堆，比如求 TOP10 大，我们首先取前 10 个元素调整成最小堆，如果发现，然后扫描后面的数据，并与堆顶元素比较，如果比堆顶元素大，那么用该元素替换堆顶，然后再调整为最小堆。最后堆中的元素就是 TOP10 大)。 求出每台电脑上的 TOP10 后，然后把这 100 台电脑上的 TOP10 组合起来，共 1000 个数据，再利用上面类似的方法求出 TOP10 就可以了。

但如果同一个元素重复出现在不同的电脑中呢，如下例子所述, 这个时候，你可以有两种方法:

*   遍历一遍所有数据，重新 hash 取摸，如此使得同一个元素只出现在单独的一台电脑中，然后采用上面所说的方法，统计每台电脑中各个元素的出现次数找出 TOP10，继而组合 100 台电脑上的 TOP10，找出最终的 TOP10。
*   或者，暴力求解: 直接统计统计每台电脑中各个元素的出现次数，然后把同一个元素在不同机器中的出现次数相加，最终从所有数据中找出 TOP10。

#### 7.4.7. [#](#有10个文件-每个文件1g-每个文件的每一行存放的都是用户的query-每个文件的query都可能重复。要求你按照query的频度排序) 有 10 个文件，每个文件 1G，每个文件的每一行存放的都是用户的 query，每个文件的 query 都可能重复。要求你按照 query 的频度排序?

方案 1:

*   **hash 映射**: 顺序读取 10 个文件，按照 hash(query)%10 的结果将 query 写入到另外 10 个文件 (记为 a0,a1,..a9) 中。这样新生成的文件每个的大小大约也 1G(假设 hash 函数是随机的)。
*   **hash_map 统计**: 找一台内存在 2G 左右的机器，依次对用 hash_map(query, query_count) 来统计每个 query 出现的次数。注: hash_map(query,query_count) 是用来统计每个 query 的出现次数，不是存储他们的值，出现一次，则 count+1。
*   **堆 / 快速 / 归并排序**: 利用快速 / 堆 / 归并排序按照出现次数进行排序，将排序好的 query 和对应的 query_cout 输出到文件中，这样得到了 10 个排好序的文件 (记为)。最后，对这 10 个文件进行归并排序 (内排序与外排序相结合)。

方案 2: 一般 query 的总量是有限的，只是重复的次数比较多而已，可能对于所有的 query，一次性就可以加入到内存了。这样，我们就可以采用 trie 树 / hash_map 等直接来统计每个 query 出现的次数，然后按出现次数做快速 / 堆 / 归并排序就可以了。

方案 3: 与方案 1 类似，但在做完 hash，分成多个文件后，可以交给多个文件来处理，采用分布式的架构来处理 (比如 MapReduce)，最后再进行合并。

#### 7.4.8. [#](#给定a、b两个文件-各存放50亿个url-每个url各占64字节-内存限制是4g-让你找出a、b文件共同的url) 给定 a、b 两个文件，各存放 50 亿个 url，每个 url 各占 64 字节，内存限制是 4G，让你找出 a、b 文件共同的 url?

可以估计每个文件安的大小为 5G×64=320G，远远大于内存限制的 4G。所以不可能将其完全加载到内存中处理。考虑采取分而治之的方法。

*   **分而治之 / hash 映射**: 遍历文件 a，对每个 url 求取，然后根据所取得的值将 url 分别存储到 1000 个小文件 (记为，这里漏写个了 a1) 中。这样每个小文件的大约为 300M。遍历文件 b，采取和 a 相同的方式将 url 分别存储到 1000 小文件中 (记为)。这样处理后，所有可能相同的 url 都在对应的小文件() 中，不对应的小文件不可能有相同的 url。然后我们只要求出 1000 对小文件中相同的 url 即可。
*   **hash_set 统计**: 求每对小文件中相同的 url 时，可以把其中一个小文件的 url 存储到 hash_set 中。然后遍历另一个小文件的每个 url，看其是否在刚才构建的 hash_set 中，如果是，那么就是共同的 url，存到文件里面就可以了。

如果允许有一定的错误率，可以使用 Bloom filter，4G 内存大概可以表示 340 亿 bit。将其中一个文件中的 url 使用 Bloom filter 映射为这 340 亿 bit，然后挨个读取另外一个文件的 url，检查是否与 Bloom filter，如果是，那么该 url 应该是共同的 url(注意会有一定的错误率)。”

#### 7.4.9. [#](#怎么在海量数据中找出重复次数最多的一个) 怎么在海量数据中找出重复次数最多的一个?

方案: 先做 hash，然后求模映射为小文件，求出每个小文件中重复次数最多的一个，并记录重复次数。然后找出上一步求出的数据中重复次数最多的一个就是所求 (具体参考前面的题)。

#### 7.4.10. [#](#上千万或上亿数据-有重复-统计其中出现次数最多的前n个数据) 上千万或上亿数据 (有重复)，统计其中出现次数最多的前 N 个数据?

方案: 上千万或上亿的数据，现在的机器的内存应该能存下。所以考虑采用 hash_map / 搜索二叉树 / 红黑树等来进行统计次数。然后利用堆取出前 N 个出现次数最多的数据。

#### 7.4.11. [#](#一个文本文件-大约有一万行-每行一个词-要求统计出其中最频繁出现的前10个词-请给出思想-给出时间复杂度分析) 一个文本文件，大约有一万行，每行一个词，要求统计出其中最频繁出现的前 10 个词，请给出思想，给出时间复杂度分析?

方案 1: 如果文件比较大，无法一次性读入内存，可以采用 hash 取模的方法，将大文件分解为多个小文件，对于单个小文件利用 hash_map 统计出每个小文件中 10 个最常出现的词，然后再进行归并处理，找出最终的 10 个最常出现的词。

方案 2: 通过 hash 取模将大文件分解为多个小文件后，除了可以用 hash_map 统计出每个小文件中 10 个最常出现的词，也可以用 trie 树统计每个词出现的次数，时间复杂度是 O(n_le)(le 表示单词的平准长度)，最终同样找出出现最频繁的前 10 个词 (可用堆来实现)，时间复杂度是 O(n_lg10)。

#### 7.4.12. [#](#一个文本文件-找出前10个经常出现的词-但这次文件比较长-说是上亿行或十亿行-总之无法一次读入内存-问最优解) 一个文本文件，找出前 10 个经常出现的词，但这次文件比较长，说是上亿行或十亿行，总之无法一次读入内存，问最优解?

方案 1: 首先根据用 hash 并求模，将文件分解为多个小文件，对于单个文件利用上题的方法求出每个文件件中 10 个最常出现的词。然后再进行归并处理，找出最终的 10 个最常出现的词。

#### 7.4.13. [#](#_100w个数中找出最大的100个数) 100w 个数中找出最大的 100 个数?

方案 1: 采用局部淘汰法。选取前 100 个元素，并排序，记为序列 L。然后一次扫描剩余的元素 x，与排好序的 100 个元素中最小的元素比，如果比这个最小的要大，那么把这个最小的元素删除，并把 x 利用插入排序的思想，插入到序列 L 中。依次循环，知道扫描了所有的元素。复杂度为 O(100w*100)。

方案 2: 采用快速排序的思想，每次分割之后只考虑比轴大的一部分，知道比轴大的一部分在比 100 多的时候，采用传统排序算法排序，取前 100 个。复杂度为 O(100w*100)。

方案 3: 在前面的题中，我们已经提到了，用一个含 100 个元素的最小堆完成。复杂度为 O(100w*lg100)。

#### 7.4.14. [#](#_5亿个int找它们的中位数) 5 亿个 int 找它们的中位数?

*   **思路一**

这个例子比上面那个更明显。首先我们将 int 划分为 2^16 个区域，然后读取数据统计落到各个区域里的数的个数，之后我们根据统计结果就可以判断中位数落到那个区域，同时知道这个区域中的第几大数刚好是中位数。然后第二次扫描我们只统计落在这个区域中的那些数就可以了。

实际上，如果不是 int 是 int64，我们可以经过 3 次这样的划分即可降低到可以接受的程度。即可以先将 int64 分成 224 个区域，然后确定区域的第几大数，在将该区域分成 220 个子区域，然后确定是子区域的第几大数，然后子区域里的数的个数只有 2^20，就可以直接利用 direct addr table 进行统计了。

*   **思路二**

同样需要做两遍统计，如果数据存在硬盘上，就需要读取 2 次。

方法同基数排序有些像，开一个大小为 65536 的 Int 数组，第一遍读取，统计 Int32 的高 16 位的情况，也就是 0-65535，都算作 0,65536 - 131071 都算作 1。就相当于用该数除以 65536。Int32 除以 65536 的结果不会超过 65536 种情况，因此开一个长度为 65536 的数组计数就可以。每读取一个数，数组中对应的计数 + 1，考虑有负数的情况，需要将结果加 32768 后，记录在相应的数组内。

第一遍统计之后，遍历数组，逐个累加统计，看中位数处于哪个区间，比如处于区间 k，那么 0- k-1 的区间里数字的数量 sum 应该`<n/2`(2.5 亿)。而 k+1 - 65535 的计数和也`<n/2`，第二遍统计同上面的方法类似，但这次只统计处于区间 k 的情况，也就是说 (x / 65536) + 32768 = k。统计只统计低 16 位的情况。并且利用刚才统计的 sum，比如 sum = 2.49 亿，那么现在就是要在低 16 位里面找 100 万个数 (2.5 亿 - 2.49 亿)。这次计数之后，再统计一下，看中位数所处的区间，最后将高位和低位组合一下就是结果了。

#### 7.4.15. [#](#在2-5亿个整数中找出不重复的整数-注-内存不足以容纳这2-5亿个整数。) 在 2.5 亿个整数中找出不重复的整数，注，内存不足以容纳这 2.5 亿个整数。

*   方案 1: 采用 2-Bitmap(每个数分配 2bit，00 表示不存在，01 表示出现一次，10 表示多次，11 无意义) 进行，共需内存 2^32 * 2 bit=1 GB 内存，还可以接受。然后扫描这 2.5 亿个整数，查看 Bitmap 中相对应位，如果是 00 变 01，01 变 10，10 保持不变。所描完事后，查看 bitmap，把对应位是 01 的整数输出即可。
    
*   方案 2: 也可采用分治，划分小文件的方法。然后在小文件中找出不重复的整数，并排序。然后再进行归并，注意去除重复的元素。

#### 7.4.16. [#](#给40亿个不重复的unsigned-int的整数-没排过序的-然后再给一个数-如何快速判断这个数是否在那40亿个数当中) 给 40 亿个不重复的 unsigned int 的整数，没排过序的，然后再给一个数，如何快速判断这个数是否在那 40 亿个数当中?

用位图 / Bitmap 的方法，申请 512M 的内存，一个 bit 位代表一个 unsigned int 值。读入 40 亿个数，设置相应的 bit 位，读入要查询的数，查看相应 bit 位是否为 1，为 1 表示存在，为 0 表示不存在。

### 7.5. [#](#_7-5-加密算法) 7.5 加密算法

#### 7.5.1. [#](#什么是摘要算法-有哪些) 什么是摘要算法？有哪些?

消息摘要算法的主要特征是加密过程不需要密钥，并且经过加密的数据无法被解密，目前可以解密逆向的只有 CRC32 算法，只有输入相同的明文数据经过相同的消息摘要算法才能得到相同的密文。消息摘要算法不存在密钥的管理与分发问题，适合于分布式网络上使用。消息摘要算法主要应用在 “数字签名” 领域，作为对明文的摘要算法。

*   **何谓数字签名？**

数字签名主要用到了非对称密钥加密技术与数字摘要技术。数字签名技术是将摘要信息用发送者的私钥加密，与原文一起传送给接收者。接收者只有用发送者的公钥才能解密被加密的摘要信息，然后用 HASH 函数对收到的原文产生一个摘要信息，与解密的摘要信息对比。如果相同，则说明收到的信息是完整的，在传输过程中没有被修改，否则说明信息被修改过.

因此数字签名能够验证信息的完整性。

数字签名是个加密的过程，数字签名验证是个解密的过程。

*   **有哪些摘要算法**？

著名的摘要算法有 RSA 公司的 MD5 算法和 SHA-1 算法及其大量的变体

#### 7.5.2. [#](#什么是加密算法-有哪些) 什么是加密算法？有哪些？

数据加密的基本过程就是对原来为明文的文件或数据按某种算法进行处理，使其成为不可读的一段代码为 “密文”，使其只能在输入相应的密钥之后才能显示出原容，通过这样的途径来达到保护数据不被非法人窃取、阅读的目的。 该过程的逆过程为解密，即将该编码信息转化为其原来数据的过程。

**加密算法分类**

密钥加密技术的密码体制分为**对称密钥**体制和**非对称密**钥体制两种。相应地，对数据加密的技术分为两类，即对称加密 (私人密钥加密) 和非对称加密(公开密钥加密)。

对称加密以数据加密标准 (**DES**，Data Encryption Standard) 算法为典型代表，非对称加密通常以 **RSA**(Rivest Shamir Adleman) 算法为代表。

对称加密的加密密钥和解密密钥相同。非对称加密的加密密钥和解密密钥不同，加密密钥可以公开而解密密钥需要保密

#### 7.5.3. [#](#什么是国密算法-有哪些) 什么是国密算法？有哪些？

*   SM1 **为对称加密**。其加密强度与 AES 相当。该算法不公开，调用该算法时，需要通过**加密芯片的接口进行调用**。
*   SM2 **非对称加密**，基于 ECC。该算法已公开。由于该算法基于 ECC，故其签名速度与秘钥生成速度都快于 RSA。ECC 256 位（SM2 采用的就是 ECC 256 位的一种）安全强度比 RSA 2048 位高，但运算速度快于 RSA。
*   SM3 **消息摘要**。可以用 MD5 作为对比理解。该算法已公开。校验结果为 256 位。
*   SM4 无线局域网标准的**分组数据算法**。对称加密，密钥长度和分组长度均为 128 位。
*   SM7 是一种分组密码算法，分组长度为 128 比特，密钥长度为 128 比特。SM7 适用于非接触式 IC 卡，应用包括身份识别类应用 (门禁卡、工作证、参赛证)，票务类应用 (大型赛事门票、展会门票)，支付与通卡类应用（积分消费卡、校园一卡通、企业一卡通等）。
*   SM9 不需要申请数字证书，适用于互联网应用的各种新兴应用的安全保障。如基于云技术的密码服务、电子邮件安全、智能终端保护、物联网安全、云存储安全等等。这些安全应用可采用手机号码或邮件地址作为公钥，实现数据加密、身份认证、通话加密、通道加密等安全应用，并具有使用方便，易于部署的特点，从而开启了普及密码算法的大门。

## 8. [#](#_8-数据库) 8 数据库

数据库相关，包括 MySQL，Redis，MongoDB， ElasticSearch 等。

### 8.1. [#](#_8-1-原理和sql) 8.1 原理和 SQL

#### 8.1.1. [#](#什么是事务-事务基本特性acid) 什么是事务？事务基本特性 ACID？

事务指的是满足 ACID 特性的一组操作，可以通过 Commit 提交一个事务，也可以使用 Rollback 进行回滚。

![](https://pdai.tech/images/pics/185b9c49-4c13-4241-a848-fbff85c03a64.png)

*   **事务基本特性 ACID**?：
    *   **A 原子性 (atomicity)** 指的是一个事务中的操作要么全部成功，要么全部失败。
    *   **C 一致性 (consistency)** 指的是数据库总是从一个一致性的状态转换到另外一个一致性的状态。比如 A 转账给 B100 块钱，假设中间 sql 执行过程中系统崩溃 A 也不会损失 100 块，因为事务没有提交，修改也就不会保存到数据库。
    *   **I 隔离性 (isolation)** 指的是一个事务的修改在最终提交前，对其他事务是不可见的。
    *   **D 持久性 (durability)** 指的是一旦事务提交，所做的修改就会永久保存到数据库中。

#### 8.1.2. [#](#数据库中并发一致性问题) 数据库中并发一致性问题？

在并发环境下，事务的隔离性很难保证，因此会出现很多并发一致性问题。

*   **丢失修改**

T1 和 T2 两个事务都对一个数据进行修改，T1 先修改，T2 随后修改，T2 的修改覆盖了 T1 的修改。

![](https://pdai.tech/images/pics/88ff46b3-028a-4dbb-a572-1f062b8b96d3.png)

*   **读脏数据**

T1 修改一个数据，T2 随后读取这个数据。如果 T1 撤销了这次修改，那么 T2 读取的数据是脏数据。

![](https://pdai.tech/images/pics/dd782132-d830-4c55-9884-cfac0a541b8e.png)

*   **不可重复读**

T2 读取一个数据，T1 对该数据做了修改。如果 T2 再次读取这个数据，此时读取的结果和第一次读取的结果不同。

![](https://pdai.tech/images/pics/c8d18ca9-0b09-441a-9a0c-fb063630d708.png)

*   **幻影读**

T1 读取某个范围的数据，T2 在这个范围内插入新的数据，T1 再次读取这个范围的数据，此时读取的结果和和第一次读取的结果不同。

![](https://pdai.tech/images/pics/72fe492e-f1cb-4cfc-92f8-412fb3ae6fec.png)

#### 8.1.3. [#](#事务的隔离等级) 事务的隔离等级？

*   **未提交读 (READ UNCOMMITTED)** 事务中的修改，即使没有提交，对其它事务也是可见的。
*   **提交读 (READ COMMITTED)** 一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其它事务是不可见的。
*   **可重复读 (REPEATABLE READ)** 保证在同一个事务中多次读取同样数据的结果是一样的。
*   **可串行化 (SERIALIZABLE)** 强制事务串行执行。

<table><thead><tr><th>隔离级别</th><th>脏读</th><th>不可重复读</th><th>幻影读</th></tr></thead><tbody><tr><td>未提交读</td><td>√</td><td>√</td><td>√</td></tr><tr><td>提交读</td><td>×</td><td>√</td><td>√</td></tr><tr><td>可重复读</td><td>×</td><td>×</td><td>√</td></tr><tr><td>可串行化</td><td>×</td><td>×</td><td>×</td></tr></tbody></table>

#### 8.1.4. [#](#acid靠什么保证的呢) ACID 靠什么保证的呢？

*   **A 原子性 (atomicity)** 由 undo log 日志保证，它记录了需要回滚的日志信息，事务回滚时撤销已经执行成功的 sql
*   **C 一致性 (consistency)** 一般由代码层面来保证
*   **I 隔离性 (isolation)** 由 MVCC 来保证
*   **D 持久性 (durability)** 由内存 + redo log 来保证，mysql 修改数据同时在内存和 redo log 记录这次操作，事务提交的时候通过 redo log 刷盘，宕机的时候可以从 redo log 恢复

#### 8.1.5. [#](#sql-优化的实践经验) SQL 优化的实践经验？

1. 对查询进行优化，要尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。
2. 应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：

```
int dayOfWeek;
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        dayOfWeek = 6;
        break;
    case TUESDAY:
        dayOfWeek = 7;
        break;
    case THURSDAY:
    case SATURDAY:
        dayOfWeek = 8;
        break;
    case WEDNESDAY:
        dayOfWeek = 9;
        break;
    default:
        dayOfWeek = 0;
        break;
}
```

最好不要给数据库留 NULL，尽可能的使用 NOT NULL 填充数据库.

备注、描述、评论之类的可以设置为 NULL，其他的，最好不要使用 NULL。

不要以为 NULL 不需要空间，比如：char(100) 型，在字段建立时，空间就固定了， 不管是否插入值（NULL 也包含在内），都是占用 100 个字符的空间的，如果是 varchar 这样的变长字段， null 不占用空间。

可以在 num 上设置默认值 0，确保表中 num 列没有 null 值，然后这样查询：

```
int dayOfWeek = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
case WEDNESDAY              -> 9;
    default              -> 0;

};
```

3. 应尽量避免在 where 子句中使用 != 或 <> 操作符，否则将引擎放弃使用索引而进行全表扫描。
4. 应尽量避免在 where 子句中使用 or 来连接条件，如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描，如：

```
public record Person(String name, int age) {
    public static String address;

    public String getName() {
        return name;
    }
}
```

可以这样查询：

```
public final class Person extends java.lang.Record {
    private final java.lang.String name;
    private final java.lang.String age;

    public Person(java.lang.String name, java.lang.String age) { /* compiled code */ }

    public java.lang.String getName() { /* compiled code */ }

    public java.lang.String toString() { /* compiled code */ }

    public final int hashCode() { /* compiled code */ }

    public final boolean equals(java.lang.Object o) { /* compiled code */ }

    public java.lang.String name() { /* compiled code */ }

    public java.lang.String age() { /* compiled code */ }
}
```

5.in 和 not in 也要慎用，否则会导致全表扫描，如：

```
select id from t where num is null
```

对于连续的数值，能用 between 就不要用 in 了：

```
select id from t where num = 0
```

很多时候用 exists 代替 in 是一个好的选择：

```
select id from t where num=10 or Name = 'admin'
```

用下面的语句替换：

```
select id from t where num = 10
union all
select id from t where Name = 'admin'
```

6. 下面的查询也将导致全表扫描：

```
select id from t where num in(1,2,3)
```

若要提高效率，可以考虑全文检索。

7. 如果在 where 子句中使用参数，也会导致全表扫描。因为 SQL 只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然 而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：

```
select id from t where num between 1 and 3
```

可以改为强制查询使用索引：

```
select num from a where num in(select num from b)
```

. 应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：

```
select num from a where exists(select 1 from b where num=a.num)
```

应改为:

```
select id from t where name like ‘%abc%’
```

9. 应尽量避免在 where 子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：

```
select id from t where num = @num
```

应改为:

```
select id from t with(index(索引名)) where num = @num
```

10. 不要在 where 子句中的 “=” 左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。
11. 在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。
12. 不要写一些没有意义的查询，如需要生成一个空表结构：

这类代码不会返回任何结果集，但是会消耗系统资源的，应改成这样：

13.Update 语句，如果只更改 1、2 个字段，不要 Update 全部字段，否则频繁调用会引起明显的性能消耗，同时带来大量日志。

14. 对于多张大数据量（这里几百条就算大了）的表 JOIN，要先分页再 JOIN，否则逻辑读会很高，性能很差。

15.select count(*) from table；这样不带任何条件的 count 会引起全表扫描，并且没有任何业务意义，是一定要杜绝的。

16. 索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过 6 个，若太多则应考虑一些不常使用到的列上建的索引是否有 必要。
17. 应尽可能的避免更新 clustered 索引数据列，因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。
18. 尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连 接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。
19. 尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。
20. 任何地方都不要使用 select * from t ，用具体的字段列表代替 “*”，不要返回用不到的任何字段。
21. 尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）。
22.  避免频繁创建和删除临时表，以减少系统表资源的消耗。临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用表中的某个数据集时。但是，对于一次性事件， 最好使用导出表。
23. 在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table，避免造成大量 log ，以提高速度；如果数据量不大，为了缓和系统表的资源，应先 create table，然后 insert。
24. 如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样可以避免系统表的较长时间锁定。
25. 尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过 1 万行，那么就应该考虑改写。
26. 使用基于游标的方法或临时表方法之前，应先寻找基于集的解决方案来解决问题，基于集的方法通常更有效。
27. 与临时表一样，游标并不是不可使用。对小型数据集使用 FAST_FORWARD 游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括 “合计” 的例程通常要比使用游标执行的速度快。如果开发时 间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好。
28. 在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。
29. 尽量避免大事务操作，提高系统并发能力。
30. 尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。

#### 8.1.6. [#](#buffer-pool、redo-log-buffer-和undo-log、redo-log、bin-log-概念以及关系) Buffer Pool、Redo Log Buffer 和 undo log、redo log、bin log 概念以及关系？

*   Buffer Pool 是 MySQL 的一个非常重要的组件，因为针对数据库的增删改操作都是在 Buffer Pool 中完成的
*   Undo log 记录的是数据操作前的样子
*   redo log 记录的是数据被操作后的样子（redo log 是 Innodb 存储引擎特有）
*   bin log 记录的是整个操作记录（这个对于主从复制具有非常重要的意义）

#### 8.1.7. [#](#从准备更新一条数据到事务的提交的流程描述) 从准备更新一条数据到事务的提交的流程描述？

![](https://pdai.tech/images/db/mysql/db-mysql-sql-14.png)

*   首先执行器根据 MySQL 的执行计划来查询数据，先是从缓存池中查询数据，如果没有就会去数据库中查询，如果查询到了就将其放到缓存池中
*   在数据被缓存到缓存池的同时，会写入 undo log 日志文件
*   更新的动作是在 BufferPool 中完成的，同时会将更新后的数据添加到 redo log buffer 中
*   完成以后就可以提交事务，在提交的同时会做以下三件事
    *   将 redo log buffer 中的数据刷入到 redo log 文件中
    *   将本次操作记录写入到 bin log 文件中
    *   将 bin log 文件名字和更新内容在 bin log 中的位置记录到 redo log 中，同时在 redo log 最后添加 commit 标记

### 8.2. [#](#_8-2-mysql) 8.2 MySQL

#### 8.2.1. [#](#能说下myisam-和-innodb的区别吗) 能说下 myisam 和 innodb 的区别吗？

**myisam** 引擎是 5.1 版本之前的默认引擎，支持全文检索、压缩、空间函数等，但是不支持**事务**和**行级锁**，所以一般用于有大量查询少量插入的场景来使用，而且 myisam 不支持**外键**，并且索引和数据是分开存储的。

**innodb** 是基于 B+Tree 索引建立的，和 myisam 相反它支持事务、外键，并且通过 MVCC 来支持高并发，索引和数据存储在一起。

#### 8.2.2. [#](#说下mysql的索引有哪些吧) 说下 MySQL 的索引有哪些吧？

索引在什么层面？

首先，索引是在**存储引擎层实现**的，而不是在服务器层实现的，所以不同存储引擎具有不同的索引类型和实现。

有哪些？

*   **B+Tree 索引**
    
    *   是大多数 MySQL 存储引擎的默认索引类型。
*   **哈希索引**
    
    *   哈希索引能以 O(1) 时间进行查找，但是失去了有序性；
    *   InnoDB 存储引擎有一个特殊的功能叫 “自适应哈希索引”，当某个索引值被使用的非常频繁时，会在 B+Tree 索引之上再创建一个哈希索引，这样就让 B+Tree 索引具有哈希索引的一些优点，比如快速的哈希查找。
*   **全文索引**
    
    *   MyISAM 存储引擎支持全文索引，用于查找文本中的关键词，而不是直接比较是否相等。查找条件使用 MATCH AGAINST，而不是普通的 WHERE。
    *   全文索引一般使用倒排索引实现，它记录着关键词到其所在文档的映射。
    *   InnoDB 存储引擎在 MySQL 5.6.4 版本中也开始支持全文索引。
*   **空间数据索引**
    
    *   MyISAM 存储引擎支持空间数据索引 (R-Tree)，可以用于地理数据存储。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。

#### 8.2.3. [#](#什么是b-树-为什么b-树成为主要的sql数据库的索引实现) 什么是 B + 树？为什么 B + 树成为主要的 SQL 数据库的索引实现？

*   **什么是 B+Tree?**

B+ Tree 是基于 B Tree 和叶子节点顺序访问指针进行实现，它具有 B Tree 的平衡性，并且通过顺序访问指针来提高区间查询的性能。在 B+ Tree 中，一个节点中的 key 从左到右非递减排列，如果某个指针的左右相邻 key 分别是 keyi 和 keyi+1，且不为 null，则该指针指向节点的所有 key 大于等于 keyi 且小于等于 keyi+1。

![](https://pdai.tech/images/mysql/061c88c1-572f-424f-b580-9cbce903a3fe.png)

*   **为什么是 B+Tree**?
    
    *   为了减少磁盘读取次数，决定了树的高度不能高，所以必须是先 B-Tree；
    *   以页为单位读取使得一次 I/O 就能完全载入一个节点，且相邻的节点也能够被预先载入；所以数据放在叶子节点，本质上是一个 Page 页；
    *   为了支持范围查询以及关联关系， 页中数据需要有序，且页的尾部节点指向下个页的头部；
*   **B + 树索引可分为聚簇索引和非聚簇索引**?

1.  主索引就是聚簇索引（也称聚集索引，clustered index）
2.  辅助索引（有时也称非聚簇索引或二级索引，secondary index，non-clustered index）。

![](https://pdai.tech/images/db/mysql/db-mysql-index-1.png)

如上图，**主键索引的叶子节点保存的是真正的数据。而辅助索引叶子节点的数据区保存的是主键索引关键字的值**。

假如要查询 name = C 的数据，其搜索过程如下：a) 先在辅助索引中通过 C 查询最后找到主键 id = 9; b) 在主键索引中搜索 id 为 9 的数据，最终在主键索引的叶子节点中获取到真正的数据。所以通过辅助索引进行检索，需要检索两次索引。

之所以这样设计，一个原因就是：如果和 MyISAM 一样在主键索引和辅助索引的叶子节点中都存放数据行指针，一旦数据发生迁移，则需要去重新组织维护所有的索引。

#### 8.2.4. [#](#那你知道什么是覆盖索引和回表吗) 那你知道什么是覆盖索引和回表吗？

覆盖索引指的是在一次查询中，如果一个索引包含或者说覆盖所有需要查询的字段的值，我们就称之为覆盖索引，而不再需要回表查询。

而要确定一个查询是否是覆盖索引，我们只需要 explain sql 语句看 Extra 的结果是否是 “Using index” 即可。

比如：

```
select id from t where num/2 = 100
```

#### 8.2.5. [#](#什么是mvcc-说说mysql实现mvcc的原理) 什么是 MVCC？ 说说 MySQL 实现 MVCC 的原理？

*   **什么是 MVCC？**

MVCC，全称 Multi-Version Concurrency Control，即多版本并发控制。MVCC 是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存。

在 Mysql 的 InnoDB 引擎中就是指在已提交读 (READ COMMITTD) 和可重复读 (REPEATABLE READ) 这两种隔离级别下的事务对于 SELECT 操作会访问版本链中的记录的过程。

这就使得别的事务可以修改这条记录，反正每次修改都会在版本链中记录。SELECT 可以去版本链中拿记录，这就实现了读 - 写，写 - 读的并发执行，提升了系统的性能。

*   **MySQL 的 InnoDB 引擎实现 MVCC 的 3 个基础点**

1.  **隐式字段**

![](https://pdai.tech/images/db/mysql/db-mysql-mvcc-1.png)

如上图，DB_ROW_ID 是数据库默认为该行记录生成的唯一隐式主键；DB_TRX_ID 是当前操作该记录的事务 ID； 而 DB_ROLL_PTR 是一个回滚指针，用于配合 undo 日志，指向上一个旧版本；delete flag 没有展示出来。

2.  **undo log**

![](https://pdai.tech/images/db/mysql/db-mysql-mvcc-4.png)

从上面，我们就可以看出，不同事务或者相同事务的对同一记录的修改，会导致该记录的 undo log 成为一条记录版本线性表，既链表，undo log 的链首就是最新的旧记录，链尾就是最早的旧记录

3.  **ReadView**

已提交读和可重复读的区别就在于它们生成 ReadView 的策略不同。

ReadView 中主要就是有个列表来存储我们系统中当前活跃着的读写事务，也就是 begin 了还未提交的事务。通过这个列表来判断记录的某个版本是否对当前事务可见。假设当前列表里的事务 id 为 [80,100]。

a) 如果你要访问的记录版本的事务 id 为 50，比当前列表最小的 id80 小，那说明这个事务在之前就提交了，所以对当前活动的事务来说是可访问的。

b) 如果你要访问的记录版本的事务 id 为 90, 发现此事务在列表 id 最大值和最小值之间，那就再判断一下是否在列表内，如果在那就说明此事务还未提交，所以版本不能被访问。如果不在那说明事务已经提交，所以版本可以被访问。

c) 如果你要访问的记录版本的事务 id 为 110，那比事务列表最大 id100 都大，那说明这个版本是在 ReadView 生成之后才发生的，所以不能被访问。

这些记录都是去 undo log 链里面找的，先找最近记录，如果最近这一条记录事务 id 不符合条件，不可见的话，再去找上一个版本再比较当前事务的 id 和这个版本事务 id 看能不能访问，以此类推直到返回可见的版本或者结束。

*   **举个例子** ，在已提交读隔离级别下：

比如此时有一个事务 id 为 100 的事务，修改了 name, 使得的 name 等于小明 2，但是事务还没提交。则此时的版本链是

![](https://pdai.tech/images/db/mysql/db-mysql-mvcc-11.jpeg)

那此时另一个事务发起了 select 语句要查询 id 为 1 的记录，那此时生成的 ReadView 列表只有 [100]。那就去版本链去找了，首先肯定找最近的一条，发现 trx_id 是 100, 也就是 name 为小明 2 的那条记录，发现在列表内，所以不能访问。

这时候就通过指针继续找下一条，name 为小明 1 的记录，发现 trx_id 是 60，小于列表中的最小 id, 所以可以访问，直接访问结果为小明 1。

那这时候我们把事务 id 为 100 的事务提交了，并且新建了一个事务 id 为 110 也修改 id 为 1 的记录，并且不提交事务

![](https://pdai.tech/images/db/mysql/db-mysql-mvcc-12.jpeg)

这时候版本链就是

![](https://pdai.tech/images/db/mysql/db-mysql-mvcc-13.jpeg)

这时候之前那个 select 事务又执行了一次查询, 要查询 id 为 1 的记录。

**已提交读隔离级别下的事务在每次查询的开始都会生成一个独立的 ReadView, 而可重复读隔离级别则在第一次读的时候生成一个 ReadView，之后的读都复用之前的 ReadView**。

1.  如果你是已提交读隔离级别，这时候你会重新一个 ReadView，那你的活动事务列表中的值就变了，变成了 [110]。按照上的说法，你去版本链通过 trx_id 对比查找到合适的结果就是小明 2。
2.  如果你是可重复读隔离级别，这时候你的 ReadView 还是第一次 select 时候生成的 ReadView, 也就是列表的值还是 [100]。所以 select 的结果是小明 1。所以第二次 select 结果和第一次一样，所以叫可重复读！

这就是 Mysql 的 MVCC, 通过版本链，实现多版本，可并发读 - 写，写 - 读。通过 ReadView 生成策略的不同实现不同的隔离级别。

#### 8.2.6. [#](#mysql-锁的类型有哪些呢) MySQL 锁的类型有哪些呢？

**说两个维度**：

*   共享锁 (简称 S 锁) 和排他锁(简称 X 锁)
    *   **读锁**是共享的，可以通过 lock in share mode 实现，这时候只能读不能写。
    *   **写锁**是排他的，它会阻塞其他的写锁和读锁。从颗粒度来区分，可以分为表锁和行锁两种。
*   表锁和行锁
    *   **表锁**会锁定整张表并且阻塞其他用户对该表的所有读写操作，比如 alter 修改表结构的时候会锁表。
    *   **行锁**又可以分为乐观锁和悲观锁
        *   悲观锁可以通过 for update 实现
        *   乐观锁则通过版本号实现。

**两个维度结合来看**：

*   共享锁 (行锁):Shared Locks
    *   读锁 (s 锁), 多个事务对于同一数据可以共享访问, 不能操作修改
    *   使用方法:
        *   加锁: SELECT * FROM table WHERE id=1 LOCK IN SHARE MODE
        *   释锁: COMMIT/ROLLBACK
*   排他锁（行锁）：Exclusive Locks
    *   写锁 (X 锁)，互斥锁 / 独占锁, 事务获取了一个数据的 X 锁，其他事务就不能再获取该行的读锁和写锁（S 锁、X 锁），只有获取了该排他锁的事务是可以对数据行进行读取和修改
    *   使用方法:
        *   DELETE/ UPDATE/ INSERT -- 加锁
        *   SELECT * FROM table WHERE ... FOR UPDATE -- 加锁
        *   COMMIT/ROLLBACK -- 释锁
*   意向共享锁 (IS)
    *   一个数据行加共享锁前必须先取得该表的 IS 锁，意向共享锁之间是可以相互兼容的 意向排它锁 (IX) 一个数据行加排他锁前必须先取得该表的 IX 锁，意向排它锁之间是可以相互兼容的 意向锁(IS、IX) 是 InnoDB 引擎操作数据之前自动加的，不需要用户干预; 意义： 当事务操作需要锁表时，只需判断意向锁是否存在，存在时则可快速返回该表不能启用表锁
    *   意向共享锁 (IS 锁)（表锁）：Intention Shared Locks
        *   表示事务准备给数据行加入共享锁，也就是说一个数据行加共享锁 前必须先取得该表的 IS 锁。
    *   意向排它锁 (IX 锁)（表锁）：Intention Exclusive Locks
        *   表示事务准备给数据行加入排他锁，说明事务在一个数据行加排他 锁前必须先取得该表的 IX 锁。

#### 8.2.7. [#](#你们数据量级多大-分库分表怎么做的) 你们数据量级多大？分库分表怎么做的？

首先分库分表分为垂直和水平两个方式，一般来说我们拆分的顺序是先垂直后水平。

*   **垂直分库**

基于现在微服务拆分来说，都是已经做到了垂直分库了

*   **垂直分表**

垂直切分是将一张表按列切分成多个表，通常是按照列的关系密集程度进行切分，也可以利用垂直切分将经常被使用的列和不经常被使用的列切分到不同的表中。

在数据库的层面使用垂直切分将按数据库中表的密集程度部署到不同的库中，例如将原来的电商数据库垂直切分成商品数据库、用户数据库等。

![](https://pdai.tech/images/mysql/e130e5b8-b19a-4f1e-b860-223040525cf6.jpg)

*   **水平分表**

首先根据业务场景来决定使用什么字段作为分表字段 (sharding_key)，比如我们现在日订单 1000 万，我们大部分的场景来源于 C 端，我们可以用 user_id 作为 sharding_key，数据查询支持到最近 3 个月的订单，超过 3 个月的做归档处理，那么 3 个月的数据量就是 9 亿，可以分 1024 张表，那么每张表的数据大概就在 100 万左右。

比如用户 id 为 100，那我们都经过 hash(100)，然后对 1024 取模，就可以落到对应的表上了。

![](https://pdai.tech/images/mysql/63c2909f-0c5f-496f-9fe5-ee9176b31aba.jpg)

#### 8.2.8. [#](#那分表后的id怎么保证唯一性的呢) 那分表后的 ID 怎么保证唯一性的呢？

因为我们主键默认都是自增的，那么分表之后的主键在不同表就肯定会有冲突了。有几个办法考虑：

*   设定步长，比如 1-1024 张表我们分别设定 1-1024 的基础步长，这样主键落到不同的表就不会冲突了。
*   分布式 ID，自己实现一套分布式 ID 生成算法或者使用开源的比如雪花算法这种
*   分表后不使用主键作为查询依据，而是每张表单独新增一个字段作为唯一主键使用，比如订单表订单号是唯一的，不管最终落在哪张表都基于订单号作为查询依据，更新也一样。

#### 8.2.9. [#](#分表后非sharding-key的查询怎么处理呢) 分表后非 sharding_key 的查询怎么处理呢？

*   可以做一个 mapping 表，比如这时候商家要查询订单列表怎么办呢？不带 user_id 查询的话你总不能扫全表吧？所以我们可以做一个映射关系表，保存商家和用户的关系，查询的时候先通过商家查询到用户列表，再通过 user_id 去查询。
*   大宽表，一般而言，商户端对数据实时性要求并不是很高，比如查询订单列表，可以把订单表同步到离线（实时）数仓，再基于数仓去做成一张宽表，再基于其他如 es 提供查询服务。
*   数据量不是很大的话，比如后台的一些查询之类的，也可以通过多线程扫表，然后再聚合结果的方式来做。或者异步的形式也是可以的。

```
select id from t where num = 100*2
```

#### 8.2.10. [#](#mysql主从复制) MySQL 主从复制？

主要涉及三个线程: binlog 线程、I/O 线程和 SQL 线程。

*   **binlog 线程** : 负责将主服务器上的数据更改写入二进制日志中。
*   **I/O 线程** : 负责从主服务器上读取二进制日志，并写入从服务器的中继日志中。
*   **SQL 线程** : 负责读取中继日志并重放其中的 SQL 语句。

![](https://pdai.tech/images/mysql/master-slave.png)

**全同步复制**

主库写入 binlog 后强制同步日志到从库，所有的从库都执行完成后才返回给客户端，但是很显然这个方式的话性能会受到严重影响。

**半同步复制**

和全同步不同的是，半同步复制的逻辑是这样，从库写入日志成功后返回 ACK 确认给主库，主库收到至少一个从库的确认就认为写操作完成。

#### 8.2.11. [#](#mysql主从的延迟怎么解决呢) MySQL 主从的延迟怎么解决呢？

这个问题貌似真的是个无解的问题，只能是说自己来判断了，需要走主库的强制走主库查询。

#### 8.2.12. [#](#mysql读写分离方案) MySQL 读写分离方案?

主服务器处理写操作以及实时性要求比较高的读操作，而从服务器处理读操作。

读写分离能提高性能的原因在于:

*   主从服务器负责各自的读和写，极大程度缓解了锁的争用；
*   从服务器可以使用 MyISAM，提升查询性能以及节约系统开销；
*   增加冗余，提高可用性。

读写分离常用代理方式来实现，代理服务器接收应用层传来的读写请求，然后决定转发到哪个服务器。

![](https://pdai.tech/images/mysql/master-slave-proxy.png)

### 8.3. [#](#_8-3-redis) 8.3 Redis

#### 8.3.1. [#](#什么是redis-为什么用redis) 什么是 Redis，为什么用 Redis？

Redis 是一种支持 key-value 等多种数据结构的存储系统。可用于缓存，事件发布或订阅，高速队列等场景。支持网络，提供字符串，哈希，列表，队列，集合结构直接存取，基于内存，可持久化。

*   **读写性能优异**
    *   Redis 能读的速度是 110000 次 / s, 写的速度是 81000 次 / s （测试条件见下一节）。
*   **数据类型丰富**
    *   Redis 支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
*   **原子性**
    *   Redis 的所有操作都是原子性的，同时 Redis 还支持对几个操作全并后的原子性执行。
*   **丰富的特性**
    *   Redis 支持 publish/subscribe, 通知, key 过期等特性。
*   **持久化**
    *   Redis 支持 RDB, AOF 等持久化方式
*   **发布订阅**
    *   Redis 支持发布 / 订阅模式
*   **分布式**
    *   Redis Cluster

#### 8.3.2. [#](#为什么redis-是单线程的以及为什么这么快) 为什么 Redis 是单线程的以及为什么这么快？

*   redis 完全基于内存, 绝大部分请求是纯粹的内存操作, 非常快速.
*   数据结构简单, 对数据操作也简单, redis 中的数据结构是专门进行设计的
*   采用单线程模型, 避免了不必要的上下文切换和竞争条件, 也不存在多线程或者多线程切换而消耗 CPU, 不用考虑各种锁的问题, 不存在加锁, 释放锁的操作, 没有因为可能出现死锁而导致性能消耗
*   使用了多路 IO 复用模型, 非阻塞 IO
*   使用底层模型不同, 它们之间底层实现方式及与客户端之间的 通信的应用协议不一样, Redis 直接构建了自己的 VM 机制, 因为一般的系统调用系统函数的话, 会浪费一定的时间去移动和请求

#### 8.3.3. [#](#redis-一般有哪些使用场景) Redis 一般有哪些使用场景？

可以结合自己的项目讲讲，比如

*   **热点数据的缓存**

缓存是 Redis 最常见的应用场景，之所有这么使用，主要是因为 Redis 读写性能优异。而且逐渐有取代 memcached，成为首选服务端缓存的组件。而且，Redis 内部是支持事务的，在使用时候能有效保证数据的一致性。

*   **限时业务的运用**

redis 中可以使用 expire 命令设置一个键的生存时间，到时间后 redis 会删除它。利用这一特性可以运用在限时的优惠活动信息、手机验证码等业务场景。

*   **计数器相关问题**

redis 由于 incrby 命令可以实现原子性的递增，所以可以运用于高并发的秒杀活动、分布式序列号的生成、具体业务还体现在比如限制一个手机号发多少条短信、一个接口一分钟限制多少请求、一个接口一天限制调用多少次等等。

*   **分布式锁**

这个主要利用 redis 的 setnx 命令进行，setnx："set if not exists" 就是如果不存在则成功设置缓存同时返回 1，否则返回 0 ，这个特性在俞你奔远方的后台中有所运用，因为我们服务器是集群的，定时任务可能在两台机器上都会运行，所以在定时任务中首先 通过 setnx 设置一个 lock，如果成功设置则执行，如果没有成功设置，则表明该定时任务已执行。 当然结合具体业务，我们可以给这个 lock 加一个过期时间，比如说 30 分钟执行一次的定时任务，那么这个过期时间设置为小于 30 分钟的一个时间就可以，这个与定时任务的周期以及定时任务执行消耗时间相关。

在分布式锁的场景中，主要用在比如秒杀系统等。

#### 8.3.4. [#](#redis-有哪些数据类型) Redis 有哪些数据类型？

*   **5 种基础数据类型**，分别是：String、List、Set、Zset、Hash。

![](https://pdai.tech/images/db/redis/db-redis-ds-1.jpeg)

<table><thead><tr><th>结构类型</th><th>结构存储的值</th><th>结构的读写能力</th></tr></thead><tbody><tr><td><strong>String 字符串</strong></td><td>可以是字符串、整数或浮点数</td><td>对整个字符串或字符串的一部分进行操作；对整数或浮点数进行自增或自减操作；</td></tr><tr><td><strong>List 列表</strong></td><td>一个链表，链表上的每个节点都包含一个字符串</td><td>对链表的两端进行 push 和 pop 操作，读取单个或多个元素；根据值查找或删除元素；</td></tr><tr><td><strong>Set 集合</strong></td><td>包含字符串的无序集合</td><td>字符串的集合，包含基础的方法有看是否存在添加、获取、删除；还包含计算交集、并集、差集等</td></tr><tr><td><strong>Hash 散列</strong></td><td>包含键值对的无序散列表</td><td>包含方法有添加、获取、删除单个元素</td></tr><tr><td><strong>Zset 有序集合</strong></td><td>和散列一样，用于存储键值对</td><td>字符串成员与浮点数分数之间的有序映射；元素的排列顺序由分数的大小决定；包含方法有添加、获取、删除单个元素以及根据分值范围或成员来获取元素</td></tr></tbody></table>

*   **三种特殊的数据类型** 分别是 HyperLogLogs（基数统计）， Bitmaps (位图) 和 geospatial （地理位置)

#### 8.3.5. [#](#谈谈redis-的对象机制-redisobject) 谈谈 Redis 的对象机制（redisObject)？

比如说， 集合类型就可以由字典和整数集合两种不同的数据结构实现， 但是， 当用户执行 ZADD 命令时， 他 / 她应该不必关心集合使用的是什么编码， 只要 Redis 能按照 ZADD 命令的指示， 将新元素添加到集合就可以了。

这说明, **操作数据类型的命令除了要对键的类型进行检查之外, 还需要根据数据类型的不同编码进行多态处理**.

为了解决以上问题, **Redis 构建了自己的类型系统**, 这个系统的主要功能包括:

*   redisObject 对象.
*   基于 redisObject 对象的类型检查.
*   基于 redisObject 对象的显式多态函数.
*   对 redisObject 进行分配、共享和销毁的机制.

```
select id from t where substring(name,1,3) = ’abc’       -–name以abc开头的id
select id from t where datediff(day,createdate,’2005-11-30′) = 0    -–‘2005-11-30’    --生成的id
```

下图对应上面的结构

![](https://pdai.tech/images/db/redis/db-redis-object-1.png)

#### 8.3.6. [#](#redis-数据类型有哪些底层数据结构) Redis 数据类型有哪些底层数据结构？

![](https://pdai.tech/images/db/redis/db-redis-object-2-3.png)

*   简单动态字符串 - sds
*   压缩列表 - ZipList
*   快表 - QuickList
*   字典 / 哈希表 - Dict
*   整数集 - IntSet
*   跳表 - ZSkipList

#### 8.3.7. [#](#为什么要设计sds) 为什么要设计 sds？

*   **常数复杂度获取字符串长度**

由于 len 属性的存在，我们获取 SDS 字符串的长度只需要读取 len 属性，时间复杂度为 O(1)。而对于 C 语言，获取字符串的长度通常是经过遍历计数来实现的，时间复杂度为 O(n)。通过 `strlen key` 命令可以获取 key 的字符串长度。

*   **杜绝缓冲区溢出**

我们知道在 C 语言中使用 `strcat` 函数来进行两个字符串的拼接，一旦没有分配足够长度的内存空间，就会造成缓冲区溢出。而对于 SDS 数据类型，在进行字符修改的时候，**会首先根据记录的 len 属性检查内存空间是否满足需求**，如果不满足，会进行相应的空间扩展，然后在进行修改操作，所以不会出现缓冲区溢出。

*   **减少修改字符串的内存重新分配次数**

C 语言由于不记录字符串的长度，所以如果要修改字符串，必须要重新分配内存（先释放再申请），因为如果没有重新分配，字符串长度增大时会造成内存缓冲区溢出，字符串长度减小时会造成内存泄露。

而对于 SDS，由于`len`属性和`alloc`属性的存在，对于修改字符串 SDS 实现了**空间预分配**和**惰性空间释放**两种策略：

1.  **空间预分配**：对字符串进行空间扩展的时候，扩展的内存比实际需要的多，这样可以减少连续执行字符串增长操作所需的内存重分配次数。
2.  **惰性空间释放**：对字符串进行缩短操作时，程序不立即使用内存重新分配来回收缩短后多余的字节，而是使用 `alloc` 属性将这些字节的数量记录下来，等待后续使用。（当然 SDS 也提供了相应的 API，当我们有需要时，也可以手动释放这些未使用的空间。）

*   **二进制安全**

因为 C 字符串以空字符作为字符串结束的标识，而对于一些二进制文件（如图片等），内容可能包括空字符串，因此 C 字符串无法正确存取；而所有 SDS 的 API 都是以处理二进制的方式来处理 `buf` 里面的元素，并且 SDS 不是以空字符串来判断是否结束，而是以 len 属性表示的长度来判断字符串是否结束。

*   **兼容部分 C 字符串函数**

虽然 SDS 是二进制安全的，但是一样遵从每个字符串都是以空字符串结尾的惯例，这样可以重用 C 语言库`<string.h>` 中的一部分函数。

#### 8.3.8. [#](#redis-一个字符串类型的值能存储最大容量是多少) Redis 一个字符串类型的值能存储最大容量是多少？

512M

#### 8.3.9. [#](#为什么会设计stream) 为什么会设计 Stream？

用过 Redis 做消息队列的都了解，基于 Reids 的消息队列实现有很多种，例如：

*   **PUB/SUB，订阅 / 发布模式**
    *   但是发布订阅模式是无法持久化的，如果出现网络断开、Redis 宕机等，消息就会被丢弃；
*   基于 **List LPUSH+BRPOP** 或者 **基于 Sorted-Set** 的实现
    *   支持了持久化，但是不支持多播，分组消费等

**消费组消费图**

![](https://pdai.tech/images/db/redis/db-redis-stream-3.png)

#### 8.3.10. [#](#redis-stream用在什么样场景) Redis Stream 用在什么样场景？

可用作时通信等，大数据分析，异地数据备份等

![](https://pdai.tech/images/db/redis/db-redis-stream-4.png)

客户端可以平滑扩展，提高处理能力

![](https://pdai.tech/images/db/redis/db-redis-stream-5.png)

#### 8.3.11. [#](#redis-stream消息id的设计是否考虑了时间回拨的问题) Redis Stream 消息 ID 的设计是否考虑了时间回拨的问题？

XADD 生成的 1553439850328-0，就是 Redis 生成的消息 ID，由两部分组成: **时间戳 - 序号**。时间戳是毫秒级单位，是生成消息的 Redis 服务器时间，它是个 64 位整型（int64）。序号是在这个毫秒时间点内的消息序号，它也是个 64 位整型。

可以通过 multi 批处理，来验证序号的递增：

```
select id from t where name like 'abc%'
select id from t where createdate >= '2005-11-30' and createdate < '2005-12-1'
```

由于一个 redis 命令的执行很快，所以可以看到在同一时间戳内，是通过序号递增来表示消息的。

为了保证消息是有序的，因此 Redis 生成的 ID 是单调递增有序的。由于 ID 中包含时间戳部分，为了避免服务器时间错误而带来的问题（例如服务器时间延后了），Redis 的每个 Stream 类型数据都维护一个 latest_generated_id 属性，用于记录最后一个消息的 ID。**若发现当前时间戳退后（小于 latest_generated_id 所记录的），则采用时间戳不变而序号递增的方案来作为新消息 ID**（这也是序号为什么使用 int64 的原因，保证有足够多的的序号），从而保证 ID 的单调递增性质。

强烈建议使用 Redis 的方案生成消息 ID，因为这种时间戳 + 序号的单调递增的 ID 方案，几乎可以满足你全部的需求。但同时，记住 ID 是支持自定义的，别忘了！

#### 8.3.12. [#](#redis-stream消费者崩溃带来的会不会消息丢失问题) Redis Stream 消费者崩溃带来的会不会消息丢失问题?

为了解决组内消息读取但处理期间消费者崩溃带来的消息丢失问题，STREAM 设计了 Pending 列表，用于记录读取但并未处理完毕的消息。命令 XPENDIING 用来获消费组或消费内消费者的未处理完毕的消息。演示如下：

```
select col1,col2 into #t from t where 1=0
```

每个 Pending 的消息有 4 个属性：

*   消息 ID
*   所属消费者
*   IDLE，已读取时长
*   delivery counter，消息被读取次数

上面的结果我们可以看到，我们之前读取的消息，都被记录在 Pending 列表中，说明全部读到的消息都没有处理，仅仅是读取了。那如何表示消费者处理完毕了消息呢？使用命令 XACK 完成告知消息处理完成，演示如下：

```
create table #t(…)
```

有了这样一个 Pending 机制，就意味着在某个消费者读取消息但未处理后，消息是不会丢失的。等待消费者再次上线后，可以读取该 Pending 列表，就可以继续处理该消息了，保证消息的有序和不丢失。

#### 8.3.13. [#](#redis-steam-坏消息问题-死信问题) Redis Steam 坏消息问题，死信问题?

正如上面所说，如果某个消息，不能被消费者处理，也就是不能被 XACK，这是要长时间处于 Pending 列表中，即使被反复的转移给各个消费者也是如此。此时该消息的 delivery counter 就会累加（上一节的例子可以看到），当累加到某个我们预设的临界值时，我们就认为是坏消息（也叫死信，DeadLetter，无法投递的消息），由于有了判定条件，我们将坏消息处理掉即可，删除即可。删除一个消息，使用 XDEL 语法，演示如下：

```
explain select * from user where age=1; // 查询的name无法从索引数据获取
explain select id,age from user where age=1; //可以直接从索引获取
```

注意本例中，并没有删除 Pending 中的消息因此你查看 Pending，消息还会在。可以执行 XACK 标识其处理完毕！

#### 8.3.14. [#](#redis-的持久化机制是什么-各自的优缺点-一般怎么用) Redis 的持久化机制是什么？各自的优缺点？一般怎么用？

1.  RDB 持久化是把当前进程数据生成快照保存到磁盘上的过程; 针对 RDB 不适合实时持久化的问题，Redis 提供了 AOF 持久化方式来解决.
    
2.  AOF 是 “写后” 日志，Redis 先执行命令，把数据写入内存，然后才记录日志。日志里记录的是 Redis 收到的每一条命令，这些命令是以文本形式保存。
    
3.  Redis 4.0 中提出了一个**混合使用 AOF 日志和内存快照**的方法。简单来说，内存快照以一定的频率执行，在两次快照之间，使用 AOF 日志记录这期间的所有命令操作。

这样一来，快照不用很频繁地执行，这就避免了频繁 fork 对主线程的影响。而且，AOF 日志也只用记录两次快照间的操作，也就是说，不需要记录所有操作了，因此，就不会出现文件过大的情况了，也可以避免重写开销。

如下图所示，T1 和 T2 时刻的修改，用 AOF 日志记录，等到第二次做全量快照时，就可以清空 AOF 日志，因为此时的修改都已经记录到快照中了，恢复时就不再用日志了。

![](https://pdai.tech/images/db/redis/redis-x-rdb-4.jpg)

这个方法既能享受到 RDB 文件快速恢复的好处，又能享受到 AOF 只记录操作命令的简单优势, 实际环境中用的很多。

#### 8.3.15. [#](#rdb-触发方式) RDB 触发方式?

触发 rdb 持久化的方式有 2 种，分别是**手动触发**和**自动触发**。

*   **手动触发**
    *   **save 命令**：阻塞当前 Redis 服务器，直到 RDB 过程完成为止，对于内存 比较大的实例会造成长时间**阻塞**，线上环境不建议使用
    *   **bgsave 命令**：Redis 进程执行 fork 操作创建子进程，RDB 持久化过程由子 进程负责，完成后自动结束。阻塞只发生在 fork 阶段，一般时间很短

bgsave 流程图如下所示

![](https://pdai.tech/images/db/redis/redis-x-rdb-1.png)

*   **自动触发**
    *   redis.conf 中配置`save m n`，即在 m 秒内有 n 次修改时，自动触发 bgsave 生成 rdb 文件；
    *   主从复制时，从节点要从主节点进行全量复制时也会触发 bgsave 操作，生成当时的快照发送到从节点；
    *   执行 debug reload 命令重新加载 redis 时也会触发 bgsave 操作；
    *   默认情况下执行 shutdown 命令时，如果没有开启 aof 持久化，那么也会触发 bgsave 操作；

#### 8.3.16. [#](#rdb由于生产环境中我们为redis开辟的内存区域都比较大-例如6gb-那么将内存中的数据同步到硬盘的过程可能就会持续比较长的时间-而实际情况是这段时间redis服务一般都会收到数据写操作请求。那么如何保证数据一致性呢) RDB 由于生产环境中我们为 Redis 开辟的内存区域都比较大（例如 6GB），那么将内存中的数据同步到硬盘的过程可能就会持续比较长的时间，而实际情况是这段时间 Redis 服务一般都会收到数据写操作请求。那么如何保证数据一致性呢？

RDB 中的核心思路是 Copy-on-Write，来保证在进行快照操作的这段时间，需要压缩写入磁盘上的数据在内存中不会发生变化。在正常的快照操作中，一方面 Redis 主进程会 fork 一个新的快照进程专门来做这个事情，这样保证了 Redis 服务不会停止对客户端包括写请求在内的任何响应。另一方面这段时间发生的数据变化会以副本的方式存放在另一个新的内存区域，待快照操作结束后才会同步到原来的内存区域。

举个例子：如果主线程对这些数据也都是读操作（例如图中的键值对 A），那么，主线程和 bgsave 子进程相互不影响。但是，如果主线程要修改一块数据（例如图中的键值对 C），那么，这块数据就会被复制一份，生成该数据的副本。然后，bgsave 子进程会把这个副本数据写入 RDB 文件，而在这个过程中，主线程仍然可以直接修改原来的数据。

![](https://pdai.tech/images/db/redis/redis-x-aof-42.jpg)

#### 8.3.17. [#](#在进行rdb快照操作的这段时间-如果发生服务崩溃怎么办) 在进行 RDB 快照操作的这段时间，如果发生服务崩溃怎么办？

很简单，在没有将数据全部写入到磁盘前，这次快照操作都不算成功。如果出现了服务崩溃的情况，将以上一次完整的 RDB 快照文件作为恢复内存数据的参考。也就是说，在快照操作过程中不能影响上一次的备份数据。Redis 服务会在磁盘上创建一个临时文件进行数据操作，待操作成功后才会用这个临时文件替换掉上一次的备份。

#### 8.3.18. [#](#可以每秒做一次rdb快照吗) 可以每秒做一次 RDB 快照吗？

对于快照来说，所谓 “连拍” 就是指连续地做快照。这样一来，快照的间隔时间变得很短，即使某一时刻发生宕机了，因为上一时刻快照刚执行，丢失的数据也不会太多。但是，这其中的快照间隔时间就很关键了。

如下图所示，我们先在 T0 时刻做了一次快照，然后又在 T0+t 时刻做了一次快照，在这期间，数据块 5 和 9 被修改了。如果在 t 这段时间内，机器宕机了，那么，只能按照 T0 时刻的快照进行恢复。此时，数据块 5 和 9 的修改值因为没有快照记录，就无法恢复了。 　　

![](https://pdai.tech/images/db/redis/redis-x-rdb-2.jpg)

所以，要想尽可能恢复数据，t 值就要尽可能小，t 越小，就越像 “连拍”。那么，t 值可以小到什么程度呢，比如说是不是可以每秒做一次快照？毕竟，每次快照都是由 bgsave 子进程在后台执行，也不会阻塞主线程。

这种想法其实是错误的。虽然 bgsave 执行时不阻塞主线程，但是，**如果频繁地执行全量快照，也会带来两方面的开销**：

*   一方面，频繁将全量数据写入磁盘，会给磁盘带来很大压力，多个快照竞争有限的磁盘带宽，前一个快照还没有做完，后一个又开始做了，容易造成恶性循环。
*   另一方面，bgsave 子进程需要通过 fork 操作从主线程创建出来。虽然，子进程在创建后不会再阻塞主线程，但是，fork 这个创建过程本身会阻塞主线程，而且主线程的内存越大，阻塞时间越长。如果频繁 fork 出 bgsave 子进程，这就会频繁**阻塞主线程**了。

那么，有什么其他好方法吗？此时，我们可以做增量快照，就是指做了一次全量快照后，后续的快照只对修改的数据进行快照记录，这样可以避免每次全量快照的开销。这个比较好理解。

但是它需要我们使用额外的元数据信息去记录哪些数据被修改了，这会带来额外的**空间开销问题**。那么，还有什么方法既能利用 RDB 的快速恢复，又能以较小的开销做到尽量少丢数据呢？RDB 和 AOF 的混合方式。

#### 8.3.19. [#](#aof是写前日志还是写后日志) AOF 是写前日志还是写后日志？

AOF 日志采用写后日志，即**先写内存，后写日志**。

![](https://pdai.tech/images/db/redis/redis-x-aof-41.jpg)

**为什么采用写后日志**？

Redis 要求高性能，采用写日志有两方面好处：

*   **避免额外的检查开销**：Redis 在向 AOF 里面记录日志的时候，并不会先去对这些命令进行语法检查。所以，如果先记日志再执行命令的话，日志中就有可能记录了错误的命令，Redis 在使用日志恢复数据时，就可能会出错。
*   不会阻塞当前的写操作

但这种方式存在潜在风险：

*   如果命令执行完成，写日志之前宕机了，会丢失数据。
*   主线程写磁盘压力大，导致写盘慢，阻塞后续操作。

#### 8.3.20. [#](#如何实现aof的) 如何实现 AOF 的？

AOF 日志记录 Redis 的每个写命令，步骤分为：命令追加（append）、文件写入（write）和文件同步（sync）。

*   **命令追加** 当 AOF 持久化功能打开了，服务器在执行完一个写命令之后，会以协议格式将被执行的写命令追加到服务器的 aof_buf 缓冲区。
    
*   **文件写入和同步** 关于何时将 aof_buf 缓冲区的内容写入 AOF 文件中，Redis 提供了三种写回策略：

![](https://pdai.tech/images/db/redis/redis-x-aof-4.jpg)

`Always`，同步写回：每个写命令执行完，立马同步地将日志写回磁盘；

`Everysec`，每秒写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，每隔一秒把缓冲区中的内容写入磁盘；

`No`，操作系统控制的写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，由操作系统决定何时将缓冲区内容写回磁盘。

*   **三种写回策略的优缺点**

上面的三种写回策略体现了一个重要原则：**trade-off**，取舍，指在性能和可靠性保证之间做取舍。

关于 AOF 的同步策略是涉及到操作系统的 write 函数和 fsync 函数的，在《Redis 设计与实现》中是这样说明的：

```
List<Callable<List<User>>> taskList = Lists.newArrayList();
for (int shardingIndex = 0; shardingIndex < 1024; shardingIndex++) {
    taskList.add(() -> (userMapper.getProcessingAccountList(shardingIndex)));
}
List<ThirdAccountInfo> list = null;
try {
    list = taskExecutor.executeTask(taskList);
} catch (Exception e) {
    //do something
}

public class TaskExecutor {
    public <T> List<T> executeTask(Collection<? extends Callable<T>> tasks) throws Exception {
        List<T> result = Lists.newArrayList();
        List<Future<T>> futures = ExecutorUtil.invokeAll(tasks);
        for (Future<T> future : futures) {
            result.add(future.get());
        }
        return result;
    }
}
```

#### 8.3.21. [#](#什么是aof重写) 什么是 AOF 重写？

Redis 通过创建一个新的 AOF 文件来替换现有的 AOF，新旧两个 AOF 文件保存的数据相同，但新 AOF 文件没有了冗余命令。

![](https://pdai.tech/images/db/redis/redis-x-aof-1.jpg)

#### 8.3.22. [#](#aof重写会阻塞吗) AOF 重写会阻塞吗？

AOF 重写过程是由后台进程 bgrewriteaof 来完成的。主线程 fork 出后台的 bgrewriteaof 子进程，fork 会把主线程的内存拷贝一份给 bgrewriteaof 子进程，这里面就包含了数据库的最新数据。然后，bgrewriteaof 子进程就可以在不影响主线程的情况下，逐一把拷贝的数据写成操作，记入重写日志。

所以 aof 在重写时，在 fork 进程时是会阻塞住主线程的。

#### 8.3.23. [#](#aof日志何时会重写) AOF 日志何时会重写？

有两个配置项控制 AOF 重写的触发：

`auto-aof-rewrite-min-size`: 表示运行 AOF 重写时文件的最小大小，默认为 64MB。

`auto-aof-rewrite-percentage`: 这个值的计算方式是，当前 aof 文件大小和上一次重写后 aof 文件大小的差值，再除以上一次重写后 aof 文件大小。也就是当前 aof 文件比上一次重写后 aof 文件的增量大小，和上一次重写后 aof 文件大小的比值。

#### 8.3.24. [#](#aof重写日志时-有新数据写入咋整) AOF 重写日志时，有新数据写入咋整？

重写过程总结为：“一个拷贝，两处日志”。在 fork 出子进程时的拷贝，以及在重写时，如果有新数据写入，主线程就会将命令记录到两个 aof 日志内存缓冲区中。如果 AOF 写回策略配置的是 always，则直接将命令写回旧的日志文件，并且保存一份命令至 AOF 重写缓冲区，这些操作对新的日志文件是不存在影响的。（旧的日志文件：主线程使用的日志文件，新的日志文件：bgrewriteaof 进程使用的日志文件）

而在 bgrewriteaof 子进程完成会日志文件的重写操作后，会提示主线程已经完成重写操作，主线程会将 AOF 重写缓冲中的命令追加到新的日志文件后面。这时候在高并发的情况下，AOF 重写缓冲区积累可能会很大，这样就会造成阻塞，Redis 后来通过 Linux 管道技术让 aof 重写期间就能同时进行回放，这样 aof 重写结束后只需回放少量剩余的数据即可。

最后通过修改文件名的方式，保证文件切换的原子性。

在 AOF 重写日志期间发生宕机的话，因为日志文件还没切换，所以恢复数据时，用的还是旧的日志文件。

#### 8.3.25. [#](#主线程fork出子进程的是如何复制内存数据的) 主线程 fork 出子进程的是如何复制内存数据的？

fork 采用操作系统提供的写时复制（copy on write）机制，就是为了避免一次性拷贝大量内存数据给子进程造成阻塞。fork 子进程时，子进程时会拷贝父进程的页表，即虚实映射关系（虚拟内存和物理内存的映射索引表），而不会拷贝物理内存。这个拷贝会消耗大量 cpu 资源，并且拷贝完成前会阻塞主线程，阻塞时间取决于内存中的数据量，数据量越大，则内存页表越大。拷贝完成后，父子进程使用相同的内存地址空间。

但主进程是可以有数据写入的，这时候就会拷贝物理内存中的数据。如下图（进程 1 看做是主进程，进程 2 看做是子进程）：

![](https://pdai.tech/images/db/redis/redis-x-aof-3.png)

在主进程有数据写入时，而这个数据刚好在页 c 中，操作系统会创建这个页面的副本（页 c 的副本），即拷贝当前页的物理数据，将其映射到主进程中，而子进程还是使用原来的的页 c。

#### 8.3.26. [#](#在重写日志整个过程时-主线程有哪些地方会被阻塞) 在重写日志整个过程时，主线程有哪些地方会被阻塞？

1.  fork 子进程时，需要拷贝虚拟页表，会对主线程阻塞。
2.  主进程有 bigkey 写入时，操作系统会创建页面的副本，并拷贝原有的数据，会对主线程阻塞。
3.  子进程重写日志完成后，主进程追加 aof 重写缓冲区时可能会对主线程阻塞。

#### 8.3.27. [#](#为什么aof重写不复用原aof日志) 为什么 AOF 重写不复用原 AOF 日志？

两方面原因：

1.  父子进程写同一个文件会产生竞争问题，影响父进程的性能。
2.  如果 AOF 重写过程中失败了，相当于污染了原本的 AOF 文件，无法做恢复数据使用。

#### 8.3.28. [#](#redis-过期键的删除策略有哪些) Redis 过期键的删除策略有哪些?

在单机版 Redis 中，存在两种删除策略：

*   **惰性删除**：服务器不会主动删除数据，只有当客户端查询某个数据时，服务器判断该数据是否过期，如果过期则删除。
*   **定期删除**：服务器执行定时任务删除过期数据，但是考虑到内存和 CPU 的折中（删除会释放内存，但是频繁的删除操作对 CPU 不友好），该删除的频率和执行时间都受到了限制。

在主从复制场景下，为了主从节点的数据一致性，从节点不会主动删除数据，而是由主节点控制从节点中过期数据的删除。由于主节点的惰性删除和定期删除策略，都不能保证主节点及时对过期数据执行删除操作，因此，当客户端通过 Redis 从节点读取数据时，很容易读取到已经过期的数据。

Redis 3.2 中，从节点在读取数据时，增加了对数据是否过期的判断：如果该数据已过期，则不返回给客户端；将 Redis 升级到 3.2 可以解决数据过期问题。

#### 8.3.29. [#](#redis-内存淘汰算法有哪些) Redis 内存淘汰算法有哪些?

Redis 共支持八种淘汰策略，分别是 noeviction、volatile-random、volatile-ttl、volatile-lru、volatile-lfu、allkeys-lru、allkeys-random 和 allkeys-lfu 策略。

**怎么理解呢**？主要看分三类看：

*   不淘汰
    *   noeviction （v4.0 后默认的）
*   对设置了过期时间的数据中进行淘汰
    *   随机：volatile-random
    *   ttl：volatile-ttl
    *   lru：volatile-lru
    *   lfu：volatile-lfu
*   全部数据进行淘汰
    *   随机：allkeys-random
    *   lru：allkeys-lru
    *   lfu：allkeys-lfu

**LRU 算法**：LRU 算法的全称是 Least Recently Used，按照最近最少使用的原则来筛选数据。这种模式下会使用 LRU 算法筛选设置了过期时间的键值对。

Redis 优化的 **LRU 算法实现**：

Redis 会记录每个数据的最近一次被访问的时间戳。在 Redis 在决定淘汰的数据时，第一次会随机选出 N 个数据，把它们作为一个候选集合。接下来，Redis 会比较这 N 个数据的 lru 字段，把 lru 字段值最小的数据从缓存中淘汰出去。通过随机读取待删除集合，可以让 Redis 不用维护一个巨大的链表，也不用操作链表，进而提升性能。

**LFU 算法**：LFU 缓存策略是在 LRU 策略基础上，为每个数据增加了一个计数器，来统计这个数据的访问次数。当使用 LFU 策略筛选淘汰数据时，首先会根据数据的访问次数进行筛选，把访问次数最低的数据淘汰出缓存。如果两个数据的访问次数相同，LFU 策略再比较这两个数据的访问时效性，把距离上一次访问时间更久的数据淘汰出缓存。 Redis 的 LFU 算法实现:

当 LFU 策略筛选数据时，Redis 会在候选集合中，根据数据 lru 字段的后 8bit 选择访问次数最少的数据进行淘汰。当访问次数相同时，再根据 lru 字段的前 16bit 值大小，选择访问时间最久远的数据进行淘汰。

Redis 只使用了 8bit 记录数据的访问次数，而 8bit 记录的最大值是 255，这样在访问快速的情况下，如果每次被访问就将访问次数加一，很快某条数据就达到最大值 255，可能很多数据都是 255，那么退化成 LRU 算法了。所以 Redis 为了解决这个问题，实现了一个更优的计数规则，并可以通过配置项，来控制计数器增加的速度。

#### 8.3.30. [#](#redis的内存用完了会发生什么) Redis 的内存用完了会发生什么？

如果达到设置的上限，Redis 的写命令会返回错误信息（但是读命令还可以正常返回。）或者你可以配置内存淘汰机制，当 Redis 达到内存上限时会冲刷掉旧的内容。

#### 8.3.31. [#](#redis如何做内存优化) Redis 如何做内存优化？

1.  缩减键值对象: 缩减键（key）和值（value）的长度，

key 长度：如在设计键时，在完整描述业务情况下，键值越短越好。

value 长度：值对象缩减比较复杂，常见需求是把业务对象序列化成二进制数组放入 Redis。首先应该在业务上精简业务对象，去掉不必要的属性避免存储无效数据。其次在序列化工具选择上，应该选择更高效的序列化工具来降低字节数组大小。以 JAVA 为例，内置的序列化方式无论从速度还是压缩比都不尽如人意，这时可以选择更高效的序列化工具，如: protostuff，kryo 等，下图是 JAVA 常见序列化工具空间压缩对比。

2.  共享对象池

对象共享池指 Redis 内部维护 [0-9999] 的整数对象池。创建大量的整数类型 redisObject 存在内存开销，每个 redisObject 内部结构至少占 16 字节，甚至超过了整数自身空间消耗。所以 Redis 内存维护一个 [0-9999] 的整数对象池，用于节约内存。 除了整数值对象，其他类型如 list,hash,set,zset 内部元素也可以使用整数对象池。因此开发中在满足需求的前提下，尽量使用整数对象以节省内存。

3.  字符串优化
    
4.  编码优化
    
5.  控制 key 的数量

#### 8.3.32. [#](#redis-key-的过期时间和永久有效分别怎么设置) Redis key 的过期时间和永久有效分别怎么设置？

EXPIRE 和 PERSIST 命令

#### 8.3.33. [#](#redis-中的管道有什么用) Redis 中的管道有什么用？

一次请求 / 响应服务器能实现处理新的请求即使旧的请求还未被响应，这样就可以将多个命令发送到服务器，而不用等待回复，最后在一个步骤中读取该答复。

这就是管道（pipelining），是一种几十年来广泛使用的技术。例如许多 POP3 协议已经实现支持这个功能，大大加快了从服务器下载新邮件的过程。

#### 8.3.34. [#](#什么是redis事务) 什么是 redis 事务？

Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

总结说：redis 事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。

#### 8.3.35. [#](#redis事务相关命令) Redis 事务相关命令？

MULTI 、 EXEC 、 DISCARD 和 WATCH 是 Redis 事务相关的命令。

*   MULTI ：开启事务，redis 会将后续的命令逐个放入队列中，然后使用 EXEC 命令来原子化执行这个命令系列。
*   EXEC：执行事务中的所有操作命令。
*   DISCARD：取消事务，放弃执行事务块中的所有命令。
*   WATCH：监视一个或多个 key, 如果事务在执行前，这个 key(或多个 key) 被其他命令修改，则事务被中断，不会执行事务中的任何命令。
*   UNWATCH：取消 WATCH 对所有 key 的监视。

#### 8.3.36. [#](#redis事务的三个阶段) Redis 事务的三个阶段？

Redis 事务执行是三个阶段：

*   **开启**：以 MULTI 开始一个事务
    
*   **入队**：将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面
    
*   **执行**：由 EXEC 命令触发事务

当一个客户端切换到事务状态之后， 服务器会根据这个客户端发来的不同命令执行不同的操作：

*   如果客户端发送的命令为 EXEC 、 DISCARD 、 WATCH 、 MULTI 四个命令的其中一个， 那么服务器立即执行这个命令。
*   与此相反， 如果客户端发送的命令是 EXEC 、 DISCARD 、 WATCH 、 MULTI 四个命令以外的其他命令， 那么服务器并不立即执行这个命令， 而是将这个命令放入一个事务队列里面， 然后向客户端返回 QUEUED 回复。

![](https://pdai.tech/images/db/redis/db-redis-trans-1.png)

#### 8.3.37. [#](#redis事务其它实现) Redis 事务其它实现？

*   基于 Lua 脚本，Redis 可以保证脚本内的命令一次性、按顺序地执行，其同时也不提供事务运行错误的回滚，执行过程中如果部分命令运行错误，剩下的命令还是会继续运行完
    
*   基于中间标记变量，通过另外的标记变量来标识事务是否执行完成，读取数据时先读取该标记变量判断是否事务执行完成。但这样会需要额外写代码实现，比较繁琐

#### 8.3.38. [#](#redis事务中出现错误的处理) Redis 事务中出现错误的处理？

*   **语法错误（编译器错误）**

在开启事务后，修改 k1 值为 11，k2 值为 22，但 k2 语法错误，**最终导致事务提交失败，k1、k2 保留原值**。

```
/*
 * Redis 对象
 */
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码方式
    unsigned encoding:4;

    // LRU - 24位, 记录最末一次访问时间（相对于lru_clock）; 或者 LFU（最少使用的数据：8位频率，16位访问时间）
    unsigned lru:LRU_BITS; // LRU_BITS: 24

    // 引用计数
    int refcount;

    // 指向底层数据结构实例
    void *ptr;

} robj;
```

*   **Redis 类型错误（运行时错误）**

在开启事务后，修改 k1 值为 11，k2 值为 22，但将 k2 的类型作为 List，**在运行时检测类型错误，最终导致事务提交失败，此时事务并没有回滚，而是跳过错误命令继续执行**， 结果 k1 值改变、k2 保留原值

```
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> XADD memberMessage * msg one
QUEUED
127.0.0.1:6379> XADD memberMessage * msg two
QUEUED
127.0.0.1:6379> XADD memberMessage * msg three
QUEUED
127.0.0.1:6379> XADD memberMessage * msg four
QUEUED
127.0.0.1:6379> XADD memberMessage * msg five
QUEUED
127.0.0.1:6379> EXEC
1) "1553441006884-0"
2) "1553441006884-1"
3) "1553441006884-2"
4) "1553441006884-3"
5) "1553441006884-4"
```

#### 8.3.39. [#](#redis事务中watch是如何监视实现的呢) Redis 事务中 watch 是如何监视实现的呢？

Redis 使用 WATCH 命令来决定事务是继续执行还是回滚，那就需要在 MULTI 之前使用 WATCH 来监控某些键值对，然后使用 MULTI 命令来开启事务，执行对数据结构操作的各种命令，此时这些命令入队列。

当使用 EXEC 执行事务时，首先会比对 WATCH 所监控的键值对，如果没发生改变，它会执行事务队列中的命令，提交事务；如果发生变化，将不会执行事务中的任何命令，同时事务回滚。当然无论是否回滚，Redis 都会取消执行事务前的 WATCH 命令。

![](https://pdai.tech/images/db/redis/db-redis-trans-2.png)

#### 8.3.40. [#](#为什么-redis-不支持回滚) 为什么 Redis 不支持回滚？

以下是这种做法的优点：

*   Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
*   因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

有种观点认为 Redis 处理事务的做法会产生 bug ， 然而需要注意的是， 在通常情况下， **回滚并不能解决编程错误带来的问题**。 举个例子， 如果你本来想通过 INCR 命令将键的值加上 1 ， 却不小心加上了 2 ， 又或者对错误类型的键执行了 INCR ， 回滚是没有办法处理这些情况的。

#### 8.3.41. [#](#redis-对-acid的支持性理解) Redis 对 ACID 的支持性理解？

*   **原子性 atomicity**

首先通过上文知道 运行期的错误是不会回滚的，很多文章由此说 Redis 事务违背原子性的；而官方文档认为是遵从原子性的。

Redis 官方文档给的理解是，**Redis 的事务是原子性的：所有的命令，要么全部执行，要么全部不执行**。而不是完全成功。

*   **一致性 consistency**

redis 事务可以保证命令失败的情况下得以回滚，数据能恢复到没有执行之前的样子，是保证一致性的，除非 redis 进程意外终结。

*   **隔离性 Isolation**

redis 事务是严格遵守隔离性的，原因是 redis 是单进程单线程模式 (v6.0 之前），可以保证命令执行过程中不会被其他客户端命令打断。

但是，Redis 不像其它结构化数据库有隔离级别这种设计。

*   **持久性 Durability**

**redis 事务是不保证持久性的**，这是因为 redis 持久化策略中不管是 RDB 还是 AOF 都是异步执行的，不保证持久性是出于对性能的考虑。

#### 8.3.42. [#](#redis事务其他实现) Redis 事务其他实现？

基于 Lua 脚本，Redis 可以保证脚本内的命令一次性、按顺序地执行，其同时也不提供事务运行错误的回滚，执行过程中如果部分命令运行错误，剩下的命令还是会继续运行完

基于中间标记变量，通过另外的标记变量来标识事务是否执行完成，读取数据时先读取该标记变量判断是否事务执行完成。但这样会需要额外写代码实现，比较繁琐

#### 8.3.43. [#](#redis集群的主从复制模型是怎样的) Redis 集群的主从复制模型是怎样的？

主从复制，是指将一台 Redis 服务器的数据，复制到其他的 Redis 服务器。前者称为主节点 (master)，后者称为从节点 (slave)；数据的复制是单向的，只能由主节点到从节点。

**主从复制的作用**主要包括：

*   **数据冗余**：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
*   **故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
*   **负载均衡**：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写 Redis 数据时应用连接主节点，读 Redis 数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高 Redis 服务器的并发量。
*   **高可用基石**：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是 Redis 高可用的基础。

主从库之间采用的是**读写分离**的方式。

*   读操作：主库、从库都可以接收；
*   写操作：首先到主库执行，然后，主库将写操作同步给从库。

![](https://pdai.tech/images/db/redis/db-redis-copy-1.png)

注意：在 2.8 版本之前只有全量复制，而 2.8 版本后有全量和增量复制：

*   **全量（同步）复制**：比如第一次同步时
*   **增量（同步）复制**：只会把主从库网络断连期间主库收到的命令，同步给从库

#### 8.3.44. [#](#redis-全量复制的三个阶段) Redis 全量复制的三个阶段？

![](https://pdai.tech/images/db/redis/db-redis-copy-2.jpg)

**第一阶段是主从库间建立连接、协商同步的过程**，主要是为全量复制做准备。在这一步，从库和主库建立起连接，并告诉主库即将进行同步，主库确认回复后，主从库间就可以开始同步了。

具体来说，从库给主库发送 psync 命令，表示要进行数据同步，主库根据这个命令的参数来启动复制。psync 命令包含了主库的 runID 和复制进度 offset 两个参数。runID，是每个 Redis 实例启动时都会自动生成的一个随机 ID，用来唯一标记这个实例。当从库和主库第一次复制时，因为不知道主库的 runID，所以将 runID 设为 “？”。offset，此时设为 -1，表示第一次复制。主库收到 psync 命令后，会用 FULLRESYNC 响应命令带上两个参数：主库 runID 和主库目前的复制进度 offset，返回给从库。从库收到响应后，会记录下这两个参数。这里有个地方需要注意，FULLRESYNC 响应表示第一次复制采用的全量复制，也就是说，主库会把当前所有的数据都复制给从库。

**第二阶段，主库将所有数据同步给从库**。从库收到数据后，在本地完成数据加载。这个过程依赖于内存快照生成的 RDB 文件。

具体来说，主库执行 bgsave 命令，生成 RDB 文件，接着将文件发给从库。从库接收到 RDB 文件后，会先清空当前数据库，然后加载 RDB 文件。这是因为从库在通过 replicaof 命令开始和主库同步前，可能保存了其他数据。为了避免之前数据的影响，从库需要先把当前数据库清空。在主库将数据同步给从库的过程中，主库不会被阻塞，仍然可以正常接收请求。否则，Redis 的服务就被中断了。但是，这些请求中的写操作并没有记录到刚刚生成的 RDB 文件中。为了保证主从库的数据一致性，主库会在内存中用专门的 replication buffer，记录 RDB 文件生成后收到的所有写操作。

**第三个阶段，主库会把第二阶段执行过程中新收到的写命令，再发送给从库**。具体的操作是，当主库完成 RDB 文件发送后，就会把此时 replication buffer 中的修改操作发给从库，从库再重新执行这些操作。这样一来，主从库就实现同步了。

#### 8.3.45. [#](#redis-为什么会设计增量复制) Redis 为什么会设计增量复制？

如果主从库在命令传播时出现了网络闪断，那么，从库就会和主库重新进行一次全量复制，开销非常大。从 Redis 2.8 开始，网络断了之后，主从库会采用增量复制的方式继续同步。

#### 8.3.46. [#](#redis-增量复制的流程) Redis 增量复制的流程？

![](https://pdai.tech/images/db/redis/db-redis-copy-3.jpg)

先看两个概念： `replication buffer` 和 `repl_backlog_buffer`

`repl_backlog_buffer`：它是为了从库断开之后，如何找到主从差异数据而设计的环形缓冲区，从而避免全量复制带来的性能开销。如果从库断开时间太久，repl_backlog_buffer 环形缓冲区被主库的写命令覆盖了，那么从库连上主库后只能乖乖地进行一次全量复制，所以 **repl_backlog_buffer 配置尽量大一些，可以降低主从断开后全量复制的概率**。而在 repl_backlog_buffer 中找主从差异的数据后，如何发给从库呢？这就用到了 replication buffer。

`replication buffer`：Redis 和客户端通信也好，和从库通信也好，Redis 都需要给分配一个 内存 buffer 进行数据交互，客户端是一个 client，从库也是一个 client，我们每个 client 连上 Redis 后，Redis 都会分配一个 client buffer，所有数据交互都是通过这个 buffer 进行的：Redis 先把数据写到这个 buffer 中，然后再把 buffer 中的数据发到 client socket 中再通过网络发送出去，这样就完成了数据交互。所以主从在增量同步时，从库作为一个 client，也会分配一个 buffer，只不过这个 buffer 专门用来传播用户的写命令到从库，保证主从数据一致，我们通常把它叫做 replication buffer。

#### 8.3.47. [#](#增量复制如果在网络断开期间-repl-backlog-size环形缓冲区写满之后-从库是会丢失掉那部分被覆盖掉的数据-还是直接进行全量复制呢) 增量复制如果在网络断开期间，repl_backlog_size 环形缓冲区写满之后，从库是会丢失掉那部分被覆盖掉的数据，还是直接进行全量复制呢？

对于这个问题来说，有两个关键点：

1.  一个从库如果和主库断连时间过长，造成它在主库 repl_backlog_buffer 的 slave_repl_offset 位置上的数据已经被覆盖掉了，此时从库和主库间将进行全量复制。
    
2.  每个从库会记录自己的 slave_repl_offset，每个从库的复制进度也不一定相同。在和主库重连进行恢复时，从库会通过 psync 命令把自己记录的 slave_repl_offset 发给主库，主库会根据从库各自的复制进度，来决定这个从库可以进行增量复制，还是全量复制。

#### 8.3.48. [#](#redis-为什么不持久化的主服务器自动重启非常危险呢) Redis 为什么不持久化的主服务器自动重启非常危险呢?

*   我们设置节点 A 为主服务器，关闭持久化，节点 B 和 C 从节点 A 复制数据。
*   这时出现了一个崩溃，但 Redis 具有自动重启系统，重启了进程，因为关闭了持久化，节点重启后只有一个空的数据集。
*   节点 B 和 C 从节点 A 进行复制，现在节点 A 是空的，所以节点 B 和 C 上的复制数据也会被删除。
*   当在高可用系统中使用 Redis Sentinel，关闭了主服务器的持久化，并且允许自动重启，这种情况是很危险的。比如主服务器可能在很短的时间就完成了重启，以至于 Sentinel 都无法检测到这次失败，那么上面说的这种失败的情况就发生了。

如果数据比较重要，并且在使用主从复制时关闭了主服务器持久化功能的场景中，都应该禁止实例自动重启。

#### 8.3.49. [#](#redis-为什么主从全量复制使用rdb而不使用aof) Redis 为什么主从全量复制使用 RDB 而不使用 AOF？

1、RDB 文件内容是经过压缩的二进制数据（不同数据类型数据做了针对性优化），文件很小。而 AOF 文件记录的是每一次写操作的命令，写操作越多文件会变得很大，其中还包括很多对同一个 key 的多次冗余操作。在主从全量数据同步时，传输 RDB 文件可以尽量降低对主库机器网络带宽的消耗，从库在加载 RDB 文件时，一是文件小，读取整个文件的速度会很快，二是因为 RDB 文件存储的都是二进制数据，从库直接按照 RDB 协议解析还原数据即可，速度会非常快，而 AOF 需要依次重放每个写命令，这个过程会经历冗长的处理逻辑，恢复速度相比 RDB 会慢得多，所以使用 RDB 进行主从全量复制的成本最低。

2、假设要使用 AOF 做全量复制，意味着必须打开 AOF 功能，打开 AOF 就要选择文件刷盘的策略，选择不当会严重影响 Redis 性能。而 RDB 只有在需要定时备份和主从全量复制数据时才会触发生成一次快照。而在很多丢失数据不敏感的业务场景，其实是不需要开启 AOF 的。

#### 8.3.50. [#](#redis-为什么还有无磁盘复制模式) Redis 为什么还有无磁盘复制模式？

Redis 默认是磁盘复制，但是**如果使用比较低速的磁盘，这种操作会给主服务器带来较大的压力**。Redis 从 2.8.18 版本开始尝试支持无磁盘的复制。使用这种设置时，子进程直接将 RDB 通过网络发送给从服务器，不使用磁盘作为中间存储。

**无磁盘复制模式**：master 创建一个新进程直接 dump RDB 到 slave 的 socket，不经过主进程，不经过硬盘。适用于 disk 较慢，并且网络较快的时候。

使用`repl-diskless-sync`配置参数来启动无磁盘复制。

使用`repl-diskless-sync-delay` 参数来配置传输开始的延迟时间；master 等待一个`repl-diskless-sync-delay`的秒数，如果没 slave 来的话，就直接传，后来的得排队等了; 否则就可以一起传。

#### 8.3.51. [#](#redis-为什么还会有从库的从库的设计) Redis 为什么还会有从库的从库的设计？

通过分析主从库间第一次数据同步的过程，你可以看到，一次全量复制中，对于主库来说，需要完成两个耗时的操作：**生成 RDB 文件和传输 RDB 文件**。

如果从库数量很多，而且都要和主库进行全量复制的话，就会导致主库忙于 fork 子进程生成 RDB 文件，进行数据全量复制。fork 这个操作会阻塞主线程处理正常请求，从而导致主库响应应用程序的请求速度变慢。此外，传输 RDB 文件也会占用主库的网络带宽，同样会给主库的资源使用带来压力。那么，有没有好的解决方法可以分担主库压力呢？

其实是有的，这就是 “主 - 从 - 从” 模式。

在刚才介绍的主从库模式中，所有的从库都是和主库连接，所有的全量复制也都是和主库进行的。现在，我们可以通过 “主 - 从 - 从” 模式**将主库生成 RDB 和传输 RDB 的压力，以级联的方式分散到从库上**。

简单来说，我们在部署主从集群的时候，可以手动选择一个从库（比如选择内存资源配置较高的从库），用于级联其他的从库。然后，我们可以再选择一些从库（例如三分之一的从库），在这些从库上执行如下命令，让它们和刚才所选的从库，建立起主从关系。

这样一来，这些从库就会知道，在进行同步时，不用再和主库进行交互了，只要和级联的从库进行写操作同步就行了，这就可以减轻主库上的压力，如下图所示：

![](https://pdai.tech/images/db/redis/db-redis-copy-4.jpg)

级联的 “主 - 从 - 从” 模式好了，到这里，我们了解了主从库间通过全量复制实现数据同步的过程，以及通过 “主 - 从 - 从” 模式分担主库压力的方式。那么，一旦主从库完成了全量复制，它们之间就会一直维护一个网络连接，主库会通过这个连接将后续陆续收到的命令操作再同步给从库，这个过程也称为基于长连接的命令传播，可以避免频繁建立连接的开销。

#### 8.3.52. [#](#redis哨兵机制-哨兵实现了什么功能呢) Redis 哨兵机制？哨兵实现了什么功能呢?

哨兵的核心功能是主节点的自动故障转移。

下图是一个典型的哨兵集群监控的逻辑图：

![](https://pdai.tech/images/db/redis/db-redis-sen-1.png)

哨兵实现了什么功能呢？下面是 Redis 官方文档的描述：

*   **监控（Monitoring）**：哨兵会不断地检查主节点和从节点是否运作正常。
*   **自动故障转移（Automatic failover）**：当主节点不能正常工作时，哨兵会开始自动故障转移操作，它会将失效主节点的其中一个从节点升级为新的主节点，并让其他从节点改为复制新的主节点。
*   **配置提供者（Configuration provider）**：客户端在初始化时，通过连接哨兵来获得当前 Redis 服务的主节点地址。
*   **通知（Notification）**：哨兵可以将故障转移的结果发送给客户端。

其中，监控和自动故障转移功能，使得哨兵可以及时发现主节点故障并完成转移；而配置提供者和通知功能，则需要在与客户端的交互中才能体现。

#### 8.3.53. [#](#redis-哨兵集群是通过什么方式组建的) Redis 哨兵集群是通过什么方式组建的？

哨兵实例之间可以相互发现，要归功于 Redis 提供的 pub/sub 机制，也就是发布 / 订阅机制。

在主从集群中，主库上有一个名为`__sentinel__:hello`的频道，不同哨兵就是通过它来相互发现，实现互相通信的。在下图中，哨兵 1 把自己的 IP（172.16.19.3）和端口（26579）发布到`__sentinel__:hello`频道上，哨兵 2 和 3 订阅了该频道。那么此时，哨兵 2 和 3 就可以从这个频道直接获取哨兵 1 的 IP 地址和端口号。然后，哨兵 2、3 可以和哨兵 1 建立网络连接。

![](https://pdai.tech/images/db/redis/db-redis-sen-6.jpg)

通过这个方式，哨兵 2 和 3 也可以建立网络连接，这样一来，哨兵集群就形成了。它们相互间可以通过网络连接进行通信，比如说对主库有没有下线这件事儿进行判断和协商。

#### 8.3.54. [#](#redis-哨兵是如何监控redis集群的) Redis 哨兵是如何监控 Redis 集群的？

这是由哨兵向主库发送 INFO 命令来完成的。就像下图所示，哨兵 2 给主库发送 INFO 命令，主库接受到这个命令后，就会把从库列表返回给哨兵。接着，哨兵就可以根据从库列表中的连接信息，和每个从库建立连接，并在这个连接上持续地对从库进行监控。哨兵 1 和 3 可以通过相同的方法和从库建立连接。

![](https://pdai.tech/images/db/redis/db-redis-sen-7.jpg)

#### 8.3.55. [#](#redis-哨兵如何判断主库已经下线了呢) Redis 哨兵如何判断主库已经下线了呢？

首先要理解两个概念：**主观下线**和**客观下线**

*   **主观下线**：任何一个哨兵都是可以监控探测，并作出 Redis 节点下线的判断；
*   **客观下线**：有哨兵集群共同决定 Redis 节点是否下线；

当某个哨兵（如下图中的哨兵 2）判断主库 “主观下线” 后，就会给其他哨兵发送 `is-master-down-by-addr` 命令。接着，其他哨兵会根据自己和主库的连接情况，做出 Y 或 N 的响应，Y 相当于赞成票，N 相当于反对票。

![](https://pdai.tech/images/db/redis/db-redis-sen-2.jpg)

如果赞成票数（这里是 2）是大于等于哨兵配置文件中的 `quorum` 配置项（比如这里如果是 quorum=2）, 则可以判定**主库客观下线**了。

#### 8.3.56. [#](#redis-哨兵的选举机制是什么样的) Redis 哨兵的选举机制是什么样的？

*   **为什么必然会出现选举 / 共识机制**？

为了避免哨兵的单点情况发生，所以需要一个哨兵的分布式集群。作为分布式集群，必然涉及共识问题（即选举问题）；同时故障的转移和通知都只需要一个主的哨兵节点就可以了。

*   **哨兵的选举机制是什么样的**？

哨兵的选举机制其实很简单，就是一个 Raft 选举算法： **选举的票数大于等于 num(sentinels)/2+1 时，将成为领导者，如果没有超过，继续选举**

Raft 算法你可以参看这篇文章[分布式算法 - Raft 算法](https://pdai.tech/md/algorithm/alg-domain-distribute-x-raft.html)

*   **任何一个想成为 Leader 的哨兵，要满足两个条件**：
    *   第一，拿到半数以上的赞成票；
    *   第二，拿到的票数同时还需要大于等于哨兵配置文件中的 quorum 值。

以 3 个哨兵为例，假设此时的 quorum 设置为 2，那么，任何一个想成为 Leader 的哨兵只要拿到 2 张赞成票，就可以了。

#### 8.3.57. [#](#redis-1主4从-5个哨兵-哨兵配置quorum为2-如果3个哨兵故障-当主库宕机时-哨兵能否判断主库-客观下线-能否自动切换) Redis 1 主 4 从，5 个哨兵，哨兵配置 quorum 为 2，如果 3 个哨兵故障，当主库宕机时，哨兵能否判断主库 “客观下线”？能否自动切换？

经过实际测试：

1、哨兵集群可以判定主库 “主观下线”。由于 quorum=2，所以当一个哨兵判断主库“主观下线” 后，询问另外一个哨兵后也会得到同样的结果，2 个哨兵都判定“主观下线”，达到了 quorum 的值，因此，**哨兵集群可以判定主库为 “客观下线”**。

2、**但哨兵不能完成主从切换**。哨兵标记主库 “客观下线后”，在选举“哨兵领导者” 时，一个哨兵必须拿到超过多数的选票(5/2+1=3 票)。但目前只有 2 个哨兵活着，无论怎么投票，一个哨兵最多只能拿到 2 票，永远无法达到`N/2+1`选票的结果。

#### 8.3.58. [#](#主库判定客观下线了-那么如何从剩余的从库中选择一个新的主库呢) 主库判定客观下线了，那么如何从剩余的从库中选择一个新的主库呢？

*   过滤掉不健康的（下线或断线），没有回复过哨兵 ping 响应的从节点
*   选择`salve-priority`从节点优先级最高（redis.conf）的
*   选择复制偏移量最大，只复制最完整的从节点

![](https://pdai.tech/images/db/redis/db-redis-sen-3.jpg)

#### 8.3.59. [#](#新的主库选择出来后-如何进行故障的转移) 新的主库选择出来后，如何进行故障的转移？

假设根据我们一开始的图：（我们假设：判断主库客观下线了，同时选出`sentinel 3`是哨兵 leader）

![](https://pdai.tech/images/db/redis/db-redis-sen-1.png)

**故障转移流程如下**：

![](https://pdai.tech/images/db/redis/db-redis-sen-4.png)

*   将 slave-1 脱离原从节点（PS: 5.0 中应该是`replicaof no one`)，升级主节点，
*   将从节点 slave-2 指向新的主节点
*   通知客户端主节点已更换
*   将原主节点（oldMaster）变成从节点，指向新的主节点

**转移之后**

![](https://pdai.tech/images/db/redis/db-redis-sen-5.png)

#### 8.3.60. [#](#redis事件机制) Redis 事件机制?

Redis 中的事件驱动库只关注网络 IO，以及定时器。该事件库处理下面两类事件：

*   **文件事件** (file event)：用于处理 Redis 服务器和客户端之间的网络 IO。
*   **时间事件** (time eveat)：Redis 服务器中的一些操作（比如 serverCron 函数）需要在给定的时间点执行，而时间事件就是处理这类定时操作的。

事件驱动库的代码主要是在`src/ae.c`中实现的，其示意图如下所示。

![](https://pdai.tech/images/db/redis/db-redis-event-1.png)

`aeEventLoop`是整个事件驱动的核心，它管理着文件事件表和时间事件列表，不断地循环处理着就绪的文件事件和到期的时间事件。

#### 8.3.61. [#](#redis文件事件的模型) Redis 文件事件的模型？

Redis 基于 **Reactor 模式**开发了自己的网络事件处理器，也就是文件事件处理器。文件事件处理器使用 **IO 多路复用技术**，同时监听多个套接字，并为套接字关联不同的事件处理函数。当套接字的可读或者可写事件触发时，就会调用相应的事件处理函数。

*   **1. 为什么单线程的 Redis 能那么快**？

Redis 的瓶颈主要在 IO 而不是 CPU，所以为了省开发量，在 6.0 版本前是单线程模型；其次，Redis 是单线程主要是指 **Redis 的网络 IO 和键值对读写是由一个线程来完成的**，这也是 Redis 对外提供键值存储服务的主要流程。（但 Redis 的其他功能，比如持久化、异步删除、集群数据同步等，其实是由额外的线程执行的）。

Redis 采用了多路复用机制使其在网络 IO 操作中能并发处理大量的客户端请求，实现高吞吐率。

*   **2. Redis 事件响应框架 ae_event 及文件事件处理器**

Redis 并没有使用 libevent 或者 libev 这样的成熟开源方案，而是自己实现一个非常简洁的事件驱动库 ae_event。@pdai

Redis 使用的 IO 多路复用技术主要有：`select`、`epoll`、`evport`和`kqueue`等。每个 IO 多路复用函数库在 Redis 源码中都对应一个单独的文件，比如`ae_select.c`，`ae_epoll.c`， `ae_kqueue.c`等。Redis 会根据不同的操作系统，按照不同的优先级选择多路复用技术。事件响应框架一般都采用该架构，比如 netty 和 libevent。

![](https://pdai.tech/images/db/redis/db-redis-event-2.png)

如下图所示，文件事件处理器有四个组成部分，它们分别是套接字、I/O 多路复用程序、文件事件分派器以及事件处理器。

![](https://pdai.tech/images/db/redis/db-redis-event-3.png)

文件事件是对套接字操作的抽象，每当一个套接字准备好执行 `accept`、`read`、`write`和 `close` 等操作时，就会产生一个文件事件。因为 Redis 通常会连接多个套接字，所以多个文件事件有可能并发的出现。

I/O 多路复用程序负责监听多个套接字，并向文件事件派发器传递那些产生了事件的套接字。

尽管多个文件事件可能会并发地出现，但 I/O 多路复用程序总是会将所有产生的套接字都放到同一个队列 (也就是后文中描述的 aeEventLoop 的 fired 就绪事件表) 里边，然后文件事件处理器会以有序、同步、单个套接字的方式处理该队列中的套接字，也就是处理就绪的文件事件。

![](https://pdai.tech/images/db/redis/db-redis-event-4.png)

#### 8.3.62. [#](#什么是redis发布订阅) 什么是 Redis 发布订阅？

Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。

Redis 的 SUBSCRIBE 命令可以让客户端订阅任意数量的频道， 每当有新信息发送到被订阅的频道时， 信息就会被发送给所有订阅指定频道的客户端。

作为例子， 下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![](https://pdai.tech/images/db/redis/db-redis-sub-1.svg)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![](https://pdai.tech/images/db/redis/db-redis-sub-2.svg)

#### 8.3.63. [#](#redis发布订阅有哪两种方式) Redis 发布订阅有哪两种方式？

*   **基于频道 (Channel) 的发布 / 订阅**

"发布 / 订阅" 模式包含两种角色，分别是发布者和订阅者。发布者可以向指定的频道 (channel) 发送消息; 订阅者可以订阅一个或者多个频道(channel), 所有订阅此频道的订阅者都会收到此消息。

![](https://pdai.tech/images/db/redis/db-redis-sub-8.png)

*   **基于模式 (pattern) 的发布 / 订阅**

下图展示了一个带有频道和模式的例子， 其中 tweet.shop.* 模式匹配了 tweet.shop.kindle 频道和 tweet.shop.ipad 频道， 并且有不同的客户端分别订阅它们三个：

![](https://pdai.tech/images/db/redis/db-redis-sub-5.svg)

当有信息发送到 tweet.shop.kindle 频道时， 信息除了发送给 clientX 和 clientY 之外， 还会发送给订阅 tweet.shop.* 模式的 client123 和 client256 ：

![](https://pdai.tech/images/db/redis/db-redis-sub-6.svg)

另一方面， 如果接收到信息的是频道 tweet.shop.ipad ， 那么 client123 和 client256 同样会收到信息：

![](https://pdai.tech/images/db/redis/db-redis-sub-7.svg)

#### 8.3.64. [#](#什么是redis-cluster) 什么是 Redis Cluster？

Redis-cluster 是一种服务器 Sharding 技术，Redis3.0 以后版本正式提供支持。

#### 8.3.65. [#](#说说redis哈希槽的概念-为什么是16384个) 说说 Redis 哈希槽的概念？为什么是 16384 个？

Redis-cluster 没有使用一致性 hash，而是引入了**哈希槽**的概念。Redis-cluster 中有 16384(即 2 的 14 次方）个哈希槽，每个 key 通过 CRC16 校验后对 16383 取模来决定放置哪个槽。Cluster 中的每个节点负责一部分 hash 槽（hash slot）。

比如集群中存在三个节点，则可能存在的一种分配如下：

1.  节点 A 包含 0 到 5500 号哈希槽；
2.  节点 B 包含 5501 到 11000 号哈希槽；
3.  节点 C 包含 11001 到 16384 号哈希槽。

*   **为什么是 16384 个**

在 redis 节点发送心跳包时需要把所有的槽放到这个心跳包里，以便让节点知道当前集群信息，16384=16k，在发送心跳包时使用 char 进行 bitmap 压缩后是 2k（2 * 8 (8 bit) * 1024(1k) = 16K），也就是说使用 2k 的空间创建了 16k 的槽数。

虽然使用 CRC16 算法最多可以分配 65535（2^16-1）个槽位，65535=65k，压缩后就是 8k（8 * 8 (8 bit) * 1024(1k) =65K），也就是说需要需要 8k 的心跳包，作者认为这样做不太值得；并且一般情况下一个 redis 集群不会有超过 1000 个 master 节点，所以 16k 的槽位是个比较合适的选择。

#### 8.3.66. [#](#redis集群会有写操作丢失吗-为什么) Redis 集群会有写操作丢失吗？为什么？

Redis 并不能保证数据的强一致性，这意味这在实际中集群在特定的条件下可能会丢失写操作。

#### 8.3.67. [#](#redis-客户端有哪些) Redis 客户端有哪些？

Redisson、Jedis、lettuce 等等，官方推荐使用 Redisson。

Redisson 是一个高级的分布式协调 Redis 客服端，能帮助用户在分布式环境中轻松实现一些 Java 的对象 (Bloom filter, BitSet, Set, SetMultimap, ScoredSortedSet, SortedSet, Map, ConcurrentMap, List, ListMultimap, Queue, BlockingQueue, Deque, BlockingDeque, Semaphore, Lock, ReadWriteLock, AtomicLong, CountDownLatch, Publish / Subscribe, HyperLogLog)。

#### 8.3.68. [#](#redis如何做大量数据插入) Redis 如何做大量数据插入？

Redis2.6 开始 redis-cli 支持一种新的被称之为 pipe mode 的新模式用于执行大量数据插入工作。

#### 8.3.69. [#](#redis实现分布式锁实现-什么是-redlock) Redis 实现分布式锁实现? 什么是 RedLock?

*   **常规**

加锁： SET NX PX + 校验唯一随机值

解锁： Lua 脚本

*   **RedLock**

搞多个 Redis master 部署，以保证它们不会同时宕掉。并且这些 master 节点是完全相互独立的，相互之间不存在数据同步。同时，需要确保在这多个 master 实例上，是与在 Redis 单实例，使用相同方法来获取和释放锁。

*   **Redisson 框架**

Redisson watchdog 或者它实现了 RedLock 方式

具体可以看后文分布式锁中实现方式。

#### 8.3.70. [#](#redis缓存有哪些问题-如何解决) Redis 缓存有哪些问题，如何解决？

当缓存库出现时，必须要考虑如下问题：

*   **缓存穿透**
    
    *   **问题来源**: 缓存穿透是指**缓存和数据库中都没有的数据**，而用户不断发起请求。由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。
    *   **解决方案**
        *   接口层增加校验，如用户鉴权校验，id 做基础校验，id<=0 的直接拦截；
        *   从缓存取不到的数据，在数据库中也没有取到，这时也可以将 key-value 对写为 key-null，缓存有效时间可以设置短点，如 30 秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个 id 暴力攻击
        *   布隆过滤器。bloomfilter 就类似于一个 hash set，用于快速判某个元素是否存在于集合中，其典型的应用场景就是快速判断一个 key 是否存在于某容器，不存在就直接返回。布隆过滤器的关键就在于 hash 算法和容器大小
*   **缓存穿击**
    
    *   **问题来源**: 缓存击穿是指**缓存中没有但数据库中有的数据**（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。
    *   **解决方案**
        *   设置热点数据永远不过期。
        *   接口限流与熔断，降级。重要的接口一定要做好限流策略，防止用户恶意刷接口，同时要降级准备，当接口中的某些 服务 不可用时候，进行熔断，失败快速返回机制。
        *   加互斥锁
*   **缓存雪崩**
    
    *   **问题来源**: 缓存雪崩是指缓存中**数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至 down 机**。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。
    *   **解决方案**
        *   缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
        *   如果缓存数据库是分布式部署，将热点数据均匀分布在不同的缓存数据库中。
        *   设置热点数据永远不过期。
*   **缓存污染**（或者满了）

缓存污染问题说的是缓存中一些只会被访问一次或者几次的的数据，被访问完后，再也不会被访问到，但这部分数据依然留存在缓存中，消耗缓存空间。

缓存污染会随着数据的持续增加而逐渐显露，随着服务的不断运行，缓存中会存在大量的永远不会再次被访问的数据。缓存空间是有限的，如果缓存空间满了，再往缓存里写数据时就会有额外开销，影响 Redis 性能。这部分额外开销主要是指写的时候判断淘汰策略，根据淘汰策略去选择要淘汰的数据，然后进行删除操作。

#### 8.3.71. [#](#redis性能问题有哪些-如何分析定位解决) Redis 性能问题有哪些，如何分析定位解决?

举几个例子

*   **看延迟** 60 秒内的最大响应延迟：

```
127.0.0.1:6379> XPENDING mq mqGroup # mpGroup的Pending情况
1) (integer) 5 # 5个已读取但未处理的消息
2) "1553585533795-0" # 起始ID
3) "1553585533795-4" # 结束ID
4) 1) 1) "consumerA" # 消费者A有3个
      2) "3"
   2) 1) "consumerB" # 消费者B有1个
      2) "1"
   3) 1) "consumerC" # 消费者C有1个
      2) "1"

127.0.0.1:6379> XPENDING mq mqGroup - + 10 # 使用 start end count 选项可以获取详细信息
1) 1) "1553585533795-0" # 消息ID
   2) "consumerA" # 消费者
   3) (integer) 1654355 # 从读取到现在经历了1654355ms，IDLE
   4) (integer) 5 # 消息被读取了5次，delivery counter
2) 1) "1553585533795-1"
   2) "consumerA"
   3) (integer) 1654355
   4) (integer) 4
# 共5个，余下3个省略 ...

127.0.0.1:6379> XPENDING mq mqGroup - + 10 consumerA # 在加上消费者参数，获取具体某个消费者的Pending列表
1) 1) "1553585533795-0"
   2) "consumerA"
   3) (integer) 1641083
   4) (integer) 5
# 共3个，余下2个省略 ...
```

*   **慢日志**（slowlog）

慢查询，就会导致后面的请求发生排队，对于客户端来说，响应延迟也会变长。

![](https://pdai.tech/images/db/redis/redis-performance-2.jpeg)

*   **bigkey**

大对象

*   **集中过期**

一般有两种方案来规避这个问题：

1.  集中过期 key 增加一个随机过期时间，把集中过期的时间打散，降低 Redis 清理过期 key 的压力
2.  如果你使用的 Redis 是 4.0 以上版本，可以开启 lazy-free 机制，当删除过期 key 时，把释放内存的操作放到后台线程中执行，避免阻塞主线程

*   **fork 耗时严重**

主进程创建子进程，会调用操作系统提供的 fork 函数

*   **使用 Swap**

当内存中的数据被换到磁盘上后，Redis 再访问这些数据时，就需要从磁盘上读取，访问磁盘的速度要比访问内存慢几百倍！

*   **内存碎片**

Redis 4.0 版本，它正好提供了自动碎片整理的功能，可以通过配置开启碎片自动整理

#### 8.3.72. [#](#redis单线程模型-在6-0之前如何提高多核cpu的利用率) Redis 单线程模型？ 在 6.0 之前如何提高多核 CPU 的利用率？

可以在同一个服务器部署多个 Redis 的实例，并把他们当作不同的服务器来使用，在某些时候，无论如何一个服务器是不够的， 所以，如果你想使用多个 CPU，你可以考虑一下分片（shard）。

#### 8.3.73. [#](#redis6-0之前的版本真的是单线程的吗) Redis6.0 之前的版本真的是单线程的吗?

Redis 在处理客户端请求时, 包括获取 (socket 读)、解析、执行、内容返回(socket 写) 等都是由一个顺序串行的主线程执行的, 这就是所谓的 单线程. 单如果严格讲, 从 Redis4.0 之后并不是单线程, 除了主线程之外, 它也有后台线程在处理一些较为缓慢的操作, 例如 清理脏数据, 无用链接的释放, 大 key 的删除, 数据持久化 bgsave,bgrewriteaof 等, 都是在主线程之外的子线程单独执行的.

#### 8.3.74. [#](#redis6-0之前为什么一致不用多线程) Redis6.0 之前为什么一致不用多线程?

官方曾做过类似问题的回复：使用 Redis 时，几乎不存在 CPU 成为瓶颈的情况， Redis 主要受限于内存和网络。例如在一个普通的 Linux 系统上，Redis 通过使用 pipelining 每秒可以处理 100 万个请求，所以如果应用程序主要使用 O(N) 或 O(log(N)) 的命令，它几乎不会占用太多 CPU。

使用了单线程后，可维护性高。多线程模型虽然在某些方面表现优异，但是它却引入了程序执行顺序的不确定性，带来了并发读写的一系列问题，增加了系统复杂度、同时可能存在线程切换、甚至加锁解锁、死锁造成的性能损耗。Redis 通过 AE 事件模型以及 IO 多路复用等技术，处理性能非常高，因此没有必要使用多线程。单线程机制使得 Redis 内部实现的复杂度大大降低，Hash 的惰性 Rehash、Lpush 等等 “线程不安全” 的命令都可以无锁进行。

#### 8.3.75. [#](#redis6-0为什么要引入多线程呢) Redis6.0 为什么要引入多线程呢？

Redis 将所有数据放在内存中，内存的响应时长大约为 100 纳秒，对于小数据包，Redis 服务器可以处理 80,000 到 100,000 QPS，这也是 Redis 处理的极限了，对于 80% 的公司来说，单线程的 Redis 已经足够使用了。

但随着越来越复杂的业务场景，有些公司动不动就上亿的交易量，因此需要更大的 QPS。常见的解决方案是在分布式架构中对数据进行分区并采用多个服务器，但该方案有非常大的缺点，例如要管理的 Redis 服务器太多，维护代价大；某些适用于单个 Redis 服务器的命令不适用于数据分区；数据分区无法解决热点读 / 写问题；数据偏斜，重新分配和放大 / 缩小变得更加复杂等等。

从 Redis 自身角度来说，因为读写网络的 read/write 系统调用占用了 Redis 执行期间大部分 CPU 时间，瓶颈主要在于网络的 IO 消耗, 优化主要有两个方向:

*   提高网络 IO 性能，典型的实现比如使用 DPDK 来替代内核网络栈的方式
*   使用多线程充分利用多核，典型的实现比如 Memcached

协议栈优化的这种方式跟 Redis 关系不大，支持多线程是一种最有效最便捷的操作方式。所以总结起来，redis 支持多线程主要就是两个原因：

*   可以充分利用服务器 CPU 资源，目前主线程只能利用一个核
*   多线程任务可以分摊 Redis 同步 IO 读写负荷

#### 8.3.76. [#](#redis6-0默认是否开启了多线程) Redis6.0 默认是否开启了多线程？

Redis6.0 的多线程默认是禁用的，只使用主线程。如需开启需要修改 redis.conf 配置文件：io-threads-do-reads yes

#### 8.3.77. [#](#redis6-0多线程开启时-线程数如何设置) Redis6.0 多线程开启时，线程数如何设置？

开启多线程后，还需要设置线程数，否则是不生效的。同样修改 redis.conf 配置文件 io-threads4

关于线程数的设置，官方有一个建议：4 核的机器建议设置为 2 或 3 个线程，8 核的建议设置为 6 个线程，线程数一定要小于机器核数。还需要注意的是，线程数并不是越大越好，官方认为超过了 8 个基本就没什么意义了。

#### 8.3.78. [#](#redis6-0多线程的实现机制) Redis6.0 多线程的实现机制？

核心思路是，将主线程的 IO 读写任务拆分出来给一组独立的线程执行，使得多个 socket 的读写可以并行化

*   主线程负责接收建立连接的请求, 获取 socket 放到全局等待处理队列
*   主线程处理完读事件之后, 通过 Round Robin 将这些连接分配给 IO 线程 (并不会等待队列满)
*   主线程阻塞等待 IO 线程读取 socket 完毕
*   主线程通过单线程的方式执行请求命令，请求数据读取并解析完成，但并不执行
*   主线程阻塞等待 IO 线程将数据回写 socket 完毕
*   解除绑定, 清空等待队列

该线程有如下特点:

*   IO 线程要么同时在读 socket，要么同时在写，不会同时读或写
*   IO 线程只负责读写 socket 解析命令，不负责命令处理（主线程串行执行命令）

#### 8.3.79. [#](#开启多线程后-是否会存在线程并发安全问题) 开启多线程后，是否会存在线程并发安全问题？

Redis 的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程顺序执行, 因此不存在线程的并发安全问题

### 8.4. [#](#_8-4-mongodb) 8.4 MongoDB

#### 8.4.1. [#](#什么是mongodb-为什么使用mongodb) 什么是 MongoDB？为什么使用 MongoDB？

MongoDB 是面向文档的 NoSQL 数据库，用于大量数据存储。MongoDB 是一个在 2000 年代中期问世的数据库。属于 NoSQL 数据库的类别。以下是一些为什么应该开始使用 MongoDB 的原因

*   **面向文档的**–由于 MongoDB 是 NoSQL 类型的数据库，它不是以关系类型的格式存储数据，而是将数据存储在文档中。这使得 MongoDB 非常灵活，可以适应实际的业务环境和需求。
*   **临时查询** -MongoDB 支持按字段，范围查询和正则表达式搜索。可以查询返回文档中的特定字段。
*   **索引** - 可以创建索引以提高 MongoDB 中的搜索性能。MongoDB 文档中的任何字段都可以建立索引。
*   **复制** -MongoDB 可以提供副本集的高可用性。副本集由两个或多个 mongo 数据库实例组成。每个副本集成员可以随时充当主副本或辅助副本的角色。主副本是与客户端交互并执行所有读 / 写操作的主服务器。辅助副本使用内置复制维护主数据的副本。当主副本发生故障时，副本集将自动切换到辅助副本，然后它将成为主服务器。
*   **负载平衡** -MongoDB 使用分片的概念，通过在多个 MongoDB 实例之间拆分数据来水平扩展。MongoDB 可以在多台服务器上运行，以平衡负载或复制数据，以便在硬件出现故障时保持系统正常运行。

#### 8.4.2. [#](#mongodb与rdbms区别-有哪些术语) MongoDB 与 RDBMS 区别？有哪些术语？

下表将帮助您更容易理解 Mongo 中的一些概念：

<table><thead><tr><th>SQL 术语 / 概念</th><th>MongoDB 术语 / 概念</th><th>解释 / 说明</th></tr></thead><tbody><tr><td>database</td><td>database</td><td>数据库</td></tr><tr><td>table</td><td>collection</td><td>数据库表 / 集合</td></tr><tr><td>row</td><td>document</td><td>数据记录行 / 文档</td></tr><tr><td>column</td><td>field</td><td>数据字段 / 域</td></tr><tr><td>index</td><td>index</td><td>索引</td></tr><tr><td>table joins</td><td></td><td>表连接, MongoDB 不支持</td></tr><tr><td>primary key</td><td>primary key</td><td>主键, MongoDB 自动将_id 字段设置为主键</td></tr></tbody></table>

![](https://pdai.tech/images/db/mongo/mongo-y-arch-2.png)

#### 8.4.3. [#](#mongodb聚合的管道方式) MongoDB 聚合的管道方式？

Aggregation Pipline： 类似于将 SQL 中的 group by + order by + left join ... 等操作管道化。MongoDB 的聚合管道（Pipline）将 MongoDB 文档在一个阶段（Stage）处理完毕后将结果传递给下一个阶段（Stage）处理。**阶段（Stage）操作是可以重复的**。

![](https://pdai.tech/images/db/mongo/mongo-x-usage-11.png)

#### 8.4.4. [#](#mongodb聚合的map-reduce方式) MongoDB 聚合的 Map Reduce 方式？

![](https://pdai.tech/images/db/mongo/mongo-x-usage-12.png)

#### 8.4.5. [#](#spring-data-和mongodb集成) Spring Data 和 MongoDB 集成？

![](https://pdai.tech/images/db/mongo/mongo-x-usage-spring-5.png)

*   引入`mongodb-driver`, 使用最原生的方式通过 Java 调用 mongodb 提供的 Java driver;
*   引入`spring-data-mongo`, 自行配置使用 spring data 提供的对 MongoDB 的封装
    *   使用`MongoTemplate` 的方式
    *   使用`MongoRespository` 的方式
*   引入`spring-data-mongo-starter`, 采用 spring autoconfig 机制自动装配，然后再使用`MongoTemplate`或者`MongoRespository`方式。

#### 8.4.6. [#](#mongodb-有哪几种存储引擎) MongoDB 有哪几种存储引擎？

MongoDB 一共提供了三种存储引擎：WiredTiger，MMAPV1 和 In Memory；在 MongoDB3.2 之前采用的是 MMAPV1; 后续版本 v3.2 开始默认采用 WiredTiger； 且在 v4.2 版本中移除了 MMAPV1 的引擎。

#### 8.4.7. [#](#谈谈你对mongodb-wt存储引擎的理解) 谈谈你对 MongoDB WT 存储引擎的理解？

从几个方面回答，比如：

*   **插件式存储引擎架构**

实现了 Server 层和存储引擎层的解耦，可以支持多种存储引擎，如 MySQL 既可以支持 B-Tree 结构的 InnoDB 存储引擎，还可以支持 LSM 结构的 RocksDB 存储引擎。

*   **B-Tree + Page**

![](https://pdai.tech/images/db/mongo/mongo-y-ds-3.jpg)

上图是 WiredTiger 在内存里面的大概布局图，通过它我们可梳理清楚存储引擎是如何将数据加载到内存，然后如何通过相应数据结构来支持查询、插入、修改操作的。

内存里面 B-Tree 包含三种类型的 page，即 rootpage、internal page 和 leaf page，前两者包含指向其子页的 page index 指针，不包含集合中的真正数据，leaf page 包含集合中的真正数据即 keys/values 和指向父页的 home 指针；

*   **为什么是 Page**？

数据以 page 为单位加载到 cache、cache 里面又会生成各种不同类型的 page 及为不同类型的 page 分配不同大小的内存、eviction 触发机制和 reconcile 动作都发生在 page 上、page 大小持续增加时会被分割成多个小 page，所有这些操作都是围绕一个 page 来完成的。

Page 的典型生命周期如下图所示：

![](https://pdai.tech/images/db/mongo/mongo-y-page-1.png)

*   **什么是 CheckPoint**?

本质上来说，Checkpoint 相当于一个日志，记录了上次 Checkpoint 后相关数据文件的变化。作用： 一是将内存里面发生修改的数据写到数据文件进行持久化保存，确保数据一致性； 二是实现数据库在某个时刻意外发生故障，再次启动时，缩短数据库的恢复时间，WiredTiger 存储引擎中的 Checkpoint 模块就是来实现这个功能的。

一个 Checkpoint 包含关键信息如下图所示：

![](https://pdai.tech/images/db/mongo/mongo-x-checkpoint-1.png)

每个 checkpoint 包含一个 root page、三个指向磁盘具体位置上 pages 的列表以及磁盘上文件的大小。

*   **如何理解 WT 事务机制**？

要了解实现先要知道它的事务的构造和使用相关的技术，WT 在实现事务的时使用主要是使用了三个技术：**snapshot(事务快照)**、**MVCC (多版本并发控制) **和** redo log(重做日志)**，为了实现这三个技术，它还定义了一个基于这三个技术的事务对象和**全局事务管理器**。

*   **如何理解 WT 缓存淘汰**？

eviction cache 是一个 LRU cache，即页面置换算法缓冲区，它对数据页采用的是**分段局部扫描和淘汰**，而不是对内存中所有的数据页做全局管理。基本思路是一个线程阶段性的去扫描各个 btree，并把 btree 可以进行淘汰的数据页添加到一个 lru queue 中，当 queue 填满了后记录下这个过程当前的 btree 对象和 btree 的位置（这个位置是为了作为下次阶段性扫描位置）, 然后对 queue 中的数据页按照访问热度排序，最后各个淘汰线程按照淘汰优先级淘汰 queue 中的数据页，整个过程是周期性重复。WT 的这个 evict 过程涉及到多个 eviction thread 和 hazard pointer 技术。

**WT 的 evict 过程都是以 page 为单位做淘汰，而不是以 K/V。这一点和 memcache、redis 等常用的缓存 LRU 不太一样，因为在磁盘上数据的最小描述单位是 page block，而不是记录**。

#### 8.4.8. [#](#mongodb为什么要引入复制集-有哪些成员) MongoDB 为什么要引入复制集？有哪些成员？

*   **什么是复制集**？

保证数据在生产部署时的**冗余和可靠性**，通过在不同的机器上保存副本来保证数据的不会因为单点损坏而丢失。能够随时应对数据丢失、机器损坏带来的风险。换一句话来说，还能提高读取能力，用户的读取服务器和写入服务器在不同的地方，而且，由不同的服务器为不同的用户提供服务，提高整个系统的负载。

在 **MongoDB 中就是复制集（replica set)**： 一组复制集就是一组 mongod 实例掌管同一个数据集，实例可以在不同的机器上面。实例中包含一个主导，接受客户端所有的写入操作，其他都是副本实例，从主服务器上获得数据并保持同步。

![](https://pdai.tech/images/db/mongo/mongo-z-rep-1.png)

*   **基本的成员**?
    *   **主节点（Primary）** 包含了所有的写操作的日志。但是副本服务器集群包含有所有的主服务器数据，因此当主服务器挂掉了，就会在副本服务器上重新选取一个成为主服务器。MongoDB 还细化将从节点（Primary）进行了细化
        *   **Priority0** Priority0 节点的选举优先级为 0，不会被选举为 Primary
        *   **Hidden** 隐藏节点将不会收到来自应用程序的请求, 可使用 Hidden 节点做一些数据备份、离线计算的任务，不会影响复制集的服务
            *   **Delayed** Delayed 节点必须是 Hidden 节点，并且其数据落后与 Primary 一段时间（可配置，比如 1 个小时）；当错误或者无效的数据写入 Primary 时，可通过 Delayed 节点的数据来恢复到之前的时间点。
    *   **从节点（Seconary）** 正常情况下，复制集的 Seconary 会参与 Primary 选举（自身也可能会被选为 Primary），并从 Primary 同步最新写入的数据，以保证与 Primary 存储相同的数据；增加 Secondary 节点可以提供复制集的读服务能力，同时提升复制集的可用性。
    *   **仲裁节点（Arbiter）** Arbiter 节点只参与投票，不能被选为 Primary，并且不从 Primary 同步数据。比如你部署了一个 2 个节点的复制集，1 个 Primary，1 个 Secondary，任意节点宕机，复制集将不能提供服务了（无法选出 Primary），这时可以给复制集添加一个 Arbiter 节点，即使有节点宕机，仍能选出 Primary。

#### 8.4.9. [#](#mongodb复制集常见部署架构) MongoDB 复制集常见部署架构？

分别从三节点的单数据中心，和五节点的多数据中心来说：

三节点的单数据中心

*   **三节点：一主两从方式**
    *   一个主节点；
    *   两个从节点组成，主节点宕机时，这两个从节点都可以被选为主节点。

![](https://pdai.tech/images/db/mongo/mongo-z-rep-5.png)

当主节点宕机后, 两个从节点都会进行竞选，其中一个变为主节点，当原主节点恢复后，作为从节点加入当前的复制集群即可。

![](https://pdai.tech/images/db/mongo/mongo-z-rep-6.png)

*   **一主一从一仲裁方式**
    *   一个主节点
    *   一个从节点，可以在选举中成为主节点
    *   一个仲裁节点，在选举中，只进行投票，不能成为主节点

![](https://pdai.tech/images/db/mongo/mongo-z-rep-7.png)

当主节点宕机时，将会选择从节点成为主，主节点修复后，将其加入到现有的复制集群中即可。

![](https://pdai.tech/images/db/mongo/mongo-z-rep-8.png)

对于具有 5 个成员的复制集，成员的一些可能的分布包括（相关注意事项和三个节点一致，这里仅展示分布方案）：

*   **两个数据中心**：数据中心 1 的三个成员和数据中心 2 的两个成员。
    *   如果数据中心 1 发生故障，则复制集将变为只读。
    *   如果数据中心 2 发生故障，则复制集将保持可写状态，因为数据中心 1 中的成员可以创建多数。
*   **三个数据中心**：两个成员是数据中心 1，两个成员是数据中心 2，一个成员是站点数据中心 3。
    *   如果任何数据中心发生故障，复制集将保持可写状态，因为其余成员可以举行选举。

例如，以下 5 个成员复制集将其成员分布在三个数据中心中。

![](https://pdai.tech/images/db/mongo/mongo-z-rep-9.png)

#### 8.4.10. [#](#mongodb复制集是如何保证数据高可用的) MongoDB 复制集是如何保证数据高可用的？

通过两方面阐述：一个是选举机制，另一个是故障转移期间的回滚。

*   **如何选出 Primary 主节点的?**

假设复制集内**能够投票的成员**数量为 N，则大多数为 N/2 + 1，当复制集内存活成员数量不足大多数时，整个复制集将**无法选举出 Primary，复制集将无法提供写服务，处于只读状态**。

举例：3 投票节点需要 2 个节点的赞成票，容忍选举失败次数为 1；5 投票节点需要 3 个节点的赞成票，容忍选举失败次数为 2；通常投票节点为奇数，这样可以减少选举失败的概率。

在以下的情况将触发选举机制：

1.  往复制集中新加入节点
2.  初始化复制集时
3.  对复制集进行维护时，比如`rs.stepDown()`或者`rs.reconfig()`操作时
4.  从节点失联时，比如超时（默认是 10 秒）

*   **故障转移期间的回滚**

当成员在故障转移后重新加入其复制集时，回滚将还原以前的主在数据库上的写操作。 **本质上就是保证数据的一致性**。

仅当主服务器接受了在主服务器降级之前辅助服务器未成功复制的写操作时，才需要回滚。 当主数据库作为辅助数据库重新加入集合时，它会还原或 “回滚” 其写入操作，以保持数据库与其他成员的一致性。

#### 8.4.11. [#](#mongodb复制集如何同步数据) MongoDB 复制集如何同步数据？

复制集中的数据同步是为了维护共享数据集的最新副本，包括复制集的辅助成员同步或复制其他成员的数据。 MongoDB 使用两种形式的数据同步：

*   **初始同步 (Initial Sync)** 以使用完整的数据集填充新成员, 即**全量同步**
    *   新节点加入，无任何 oplog，此时需先进性 initial sync
    *   initial sync 开始时，会主动将_initialSyncFlag 字段设置为 true，正常结束后再设置为 false；如果节点重启时，发现_initialSyncFlag 为 true，说明上次全量同步中途失败了，此时应该重新进行 initial sync
    *   当用户发送 resync 命令时，initialSyncRequested 会设置为 true，此时会重新开始一次 initial sync
*   **复制 (Replication)** 以将正在进行的更改应用于整个数据集, 即**增量同步**
    *   initial sync 结束后，接下来 Secondary 就会『不断拉取主上新产生的 optlog 并重放』

#### 8.4.12. [#](#mongodb为什么要引入分片) MongoDB 为什么要引入分片？

高数据量和吞吐量的数据库应用会对单机的性能造成较大压力, 大的查询量会将单机的 CPU 耗尽, 大的数据量对单机的存储压力较大, 最终会耗尽系统的内存而将压力转移到磁盘 IO 上。

为了解决这些问题, 有两个基本的方法: 垂直扩展和水平扩展。

*   垂直扩展：增加更多的 CPU 和存储资源来扩展容量。
*   水平扩展：将数据集分布在多个服务器上。**MongoDB 的分片就是水平扩展的体现**。

**分片设计思想**

分片为应对高吞吐量与大数据量提供了方法。使用分片减少了每个分片需要处理的请求数，因此，通过水平扩展，集群可以提高自己的存储容量和吞吐量。举例来说，当插入一条数据时，应用只需要访问存储这条数据的分片.

**分片目的**

*   读 / 写能力提升
*   存储容量扩容
*   高可用性

#### 8.4.13. [#](#mongodb分片集群的结构) MongoDB 分片集群的结构？

一个 MongoDB 的分片集群包含如下组件：

*   `shard`: 即分片，真正的数据存储位置，以 chunk 为单位存数据；每个分片可以部署为一个复制集。
*   `mongos`: 查询的路由, 提供客户端和分片集群之间的接口。
*   `config servers`: 存储元数据和配置数据。

![](https://pdai.tech/images/db/mongo/mongo-z-shard-1.png)

这里要注意 mongos 提供的是客户端 application 与 MongoDB 分片集群的路由功能，这里分片集群包含了分片的 collection 和非分片的 collection。如下展示了通过路由访问分片的 collection 和非分片的 collection:

![](https://pdai.tech/images/db/mongo/mongo-z-shard-2.png)

#### 8.4.14. [#](#mongodb分片的内部是如何管理数据的呢) MongoDB 分片的内部是如何管理数据的呢？

在一个 shard server 内部，MongoDB 还是会**把数据分为 chunks**，每个 chunk 代表这个 shard server 内部一部分数据。chunk 的产生，会有以下两个用途：

*   `Splitting`：当一个 chunk 的大小超过配置中的 chunk size 时，MongoDB 的后台进程会把这个 chunk 切分成更小的 chunk，从而避免 chunk 过大的情况
*   `Balancing`：在 MongoDB 中，balancer 是一个后台进程，负责 chunk 的迁移，从而均衡各个 shard server 的负载，系统初始 1 个 chunk，chunk size 默认值 64M, 生产库上选择适合业务的 chunk size 是最好的。MongoDB 会自动拆分和迁移 chunks。

分片集群的数据分布（shard 节点）

*   使用 chunk 来存储数据
*   进群搭建完成之后，默认开启一个 chunk，大小是 64M，
*   存储需求超过 64M，chunk 会进行分裂，如果单位时间存储需求很大，设置更大的 chunk
*   chunk 会被自动均衡迁移。

**chunk 分裂及迁移**

随着数据的增长，其中的数据大小超过了配置的 chunk size，默认是 64M，则这个 chunk 就会分裂成两个。数据的增长会让 chunk 分裂得越来越多。

![](https://pdai.tech/images/db/mongo/mongo-z-shard-4.png)

这时候，各个 shard 上的 chunk 数量就会不平衡。这时候，mongos 中的一个组件 balancer 就会执行自动平衡。把 chunk 从 chunk 数量最多的 shard 节点挪动到数量最少的节点。

![](https://pdai.tech/images/db/mongo/mongo-z-shard-5.png)

#### 8.4.15. [#](#mongodb-中collection的数据是根据什么进行分片的呢) MongoDB 中 Collection 的数据是根据什么进行分片的呢？

MongoDB 中 Collection 的数据是根据什么进行分片的呢？这就是我们要介绍的**分片键（Shard key）**；那么又是采用过了什么算法进行分片的呢？这就是紧接着要介绍的**范围分片（range sharding）**和**哈希分片（Hash Sharding)**。

*   **哈希分片（Hash Sharding)**

分片过程中利用哈希索引作为分片，基于哈希片键最大的好处就是保证数据在各个节点分布基本均匀。

对于基于哈希的分片，MongoDB 计算一个字段的哈希值，并用这个哈希值来创建数据块。在使用基于哈希分片的系统中，拥有**相近分片键**的文档很可能不会存储在同一个数据块中，因此数据的分离性更好一些。

![](https://pdai.tech/images/db/mongo/mongo-z-shard-6.png)

*   **范围分片（range sharding）**

将单个 Collection 的数据分散存储在多个 shard 上，用户可以指定根据集合内文档的某个字段即 shard key 来进行范围分片（range sharding）。

对于基于范围的分片，MongoDB 按照片键的范围把数据分成不同部分:

![](https://pdai.tech/images/db/mongo/mongo-z-shard-7.png)

在使用片键做范围划分的系统中，拥有**相近分片键**的文档很可能存储在同一个数据块中，因此也会存储在同一个分片中。

*   **哈希和范围的结合**

如下是基于 X 索引字段进行范围分片，但是随着 X 的增长，大于 20 的数据全部进入了 Chunk C, 这导致了数据的不均衡。

![](https://pdai.tech/images/db/mongo/mongo-z-shard-111.png)

这时对 X 索引字段建哈希索引：

![](https://pdai.tech/images/db/mongo/mongo-z-shard-112.png)

#### 8.4.16. [#](#mongodb如何做备份恢复) MongoDB 如何做备份恢复？

*   JSON 格式：mongoexport/mongoimport

JSON 可读性强但体积较大，JSON 虽然具有较好的跨版本通用性，但其只保留了数据部分，不保留索引，账户等其他基础信息。

*   BSON 格式：mongoexport/mongoimport

BSON 则是二进制文件，体积小但对人类几乎没有可读性。

#### 8.4.17. [#](#mongodb如何设计文档模型) MongoDB 如何设计文档模型？

举几个例子：

*   **多态模式**

当集合中的所有文档都具有**相似但不相同的结构**时，我们将其称为多态模式； 比如运动员的运动记录：

![](https://pdai.tech/images/db/mongo/mongo-y-doc-2.png)

![](https://pdai.tech/images/db/mongo/mongo-y-doc-2.gif)

*   **属性模式**

出于性能原因考虑，为了优化搜索我们可能需要许多索引以照顾到所有子集。创建所有这些索引可能会降低性能。属性模式为这种情况提供了一个很好的解决方案。

假设现在有一个关于电影的集合。其中所有文档中可能都有类似的字段：标题、导演、制片人、演员等等。假如我们希望在上映日期这个字段进行搜索，这时面临的挑战是 “哪个上映日期”？在不同的国家，电影通常在不同的日期上映。

```
127.0.0.1:6379> XACK mq mqGroup 1553585533795-0 # 通知消息处理结束，用消息ID标识
(integer) 1

127.0.0.1:6379> XPENDING mq mqGroup # 再次查看Pending列表
1) (integer) 4 # 已读取但未处理的消息已经变为4个
2) "1553585533795-1"
3) "1553585533795-4"
4) 1) 1) "consumerA" # 消费者A，还有2个消息处理
      2) "2"
   2) 1) "consumerB"
      2) "1"
   3) 1) "consumerC"
      2) "1"
127.0.0.1:6379>
```

使用属性模式，我们可以将此信息移至数组中并减少对索引需求。我们将这些信息转换成一个包含键值对的数组：

```
# 删除队列中的消息
127.0.0.1:6379> XDEL mq 1553585533795-1
(integer) 1
# 查看队列中再无此消息
127.0.0.1:6379> XRANGE mq - +
1) 1) "1553585533795-0"
   2) 1) "msg"
      2) "1"
2) 1) "1553585533795-2"
   2) 1) "msg"
      2) "3"
```

*   **桶模式**

这种模式在处理物联网（IOT）、实时分析或通用时间序列数据时特别有效。通过将数据放在一起，我们可以更容易地将数据组织成特定的组，提高发现历史趋势或提供未来预测的能力，同时还能对存储进行优化。

随着数据在一段时间内持续流入（时间序列数据），我们可能倾向于将每个测量值存储在自己的文档中。然而，这种倾向是一种非常偏向于关系型数据处理的方式。如果我们有一个传感器每分钟测量温度并将其保存到数据库中，我们的数据流可能看起来像这样：

```
为了提高文件写入效率，在现代操作系统中，当用户调用write函数，将一些数据写入文件时，操作系统通常会将数据暂存到一个内存缓冲区里，当缓冲区的空间被填满或超过了指定时限后，才真正将缓冲区的数据写入到磁盘里。

这样的操作虽然提高了效率，但也为数据写入带来了安全问题：如果计算机停机，内存缓冲区中的数据会丢失。为此，系统提供了fsync、fdatasync同步函数，可以强制操作系统立刻将缓冲区中的数据写入到硬盘里，从而确保写入数据的安全性。
```

通过将桶模式应用于数据模型，我们可以在节省索引大小、简化潜在的查询以及在文档中使用预聚合数据的能力等方面获得一些收益。获取上面的数据流并对其应用桶模式，我们可以得到：

```
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> sets k2 22
(error) ERR unknown command `sets`, with args beginning with: `k2`, `22`, 
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379>
```

#### 8.4.18. [#](#mongodb如何进行性能优化) MongoDB 如何进行性能优化？

*   **慢查询**

为了定位查询，需要查看当前 mongo profile 的级别, profile 的级别有 0|1|2，分别代表意思: 0 代表关闭，1 代表记录慢命令，2 代表全部

显示为 0， 表示默认下是没有记录的。

设置 profile 级别，设置为记录慢查询模式, 所有超过 1000ms 的查询语句都会被记录下来

```
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k1 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> lpush k2 22
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> get k1
"11"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379>
```

### 8.5. [#](#_8-5-elasticsearch) 8.5 ElasticSearch

#### 8.5.1. [#](#elasticsearch是什么-基于lucene的-那么为什么不是直接使用lucene呢) ElasticSearch 是什么？基于 Lucene 的，那么为什么不是直接使用 Lucene 呢？

Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库。Elasticsearch 也是使用 Java 编写的，它的内部使用 Lucene 做索引与搜索，但是它的目的是使全文检索变得简单，**通过隐藏 Lucene 的复杂性，取而代之的提供一套简单一致的 RESTful API**。

然而，Elasticsearch 不仅仅是 Lucene，并且也不仅仅只是一个全文搜索引擎：

*   一个分布式的实时文档存储，每个字段 可以被索引与搜索
*   一个分布式实时分析搜索引擎
*   能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据

一个 ES 和数据库的对比

![](https://pdai.tech/images/db/es/es-introduce-1-3.png)

#### 8.5.2. [#](#elk-技术栈的常见应用场景) ELK 技术栈的常见应用场景？

*   日志系统

![](https://pdai.tech/images/db/es/es-introduce-2-5.png)

增加数据源，和使用 MQ

![](https://pdai.tech/images/db/es/es-introduce-2-6.png)

*   Metric 收集和 APM 性能监控

![](https://pdai.tech/images/db/es/es-introduce-2-7.png)

#### 8.5.3. [#](#es中索引模板是什么) ES 中索引模板是什么？

索引模板是一种告诉 Elasticsearch 在创建索引时如何配置索引的方法。

*   **使用方式**

在创建索引之前可以先配置模板，这样在创建索引（手动创建索引或通过对文档建立索引）时，模板设置将用作创建索引的基础。

*   **模板类型**

1.  **组件模板**是可重用的构建块，用于配置映射，设置和别名；它们不会直接应用于一组索引。
2.  **索引模板**可以包含组件模板的集合，也可以直接指定设置，映射和别名。

#### 8.5.4. [#](#es中索引的生命周期管理) ES 中索引的生命周期管理？

*   **为什么会引入**？

随着时间的增长索引的数量也会持续增长，然而这些场景基本上只有最近一段时间的数据有使用价值或者会被经常使用（热数据），而历史数据几乎没有作用或者很少会被使用（冷数据），这个时候就需要对索引进行一定策略的维护管理甚至是删除清理，否则随着数据量越来越多除了浪费磁盘与内存空间之外，还会严重影响 Elasticsearch 的性能。

*   **哪个版本引入的**？

在 Elastic Stack 6.6 版本后推出了新功能 Index Lifecycle Management(索引生命周期管理)，支持针对索引的全生命周期托管管理，并且在 Kibana 上也提供了一套 UI 界面来配置策略。

*   **索引生命周期常见的阶段**？
    *   hot: 索引还存在着大量的读写操作。
    *   warm: 索引不存在写操作，还有被查询的需要。
    *   cold: 数据不存在写操作，读操作也不多。
    *   delete: 索引不再需要，可以被安全删除。

#### 8.5.5. [#](#es查询和聚合的有哪些方式) ES 查询和聚合的有哪些方式？

*   **DSL**
    *   基于文本 - match, query string
    *   基于词项 - term
    *   复合查询 - 5 种
*   **EQL** Elastic Query Language
    *   bucket
    *   metric
    *   pipline
*   **SQL**

#### 8.5.6. [#](#es查询中query和filter的区别) ES 查询中 query 和 filter 的区别？

query 是查询 + score, 而 filter 仅包含查询， 比如在复合查询中 constant_score 查询无需计算 score，所以对应查询是 filter 而不是 query。

比如

```
replicaof 所选从库的IP 6379
```

#### 8.5.7. [#](#es查询中match和term的区别) ES 查询中 match 和 term 的区别？

term 是基于索引的词项，而 match 基于文本。

比如：

```
$ redis-cli -h 127.0.0.1 -p 6379 --intrinsic-latency 60
Max latency so far: 1 microseconds.
Max latency so far: 15 microseconds.
Max latency so far: 17 microseconds.
Max latency so far: 18 microseconds.
Max latency so far: 31 microseconds.
Max latency so far: 32 microseconds.
Max latency so far: 59 microseconds.
Max latency so far: 72 microseconds.
 
1428669267 total runs (avg latency: 0.0420 microseconds / 42.00 nanoseconds per run).
Worst run took 1429x longer than the average latency.
```

等同于

```
{
    title: "Star Wars",
    director: "George Lucas",
    ...
    release_US: ISODate("1977-05-20T01:00:00+01:00"),
    release_France: ISODate("1977-10-19T01:00:00+01:00"),
    release_Italy: ISODate("1977-10-20T01:00:00+01:00"),
    release_UK: ISODate("1977-12-27T01:00:00+01:00"),
    ...
}
```

也等同于

```
{
    title: "Star Wars",
    director: "George Lucas",
    …
    releases: [
        {
        location: "USA",
        date: ISODate("1977-05-20T01:00:00+01:00")
        },
        {
        location: "France",
        date: ISODate("1977-10-19T01:00:00+01:00")
        },
        {
        location: "Italy",
        date: ISODate("1977-10-20T01:00:00+01:00")
        },
        {
        location: "UK",
        date: ISODate("1977-12-27T01:00:00+01:00")
        },
        … 
    ],
    … 
}
```

#### 8.5.8. [#](#es查询中should和must的区别) ES 查询中 should 和 must 的区别？

should 是任意匹配，must 是同时匹配。

比如，接上面的例子是 should(任意匹配）， 而如下查询是 must(同时匹配)：

```
{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:00:00.000Z"),
   temperature: 40
}

{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:01:00.000Z"),
   temperature: 40
}

{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:02:00.000Z"),
   temperature: 41
}
```

等同于

```
{
    sensor_id: 12345,
    start_date: ISODate("2019-01-31T10:00:00.000Z"),
    end_date: ISODate("2019-01-31T10:59:59.000Z"),
    measurements: [
       {
       timestamp: ISODate("2019-01-31T10:00:00.000Z"),
       temperature: 40
       },
       {
       timestamp: ISODate("2019-01-31T10:01:00.000Z"),
       temperature: 40
       },
       … 
       {
       timestamp: ISODate("2019-01-31T10:42:00.000Z"),
       temperature: 42
       }
    ],
   transaction_count: 42,
   sum_temperature: 2413
}
```

#### 8.5.9. [#](#es查询中match-match-phrase和match-phrase-prefix有什么区别) ES 查询中 match，match_phrase 和 match_phrase_prefix 有什么区别？

match 本质上是对 term 组合，match_phrase 本质是连续的 term 的查询（and 关系），match_phrase_prefix 在 match_phrase 基础上提供了一种可以查最后一个词项是前缀的方法

比如：

某个字段内容是 “quick brown fox”, 如果我们需要查询包含 “quick brown f”，就需要使用 match_phrase_prefix，因为 f 不是完整的 term 分词，不能用 match_phrase。

#### 8.5.10. [#](#es查询中什么是复合查询-有哪些复合查询方式) ES 查询中什么是复合查询？有哪些复合查询方式？

在查询中会有多种条件组合的查询，在 ElasticSearch 中叫复合查询。它提供了 5 种复合查询方式：

*   **bool query(布尔查询)**
    *   通过布尔逻辑将较小的查询组合成较大的查询。
*   **boosting query(提高查询)**
    *   不同于 bool 查询，bool 查询中只要一个子查询条件不匹配那么搜索的数据就不会出现。而 boosting query 则是降低显示的权重 / 优先级（即 score)。
*   **constant_score（固定分数查询）**
    *   查询某个条件时，固定的返回指定的 score；显然当不需要计算 score 时，只需要 filter 条件即可，因为 filter context 忽略 score。
*   **dis_max(最佳匹配查询）**
    *   分离最大化查询（Disjunction Max Query）指的是： 将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回 。
*   **function_score(函数查询）**
    *   简而言之就是用自定义 function 的方式来计算_score。

#### 8.5.11. [#](#es聚合中的bucket聚合有哪些-如何理解) ES 聚合中的 Bucket 聚合有哪些？如何理解？

设计上大概分为三类（当然有些是第二和第三类的融合）

![](https://pdai.tech/images/db/es/es-agg-bucket-1.png)

#### 8.5.12. [#](#es聚合中的metric聚合有哪些-如何理解) ES 聚合中的 Metric 聚合有哪些？如何理解？

如何理解？

1.  **从分类看**：Metric 聚合分析分为**单值分析**和**多值分析**两类
2.  **从功能看**：根据具体的应用场景设计了一些分析 api, 比如地理位置，百分数等等

*   **单值分析**: 只输出一个分析结果
    *   标准 stat 型
        *   `avg` 平均值
        *   `max` 最大值
        *   `min` 最小值
        *   `sum` 和
        *   `value_count` 数量
    *   其它类型
        *   `cardinality` 基数（distinct 去重）
        *   `weighted_avg` 带权重的 avg
        *   `median_absolute_deviation` 中位值
*   **多值分析**: 单值之外的
    *   stats 型
        *   `stats` 包含 avg,max,min,sum 和 count
        *   `matrix_stats` 针对矩阵模型
        *   `extended_stats`
        *   `string_stats` 针对字符串
    *   百分数型
        *   `percentiles` 百分数范围
        *   `percentile_ranks` 百分数排行
    *   地理位置型
        *   `geo_bounds` Geo bounds
        *   `geo_centroid` Geo-centroid
        *   `geo_line` Geo-Line
    *   Top 型
        *   `top_hits` 分桶后的 top hits
        *   `top_metrics`

#### 8.5.13. [#](#es聚合中的管道聚合有哪些-如何理解) ES 聚合中的管道聚合有哪些？如何理解？

简单而言：让上一步的聚合结果成为下一个聚合的输入，这就是管道。

如何理解？

**第一个维度**：管道聚合有很多不同**类型**，每种类型都与其他聚合计算不同的信息，但是可以将这些类型分为两类：

*   **父级** 父级聚合的输出提供了一组管道聚合，它可以计算新的存储桶或新的聚合以添加到现有存储桶中。
*   **兄弟** 同级聚合的输出提供的管道聚合，并且能够计算与该同级聚合处于同一级别的新聚合。

**第二个维度**：根据**功能设计**的意图

比如前置聚合可能是 Bucket 聚合，后置的可能是基于 Metric 聚合，那么它就可以成为一类管道

进而引出了：`xxx bucket`

*   **Bucket 聚合 -> Metric 聚合**： bucket 聚合的结果，成为下一步 metric 聚合的输入
    *   Average bucket
    *   Min bucket
    *   Max bucket
    *   Sum bucket
    *   Stats bucket
    *   Extended stats bucket

...

#### 8.5.14. [#](#如何理解es的结构和底层实现) 如何理解 ES 的结构和底层实现？

*   **ES 的整体结构**?
    *   一个 ES Index 在集群模式下，有多个 Node （节点）组成。每个节点就是 ES 的 Instance (实例)。
    *   每个节点上会有多个 shard （分片）， P1 P2 是主分片, R1 R2 是副本分片
    *   每个分片上对应着就是一个 Lucene Index（底层索引文件）
    *   Lucene Index 是一个统称
        *   由多个 Segment （段文件，就是倒排索引）组成。每个段文件存储着就是 Doc 文档。
        *   commit point 记录了所有 segments 的信息

![](https://pdai.tech/images/db/es/es-th-2-3.png)

*   **底层和数据文件**？
    *   倒排索引（词典 + 倒排表）
    *   doc values - 列式存储
    *   正向文件 - 行式存储

![](https://pdai.tech/images/db/es/es-th-2-2.png)

文件的关系如下：

![](https://pdai.tech/images/db/es/es-th-3-2.jpeg)

#### 8.5.15. [#](#es内部读取文档是怎样的-如何实现的) ES 内部读取文档是怎样的？如何实现的？

*   **主分片或者副本分片检索文档的步骤顺序**：

![](https://pdai.tech/images/db/es/es-th-2-21.png)

1.  客户端向 Node 1 发送获取请求。
2.  节点使用文档的 _id 来确定文档属于分片 0 。分片 0 的副本分片存在于所有的三个节点上。 在这种情况下，它将请求转发到 Node 2 。
3.  Node 2 将文档返回给 Node 1 ，然后将文档返回给客户端。

在处理读取请求时，协调结点在每次请求的时候都会通过轮询所有的副本分片来达到负载均衡。

*   **读取文档的两阶段查询**？

所有的搜索系统一般都是两阶段查询，第一阶段查询到匹配的 DocID，第二阶段再查询 DocID 对应的完整文档，这种在 Elasticsearch 中称为 query_then_fetch。（这里主要介绍最常用的 2 阶段查询）。

![](https://pdai.tech/images/db/es/es-th-2-32.jpeg)

1.  在初始查询阶段时，查询会广播到索引中每一个分片拷贝（主分片或者副本分片）。 每个分片在本地执行搜索并构建一个匹配文档的大小为 from + size 的优先队列。PS：在 2. 搜索的时候是会查询 Filesystem Cache 的，但是有部分数据还在 Memory Buffer，所以搜索是近实时的。
2.  每个分片返回各自优先队列中 所有文档的 ID 和排序值 给协调节点，它合并这些值到自己的优先队列中来产生一个全局排序后的结果列表。
3.  接下来就是 取回阶段，协调节点辨别出哪些文档需要被取回并向相关的分片提交多个 GET 请求。每个分片加载并丰富文档，如果有需要的话，接着返回文档给协调节点。一旦所有的文档都被取回了，协调节点返回结果给客户端。

#### 8.5.16. [#](#es内部索引文档是怎样的-如何实现的) ES 内部索引文档是怎样的？如何实现的？

*   **新建单个文档所需要的步骤顺序**：

![](https://pdai.tech/images/db/es/es-th-2-4.png)

1.  客户端向 Node 1 发送新建、索引或者删除请求。
2.  节点使用文档的 _id 确定文档属于分片 0 。请求会被转发到 Node 3，因为分片 0 的主分片目前被分配在 Node 3 上。
3.  Node 3 在主分片上面执行请求。如果成功了，它将请求并行转发到 Node 1 和 Node 2 的副本分片上。一旦所有的副本分片都报告成功, Node 3 将向协调节点报告成功，协调节点向客户端报告成功。

*   **看下整体的索引流程**

![](https://pdai.tech/images/db/es/es-th-2-5.jpeg)

1.  协调节点默认使用文档 ID 参与计算（也支持通过 routing），以便为路由提供合适的分片。

```
db.getProfilingLevel()
```

2.  当分片所在的节点接收到来自协调节点的请求后，会将请求写入到 Memory Buffer，然后定时（默认是每隔 1 秒）写入到 Filesystem Cache，这个从 Momery Buffer 到 Filesystem Cache 的过程就叫做 refresh；
3.  当然在某些情况下，存在 Momery Buffer 和 Filesystem Cache 的数据可能会丢失，ES 是通过 translog 的机制来保证数据的可靠性的。其实现机制是接收到请求后，同时也会写入到 translog 中，当 Filesystem cache 中的数据写入到磁盘中时，才会清除掉，这个过程叫做 flush。
4.  在 flush 过程中，内存中的缓冲将被清除，内容被写入一个新段，段的 fsync 将创建一个新的提交点，并将内容刷新到磁盘，旧的 translog 将被删除并开始一个新的 translog。 flush 触发的时机是定时触发（默认 30 分钟）或者 translog 变得太大（默认为 512M）时。

#### 8.5.17. [#](#es底层数据持久化的过程) ES 底层数据持久化的过程？

**通过分步骤看数据持久化过程**：**write -> refresh -> flush -> merge**

*   **write 过程**

![](https://pdai.tech/images/db/es/es-th-2-6.png)

一个新文档过来，会存储在 in-memory buffer 内存缓存区中，顺便会记录 Translog（Elasticsearch 增加了一个 translog ，或者叫事务日志，在每一次对 Elasticsearch 进行操作时均进行了日志记录）。

这时候数据还没到 segment ，是搜不到这个新文档的。数据只有被 refresh 后，才可以被搜索到。

*   **refresh 过程**

![](https://pdai.tech/images/db/es/es-th-2-7.png)

refresh 默认 1 秒钟，执行一次上图流程。ES 是支持修改这个值的，通过 index.refresh_interval 设置 refresh （冲刷）间隔时间。refresh 流程大致如下：

1.  in-memory buffer 中的文档写入到新的 segment 中，但 segment 是存储在文件系统的缓存中。此时文档可以被搜索到
2.  最后清空 in-memory buffer。注意: Translog 没有被清空，为了将 segment 数据写到磁盘
3.  文档经过 refresh 后， segment 暂时写到文件系统缓存，这样避免了性能 IO 操作，又可以使文档搜索到。refresh 默认 1 秒执行一次，性能损耗太大。一般建议稍微延长这个 refresh 时间间隔，比如 5 s。因此，ES 其实就是准实时，达不到真正的实时。

*   **flush 过程**

每隔一段时间—​例如 translog 变得越来越大—​索引被刷新（flush）；一个新的 translog 被创建，并且一个全量提交被执行

![](https://pdai.tech/images/db/es/es-th-2-9.png)

上个过程中 segment 在文件系统缓存中，会有意外故障文档丢失。那么，为了保证文档不会丢失，需要将文档写入磁盘。那么文档从文件缓存写入磁盘的过程就是 flush。写入磁盘后，清空 translog。具体过程如下：

1.  所有在内存缓冲区的文档都被写入一个新的段。
2.  缓冲区被清空。
3.  一个 Commit Point 被写入硬盘。
4.  文件系统缓存通过 fsync 被刷新（flush）。
5.  老的 translog 被删除。

*   **merge 过程**

由于自动刷新流程每秒会创建一个新的段 ，这样会导致短时间内的段数量暴增。而段数目太多会带来较大的麻烦。 每一个段都会消耗文件句柄、内存和 cpu 运行周期。更重要的是，每个搜索请求都必须轮流检查每个段；所以段越多，搜索也就越慢。

Elasticsearch 通过在后台进行 Merge Segment 来解决这个问题。小的段被合并到大的段，然后这些大的段再被合并到更大的段。

当索引的时候，刷新（refresh）操作会创建新的段并将段打开以供搜索使用。合并进程选择一小部分大小相似的段，并且在后台将它们合并到更大的段中。这并不会中断索引和搜索。

![](https://pdai.tech/images/db/es/es-th-2-10.png)

一旦合并结束，老的段被删除：

1.  新的段被刷新（flush）到了磁盘。 ** 写入一个包含新段且排除旧的和较小的段的新提交点。
2.  新的段被打开用来搜索。
3.  老的段被删除。

![](https://pdai.tech/images/db/es/es-th-2-11.png)

合并大的段需要消耗大量的 I/O 和 CPU 资源，如果任其发展会影响搜索性能。Elasticsearch 在默认情况下会对合并流程进行资源限制，所以搜索仍然 有足够的资源很好地执行。

#### 8.5.18. [#](#es遇到什么性能问题-如何优化的) ES 遇到什么性能问题，如何优化的？

分几个方向说几个点：

*   **硬件配置优化** 包括三个因素：CPU、内存和 IO。
    
    *   **CPU**： 大多数 Elasticsearch 部署往往对 CPU 要求不高； CPUs 和更多的核数之间选择，选择更多的核数更好。多个内核提供的额外并发远胜过稍微快一点点的时钟频率。
    *   **内存**：
        *   **配置**： 由于 ES 构建基于 lucene，而 lucene 设计强大之处在于 lucene 能够很好的利用操作系统内存来缓存索引数据，以提供快速的查询性能。lucene 的索引文件 segements 是存储在单文件中的，并且不可变，对于 OS 来说，能够很友好地将索引文件保持在 cache 中，以便快速访问；因此，我们很有必要将一半的物理内存留给 lucene；另**一半的物理内存留给 ES**（JVM heap）。
        *   **禁止 swap** 禁止 swap，一旦允许内存与磁盘的交换，会引起致命的性能问题。可以通过在 elasticsearch.yml 中 bootstrap.memory_lock: true，以保持 JVM 锁定内存，保证 ES 的性能。
        *   **垃圾回收器**： 已知 JDK 8 附带的 HotSpot JVM 的早期版本存在一些问题，当启用 G1GC 收集器时，这些问题可能导致索引损坏。受影响的版本早于 JDK 8u40 随附的 HotSpot 版本。如果你使用的 JDK8 较高版本，或者 JDK9+，我推荐你使用 G1 GC； 因为我们目前的项目使用的就是 G1 GC，运行效果良好，对 Heap 大对象优化尤为明显。
    *   **磁盘** 在经济压力能承受的范围下，尽量使用固态硬盘（SSD）
*   **索引方面优化**
    
    *   **批量提交** 当有大量数据提交的时候，建议采用批量提交（Bulk 操作）；此外使用 bulk 请求时，每个请求不超过几十 M，因为太大会导致内存使用过大。
    *   **增加 Refresh 时间间隔** 为了提高索引性能，Elasticsearch 在写入数据的时候，采用延迟写入的策略，即数据先写到内存中，当超过默认 1 秒（index.refresh_interval）会进行一次写入操作，就是将内存中 segment 数据刷新到磁盘中，此时我们才能将数据搜索出来，所以这就是为什么 Elasticsearch 提供的是近实时搜索功能，而不是实时搜索功能。如果我们的系统对数据延迟要求不高的话，我们可以**通过延长 refresh 时间间隔，可以有效地减少 segment 合并压力，提高索引速度**。比如在做全链路跟踪的过程中，我们就将 index.refresh_interval 设置为 30s，减少 refresh 次数。再如，在进行全量索引时，可以将 refresh 次数临时关闭，即 index.refresh_interval 设置为 - 1，数据导入成功后再打开到正常模式，比如 30s。
    *   **索引缓冲的设置可以控制多少内存分配** indices.memory.index_buffer_size 接受一个百分比或者一个表示字节大小的值。默认是 10%
    *   **translog 相关的设置** 控制数据从内存到硬盘的操作频率，以减少硬盘 IO。可将 sync_interval 的时间设置大一些。默认为 5s。也可以控制 tranlog 数据块的大小，达到 threshold 大小时，才会 flush 到 lucene 索引文件。默认为 512m。
    *   **_id 字段的使用** _id 字段的使用，应尽可能避免自定义 _id，以避免针对 ID 的版本管理；建议使用 ES 的默认 ID 生成策略或使用数字类型 ID 做为主键。
    *   **_all 字段及 _source 字段的使用** _all 字段及 _source 字段的使用，应该注意场景和需要，_all 字段包含了所有的索引字段，方便做全文检索，如果无此需求，可以禁用；_source 存储了原始的 document 内容，如果没有获取原始文档数据的需求，可通过设置 includes、excludes 属性来定义放入 _source 的字段。
    *   **合理的配置使用 index 属性** 合理的配置使用 index 属性，analyzed 和 not_analyzed，根据业务需求来控制字段是否分词或不分词。只有 groupby 需求的字段，配置时就设置成 not_analyzed，以提高查询或聚类的效率。
*   **查询方面优化**
    
    *   **Filter VS Query**
    *   **深度翻页** 使用 Elasticsearch scroll 和 scroll-scan 高效滚动的方式来解决这样的问题。也可以结合实际业务特点，文档 id 大小如果和文档创建时间是一致有序的，可以以文档 id 作为分页的偏移量，并将其作为分页查询的一个条件。
    *   **避免层级过深的聚合查询**， 层级过深的 aggregation , 会导致内存、CPU 消耗，建议在服务层通过程序来组装业务，也可以通过 pipeline 的方式来优化。
    *   **通过开启慢查询配置定位慢查询**