## 前言 

Dubbo的SPI机制是什么呢？首先，SPI全称为（Service Provider Interface），主要是被框架开发人员使用的一种技术。Dubbo的SPI机制提供的是一种将服务Provider接口定义在META-INF中，当Dubbo服务启动时，
通过SPI加载机制加载文件中的接口，从而使用那些被加载的接口。那么这么做的目的是什么呢？这么做的目的就是为了更好地达到 OCP 原则（即“对扩展开放，对修改封闭”的原则）。这种"对扩展开放，对修改封闭"的原则能
在维持内核功能稳定的基础上，更好的对系统功能进行扩展。换句话说，基于Dubbo SPI加载机制，让整个框架的接口和具体实现完全解耦，从而奠定了整个框架良好可扩展性的基础。

Dubbo SPI是参考了JDK原生的SPI机制，进行了性能优化以及功能增强。

## 正文

### 1. Java SPI

Javs SPI使用的是策略模式，一个接口多种实现。我们只负责声明接口，具体的实现并不在程序中直接确定，而是由程序之外的配置掌控，用于具体实现的装配。

Java SPI的定义及使用步骤如下：
1. 定义一个接口以及对应的方法
2. 编写该接口的一个实现类
3. 在META-INF/services/目录下，创建一个以接口全路径命名的文件，如com.test.spi.PrintService
4. 文件内容为具体实现类的全路径名，如果有多个，则用分行符分隔
5. 在代码中通过java.util.ServiceLoader来加载具体的实现类

在com.test.spi包目录下，定义了一个PrintService接口和一个PrintServiceImpl实现类，然后在resources目录下定义了一个META-INF/services/com.test.spi.PrintService，注意这里定义的是一个
全路径名称的文件。

```
public interface Printservice (
    void printlnfo();
}
```

```
public class PrintServicelmpl implements Printservice { 
    @Override
    public void printlnfo() {
        System.out.println("hello world");
    } 
}
```

```
public static void main(String[] args) ( 
    ServiceLoader<PrintService> serviceServiceLoader =
    ServiceLoader.load(PrintService.class);
    for (Printservice printservice : serviceServiceLoader) ( 
        //此处会输出：hello world 获取所有的SPI实现，循环调用
        printService.printInfo(); printlnfo()方法，会打印出 hello world
    } 
}
```

在JDK SPI中，是通过ServiceLoader来获取所有接口实现的。

### 2. Dubbo SPI

Dubbo SPI没有直接使用Java SPL而是在它的思想上又做了一定的改进，形成了一套自己的配置规范和特性。同时，Dubbo SPI又兼容Java SPL服务在启动的时候，Dubbo就会查找这些扩展点的所有实现

Dubbo SPI之于JDK SPI，做到了三点优化：
1. 不同于JDK SPI会一次性实例化扩展点所有实现，因为JDK SPI有扩展实现，则初始化会很耗时，并且如果没有用上也要加载，则会很浪费资源。而Dubbo SPI只会加载扩展点，而不会对其进行初始化，并且Dubbo SPI中
会根据不同的实现类来缓存到内存中，性能上得到了很大的优化。
2. JDK SPI如果对扩展加载失败，则连扩展的名称都获取不到，并且失败原因会被吃掉，而Dubbo SPI则会将异常堆栈保留下来，方便后序
对其异常信息进行分析。
3. Dubbo SPI增加了对IOC和AOP的支持，在Dubbo中，一个扩展点可以通过setter来注入到其他扩展点中。

未完待续....