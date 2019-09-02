# Redis分布式锁

通常的实现方式：

在一个实例中创建一个带过期时间的key。

单点故障：如果redis宕机了

增加slave，master不可用时使用它。

因为Redis的负责是异步的，因此不具备互斥性。master有可能在将key传到slave之前crash。



## 单个实例的正确实现

set key value nx px 30000

value必须在所有的客户端和所有的锁请求中唯一。这是为了告诉客户端这个锁正是我所想释放的锁，避免释放了别人的锁。

```
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

jedis实现分布式锁：

获取锁

```
/**
 *  分布式锁
 * @param key
 * @param value
 * @return
 */
public static boolean tryLock(String key, String value) {
   Jedis client = null;
   try {
      client = RedisWritePool.getClient();
      if (client.setnx(key, value) == 1) {
         return true;
      }
   } catch (Exception ex) {
      logger.error("key:" + key + " value:" + value + "." + ex.toString(),ex);
   } finally {
      RedisWritePool.quitClient(client);
   }
   return false;
}
```



释放redis锁：

```
    public static void unLock(String key, String values){
          jedis = RedisWritePool.getClient();
        String luaStr = "if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end";
        Object result = jedis.eval(luaStr, Lists.newArrayList(key), Lists.newArrayList(value));
        System.out.println(result);
    }
```



上述算法假设这个实例总是可用的。

## Redlock算法

1. 获取当前时间
2. 向N个实例获取锁，在设置各个锁时，会有一些小的超时时间，避免client在一个down掉的实例上阻塞太多时间。
3. 计算获取锁消耗的时间，如果client获取了从(N/2+1)个实例获取了锁，且消耗时间小于初始锁有效时间，认为获取了锁。
4. 锁有效时间=初始锁有效时间-消耗时间
5. 如果获取锁失败，会尝试解锁所有实例



### 重试

重试时应该有一定的延时，可能会造成所有人都无法获取锁。

理想情况下client应该使用多路复用向多个实例发送set命令。



AOF不会丢失信息， fsync可能会丢失键



因此需要保证重启后的instance在TTL不可用，以保证互斥性。

使用延迟启动可以基本保证安全性，无论是何种持久化方式。但是可能在可用性上大打折扣，比如多个实例crash，那么TTL内是不可用的。



可以使用短的有效时间，并通过一种lock extension机制。

