### 问题发现

今天在跑migrate,代码如下：
```ruby
class CreateReconciliationOrderItems < ActiveRecord::Migration[5.1]
  def change
    create_table :reconciliation_order_items do |t|
      t.string  :kind
      t.string  :month
      t.text    :content
      t.string  :status
      t.string  :type
      t.references :reconciliation
      t.references :reconciliation_result_set
      t.string  :internal_source_type
      t.string  :internal_source_id
      t.timestamps null: false
    end
  end
end
```

数据库报了一个错误：
```shell
   (0.3ms)  SELECT pg_try_advisory_lock(5675054488983918780)
   (1.4ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations"."version" ASC
Migrating to CreateReconciliationOrderItems (20180116061824)
   (0.2ms)  BEGIN
== 20180116061824 CreateReconciliationOrderItems: migrating ===================
-- create_table(:reconciliation_order_items)
   (21.0ms)  CREATE TABLE "reconciliation_order_items" ("id" bigserial primary key, "kind" character varying, "month" character varying, "content" text, "status" character varying, "type" character varying, "reconciliation_id" bigint, "reconciliation_result_set_id" bigint, "internal_source_type" character varying, "internal_source_id" character varying, "created_at" timestamp NOT NULL, "updated_at" timestamp NOT NULL)
   (1.0ms)  CREATE  INDEX  "index_reconciliation_order_items_on_reconciliation_id" ON "reconciliation_order_items"  ("reconciliation_id")
   (1.6ms)  ROLLBACK
   (0.3ms)  SELECT pg_advisory_unlock(5675054488983918780)
rake aborted!
StandardError: An error has occurred, this and all later migrations canceled:

Index name 'index_reconciliation_order_items_on_reconciliation_result_set_id' on table 'reconciliation_order_items' is too long; the limit is 63 characters
```

重点是在最后一句

```ruby
Index name 'index_reconciliation_order_items_on_reconciliation_result_set_id' on table 'reconciliation_order_items' is too long; the limit is 63 characters
```

### 解决问题

刚开始我并不知道这个问题的所在。只当是版本上周升级rails5的遗留问题，并且这个migrate在线上环境已经跑过了，问题不是特别大，也就每当回事。等闲下来之后查了一下资料，网上又很完善的介绍。

pg数据库[文档中](https://www.postgresql.org/docs/current/static/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS)有说：
```shell
The system uses no more than NAMEDATALEN-1 bytes of an identifier; longer names can be written in commands, but they will be truncated. By default, NAMEDATALEN is 64 so the maximum identifier length is 63 bytes. If this limit is problematic, it can be raised by changing the NAMEDATALEN constant in src/include/pg_config_manual.h.
```
大概意思就是SQL标识符与关键字不能超过64个字节。因此，我们只能创建64-1个字节，也就是63个字节。为此，rails中有相应指明name的方法。
代码如下

```ruby
t.references :reconciliation_result_set, index: { name: 'index_on_reconciliation_result_set_id'}
```
你可以在migrate中直接声明name的名字，也可以使用

```ruby
add_index :reconciliation_order_items, : reconciliation_result_set_id, name: 'index_on_reconciliation_result_set_id'
```
就是不使用默认的声明名称，自定义符合要求的声明名称即可。