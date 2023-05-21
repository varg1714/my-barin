一次ES使用过程中出现的Request cannot be executed; I/O reactor status: STOPPED与OOM异常排除

## 1. 背景

在服务器上部署了ES查询数据的接口，上线当晚一切正常，第二天早上的时候发现接口报错了。观察日志，发现报了Request cannot be executed; I/O reactor status: STOPPED与OOM异常

## 2. 解决过程

### 2.1. 解决ES请求错误

- 在本地启动服务，观察情况出现原因，当使用同样的数据请求时，本地正常返回，不是请求数据格式的问题
- 重启服务器服务后，服务恢复正常
- 利用jvm自带的分析工具jvisualvm(该程序在jdk8之前位于jdk的bin目录下，jdk9及其之后需要另外单独[下载](https://visualvm.github.io/download.html))，查看本地程序运行情况，发现在线程这一块，频繁的出现几十个线程被创建然后销毁的情况，进行分析发现，是每次服务调用都创建一个线程池，调用结束后销毁线程池造成的。

  ~~~java
  try (RestHighLevelClient client = esConfig.createClient()) {
      // doSomething
  	} catch (IOException e) {
      // doSomething
  }
  ~~~

  以前是这么获取连接的，这样会造成频繁的创建与销毁线程。逻辑修改成在启动时创建线程池，后续复用该线程池：

  ~~~java
      @Bean("restHighLevelClient")
      public RestHighLevelClient createClient(){
  
          final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
          credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials(username, pasword));
  
          RestClientBuilder builder = RestClient.builder(new HttpHost(host, port, scheme))
                  .setHttpClientConfigCallback(httpAsyncClientBuilder ->
                          httpAsyncClientBuilder.setDefaultCredentialsProvider(credentialsProvider));
  
          if(StringUtils.isBlank(esIndex)){
              throw new LFBizException("ES索引库配置拉取失败,请检查Apollo上ES索引库配置");
          }
  
          return new RestHighLevelClient(builder);
  
      }
  ~~~



### 2.2. 解决OOM问题

ES连接问题解决后，重启服务，利用Jemter进行并发请求测试。随着请求不断进行，发现虚拟机堆栈使用内存逐渐上升，虽然会有GC回收内存，但虚拟机可用内存却在逐渐减少，并一直在申请新内存，初步怀疑有对象没有被回收而一直占用内存。

- 在jvisualvm中观察发现，char[]类型的数据量在随着访问进行不断的增加，因此判断该类型数据是造成内存占用的原因之一。可是我们并不能知道究竟是什么在使用这些数据，因此我们需要借助一个新的工具MemoryAnalyzer，该工具由[Eclipse官方提供](https://www.eclipse.org/downloads/download.php?file=/mat/)。
- 启动jvisualvm，获取当前运行程序的快照进行分析(在File栏下点击Acquire Heap Dump可选择当前正在运行的java程序)。
	![image-20200918002343109](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20200918002343.png)

- 获取快照后工具会给出可能存在的内存泄露原因
  ![image-20200918002622785](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20200918002622.png)
- 在Overview菜单栏下选择Histogram查看堆内存对象的具体情况
  ![image-20200918002947060](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20200918002947.png)
  ![image-20200918003100862](https://varg-my-images.oss-cn-beijing.aliyuncs.com/img/20200918003100.png)
- 进行查看，发现这些对象都是一些日志对象DTO，这些DTO在接口调用时被创建，用于记录接口的调用信息。继续排查发现，这些DTO被存放在一个LinkedBlockingQueue队列中，该队列由线程池取出数据进行日志记录(排查中发现，该线程池是单线程池SingleThreadExecutor)，因此猜测是接口调用速度快于日志DTO记录速度，导致这些日志DTO来不及发送出去而一直呆在LinkedBlockingQueue队列中从而使堆内存不断膨胀。
- 接下来进行我们的猜测证实，我们将日志记录的操作注释，重新使用Jmeter测试调用我们的接口，此时堆内存分配回收恢复正常，处于一个稳定的值。

## 3. 总结

- 接口初始调用正常不代表一切都没问题，像这次接口中出现的OOM问题就是当请求速度快并且时间长问题才能体现出来的
- 线程池的创建一定要按照规范来
- 以后程序启动的时候可以使用jvisualvm进行观察程序的运行状况，会不会出现线程或者内存不稳定的情况，有这些情况要进行排查解决
- 要善于利用各种工具进行问题的分析解决，在本次问题解决过程中首次使用了这些工具进行JVM相关的问题排查，后续再出现此类情况时应当考虑查看这些运行数据，能帮我们更好的解决问题