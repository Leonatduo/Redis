# Redis常用命令

## Redis概述
Redis是典型的key-value数据库：
- **key**：一般是字符串
- **value**：支持多种不同数据类型

---

## 通用命令

| 命令                  | 描述                                                                 |
|-----------------------|----------------------------------------------------------------------|
| `KEYS pattern`        | 查找所有符合给定模式的key（生产环境慎用）                           |
| `EXISTS key`          | 检查key是否存在（存在返回1，不存在返回0）                           |
| `TYPE key`            | 返回key所存储值的类型                                               |
| `TTL key`             | 返回key的剩余生存时间（秒，-1表示未设置有效期）                     |
| `DEL key [key ...]`   | 删除指定的key（可批量删除）                                         |
| `EXPIRE key seconds`  | 设置key的有效期（到期自动删除）                                     |

**注意事项**：
- `KEYS`命令会阻塞Redis单线程，大数据量时性能差
- 删除多个key示例：`DEL name age`

---

## String类型
Redis中最简单的存储类型，value可以是：
- 普通字符串(string)
- 整数(int)
- 浮点数(float)

**特点**：
- 底层都是字节数组存储
- 最大容量512MB

### String常用命令

| 命令                     | 描述                                      |
|--------------------------|-------------------------------------------|
| `SET key value`          | 添加/修改String类型键值对                |
| `GET key`                | 获取key对应的value                       |
| `MSET key1 val1 key2 val2` | 批量设置多个键值对                      |
| `MGET key1 key2`         | 批量获取多个value                        |
| `INCR key`               | 整数值自增1                             |
| `INCRBY key increment`   | 整数值按步长自增（如`INCRBY num 2`）    |
| `INCRBYFLOAT key increment` | 浮点数按步长自增                     |
| `SETNX key value`        | key不存在时才设置（真正的添加）         |
| `SETEX key seconds value` | 设置键值对并指定有效期                |

---

## Key结构规范
Redis通过key前缀区分不同类型数据，建议格式：
**示例**：
- 用户数据：`reggie:user:1`
- 菜品数据：`reggie:dish:1`

**存储对象**：序列化为JSON字符串
```json
{
  "reggie:user:1": "{\"id\":1, \"name\":\"Jack\", \"age\":21}",
  "reggie:dish:1": "{\"id\":1, \"name\":\"鲟鱼火锅\", \"price\":4999}"
}
```

## Hash类型
value为字段-值对的集合（类似Java的HashMap）

**特点**：
- 支持单独操作对象的字段
- 适合存储对象属性

### Hash常用命令

| 命令                          | 描述                                      | 示例                                  |
|-------------------------------|-------------------------------------------|---------------------------------------|
| `HSET key field value`        | 添加/修改单个字段值                      | `HSET user:1 name John`               |
| `HGET key field`              | 获取单个字段值                           | `HGET user:1 name` → "John"           |
| `HMSET key field1 val1 field2 val2` | 批量设置多个字段值                   | `HMSET user:1 age 30 email a@test.com`|
| `HMGET key field1 field2`     | 批量获取多个字段值                      | `HMGET user:1 name age`               |
| `HGETALL key`                 | 获取所有字段和值                         | `HGETALL user:1`                      |
| `HKEYS key`                   | 获取所有字段名                           | `HKEYS user:1` → ["name", "age"]      |
| `HINCRBY key field increment` | 数值字段按步长自增                      | `HINCRBY user:1 age 1`                |
| `HSETNX key field value`      | 字段不存在时才设置                      | `HSETNX user:1 gender male`           |

---

## List类型
双向链表结构（类似Java的LinkedList）

**特点**：
- 有序存储
- 元素可重复
- 插入/删除速度快（O(1)）
- 随机访问速度一般（O(n)）

### List常用命令

| 命令                      | 描述                                      | 示例                                  |
|---------------------------|-------------------------------------------|---------------------------------------|
| `LPUSH key element ...`   | 左侧插入一个或多个元素                   | `LPUSH tasks "task1" "task2"`         |
| `LPOP key`                | 移除并返回左侧第一个元素                 | `LPOP tasks` → "task2"                |
| `RPUSH key element ...`   | 右侧插入一个或多个元素                   | `RPUSH tasks "task3"`                 |
| `RPOP key`                | 移除并返回右侧第一个元素                 | `RPOP tasks` → "task3"                |
| `LRANGE key start end`    | 返回指定索引范围内的元素（0 -1表示全部） | `LRANGE tasks 0 -1`                   |
| `BLPOP key timeout`       | 阻塞式左侧弹出（单位：秒）               | `BLPOP tasks 10`                      |
| `BRPOP key timeout`       | 阻塞式右侧弹出                           | `BRPOP tasks 5`                       |

---

## Set类型
无序集合（类似Java的HashSet）

**特点**：
- 元素不可重复
- 极快的查找速度（O(1)）
- 支持集合运算（交/并/差集）

### Set常用命令

| 命令                     | 描述                                      | 示例                                  |
|--------------------------|-------------------------------------------|---------------------------------------|
| `SADD key member ...`    | 添加一个或多个元素                       | `SADD tags "redis" "db"`              |
| `SREM key member ...`    | 移除指定元素                             | `SREM tags "db"`                      |
| `SCARD key`              | 返回集合元素个数                         | `SCARD tags` → 1                      |
| `SISMEMBER key member`   | 判断元素是否存在                         | `SISMEMBER tags "redis"` → 1          |
| `SMEMBERS key`           | 获取所有元素                             | `SMEMBERS tags` → ["redis"]           |
| `SINTER key1 key2`       | 返回多个集合的交集                       | `SINTER tags1 tags2`                  |
| `SUNION key1 key2`       | 返回多个集合的并集                       | `SUNION tags1 tags2`                  |
| `SDIFF key1 key2`        | 返回key1有而key2没有的元素               | `SDIFF tags1 tags2`                   |
