## 在 rails 5 中 uniq 的使用习惯已经变化了

```ruby
Use 'pluck("distinct(organization_id)")' instead of 'uniq.pluck(:organization_id)'
```

在 rails 5 中 uniq 已经不是 distinct 查询了, 它是数组方法

```ruby
User.where(id: 1).pluck("distinct(email)")
DEPRECATION WARNING: Dangerous query method (method whose arguments are used as raw SQL) called with non-attribute argument(s): "distinct(email)". Non-attribute arguments will be disallowed in Rails 6.0. This method should not be called with user-provided values, such as request parameters or model attributes. Known-safe values can be passed by wrapping them in Arel.sql(). (called from irb_binding at (irb):3)
   (2.2ms)  SELECT distinct(email) FROM "users" WHERE "users"."id" = $1  [["id", 1]]
 => [nil]
User.where(id: 1).uniq.pluck("email")
  User Load (0.6ms)  SELECT "users".* FROM "users" WHERE "users"."id" = $1  [["id", 1]]
 => [nil]

```