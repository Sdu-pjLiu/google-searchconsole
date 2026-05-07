---
icon: lucide/book-open
---

# API 参考

Google Search Console 库中所有类、方法与函数的完整参考说明。

## 模块：`searchconsole`

### `authenticate()`

主认证函数，返回已认证的 `Account` 对象。

```python
searchconsole.authenticate(
    client_config=None,
    credentials=None,
    serialize=None,  # 已弃用（DEPRECATED）
    flow="web",
    service_account=None
)
```

**参数：**

- **`client_config`** (str or dict, optional): OAuth2 客户端配置
    - 包含 client secrets 的 JSON 文件路径
    - 客户端配置字典
    - 使用 OAuth2 时必填（除非改用服务账号）

- **`credentials`** (str or dict, optional): OAuth2 用户凭据
    - 已保存凭据的 JSON 文件路径
    - 凭据数据字典
    - 若未提供，将启动 OAuth 流程

- **`serialize`** (str, optional): **已弃用** — 请改用 `Account.serialize_credentials()`
    - 认证成功后保存凭据的路径

- **`flow`** (str, default="web"): OAuth2 流程类型
    - `"web"`: 在浏览器中完成认证（默认）
    - `"console"`: 打印 URL 供手动认证（适用于远程服务器）

- **`service_account`** (str or dict, optional): 服务账号凭据
    - 服务账号 JSON 密钥文件路径
    - 服务账号数据字典
    - 不能与 `client_config` 或 `credentials` 同时使用

**返回值：**

- **`Account`**: 已认证账号对象，可访问各网站资源

**可能抛出：**

- **`ValueError`**: 同时提供了 OAuth2 与服务账号凭据，或未提供任何凭据

**示例：**

```python
# 首次 OAuth2 认证
account = searchconsole.authenticate(client_config='client_secrets.json')

# 使用已保存凭据
account = searchconsole.authenticate(
    client_config='client_secrets.json',
    credentials='credentials.json'
)

# 控制台流程
account = searchconsole.authenticate(
    client_config='client_secrets.json',
    flow='console'
)

# 服务账号
account = searchconsole.authenticate(
    service_account='service_account.json'
)

# 使用字典传参
account = searchconsole.authenticate(
    client_config={'installed': {...}},
    credentials={'token': '...', ...}
)
```

---

## 类：`Account`

表示可访问多个网站资源的 Google Search Console 账号。

!!! note
    `Account` 由 `authenticate()` 创建，请勿直接实例化。

### 属性

#### `webproperties`

当前账号可访问的全部网站资源列表。

```python
account.webproperties  # List[WebProperty]，资源列表
```

**返回值：** `WebProperty` 对象列表

**示例：**

```python
for prop in account.webproperties:
    print(f"{prop.url} - {prop.permission}")
```

### 方法

#### `serialize_credentials()`

将 OAuth2 凭据保存到文件，供后续使用。

```python
account.serialize_credentials(path)
```

**参数：**

- **`path`** (str): 保存凭据的文件路径

**返回值：** 序列化操作的结果

**可能抛出：**

- **`ValueError`**: 使用服务账号时（服务账号凭据无法通过本方法序列化）

**示例：**

```python
account = searchconsole.authenticate(client_config='client_secrets.json')
account.serialize_credentials('credentials.json')
```

### 索引访问

通过整数下标或 URL 访问网站资源：

```python
# 按下标
webproperty = account[0]  # 第一个资源
webproperty = account[1]  # 第二个资源

# 按 URL 字符串
webproperty = account['https://www.example.com/']
webproperty = account['sc-domain:example.com']
```

**参数：**

- **`item`** (int or str): 整数下标或精确 URL 字符串

**返回值：** `WebProperty` 对象

**可能抛出：**

- **`IndexError`**: 整数下标越界
- **`KeyError`**: URL 不在可访问资源列表中

---

## 类：`WebProperty`

表示 Google Search Console 中的某一网站资源。

!!! note
    `WebProperty` 通过 `Account` 的索引访问获得，请勿直接构造。

### 属性

#### `account`

父级 `Account` 对象。

```python
webproperty.account  # Account 实例
```

#### `url`

资源 URL。

```python
webproperty.url  # str 类型
```

**示例：**

- `"https://www.example.com/"`
- `"sc-domain:example.com"`（网域资源）
- `"android-app://com.example.app/"`（移动应用）

#### `permission`

你对该资源的权限级别。

```python
webproperty.permission  # str 类型
```

**可能取值：**

- `"siteOwner"` — 所有者
- `"siteFullUser"` — 完全用户
- `"siteRestrictedUser"` — 受限用户
- `"siteUnverifiedUser"` — 未验证用户

#### `query`

该资源的查询构造器，是所有 Search Analytics 查询的起点。

```python
webproperty.query  # Query 实例
```

**示例：**

```python
report = webproperty.query.range('today', days=-7).dimension('query').get()
```

#### `raw`

该资源对应的原始 API 响应数据。

```python
webproperty.raw  # dict 类型
```

---

## 类：`Query`

用于 Search Analytics 数据的流式、不可变查询构造器。

!!! info "不可变设计"
    所有查询方法都返回**新的** `Query` 对象，而不会修改原对象，便于安全复用查询片段：
    
    ```python
    base = webproperty.query.range('today', days=-7)
    queries_report = base.dimension('query').get()
    pages_report = base.dimension('page').get()
    # base 未被修改
    ```

### 查询构造方法

#### `range()`

定义查询的日期范围。

```python
query.range(start=None, stop=None, months=0, days=0)
```

**参数：**

- **`start`** (str or datetime.date, optional): 起始日期
    - 日期字符串：`"2024-01-01"`、`"2024/01/01"`
    - 特殊字符串：`"today"`、`"yesterday"`
    - `datetime.date` 对象
    - 默认：昨天

- **`stop`** (str or datetime.date, optional): 结束日期
    - 格式与 `start` 相同
    - 默认：与 `start` 相同（单日）

- **`months`** (int, default=0): 相对起始日期的月份偏移
    - 正数：未来
    - 负数：过去
    - 不能与同时指定 `start` 与 `stop` 组合使用

- **`days`** (int, default=0): 相对起始日期的天数偏移
    - 正数：未来
    - 负数：过去
    - 不能与同时指定 `start` 与 `stop` 组合使用

**返回值：** 新的 `Query` 对象

**示例：**

```python
# 最近 7 天
query.range('today', days=-7)

# 最近 30 天
query.range('today', days=-30)

# 指定日期区间
query.range('2024-01-01', '2024-01-31')

# 单日
query.range('yesterday')
query.range('2024-01-15')

# 最近 3 个月
query.range('today', months=-3)

# 使用 datetime 对象
import datetime
query.range(
    start=datetime.date(2024, 1, 1),
    stop=datetime.date(2024, 1, 31)
)

# 从指定日期起向后 28 天
query.range('2024-01-01', days=28)
```

!!! warning "数据保留期限"
    API 仅保留约 16 个月数据；超出该范围的查询将没有数据返回。

---

#### `dimension()`

指定报告中包含的维度。

```python
query.dimension(*dimensions)
```

**参数：**

- **`*dimensions`** (str): 一个或多个维度名称

**可用维度：**

| Dimension | 说明 | 示例值 |
|-----------|------|--------|
| `query` | 搜索查询字符串 | "python tutorial", "learn python" |
| `page` | 着陆页 URL | "https://example.com/blog/post" |
| `date` | 搜索日期 | "2024-01-15" |
| `country` | 国家/地区代码（ISO 3166-1 alpha-2） | "usa", "gbr", "fra" |
| `device` | 设备类型 | "desktop", "mobile", "tablet" |
| `searchAppearance` | 结果展示形态 | "RICH_RESULT", "AMP_BLUE_LINK" |

**返回值：** 新的 `Query` 对象

**示例：**

```python
# 单个维度
query.dimension('query')
query.dimension('page')

# 多个维度
query.dimension('query', 'page')
query.dimension('query', 'date')
query.dimension('page', 'country', 'device')

# 全部维度
query.dimension('query', 'page', 'date', 'country', 'device', 'searchAppearance')
```

!!! info "维度基数"
    维度越多，行数通常越多。使用 `dimension('query', 'date')` 会比仅 `dimension('query')` 产生更多行，因为同一查询会按日期拆分。

---

#### `filter()`

按维度取值过滤结果。

```python
query.filter(dimension, expression, operator="equals", group_type="and")
```

**参数：**

- **`dimension`** (str): 要过滤的维度
    - 取值为：`query`、`page`、`date`、`country`、`device`、`searchAppearance` 之一

- **`expression`** (str): 过滤所用的值
    - `equals` / `notEquals` 使用精确字符串
    - `contains` / `notContains` 使用子串
    - `includingRegex` / `excludingRegex` 使用正则模式

- **`operator`** (str, default="equals"): 过滤运算符

**过滤运算符：**

| Operator | 说明 | 示例 |
|----------|------|------|
| `equals` | 精确匹配 | `filter('country', 'usa', 'equals')` |
| `contains` | 包含子串 | `filter('query', 'python', 'contains')` |
| `notEquals` | 不等于 | `filter('device', 'tablet', 'notEquals')` |
| `notContains` | 不包含 | `filter('page', '/admin/', 'notContains')` |
| `includingRegex` | 匹配正则 | `filter('page', r'/blog/\d+/', 'includingRegex')` |
| `excludingRegex` | 不匹配 | `filter('query', r'^test', 'excludingRegex')` |

- **`group_type`** (str, default="and"): 多个过滤条件的组合方式
    - 目前 API 仅支持 `"and"`

**返回值：** 新的 `Query` 对象

**示例：**

```python
# contains 过滤
query.filter('query', 'python', 'contains')

# 页面过滤
query.filter('page', '/blog/', 'contains')
query.filter('page', 'https://www.example.com/about/', 'equals')

# 国家/地区过滤
query.filter('country', 'usa', 'equals')

# 设备过滤
query.filter('device', 'mobile', 'equals')

# 正则过滤（RE2 语法）
query.filter('page', r'/blog/\d{4}/\d{2}/', 'includingRegex')
query.filter('query', r'^(buy|purchase|order)', 'includingRegex')

# 排除类过滤
query.filter('page', '/admin/', 'notContains')
query.filter('query', 'test', 'notContains')

# 多个过滤（链式调用）
(
    query
    .filter('query', 'python', 'contains')
    .filter('page', '/blog/', 'contains')
    .filter('country', 'usa', 'equals')
    .filter('device', 'mobile', 'equals')
)
```

!!! note "正则语法"
    正则模式须遵循 [RE2 语法](https://github.com/google/re2/wiki/Syntax)，与 Python `re` 模块相近但不完全相同。

---

#### `search_type()`

按搜索类型过滤。

```python
query.search_type(search_type)
```

**参数：**

- **`search_type`** (str): 搜索类型

**可用搜索类型：**

| Type | 说明 | 是否含排名数据 |
|------|------|----------------|
| `web` | 常规网页搜索（默认） | ✅ 有 |
| `image` | 图片搜索 | ✅ 有 |
| `video` | 视频搜索 | ✅ 有 |
| `news` | 新闻搜索 | ✅ 有 |
| `discover` | Google Discover | ❌ 无 |
| `googleNews` | Google 新闻 | ❌ 无 |

**返回值：** 新的 `Query` 对象

**示例：**

```python
# 图片搜索
query.search_type('image')

# 视频搜索
query.search_type('video')

# Google Discover（无排名数据）
query.search_type('discover')

# 新闻搜索
query.search_type('news')
```

!!! warning "排名指标"
    `discover` 与 `googleNews` **不**返回排名数据，报告中将不提供 `position` 指标。

---

#### `data_state()`

在结果中包含新鲜（尚未最终确定）的数据。

```python
query.data_state(data_state)
```

**参数：**

- **`data_state`** (str): 数据新鲜度级别
    - `"final"`（默认）：仅最终数据
    - `"all"`：最终数据与新鲜数据（约 1 天内）均包含

**返回值：** 新的 `Query` 对象

**示例：**

```python
# 包含新鲜数据
query.data_state('all')

# 仅最终数据（默认）
query.data_state('final')
```

!!! info "数据新鲜度"
    新鲜数据可能在数日后被最终数据替换。需要尽可能新数据时使用 `"all"`。

---

#### `limit()`

限制返回的行数。

```python
query.limit(limit)
query.limit(limit, start)
```

**参数：**

- **`limit`** (int): 返回的最大行数
- **`start`** (int, optional): 起始位置（从 0 开始）

**返回值：** 新的 `Query` 对象

**示例：**

```python
# 前 10 行
query.limit(10)

# 前 100 行
query.limit(100)

# 第 100–199 行（分页）
query.limit(100, 100)

# 第 200–299 行
query.limit(100, 200)

# 大 limit（自动分页多次请求）
query.limit(50000)  # 将发起多次 API 调用
```

!!! note "API 行数上限"
    单次请求最多返回 25,000 行；若 `limit` 更大，本库会自动分页请求。

---

### 查询执行方法

#### `get()`

执行查询并返回全部结果。

```python
query.get()
```

**返回值：** 包含全部匹配行的 `Report` 对象

**示例：**

```python
# 构造并执行
report = (
    webproperty.query
    .range('today', days=-7)
    .dimension('query')
    .get()
)

# 多步链式调用
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('query', 'page')
    .filter('query', 'python', 'contains')
    .limit(1000)
    .get()
)
```

!!! info "自动分页"
    当结果超过单次请求上限或你设置的 `limit` 需要多页时，`get()` 会自动处理分页。

---

#### `execute()`

执行单次请求（仅一页结果）。

```python
query.execute()
```

**返回值：** 仅含单页结果的 `Report` 对象

!!! note
    多数场景应使用 `get()`，其会自动拉取所有分页。

---

### 工具方法

#### `build()`

构造查询参数但不发起请求。

```python
query.build(copy=True)
```

**参数：**

- **`copy`** (bool, default=True): 是否返回参数字典的副本

**返回值：** 查询参数字典

**示例：**

```python
params = (
    webproperty.query
    .range('today', days=-7)
    .dimension('query')
    .filter('page', '/blog/', 'contains')
    .build()
)

print(params)
# {
#     'startDate': '2024-11-08',
#     'endDate': '2024-11-15',
#     'dimensions': ['query'],
#     'dimensionFilterGroups': [{
#         'filters': [{
#             'dimension': 'page',
#             'expression': '/blog/',
#             'operator': 'contains'
#         }]
#     }],
#     'startRow': 0,
#     'rowLimit': 25000
# }
```

---

## 类：`Report`

表示已执行查询的结果。

!!! note
    `Report` 由 `Query.get()` 或 `Query.execute()` 生成，请勿直接实例化。

### 属性

#### `dimensions`

报告中的维度名称列表。

```python
report.dimensions  # List[str]，维度名列表
```

**示例：**

```python
report = query.dimension('query', 'page').get()
print(report.dimensions)  # ['query', 'page']
```

#### `metrics`

报告中的指标名称列表。

```python
report.metrics  # List[str]，指标名列表
```

**标准指标：**

- `clicks` (int)
- `impressions` (int)
- `ctr` (float)
- `position` (float) — discover / googleNews 不可用

**示例：**

```python
print(report.metrics)  # ['clicks', 'impressions', 'ctr', 'position']
```

#### `columns`

全部列名（维度 + 指标）。

```python
report.columns  # List[str]，列名列表
```

**示例：**

```python
print(report.columns)  # ['query', 'page', 'clicks', 'impressions', 'ctr', 'position']
```

#### `rows`

数据行列表，每行为命名元组。

```python
report.rows  # List[Row]，行数据列表
```

**示例：**

```python
for row in report.rows:
    print(row.query, row.clicks)
```

#### `Row`

行的命名元组类型，支持属性访问与字典式访问。

```python
row = report.rows[0]

# 属性访问
print(row.query)
print(row.clicks)
print(row.impressions)

# 字典式访问
print(row['query'])
print(row['clicks'])
```

#### `is_complete`

是否已拉取全部数据。

```python
report.is_complete  # bool，是否已完整拉取
```

### 属性（计算属性）

#### `first`

报告中的第一行；若无数据则为 `None`。

```python
report.first  # Row 或 None
```

**示例：**

```python
if report.first:
    print(f"Top query: {report.first.query}")
```

#### `last`

报告中的最后一行；若无数据则为 `None`。

```python
report.last  # Row 或 None
```

### 方法

#### `to_dict()`

将报告转换为字典列表。

```python
report.to_dict()
```

**返回值：** 每行一个字典的列表

**示例：**

```python
data = report.to_dict()
print(data[0])
# {
#     'query': 'python tutorial',
#     'clicks': 150,
#     'impressions': 2500,
#     'ctr': 0.06,
#     'position': 5.2
# }

# 保存为 JSON
import json
with open('data.json', 'w') as f:
    json.dump(data, f, indent=2)
```

---

#### `to_dataframe()`

将报告转换为 pandas DataFrame。

```python
report.to_dataframe()
```

**返回值：** `pandas.DataFrame`

**依赖：** 需安装 pandas

**示例：**

```python
df = report.to_dataframe()

print(df.head())
print(df.info())
print(df.describe())

# pandas 运算
print(df['clicks'].sum())
print(df.groupby('country')['clicks'].sum())

# 保存为 CSV
df.to_csv('report.csv', index=False)

# 保存为 Excel
df.to_excel('report.xlsx', index=False)
```

### 迭代与索引

#### 迭代

```python
for row in report:
    print(row.query, row.clicks)
```

#### 长度

```python
num_rows = len(report)
print(f"Report has {num_rows} rows")
```

#### 索引

```python
# 第一行
first = report[0]

# 最后一行
last = report[-1]

# 切片
first_ten = report[:10]
next_ten = report[10:20]
```

#### 成员检测

```python
if some_row in report:
    print("Row found in report")
```

---

## 工具函数

### `serialize()`

将日期对象序列化为 ISO 格式字符串。

```python
searchconsole.utils.serialize(date)
```

**参数：**

- **`date`** (datetime.date or str): 待序列化的日期

**返回值：** ISO 格式字符串（YYYY-MM-DD），或原始字符串

---

### `normalize()`

将多种日期表示规范为 `datetime.date`。

```python
searchconsole.utils.normalize(obj)
```

**参数：**

- **`obj`** (str, datetime.date, or None): 多种格式的日期

**返回值：** `datetime.date` 对象，或 `None`

**支持的格式：**

- `None` → `None`
- `datetime.date` objects → returned as-is
- `datetime.datetime` objects → converted to date
- Date strings: `"2024-01-01"`, `"2024/01/01"`, etc.
- Special strings: `"today"`, `"yesterday"`

---

### `daterange()`

根据偏移量计算日期范围。

```python
searchconsole.utils.daterange(start=None, stop=None, days=0, months=0)
```

**参数：**

- **`start`** (str, datetime.date, or None): 起始日期（默认：昨天）
- **`stop`** (str, datetime.date, or None): 结束日期（默认：与 start 相同）
- **`days`** (int): 天数偏移
- **`months`** (int): 月份偏移

**返回值：** 两个 ISO 日期字符串组成的元组 `(start, stop)`，按时间先后排序

**示例：**

```python
from searchconsole.utils import daterange

# 最近 7 天
start, end = daterange('today', days=-7)
print(start, end)  # ('2024-11-08', '2024-11-15')
```

---

## 完整示例

综合使用上述 API 的示例：

```python
import searchconsole

# 1. 认证
account = searchconsole.authenticate(
    client_config='client_secrets.json',
    credentials='credentials.json'
)

# 2. 选择资源
webproperty = account['https://www.example.com/']
print(f"Permission: {webproperty.permission}")

# 3. 构造复杂查询
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('query', 'page', 'country')
    .filter('query', 'python', 'contains')
    .filter('page', '/blog/', 'contains')
    .filter('country', 'usa', 'equals')
    .search_type('web')
    .data_state('all')
    .limit(1000)
    .get()
)

# 4. 分析结果
print(f"Total rows: {len(report)}")
print(f"Dimensions: {report.dimensions}")
print(f"Metrics: {report.metrics}")

# 5. 读取数据
for row in report[:10]:
    print(f"{row.query} on {row.page}")
    print(f"  Country: {row.country}")
    print(f"  Clicks: {row.clicks}, Impressions: {row.impressions}")
    print(f"  CTR: {row.ctr:.2%}, Position: {row.position:.1f}\n")

# 6. 导出
df = report.to_dataframe()
df.to_csv('search_console_data.csv', index=False)

data = report.to_dict()
import json
with open('search_console_data.json', 'w') as f:
    json.dump(data, f, indent=2)
```
