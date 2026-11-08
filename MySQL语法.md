# MySQL 常用语法速查

> 示例以 MySQL 8.0 为基础。执行时请替换数据库名、表名和参数；带 `?` 的语句表示应用程序中的参数化查询占位符，不能直接作为普通 SQL 粘贴执行。

## 快捷索引

- [1. 基础约定与连接](#s01)
- [2. 数据库操作](#s02)
- [3. 建表与常用数据类型](#s03)
- [4. 查看与修改表结构](#s04)
- [5. 插入数据](#s05)
- [6. 查询、筛选与排序](#s06)
- [7. 聚合、分组与去重](#s07)
- [8. 多表连接](#s08)
- [9. 子查询与集合操作](#s09)
- [10. 更新与删除](#s10)
- [11. 常用函数与条件表达式](#s11)
- [12. 索引与执行计划](#s12)
- [13. 事务与行锁](#s13)
- [14. 视图、CTE 与窗口函数](#s14)
- [15. 用户与权限](#s15)
- [16. 导入与备份](#s16)
- [17. 常见易错点](#s17)

<a id="s01"></a>
## 1. 基础约定与连接

在终端执行：

```bash
# -p 后不写密码，通过提示输入；-P 是端口，-p 是密码选项
mysql -h 127.0.0.1 -P 3306 -u root -p

# 直接连接指定数据库
mysql -h 127.0.0.1 -u root -p demo_db
```

进入 MySQL 客户端后：

```sql
SELECT VERSION();                  -- 查看版本
SELECT DATABASE();                 -- 当前数据库
SELECT CURRENT_USER();             -- 当前认证账户
SHOW VARIABLES LIKE 'sql_mode';    -- SQL 模式
SHOW VARIABLES LIKE 'autocommit';  -- 自动提交设置

-- 单行注释：两个减号后需要空格
# MySQL 单行注释
/* 多行注释 */

SELECT 'hello';                    -- 字符串用单引号
SELECT `name` FROM `users`;         -- 标识符可用反引号

-- 退出客户端
QUIT;
```

<a id="s02"></a>
## 2. 数据库操作

```sql
SHOW DATABASES;

CREATE DATABASE IF NOT EXISTS demo_db
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_0900_ai_ci;

USE demo_db;
SHOW CREATE DATABASE demo_db;

-- 修改默认字符集，不会自动转换已有表和列
ALTER DATABASE demo_db CHARACTER SET utf8mb4;

-- 危险：删除数据库及其中的数据
DROP DATABASE IF EXISTS demo_db;
```

<a id="s03"></a>
## 3. 建表与常用数据类型

后续大部分示例使用 `users` 表；连接示例另外使用 `orders` 表。

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(255) NOT NULL,
    age TINYINT UNSIGNED DEFAULT NULL,
    city VARCHAR(100) DEFAULT NULL,
    balance DECIMAL(12, 2) NOT NULL DEFAULT 0.00,
    status TINYINT NOT NULL DEFAULT 1 COMMENT '1启用，0禁用',
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
    UNIQUE KEY uk_users_email (email),
    KEY idx_users_city_age (city, age)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4;

CREATE TABLE orders (
    id BIGINT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    amount DECIMAL(12, 2) NOT NULL,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    KEY idx_orders_user_id (user_id),
    CONSTRAINT fk_orders_user
        FOREIGN KEY (user_id) REFERENCES users(id)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4;
```

| 类型 | 常见用途 | 注意点 |
| --- | --- | --- |
| `INT` / `BIGINT` | 整数、主键 | `UNSIGNED` 表示无符号 |
| `DECIMAL(p, s)` | 金额、精确小数 | `p` 是总位数，`s` 是小数位数 |
| `FLOAT` / `DOUBLE` | 近似浮点数 | 不适合要求精确计算的金额 |
| `CHAR(n)` | 定长字符串 | 适合固定长度编码 |
| `VARCHAR(n)` | 变长字符串 | `n` 表示字符数，存储还受字节限制影响 |
| `TEXT` | 较长文本 | 按内容规模选择具体类型 |
| `DATE` | 日期 | 如 `'2026-11-07'` |
| `DATETIME` | 日期和时间 | 不进行会话时区转换 |
| `TIMESTAMP` | 时间戳 | 存取涉及会话时区转换 |
| `JSON` | JSON 数据 | 可使用 JSON 函数查询 |
| `BOOLEAN` | 布尔标记 | MySQL 中是 `TINYINT(1)` 的别名 |

常见约束：`PRIMARY KEY` 主键、`NOT NULL` 非空、`UNIQUE` 唯一、`DEFAULT` 默认值、`FOREIGN KEY` 外键。

<a id="s04"></a>
## 4. 查看与修改表结构

以下修改语句为独立示例，请按需选择执行。

```sql
SHOW TABLES;
DESC users;
SHOW CREATE TABLE users;
SHOW INDEX FROM users;

ALTER TABLE users ADD COLUMN phone VARCHAR(20) DEFAULT NULL;
ALTER TABLE users MODIFY COLUMN name VARCHAR(100) NOT NULL;
ALTER TABLE users RENAME COLUMN phone TO mobile;
ALTER TABLE users DROP COLUMN mobile;

RENAME TABLE users TO app_users;

-- 复制表结构，不复制数据
CREATE TABLE users_copy LIKE app_users;

-- 危险：删除整个表
DROP TABLE IF EXISTS users_copy;
```

<a id="s05"></a>
## 5. 插入数据

```sql
-- 推荐明确指定列名
INSERT INTO users (name, email, age, city)
VALUES ('张三', 'zhangsan@example.com', 20, '西安');

-- 批量插入
INSERT INTO users (name, email, age, city)
VALUES
    ('李四', 'lisi@example.com', 22, '北京'),
    ('王五', 'wangwu@example.com', 24, '上海');

SELECT LAST_INSERT_ID();  -- 当前连接最近一次插入产生的自增 ID

-- 主键或唯一键冲突时更新（这里 email 有唯一索引）
INSERT INTO users (name, email, age)
VALUES ('张三', 'zhangsan@example.com', 21)
ON DUPLICATE KEY UPDATE age = 21;

-- 将查询结果插入另一张结构相容的表
CREATE TABLE users_archive LIKE users;
INSERT INTO users_archive (id, name, email, age, city, balance, status,
                           created_at, updated_at)
SELECT id, name, email, age, city, balance, status, created_at, updated_at
FROM users
WHERE status = 0;
```

<a id="s06"></a>
## 6. 查询、筛选与排序

```sql
SELECT id, name, age FROM users;
SELECT name AS 姓名, age AS 年龄 FROM users;

SELECT * FROM users WHERE age >= 18 AND status = 1;
SELECT * FROM users WHERE city = '西安' OR city = '北京';
SELECT * FROM users WHERE city IN ('西安', '北京');
SELECT * FROM users WHERE age BETWEEN 18 AND 25;  -- 包含两端
SELECT * FROM users WHERE age <> 20;             -- 不等于，也可用 !=

-- 模糊匹配：% 匹配任意数量字符，_ 匹配一个字符
SELECT * FROM users WHERE name LIKE '张%';
SELECT * FROM users WHERE name LIKE '_三';

-- NULL 的判断不能用 = NULL 或 <> NULL
SELECT * FROM users WHERE age IS NULL;
SELECT * FROM users WHERE age IS NOT NULL;

-- DESC 降序；ASC 升序（默认）；id 用于稳定相同年龄的排序
SELECT id, name, age FROM users ORDER BY age DESC, id ASC;

-- 第 1 页，每页 10 条
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 0;

-- 第 3 页，每页 10 条：偏移量 = (页码 - 1) × 每页条数
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

-- 游标分页：100 为上一页最后一条记录的 ID
SELECT * FROM users WHERE id > 100 ORDER BY id LIMIT 10;

-- 时间范围：左闭右开，覆盖整天并避免对列使用函数
SELECT * FROM users
WHERE created_at >= '2026-11-07 00:00:00'
  AND created_at <  '2026-11-08 00:00:00';
```

常见书写顺序：

```sql
SELECT 列
FROM 表
WHERE 行筛选条件
GROUP BY 分组列
HAVING 分组筛选条件
ORDER BY 排序列
LIMIT 条数 OFFSET 偏移量;
```

<a id="s07"></a>
## 7. 聚合、分组与去重

```sql
SELECT DISTINCT city FROM users;

SELECT
    COUNT(*) AS total_users,         -- 所有行
    COUNT(age) AS users_with_age,    -- age 非 NULL 的行
    SUM(balance) AS total_balance,
    AVG(age) AS average_age,
    MAX(age) AS oldest,
    MIN(age) AS youngest
FROM users;

-- WHERE 先筛选行，HAVING 再筛选分组结果
SELECT city, COUNT(*) AS user_count, AVG(age) AS average_age
FROM users
WHERE status = 1
GROUP BY city
HAVING COUNT(*) >= 2
ORDER BY user_count DESC;
```

分组查询中，选择的非聚合列通常应写在 `GROUP BY` 中，或由分组列通过约束唯一确定。`SUM`、`AVG` 等聚合通常忽略 `NULL`；没有匹配行时 `COUNT` 返回 0，`SUM` 可返回 `NULL`。

<a id="s08"></a>
## 8. 多表连接

```sql
-- 内连接：只返回有匹配订单的用户及订单
SELECT u.id, u.name, o.id AS order_id, o.amount
FROM users AS u
INNER JOIN orders AS o ON o.user_id = u.id;

-- 左连接：保留所有用户；没有订单时订单列为 NULL
SELECT u.id, u.name, o.id AS order_id, o.amount
FROM users AS u
LEFT JOIN orders AS o ON o.user_id = u.id;

-- 查询没有订单的用户
SELECT u.id, u.name
FROM users AS u
LEFT JOIN orders AS o ON o.user_id = u.id
WHERE o.id IS NULL;

-- 只匹配金额大于 100 的订单，同时保留所有用户
SELECT u.id, u.name, o.amount
FROM users AS u
LEFT JOIN orders AS o
    ON o.user_id = u.id AND o.amount > 100;

-- 每位用户的订单数与总金额
SELECT u.id, u.name, COUNT(o.id) AS order_count,
       COALESCE(SUM(o.amount), 0) AS total_amount
FROM users AS u
LEFT JOIN orders AS o ON o.user_id = u.id
GROUP BY u.id, u.name;
```

左连接后在 `WHERE` 中写 `o.amount > 100` 会排除右表为 `NULL` 的行。统计左连接的订单数应使用 `COUNT(o.id)`，因为 `COUNT(*)` 也会计算没有订单时保留的那一行。

<a id="s09"></a>
## 9. 子查询与集合操作

```sql
-- 标量子查询
SELECT * FROM users WHERE age > (SELECT AVG(age) FROM users);

-- IN 子查询
SELECT * FROM users
WHERE id IN (SELECT user_id FROM orders WHERE amount >= 100);

-- EXISTS：判断关联记录是否存在
SELECT u.* FROM users AS u
WHERE EXISTS (SELECT 1 FROM orders AS o WHERE o.user_id = u.id);

-- NOT EXISTS：查询没有订单的用户
SELECT u.* FROM users AS u
WHERE NOT EXISTS (SELECT 1 FROM orders AS o WHERE o.user_id = u.id);

-- UNION 去重，UNION ALL 保留重复；各查询的列数和类型应相容
SELECT name FROM users WHERE city = '西安'
UNION
SELECT name FROM users WHERE city = '北京';

SELECT id, name FROM users WHERE city = '西安'
UNION ALL
SELECT id, name FROM users WHERE city = '北京';
```

如果 `NOT IN` 的子查询结果包含 `NULL`，可能得不到预期结果；排除关联记录时通常可用 `NOT EXISTS`。

<a id="s10"></a>
## 10. 更新与删除

先用相同的 `WHERE` 执行 `SELECT`，确认目标记录后再更新或删除。

```sql
SELECT * FROM users WHERE id = 1;

UPDATE users
SET age = 23, city = '西安'
WHERE id = 1;

-- 基于原值更新
UPDATE users SET balance = balance + 100.00 WHERE id = 1;

-- 删除满足条件的行；如果被外键引用，可能被阻止
DELETE FROM users WHERE id = 1;

-- 危险：清空所有行（InnoDB 中可在未提交事务内回滚）
DELETE FROM users;

-- 危险：快速清空表，通常重置自增值；会隐式提交，不能事务回滚
-- users 被 orders 外键引用时，TRUNCATE 会受到限制
TRUNCATE TABLE users_archive;
```

| 语句 | 作用 | 表结构 | InnoDB 事务回滚 |
| --- | --- | --- | --- |
| `DELETE FROM t WHERE ...` | 删除指定行 | 保留 | 未提交时可以 |
| `TRUNCATE TABLE t` | 清空表 | 保留 | 不可以 |
| `DROP TABLE t` | 删除整个表 | 删除 | 不可以 |

`UPDATE` 和 `DELETE` 不写 `WHERE` 会作用于所有行。

<a id="s11"></a>
## 11. 常用函数与条件表达式

```sql
-- 字符串
SELECT CONCAT('My', 'SQL');                  -- MySQL
SELECT CONCAT_WS('-', '2026', '11', '07');   -- 2026-11-07
SELECT CHAR_LENGTH('你好');                  -- 字符数：2
SELECT LENGTH('你好');                       -- 字节数，取决于字符编码
SELECT UPPER('mysql'), LOWER('MYSQL');
SELECT TRIM('  hello  ');
SELECT SUBSTRING('abcdef', 2, 3);            -- bcd，位置从 1 开始
SELECT REPLACE('a-b-c', '-', '/');

-- 数值
SELECT ROUND(123.456, 2);                    -- 123.46
SELECT CEIL(1.2), FLOOR(1.8), ABS(-5);
SELECT MOD(10, 3);                          -- 1

-- 日期时间
SELECT NOW(), CURDATE(), CURTIME();
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s'); -- %i 是分钟
SELECT DATE_ADD(CURDATE(), INTERVAL 7 DAY);
SELECT DATE_SUB(CURDATE(), INTERVAL 1 MONTH);
SELECT DATEDIFF('2026-11-07', '2026-11-01');  -- 6 天
SELECT TIMESTAMPDIFF(YEAR, '2000-01-01', CURDATE());

-- 空值处理
SELECT IFNULL(NULL, 0);                     -- 0
SELECT COALESCE(NULL, NULL, '默认值');       -- 第一个非 NULL 值
SELECT 10 / NULLIF(0, 0);                   -- 分母为 0 时转为 NULL

-- 条件表达式
SELECT name,
       CASE
           WHEN age IS NULL THEN '未知'
           WHEN age < 18 THEN '未成年'
           ELSE '成年'
       END AS age_group,
       IF(status = 1, '启用', '禁用') AS status_text
FROM users;

-- JSON
SELECT JSON_EXTRACT('{"name":"张三","age":20}', '$.name');
SELECT JSON_UNQUOTE(JSON_EXTRACT('{"name":"张三"}', '$.name'));
```

<a id="s12"></a>
## 12. 索引与执行计划

```sql
CREATE INDEX idx_users_name ON users(name);
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at);

SHOW INDEX FROM users;
DROP INDEX idx_users_name ON users;

EXPLAIN SELECT * FROM users WHERE email = 'zhangsan@example.com';
EXPLAIN SELECT * FROM orders
WHERE user_id = 1 ORDER BY created_at DESC LIMIT 10;
```

- 主键索引：`PRIMARY KEY`；唯一索引：`UNIQUE`；普通索引：`INDEX` / `KEY`。
- 联合索引 `(city, age)` 通常适合以 `city` 开头的查询条件；是否采用索引由优化器决定。
- `LIKE '张%'` 可能使用索引；`LIKE '%张%'` 通常不能用普通 B-tree 索引做有效范围查找。
- 对索引列使用函数或发生隐式类型转换，可能影响索引使用。
- 索引会增加存储和写入开销，应结合真实查询与 `EXPLAIN` 判断。

<a id="s13"></a>
## 13. 事务与行锁

适用于 InnoDB 表。一个事务内的语句应在同一个数据库连接中执行。

```sql
START TRANSACTION;

-- 示例：转账前锁定并检查双方记录及余额
SELECT id, balance FROM users WHERE id IN (1, 2) ORDER BY id FOR UPDATE;

-- 应用程序确认两条记录都存在、付款方余额足够后再执行
UPDATE users SET balance = balance - 100.00 WHERE id = 1;
UPDATE users SET balance = balance + 100.00 WHERE id = 2;

COMMIT;
-- 检查不通过或发生错误时，应改为执行 ROLLBACK;
```

保存点示例：

```sql
START TRANSACTION;
UPDATE users SET city = '西安' WHERE id = 1;
SAVEPOINT sp1;
UPDATE users SET age = 25 WHERE id = 1;
ROLLBACK TO SAVEPOINT sp1;  -- 撤销保存点之后的修改
COMMIT;                    -- 提交 city 的修改
```

`FOR UPDATE` 应放在显式事务中使用，锁通常持续到提交或回滚。`CREATE`、`ALTER`、`DROP`、`TRUNCATE` 等常见 DDL 会隐式提交，不能按普通数据修改的方式回滚。

<a id="s14"></a>
## 14. 视图、CTE 与窗口函数

```sql
-- 视图：保存查询定义
CREATE OR REPLACE VIEW active_users AS
SELECT id, name, city FROM users WHERE status = 1;

SELECT * FROM active_users;
DROP VIEW IF EXISTS active_users;

-- CTE：公共表表达式（MySQL 8.0+）
WITH user_totals AS (
    SELECT user_id, SUM(amount) AS total_amount
    FROM orders
    GROUP BY user_id
)
SELECT u.name, t.total_amount
FROM users AS u
JOIN user_totals AS t ON t.user_id = u.id
WHERE t.total_amount >= 1000;

-- 窗口函数：按城市分别编号，不合并原始行（MySQL 8.0+）
SELECT id, name, city, age,
       ROW_NUMBER() OVER (
           PARTITION BY city ORDER BY age DESC, id ASC
       ) AS row_num
FROM users;

-- 每个用户的订单按时间累计金额
SELECT id, user_id, amount,
       SUM(amount) OVER (
           PARTITION BY user_id ORDER BY created_at, id
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM orders;
```

`ROW_NUMBER()` 连续编号且不并列；`RANK()` 并列后跳号；`DENSE_RANK()` 并列后不跳号。是否并列由窗口中的全部排序表达式决定。

<a id="s15"></a>
## 15. 用户与权限

需要使用有相应管理权限的账户执行。账户由“用户名 + 来源主机”共同确定。

```sql
CREATE USER 'app_user'@'localhost' IDENTIFIED BY '替换为强密码';

GRANT SELECT, INSERT, UPDATE, DELETE
ON demo_db.* TO 'app_user'@'localhost';

SHOW GRANTS FOR 'app_user'@'localhost';

ALTER USER 'app_user'@'localhost' IDENTIFIED BY '替换为新的强密码';

REVOKE DELETE ON demo_db.* FROM 'app_user'@'localhost';
DROP USER 'app_user'@'localhost';
```

通过 `CREATE USER`、`GRANT`、`REVOKE` 修改账户和权限后，通常不需要执行 `FLUSH PRIVILEGES`。

<a id="s16"></a>
## 16. 导入与备份

以下命令在系统终端执行，`--result-file` 可避免依赖 shell 的输出重定向编码行为：

```bash
# 备份数据库；--single-transaction 主要适用于 InnoDB
mysqldump -u root -p --single-transaction --result-file=demo_db.sql demo_db

# 只备份表结构
mysqldump -u root -p --no-data --result-file=schema.sql demo_db

# 备份指定表
mysqldump -u root -p --result-file=users.sql demo_db users
```

恢复到已创建的数据库：

```bash
mysql -u root -p demo_db
```

然后在 MySQL 客户端执行（Windows 路径可使用正斜杠）：

```sql
SOURCE E:/backup/demo_db.sql;
```

恢复脚本可能包含删除或重建表的语句，应先确认目标数据库。备份期间避免并发修改表结构，以免影响一致性。

<a id="s17"></a>
## 17. 常见易错点

- SQL 语句通常用分号结束；MySQL 客户端的 `SOURCE` 等属于客户端命令。
- 字符串使用单引号；不要依赖双引号的默认行为，因为 `ANSI_QUOTES` 模式会改变其含义。
- `NULL` 表示缺失值，与空字符串、数字 0 不同；使用 `IS NULL` 或 `IS NOT NULL` 判断。
- `AND` 优先级高于 `OR`，复杂条件应加括号，如 `WHERE status = 1 AND (city = '西安' OR city = '北京')`。
- 分页应搭配确定的 `ORDER BY`，必要时加入唯一列保证排序稳定。
- `WHERE` 筛选行；`HAVING` 筛选分组结果；聚合条件通常写在 `HAVING` 中。
- 表名在不同操作系统与配置下可能有不同的大小写行为，项目中应统一命名。
- 应用程序应使用参数化查询，如 `SELECT * FROM users WHERE email = ?`，不要拼接用户输入。
- 示例展示多种独立操作，包含重命名和删除，不能按全文顺序直接执行。
