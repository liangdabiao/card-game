---
layout: post
title: 用 Python 实现 FireSQL
date: 2022-04-29 12:00:00.000000000 -00:00
permalink: firesql-in-python/
author: Benny Cheung
artwork:
  author: Unknown
tags:
- Python
- Parsing
- Firestore
- SQL
- FireSQL
category: post
comments: true
image: images/firesql-in-python/cover2x.jpg
images:
  cover: images/firesql-in-python/cover.jpg
  header: images/firesql-in-python/header.jpg
published: true
---

<!--excerpt.start-->
[PyFireSQL](https://github.com/bennycheung/PyFireSQL) 是一个类 SQL 的编程接口，用 Python 查询 [Cloud Firestore](https://firebase.google.com/products/firestore) 集合。Cloud Firestore 是一个 NoSQL、面向文档的数据库。与 SQL 数据库不同，没有表或行。数据存储在文档中，文档组织成集合。
<!--excerpt.end-->

Cloud Firestore——这种 NoSQL 集合/文档结构——没有正式的查询语言。许多时候，我们只能用那个有用却笨重的 Firestore UI，在无穷无尽的记录中导航、滚动和筛选。用 UI，我们没有办法提取找到的文档。即便我们试着为特定任务写一个专门的程序来提取和更新，我们也感觉许多脚本几乎一模一样——必须做点什么来终结这种无休止的写程序。如果我们能用类 SQL 语句来做数据提取——既正式又可复用——会怎样？这个想法正是 FireSQL 语言的动机！

----
> [2024/10/05] 我们可以把本文当作 **Podcast Discussion** 来听，它由 Google 的实验性产品 _NotebookLM_ 生成。两位 AI 主持人既风趣又有洞见。

{{ site.baseurl }}/images/firesql-in-python/Podcast_FireSQL_In_Python.mp3
{% include open-embed.html %}
----

![结构化数据]({{ site.baseurl }}/images/firesql-in-python/structure-data.png)

虽然我们看不到（表、行、列）的关系数据模型，但在 Firestore 数据模型中很容易看到对应关系：table -> collection（集合），row -> document（文档），column -> field（字段）。类 SQL 语句可以相应地转换。

有一个类似的项目 [FireSQL](https://github.com/jsayol/firesql)，用 Typescript 和 [pegis](https://pegjs.org/) 解析器生成器编写，以类 SQL 语言对接 Firestore。我们强烈感觉 Python 加 [lark](https://lark-parser.readthedocs.io/en/latest/) 解析器生成器的组合更适合后端处理与分析工具链。尤其，提取的数据可以直接导入下游 Pandas 的 Dataframe 做后处理，前景极其诱人。

## FireSQL 语句
已实现的类 SQL DML（数据操纵语言）语句集合为，

| FireSQL 语句 | 描述 |
|---------------|-------------|
| SELECT | 从集合中选择文档
| INSERT | 在集合中插入新文档
| UPDATE | 修改集合中的现有文档
| DELETE | 删除集合中的现有文档

本文聚焦 FireSQL 的 `SELECT` 语句。有兴趣了解细节的读者，
请阅读对应的 [PyFireSQL Documentation @readthedocs](https://pyfiresql.readthedocs.io/en/latest/)。

## FireSQL 解析器详解
FireSQL 解析器由两部分组成：词法扫描器和语法规则模块。Python 解析器生成器 [Lark](https://lark-parser.readthedocs.io/en/latest/) 用于提供词法扫描器和语法规则来解析 FireSQL 语句。最后，解析器执行生成解析树，又称 AST（抽象语法树）。FireSQL 语法的复杂性要求一个同样复杂的结构来高效存储执行每种可能 FireSQL 语句所需的信息。

例如，FireSQL 语句的 AST 解析树
```sql
SELECT id, date, email
  FROM Bookings
  WHERE date = '2022-04-04T00:00:00'
```

![SQL 解析树示例]({{ site.baseurl }}/images/firesql-in-python/sql_parse_tree.jpg)

*图：`lark` 生成的解析树图示*

使用 `lark` 令人愉悦，因为它把语法规范与处理清晰地分离开。处理由 Visitor 或 Transformer 组件应用到解析树上。

### Visitor 与 Transformer
Visitor 与 Transformer 提供了处理 Lark 返回的解析树的便捷接口。`lark` 文档定义如下：

* **Visitor** - 访问树的每个节点，按节点的 data 运行相应的方法。它们自底向上工作：从叶子开始，到树根结束。
* **Transformer** - 自底向上（即深度优先）工作：从访问叶子开始，一路向上直到树根结束。
  * 对每个被访问的节点，transformer 会按节点的 `data` 调用相应的方法（回调），并用返回值替换节点，从而创建新的树结构。
  * Transformer 可用于实现 map & reduce 模式。因为节点是从叶子向根部归约的，回调在任何时刻都可以假设子节点已被转换。

> 一开始用 Visitor 很简单，但你需要确切知道自己要取什么；根据生成解析树的语法不同，子节点链可能很难导航。

我们决定使用 Transformer 把解析树转换为相应的 SQL 组件对象，让后续处理可以轻松消费。

例如，前述示例的解析树被转换为如下 SQL 组件：

```
SQL_Select(
  columns=[SQL_ColumnRef(table=None, column='id'),
           SQL_ColumnRef(table=None, column='date'),
           SQL_ColumnRef(table=None, column='email')],
  froms=[SQL_SelectFrom(part='Bookings', alias=None)],
  where=SQL_BinaryExpression(operator='==',
                             left=SQL_ColumnRef(table=None, column='date'),
                             right=SQL_ValueString(value='2022-04-04T00:00:00'))
)
```

有了这个转换后的数据结构，我们就可以编写处理器遍历这些组件，为相应的 Firestore 查询生成执行计划。

## FireSQL 所需的刚刚好的 SQL
起步阶段，我们不需要 DML（数据操纵语言）的完整 SQL 解析器和转换器。我们只定义 `SELECT` 语句，刚好满足 Firestore 集合查询的当前需要。

### FireSQL 语法
语法是对语言的形式化描述，可用于识别其结构。描述语法最常用的格式是**扩展巴科斯-诺尔范式**（EBNF）。巴科斯-诺尔语法中的一条典型规则形如：

```
  where_clause ::= bool_expression
  bool_expression ::= bool_parentheses
                      | bool_expression "AND" bool_parentheses
                      | bool_expression "OR" bool_parentheses
  bool_parentheses ::= comparison_type
                       | "(" bool_expression "AND" comparison_type ")"
                       | "(" bool_expression "OR" comparison_type ")"
  ...
  CNAME ::= ("_"|"/"|LETTER) ("_"|"/"|LETTER|DIGIT)*
  ...
```

`where_clause` 通常是非终结符（nonterminal），即它可以被右侧的元素组 `bool_expression` 替换。元素 `bool_expression` 可以包含其他非终结符或终结符。终结符就是那些在语法中不以 `<symbol>` 形式出现、且大写的符号。终结符的典型示例是字符串，如 "("、")"、"AND"、"OR"、"CNAME"。

### SELECT 语句
通过 `lark` 的[类 EBNF 语法](https://github.com/bennycheung/PyFireSQL/blob/main/firesql/sql/grammar/firesql.lark)，
我们编码了核心的 `SELECT` 语句，它随后被转换为待执行的 Firestore 集合查询。

- SELECT 列，用于集合字段的投影
  - DISTINCT 修饰符限制结果只包含唯一的字段值
- FROM 子句指定集合
- FROM/JOIN 子句用于连接集合（限 1 个 join）
- WHERE 子句带布尔代数表达式，对每个集合按字段值查询
  - 布尔算子：AND（当前未实现 OR）
  - 比较算子：=、!=、>、<、<=、>=
  - 容器表达式：IN、NOT IN
  - 数组包含表达式：CONTAIN、ANY CONTAIN
  - 过滤表达式：LIKE、NOT LIKE
  - 空值表达式：IS NULL、IS NOT NULL
- 聚合函数应用于结果集
  - COUNT 用于任意字段
  - SUM、AVG、MIN、MAX 用于数值字段

但处理器有以下限制，我们可以对查询结果集做后处理来弥补。
- 无 ORDER BY 子句
- 无 GROUP BY/HAVING 子句
- 无 WINDOW 子句

#### 示例
例如，可以表达如下语句：
> 所有关键字不区分大小写。解析器忽略所有空白。

> `docid` 是一个特殊字段名，用于提取所选文档的 Id
```sql
  SELECT docid, email, state
    FROM
      Users
    WHERE
      state = 'ACTIVE'
```

> `*` 选择所有字段，布尔算子 'AND' 指定多个查询条件。
```sql
  SELECT *
    FROM
      Users
    WHERE
      state IN ('ACTIVE') AND
      u.email LIKE '%benny%'
```

> 字段-子字段可以用 `"` 转义含有 `.` 的字段名。
```sql
  SELECT *
    FROM
      Users as u
    WHERE
      u.state IN ('ACTIVE') AND
      u."location.displayName" = 'Work From Home'
```

> `JOIN` 表达式把 2 个集合连接起来
```sql
SELECT u.email, u.state, b.date, b.state
  FROM
    Users as u JOIN Bookings as b
    ON u.email = b.email
  WHERE 
      u.state = 'ACTIVE' AND
      u.email LIKE '%benny%' AND
      b.state IN ('CHECKED_IN', 'CHECKED_OUT') AND
      b.date >= '2022-03-18T04:00:00'
```

> `COUNT`、`MIN`、`MAX`、`SUM`、`AVG` 是针对结果集计算的聚合函数。
> 只有数值字段（例如此处的 `cost`）对 `MIN`、`MAX`、`SUM`、`AVG` 计算才是有效值。
```sql
SELECT COUNT(*), MIN(b.cost), MAX(b.cost), SUM(b.cost), AVG(b.cost)
  FROM
    Users as u JOIN Bookings as b
    ON u.email = b.email
  WHERE 
      u.state = 'ACTIVE' AND
      u.email LIKE '%benny%' AND
      b.state IN ('CHECKED_IN', 'CHECKED_OUT') AND
      b.date >= '2022-03-18T04:00:00'
```

> `DISTINCT` 修饰符只选择唯一的字段。
```sql
SELECT DISTINCT email
  FROM
    Bookings
  WHERE
    date > '2022-04-01T00:00:00'
```

> FireSQL 语法规范见 [firesql.lark](https://github.com/bennycheung/PyFireSQL/blob/main/firesql/sql/grammar/firesql.lark)。

### 集合路径
Firestore 集合拥有一组文档。每个文档可以嵌套更多集合。Firestore 用路径标识一个集合，形如 `Companies/bennycorp/Users` 表示 `Companies` 集合有一个文档 `bennycorp`，后者拥有 `Users` 集合。

如果想查询嵌套集合，可以把集合名指定为路径。
路径可以很长，但我们可以用 `AS` 关键字定义别名。

例如，子集合 `Users` 和 `Bookings` 挂在 `Companies/bennycorp` 文档之下。

```sql
SELECT u.email, u.state, b.date, b.state
  FROM
    Companies/bennycorp/Users as u JOIN Companies/bennycorp/Bookings as b
    ON u.email = b.email
  WHERE 
      u.state = 'ACTIVE' AND
      b.date >= '2022-03-18T04:00:00'
```

> Firestore 有趣的事实：集合路径的段数必须是奇数。

### 文档字段与子字段
由于 Firestore 文档字段可以嵌套子字段，FireSQL 语句的列引用可以通过带引号的字符串到达文档子字段——用 `"` 转义含有 `.` 的字段名。凡是允许列引用的地方都可以使用带引号的字符串。

例如，`Users` 文档的 `location` 字段拥有子字段 `displayName`。子字段可以通过 `"location.displayName"` 访问

```sql
  SELECT email, "location.displayName"
  FROM Users
  WHERE "location.displayName" = 'Work From Home'
```

### 文档 ID
Firestore 为每个文档关联一个唯一的"文档 ID"。文档 ID 不属于文档字段，需要特殊处理才能访问。FireSQL 引入了特殊字段 `docid`，让任何语句都能引用唯一的"文档 ID"。

例如，我们可以在 `Users` 集合中选择文档等于特定 `docid` 的记录。即使文档没有 `docid` 字段，我们也可以在输出中投影 `docid` 值。

```sql
  SELECT docid, email
  FROM Users
  WHERE docid = '4LLlLw6tZicB40HrjhDJNmvaTYw1'
```

由于 Firestore admin API 的限制，与 `docid` 一起只能表达 `=` 等于或 `IN` 算子。
例如，以下语句会找到 `docid` 在指定数组中的文档。

```sql
  SELECT docid, email
  FROM Users
  WHERE docid IN ('4LLlLw6tZicB40HrjhDJNmvaTYw1', '74uWntZuVPeYcLVcoS0pFApGPdr2')
```

更有趣的是，如果想投影全部字段，包括 `docid`。可以这样写 select 语句：
`docid` 和 `*` 都投影到输出。

```sql
  SELECT docid, *
  FROM Users
  WHERE "location.displayName" = 'Work From Home'
```

### DateTime 类型
一致地描述日期时间是个大话题，我们做了一个务实的设计选择。
我们使用 [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) 以字符串表达日期时间，
而 Firestore 以 UTC 的 `Timestamp` 数据类型存储日期时间。
例如：
- "2022 年 3 月 18 日 UTC 时间 4 点" 的日期时间字符串写作 "2022-03-18T04:00:00"。
- "2022 年 3 月 18 日 多伦多时间 EDT（-4 小时）0 点" 的日期时间字符串写作 "2022-03-18T00:00:00-04"。

如有疑问，我们使用以下常量来转换、匹配并以 ISO-8601 字符串渲染日期时间值。

```python
DATETIME_ISO_FORMAT = "%Y-%m-%dT%H:%M:%S"
DATETIME_ISO_FORMAT_REGEX = r'^(-?(?:[1-9][0-9]*)?[0-9]{4})-(1[0-2]|0[1-9])-(3[01]|0[1-9]|[12][0-9])T(2[0-3]|[01][0-9]):([0-5][0-9]):([0-5][0-9])(\.[0-9]+)?(Z|[+-](?:2[0-3]|[01][0-9]):[0-5][0-9])?$'
```

### 用 LIKE 做模式匹配
SQL 表达式 `LIKE` 或 `NOT LIKE` 可用于匹配字符串数据。

```sql
SELECT docid, email, state
  FROM
    Users
  WHERE
    state IN ('ACTIVE') AND
    email LIKE '%benny%'
```

在 Firebase 查询之后，模式匹配作为过滤表达式使用。SQL 处理器支持的模式有：
- 前缀匹配 `pattern%`
- 后缀匹配 `%pattern`
- 中缀匹配 `%pattern%`

### JSON 数据
PyFireSQL 提供 JSON 数据支持，尤其面向必须接受复杂数据类型的 `INSERT` 与 `UPDATE` 语句。
当字段值需要接受复杂数据类型（如数组或映射，即 Python dict）时，
这些复杂数据类型必须编码在 JSON 封装内。JSON 封装可以解释任何合法的 JSON 对象；
随后转换为 Firestore 支持的相应数据类型。

例如，

```sql
INSERT INTO Companies/bennycorp/Visits
  ( email, event )
  VALUES
    ( 'btscheung+test1@gmail.com', JSON(["event1","event2","event3"]) )
```

当集合 `Visits` 有一个字段 `event` 接受事件名数组时，
我们用 `JSON` 封装把数组 `["event1","event2","event3"]` 编码为 JSON 字符串，赋给 `event` 字段。

既然我们把 Firestore 当作无 schema 的文档结构来处理，
我们可以把 JSON 映射中的全部键值对插入集合。

例如，以下 insert 语句——列规格用 `*` 表示全部字段。
我们插入一组
`email`、`firstName`、`lastName`、`groups`（数组）、`roles`（数组）、`vaccination`、`access`（映射）。

```sql
INSERT INTO Companies/bennycorp/Users
  ( * )
  VALUES (
    JSON(
      {
        "email": "btscheung+twotwo@gmail.com",
        "name": "Benny TwoTwo",
        "groups": [],
        "roles": [
            "ADMIN"
        ],
        "vaccination": null,
        "access": {
          "hasAccess": true
        }
      }
    )
  )
```

## FireSQL 到 Firebase 查询
我们提供了一个简单的 FireSQL 接口类，可以接受 FireSQL 语句来查询 Firestore 集合。

### 安装方法
从 PyPi 安装 PyFireSQL，

```
pip install pyfiresql
```

从 [PyFireSQL 源码](https://github.com/bennycheung/PyFireSQL)安装，检出项目：
```sh
cd PyFireSQL
# install require packages
pip install -r requirements.txt
# install (optional) development require packages
pip install -r requirements_dev.txt

python setup.py install
```

### 编程接口
PyFireSQL 提供一个简单的编程接口来解析并执行 firebase SQL。
请查阅 [Firebase Admin SDK Documentation](https://firebase.google.com/docs/admin/setup) 生成项目的服务账户 `credentials.json` 文件。

```python
from firesql.firebase import FirebaseClient
from firesql.sql.sql_fire_client import FireSQLClient
from firesql.sql import FireSQL

# make connection to Cloud Firestore
client = FirebaseClient()
client.connect(credentials_json='credentials.json')
# wrapped as FireSQL client interface
sqlClient = FireSQLClient(client)

# query via the FireSQL interface - the results are in list of docs (Dict)
query = "SELECT * FROM Users WHERE state = 'ACTIVE'"
fireSQL = FireSQL()
docs = fireSQL.execute(sqlClient, query)
```

`fireSQL.execute()` 查询完成后，结果是一个满足查询的文档列表（Dict 形式）。
然后我们可以把文档列表传递给任何输出格式的渲染，就我们而言，`DocPrinter` 对象可以按所选字段输出 `csv` 或 `json`。

```python
from firesql.sql.doc_printer import DocPrinter

docPrinter = DocPrinter()
if format == 'csv':
  docPrinter.printCSV(docs, fireSQL.select_fields())
else:
  docPrinter.printJSON(docs, fireSQL.select_fields())
```

如需进一步后处理，我们可以用 Pandas 的 Dataframe 执行任何数据分析、分组、排序和计算。文档列表（Dict 形式）可以直接导入 Dataframe！非常方便。

```python
import pandas as pd

df = pd.DataFrame(docs)
```

### 查询脚本

此外，我们提供接口脚本 `firesql-query.py` 接受 FireSQL 语句。

```
usage: firesql-query.py [-h] [-c CREDENTIALS] [-f FORMAT] [-i INPUT]
                        [-q QUERY]

optional arguments:
  -h, --help            show this help message and exit
  -c CREDENTIALS, --credentials CREDENTIALS
                        credentials JSON path
  -f FORMAT, --format FORMAT
                        output format (csv|json)
  -i INPUT, --input INPUT
                        FireSQL query input file (required)
  -q QUERY, --query QUERY
                        FireSQL query (required)
```
例如，从 Users 集合找出全部 `ACTIVE` 用户

```
python firesql-query.py -c credential.json \
  -q "SELECT docid, email, state FROM Users WHERE state IN ('ACTIVE')"
```
> `docid` 是特殊的列名，用于投影 Firestore 文档 ID。

默认查询结果以 "csv" 输出格式渲染。

```
"docid","email","state"
"0r6YWowe9rW65yB1qTKsCe83cCm2","btscheung+real@gmail.com","ACTIVE"
"1utcUa9fdheOlrMe9GOCjrJ3wjh1","btscheung+bennycorp@gmail.com","ACTIVE"
"7CUJOqe6rlOTQuatc27EQGivZfn2","btscheung+twotwo@gmail.com","ACTIVE"
...
```

或者，指定 `-f json` 输出格式，结果为，

```json
[
  {"docid": "0r6YWowe9rW65yB1qTKsCe83cCm2", "email": "btscheung+real@gmail.com", "state": "ACTIVE"},
  {"docid": "1utcUa9fdheOlrMe9GOCjrJ3wjh1", "email": "btscheung+bennycorp@gmail.com", "state": "ACTIVE"},
  {"docid": "7CUJOqe6rlOTQuatc27EQGivZfn2", "email": "btscheung+twotwo@gmail.com", "state": "ACTIVE"},
  ...
]
```
#### SQL 输入文件
对更复杂的 SQL，可以用 `-i input.sql` 指定 SQL 输入文件。

`input.sql` 文件：
```sql
SELECT u.email, u.state, b.date, b.state
  FROM
    Users as u JOIN Bookings as b
    ON u.email = b.email
  WHERE 
      u.state IN ('ACTIVE') and
      b.state IN ('CHECKED_IN', 'CHECKED_OUT') and
      b.date >= '2022-03-18T04:00:00'
```

执行输入文件

```
python firesql-query.py -c credentials.json -i input.sql
```

结果为，
> 注意：来自 `Users` 的 `state` 列会自动消歧，追加别名前缀 `u_state`。

```
"email","u_state","date","state"
"btscheung+bennycorp@gmail.com","ACTIVE","2022-03-18T04:00:00","CHECKED_IN"
"btscheung+bennycorp@gmail.com","ACTIVE","2022-03-18T04:00:00","CHECKED_IN"
"btscheung+hill6@gmail.com","ACTIVE","2022-03-31T04:00:00","CHECKED_IN"
...
```

## 结语
如果你读到了这里，说明你正承受着与 Cloud Firestore 查询相同的痛苦。希望本文能促使你
试用 PyFireSQL。它可以成为你用 Python 对接 Firestore 的首选编程接口！

FireSQL 还有许多有待实现的改进。仅举几项未来改进：
- FROM 子句中的多条 JOIN 语句
- WHERE 子句允许 OR 布尔表达式
- 在向 Firestore 发送查询之前优化查询计划
- SELECT 子句支持子查询

欢迎加入 [PyFireSQL](https://github.com/bennycheung/PyFireSQL) 开源项目，或提供反馈以改进 FireSQL 工具！

## 参考文献

### FireSQL
- PyFireSQL <https://github.com/bennycheung/PyFireSQL>
  - PyPi <https://pypi.org/project/pyfiresql/>
  - Documentation @readthedocs <https://pyfiresql.readthedocs.io/en/latest/>

### Firebase Python
- Google Cloud Firestore <https://firebase.google.com/products/firestore>
- Google Cloud Firestore Python Client SDK <https://googleapis.dev/python/firestore/latest/client.html>
- Firebase Admin SDK Documentation <https://firebase.google.com/docs/admin/setup>

### 语言解析
- Gabriele Tomassetti, Parsing In Python: Tools And Libraries, <https://tomassetti.me/parsing-in-python/>
- Lark Documentation <https://lark-parser.readthedocs.io/en/latest/>
  - 代码仓库：[lark-parser](https://github.com/lark-parser/lark) 

### 类似项目
以下项目启发了 PyFireSQL 的开发。它们的用途或基础语言不同。
- From SQL to Ibis Parsing - sql_to_ibis <https://github.com/zbrookle/sql_to_ibis>
  - `sql_to_ibis` 是一个把 SQL 语法翻译为 [ibis](https://github.com/ibis-project/ibis) 表达式的 Python 包。它提供只用一种 SQL 方言面向多种不同后端的能力。
- Ibis: Python data analysis framework for Hadoop and SQL engines, <https://ibis-project.org/docs/dev/>
  - 代码仓库：<https://github.com/ibis-project/ibis>
  - Ibis 是一个 Python 框架，以标准方式从不同来源访问数据并执行分析计算。
- FireSQL (Node.js - Typescript) Project, <https://firebaseopensource.com/projects/jsayol/firesql/>
  - 代码仓库：<https://github.com/jsayol/firesql>
  - FireSQL 是构建在官方 Firebase SDK 之上的库，允许你用 SQL 语法查询 Cloud Firestore。它足够聪明，会向 Firestore 服务器发出获取所需数据所必需的最少查询。
