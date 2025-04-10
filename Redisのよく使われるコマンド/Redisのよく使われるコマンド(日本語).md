# Redis常用コマンド

## Redis概要
Redisは典型的なkey-valueデータベースです：
- **key**：通常は文字列
- **value**：複数の異なるデータ型をサポート
---

## 汎用コマンド

| コマンド                  | 説明                                                                 |
|-----------------------|----------------------------------------------------------------------|
| `KEYS pattern`        | 指定したパターンに一致する全てのkeyを検索（本番環境では注意）       |
| `EXISTS key`          | keyの存在確認（存在=1, 不存在=0）                                   |
| `TYPE key`            | keyに格納されている値の型を返す                                     |
| `TTL key`             | keyの残存有効時間（秒、-1は有効期限未設定）                         |
| `DEL key [key ...]`   | 指定keyの削除（複数削除可能）                                       |
| `EXPIRE key seconds`  | keyの有効期限設定（期限切れで自動削除）                             |

**注意事項**：
- `KEYS`コマンドはRedisのシングルスレッドをブロックするため、大量データ時は性能低下
- 複数key削除例：`DEL name age`

---

## String型
Redisで最もシンプルな保存形式、valueは以下可能：
- 通常文字列(string)
- 整数(int)
- 浮動小数点数(float)

**特徴**：
- 内部はバイト配列で保存
- 最大容量512MB

### String型常用コマンド

| コマンド                     | 説明                                      |
|--------------------------|-------------------------------------------|
| `SET key value`          | String型key-valueの追加/修正             |
| `GET key`                | keyに対応するvalueを取得                 |
| `MSET key1 val1 key2 val2` | 複数key-valueを一括設定                |
| `MGET key1 key2`         | 複数valueを一括取得                      |
| `INCR key`               | 整数値を1増加                           |
| `INCRBY key increment`   | 整数値を指定増分で増加（例：`INCRBY num 2`） |
| `INCRBYFLOAT key increment` | 浮動小数点数を指定増分で増加         |
| `SETNX key value`        | keyが存在しない場合のみ設定（真の追加）  |
| `SETEX key seconds value` | key-valueを設定し有効期限を指定       |

---

## Key構造の規範
Redisではkeyプレフィックスでデータ種別を区別、推奨形式：
**例**：
- ユーザーデータ：`reggie:user:1`
- 料理データ：`reggie:dish:1`

**オブジェクト保存**：JSON文字列にシリアライズ
```json
{
  "reggie:user:1": "{\"id\":1, \"name\":\"Jack\", \"age\":21}",
  "reggie:dish:1": "{\"id\":1, \"name\":\"チョウザメ鍋\", \"price\":4999}"
}
```
## Hash型
valueがフィールド-値ペアの集合（JavaのHashMapに類似）

**特徴**：
- オブジェクトのフィールド単位で操作可能
- オブジェクト属性の保存に適す

### Hash型常用コマンド

| コマンド                          | 説明                                      | 例                                  |
|-------------------------------|-------------------------------------------|---------------------------------------|
| `HSET key field value`        | 単一フィールド値の追加/修正              | `HSET user:1 name John`               |
| `HGET key field`              | 単一フィールド値を取得                   | `HGET user:1 name` → "John"           |
| `HMSET key field1 val1 field2 val2` | 複数フィールド値を一括設定           | `HMSET user:1 age 30 email a@test.com`|
| `HMGET key field1 field2`     | 複数フィールド値を一括取得              | `HMGET user:1 name age`               |
| `HGETALL key`                 | 全フィールドと値を取得                   | `HGETALL user:1`                      |
| `HKEYS key`                   | 全フィールド名を取得                     | `HKEYS user:1` → ["name", "age"]      |
| `HINCRBY key field increment` | 数値フィールドを指定増分で増加          | `HINCRBY user:1 age 1`                |
| `HSETNX key field value`      | フィールドが存在しない場合のみ設定      | `HSETNX user:1 gender male`           |

---

## List型
双方向リンクリスト構造（JavaのLinkedListに類似）

**特徴**：
- 順序保持
- 要素重複可能
- 挿入/削除が高速（O(1)）
- ランダムアクセスは一般速度（O(n)）

### List型常用コマンド

| コマンド                      | 説明                                      | 例                                  |
|---------------------------|-------------------------------------------|---------------------------------------|
| `LPUSH key element ...`   | 左側に1つ以上の要素を挿入                | `LPUSH tasks "task1" "task2"`         |
| `LPOP key`                | 左側最初の要素を削除して返す             | `LPOP tasks` → "task2"                |
| `RPUSH key element ...`   | 右側に1つ以上の要素を挿入                | `RPUSH tasks "task3"`                 |
| `RPOP key`                | 右側最初の要素を削除して返す             | `RPOP tasks` → "task3"                |
| `LRANGE key start end`    | 指定インデックス範囲の要素を返す（0 -1は全件） | `LRANGE tasks 0 -1`              |
| `BLPOP key timeout`       | ブロッキング左側ポップ（単位：秒）       | `BLPOP tasks 10`                      |
| `BRPOP key timeout`       | ブロッキング右側ポップ                   | `BRPOP tasks 5`                       |

---

## Set型
順序なし集合（JavaのHashSetに類似）

**特徴**：
- 要素重複不可
- 検索が極めて高速（O(1)）
- 集合演算をサポート（積/和/差集合）

### Set型常用コマンド

| コマンド                     | 説明                                      | 例                                  |
|--------------------------|-------------------------------------------|---------------------------------------|
| `SADD key member ...`    | 1つ以上の要素を追加                      | `SADD tags "redis" "db"`              |
| `SREM key member ...`    | 指定要素を削除                           | `SREM tags "db"`                      |
| `SCARD key`              | 集合の要素数を返す                       | `SCARD tags` → 1                      |
| `SISMEMBER key member`   | 要素の存在確認                           | `SISMEMBER tags "redis"` → 1          |
| `SMEMBERS key`           | 全要素を取得                             | `SMEMBERS tags` → ["redis"]           |
| `SINTER key1 key2`       | 複数集合の積集合を返す                   | `SINTER tags1 tags2`                  |
| `SUNION key1 key2`       | 複数集合の和集合を返す                   | `SUNION tags1 tags2`                  |
| `SDIFF key1 key2`        | key1にありkey2にない要素を返す           | `SDIFF tags1 tags2`                   |
