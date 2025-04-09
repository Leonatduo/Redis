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
