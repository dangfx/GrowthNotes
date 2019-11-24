## Java Lock

### 1.进程内锁

>锁的原理：所有的线程必须使用同一把锁(即：标识锁的内容不能变)。
>
>```
>/**
>* 1.public synchronized int incr(){} 方法加锁 能锁住
>* 2.synchronized (this) this锁 能锁住
>*      this 代表 RedisIncrService实例,spring默认加载方式为单例
>* 3.synchronized (redisTemplate) @Autowired spring组件默认是单例的
>* 4.synchronized (obj) 对象锁 能锁住
>* 5.synchronized (RedisIncrService.class) class锁 能锁住
>*      classloader 只会加载一次class文件
>* 6.synchronized (obj()) 方法 能锁住
>* 7.ReentrantLock 同一个对象能锁住
>*      reentrantLock.lock();
>*      reentrantLock.unlock();
>*/
>```
>JVM 对 synchronized优化：
>
>1. 偏向锁：单线程访问时，不用加锁
>2. 轻量级锁：多线程访问时，获取不到锁的线程通过自旋尝试获取锁
>3. 重量级锁：如果自旋时间很长，JVM将自旋改为等待，让出系统资源执行其他操作

```java
@Service
public class RedisIncrService {
    private Object obj = new Object();
    private ReentrantLock reentrantLock = new ReentrantLock();
    @Autowired
    StringRedisTemplate redisTemplate;
    
    public int incr()/*public synchronized int incr()*/{
        /*synchronized (this)*/
        /*synchronized (obj)*/
        /*synchronized (redisTemplate)*/
        /*synchronized (RedisIncrService.class)*/
        synchronized (obj()){
            // reentrantLock.lock();
            int result = 0;
            ValueOperations<String, String> values = redisTemplate.opsForValue();
            String num = values.get("num");
            if(num != null){
                int parseInt = Integer.parseInt(num);
                result = parseInt ++;
                values.set("num", String.valueOf(parseInt));
            }
            // reentrantLock.unlock();
            return result;
        }
    }

    private Object obj(){
        return obj;
    }
```

### 2.进程间锁

>RPC：为进程间通信的而生，解决进程间锁问题，使用分布式锁。
>
>分布式锁的核心：
>
> 	1.  加锁要是原子的
> 	2. 锁要自动超时
> 	3. 解锁要是原子的

```java
// 阶段一 （伪代码）
// 问题：判断与设置值不是原子操作
public void lock(){
    String lock = getFromRedis("lock");
    if(lock == null){
        setRedisKey("lock", "123");
        // 业务代码
        deleteKey("lock");
        return ;
    }else{
		lock();// 自旋
    }
}

// 阶段二 （伪代码）
// 解决：setnx()同时只能由一个client设置成功
// 问题：设置锁成功，由于各种原因解锁失败，例：重启，断电
public void lock(){
    Integer lock = setnx("lock", "123");
    if(lock != null){
        // 业务代码
        deleteKey("lock");
        return ;
    }else{
		lock();// 自旋
    }
}

// 阶段三 （伪代码）
// 解决：deleteKey()未执行
// 问题：加锁与加超时不是原子操作
public void lock(){
    Integer lock = setnx("lock", "123");
    if(lock != null){
        expire("lock", 10L, TimeUnit.SECONDS)
        // 业务代码
        deleteKey("lock");
        return ;
    }else{
		lock();// 自旋
    }
}

// 阶段四 （伪代码）
// 解决：超时命令未执行
// 问题：业务逻辑执行时间超过redis过期时间，锁失效
// set在Redis版本2.6.12 提供了EX 、PX 、NX 、XX参数用于取代SETEX、SETNX和PSETEX
public void lock(){
    Integer lock = set("lock", "123", 10);
    if(lock != null){
        // 业务代码
        deleteKey("lock");
        return ;
    }else{
		lock();// 自旋
    }
}

// 阶段五 （伪代码）
// 解决：一个线程由于超时删除另一个线程的锁
// 问题：判断锁与删除锁不是原子操作，执行get("lock")时，锁失效
public void lock(){
    String token = UUID;
    Integer lock = set("lock", token, 10);
    if(lock != null){
        // 业务代码
        if(get("lock") == token)
        	deleteKey("lock");
        return ;
    }else{
		lock();// 自旋
    }
}

// 阶段六 （伪代码）
// 解决：使用LUA脚本将多条命令打包成一条执行
public void lock(){
    String token = UUID;
    Integer lock = set("lock", token, 10);
    if(lock != null){
        // 业务代码
        // 使用 LUA 脚本执行2条命令
        return ;
    }else{
		lock();// 自旋
    }
}

// LUA 脚本执行
String script = "
if redis.call('get', KEYS[1]) == ARGV[1] then 
    return redis.call('del', KEYS[1]) 
else 
    return 0 end
";
jedis.eval(script, Collections.singletonList(key), CollectionssingletonList(token));
```

>锁的更多考虑：
>
>1. 自旋次数
>2. 自旋超时
>3. 锁的粒度：分析好锁的粒度，不要锁无关数据，一种数据一种锁，一条数据一个锁。
>
>Redis分布式锁框架：[Redisson](https://github.com/redisson/redisson)

使用Redisson框架

>Maven

```xml
<dependency>
   <groupId>org.redisson</groupId>
   <artifactId>redisson</artifactId>
   <version>3.11.5</version>
</dependency>  
```

>Java connect

```java
Config config = new Config();
// 默认连接地址 127.0.0.1:6379
config.useSingleServer().setAddress("redis://127.0.0.1:6379");
RedissonClient redisson = Redisson.create(config);
```

>Java user Reentrant Lock 
>
>可重入锁：在同一个线程中，可以多次获取锁，不会造成死锁。

```
RLock lock = redisson.getLock("anyLock");

// lock.lock(10, TimeUnit.SECONDS);
lock.lock();
// 业务代码
lock.unlock();

// 尝试加锁，最多等待100秒，上锁以后10秒自动解锁
boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);
if (res) {
   try {
     ...
   } finally {
       lock.unlock();
   }
}

// 可重入锁测试，都可以获取到，不会死锁
lock.lock();
System.out.println("第一次获取锁");
lock.lock();
System.out.println("第二次获取锁");
lock.lock();
System.out.println("第三次获取锁");
```

>Java use RReadWriteLock 读写锁
>
>先读后写正常，先写后读读等待

```java
RReadWriteLock rwlock = redisson.getReadWriteLock("anyRWLock");
// 最常见的使用方法
rwlock.readLock().lock();
// 或
rwlock.writeLock().lock();
```

>Java use Semaphore 信号量
>
>统一个资源，获取与释放场景(车位)

```java
RSemaphore semaphore = redisson.getSemaphore("semaphore");
// 设置许可
semaphore.tryAcquire(23, TimeUnit.SECONDS); 
// 获得一个许可
semaphore.acquire();
// 释放一个许可
// semaphore.release(10);
semaphore.release();
```

