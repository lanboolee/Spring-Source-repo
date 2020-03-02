## spring

## 1. 请说明一下springIOC原理是什么？如果你要实现IOC需要怎么做？请简单描述一下实现步骤？

    a) ioc(控制反转)就是DI(依赖注入), 用来解决对象之间的耦合问题, ioc容器存放着对象.
    b) 实现IOC的步骤
       
       定义用来描述bean的配置的Java类
       
       解析bean的配置，將bean的配置信息转换为上面的BeanDefinition对象保存在内存中，spring中采用HashMap进行对象存储，其中会用到一些xml解析技术
       
       遍历存放BeanDefinition的HashMap对象，逐条取出BeanDefinition对象，获取bean的配置信息，利用Java的反射机制实例化对象，將实例化后的对象保存在另外一个Map中即可。
##   2.  spring是如何解决循环依赖问题的?
    protected Object getSingleton(String beanName, boolean allowEarlyReference) {
        /**
         * 第一步:我们尝试去一级缓存(单例缓存池中去获取对象,一般情况从该map中获取的对象是直接可以使用的)
         * IOC容器初始化加载单实例bean的时候第一次进来的时候 该map中一般返回空
         */
        Object singletonObject = this.singletonObjects.get(beanName);
        /**
         * 若在第一级缓存中没有获取到对象,并且singletonsCurrentlyInCreation这个list包含该beanName
         * IOC容器初始化加载单实例bean的时候第一次进来的时候 该list中一般返回空,但是循环依赖的时候可以满足该条件
         */
        if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
            synchronized (this.singletonObjects) {
                /**
                 * 尝试去二级缓存中获取对象(二级缓存中的对象是一个早期对象)
                 * 何为早期对象:就是bean刚刚调用了构造方法，还来不及给bean的属性进行赋值的对象
                 * 就是早期对象
                 */
                singletonObject = this.earlySingletonObjects.get(beanName);
                /**
                 * 二级缓存中也没有获取到对象,allowEarlyReference为true(参数是有上一个方法传递进来的true)
                 */
                if (singletonObject == null && allowEarlyReference) {
                    /**
                     * 直接从三级缓存中获取 ObjectFactory对象 这个对接就是用来解决循环依赖的关键所在
                     * 在ioc后期的过程中,当bean调用了构造方法的时候,把早期对象包裹成一个ObjectFactory
                     * 暴露到三级缓存中
                     */
                    ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                    //从三级缓存中获取到对象不为空
                    if (singletonFactory != null) {
                        /**
                         * 在这里通过暴露的ObjectFactory 包装对象中,通过调用他的getObject()来获取我们的早期对象
                         * 在这个环节中会调用到 getEarlyBeanReference()来进行后置处理
                         */
                        singletonObject = singletonFactory.getObject();
                        //把早期对象放置在二级缓存,
                        this.earlySingletonObjects.put(beanName, singletonObject);
                        //ObjectFactory 包装对象从三级缓存中删除掉
                        this.singletonFactories.remove(beanName);
                    }
                }
            }
        }
        return singletonObject;
    }

## 3. AOP
    a) 为bean创建一个proxy，JDKproxy或者CGLIBproxy，然后在调用bean的方法时，会通过proxy来调用bean方法
       
           重点过程可分为：
       
           1）通过AspectJAutoProxyBeanDefinitionParser类将AnnotationAwareAspectJAutoProxyCreator注册到Spring容器中
       
           2）AnnotationAwareAspectJAutoProxyCreator类的postProcessAfterInitialization()方法将所有有advice的bean重新包装成proxy
       
           3）调用bean方法时通过proxy来调用，proxy依次调用增强器的相关方法，来实现方法切入
## 4. spring事务
    a) 七种传播行为 REQUIRED，SUPPORTS，MANDATORY，REQUIRES_NEW，NOT_SUPPORTED，NEVER，NESTED 默认第一个
    
    b) 四种隔离级别 DEFAULT，READ_UNCOMMITTED，READ_COMMITTED，REPEATABLE_READ，SERIALIZABLE, default是根据数据库默认的, mysql默认RR
    
    c) 事务源码解析@Transactional 是通过aop实现的, 在AutoProxyRegistrar类里执行registerBeanDefinitions 去注册benaDefinitions  
     在ProxyTransactionManagementConfiguration导入切面信息, setAdvice, advice是顶层接口, 主要aop用到, advice就是增强器接口
## 5. bean的生命周期
    a) ①通过构造器或工厂方法创建bean实例
       
       ②为bean的属性设置值和对其他bean的引用
       
       ③将bean实例传递给bean后置处理器的postProcessBeforeInitialization()方法
       
       ④调用bean的初始化方法
       
       ⑤将bean实例传递给bean后置处理器的postProcessAfterInitialization()方法
       
       ⑥bean可以使用了
       
       ⑦当容器关闭时调用bean的销毁方法
# JUC

## 1. 线程池
### 1. 什么情况下适合线程池去处理?
    单个任务处理时间短
    需要处理的任务量很大
### 2. 优劣
    重用存在的线程, 减少线程创建和消亡的开销
    提高相应速度, 任务不需要等待现成的创建
    提高线程的管理性, 无限制的创建线程会降低系统稳定性, 消耗系统资源, 使用线程池可以进行统一分配, 调优和监控
### 3. 线程的实现方式
    Runable, Thread, Callable
    Exceutor框架
### 4. 线程池(不是綫程)的五个状态
    private final AtomicInteger ctl = new AtomicInteger(ctlOf(Running, 0))
    private static final int COUNT_BITS = Integer.SIZE(也就是32位) - 3;
    private static final int CAPACITY = (1 << COUNT_BITS) - 1; 用来记录线程数量, 最多5亿个左右
        ctl是对线程池的运行状态中有效线程的数量进行控制的一个字段, 高三位记录   线程的运行状态
    
    RUNNING      =   -1 << COUNT_BITS // 高三位为111
        线程初始状态, 可以接收任务
    SHUTDOWN     =   0  << COUNT_BITS // 高三位为000
    // 停止任务, 但是不会停止正在进行的任务
    STOP         =   1 << COUNT_BITS  // 高三位为001
    // 停止所有任务
    TIDYING      =   2 << COUNT_BITS //  高三位为010
    // 调用线程的钩子函数
    TERMINATED   =   3 << COUNT_BITS // 高三位为011
### 5. 创建方式
         public ThreadPoolExecutor(int corePoolSize,  // 核心线程数
                                   int maximumPoolSize, // 最大线程数
                                   long keepAliveTime,  
                                   TimeUnit unit,
                                   BlockingQueue<Runnable> workQueue) {
             this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
                  Executors.defaultThreadFactory(), defaultHandler);
         }
### 工作原理
         boolean workerStarted = false;
                boolean workerAdded = false;
                Worker w = null;
                try {
                // worker  继承AQS 控制并发
                    w = new Worker(firstTask);
                    final Thread t = w.thread;
                    if (t != null) {
                        final ReentrantLock mainLock = this.mainLock;
                        mainLock.lock();
                        try {
                            // Recheck while holding lock.
                            // Back out on ThreadFactory failure or if
                            // shut down before lock acquired.
                            int rs = runStateOf(ctl.get());
        
                            if (rs < SHUTDOWN ||
                                (rs == SHUTDOWN && firstTask == null)) {
                                if (t.isAlive()) // precheck that t is startable
                                    throw new IllegalThreadStateException();
                                workers.add(w);
                                int s = workers.size();
                                if (s > largestPoolSize)
                                    largestPoolSize = s;
                                workerAdded = true;
                            }
                        } finally {
                            mainLock.unlock();
                        }
                        if (workerAdded) {
                            t.start();
                            workerStarted = true;
                        }
                    }
                } finally {
                    if (! workerStarted)
                        addWorkerFailed(w);
                }
                return workerStarted;
        
        
        
        
        
        
        
        
        
        
        