---
icon: lucide/rocket
---

# 入门指南

本文帮助你在几分钟内上手 Google Search Console Python 库。

## 环境要求

- Python 3.8 或更高版本  
- 可访问 Google Search Console 的 Google 账号  
- 在 Search Console 中至少有一个已验证的网站资源  

## 安装

可直接从 GitHub 安装：

```bash
pip install git+https://github.com/joshcarty/google-searchconsole
```

## 配置 Google API 凭据 {#setup-credentials}

使用本库前，需要在 Google Developers Console 中创建 API 凭据。

### 步骤 1：创建 Google Cloud 项目

1. 打开 [Google Developers Console](https://console.developers.google.com/)  
2. 点击 **Create Project**  
3. 输入项目名称（例如「Search Console API」）  
4. 点击 **Create**  

### 步骤 2：启用 Search Console API

1. 在项目中进入 **APIs & Services** > **Library**  
2. 搜索「Google Search Console API」  
3. 打开后点击 **Enable**  

### 步骤 3：创建 OAuth2 凭据

1. 进入 **APIs & Services** > **Credentials**  
2. 点击 **Create Credentials** > **OAuth client ID**  
3. 若提示配置 OAuth 同意屏幕：  
    - 用户类型一般选 **External**（除非使用 Google Workspace 且策略允许）  
    - 填写必填项（应用名、用户支持邮箱、开发者邮箱等）  
    - 将你的邮箱添加为测试用户  
    - 在作用域与摘要步骤中按提示 **Save and Continue** 完成  
4. 回到 **Create OAuth client ID**：  
    - 应用类型选择 **Desktop app**  
    - 填写名称（例如「Search Console Python Client」）  
    - 点击 **Create**  
5. 点击下载图标下载 JSON  
6. 将文件保存为项目目录下的 `client_secrets.json`  

!!! warning "妥善保管凭据"
    切勿将 `client_secrets.json` 或 `credentials.json` 提交到版本库，请加入 `.gitignore`。

## 第一次查询 {#your-first-query}

完成凭据配置后即可认证并执行查询。

### 步骤 1：认证

创建 Python 文件并写入：

```python
import searchconsole

# 首次认证（交互式）
account = searchconsole.authenticate(client_config='client_secrets.json')
```

运行时会：

1. 打开浏览器窗口供你登录  
2. 请求你为应用授予权限  
3. 返回已认证的 `Account` 对象  

!!! tip "保存凭据"
    为避免每次运行都走 OAuth 流程，可保存凭据：  
    
    ```python
    # 首次：认证并保存
    account = searchconsole.authenticate(client_config='client_secrets.json')
    account.serialize_credentials('credentials.json')
    
    # 之后运行：加载已保存凭据
    account = searchconsole.authenticate(client_config='client_secrets.json', credentials='credentials.json')
    ```

### 步骤 2：选择网站资源

列出可用资源：

```python
# 查看全部资源
for property in account.webproperties:
    print(f"URL: {property.url}, Permission: {property.permission}")
```

选择要分析的资源：

```python
# 按 URL 选择
webproperty = account['https://www.example.com/']

# 或按下标选择
webproperty = account[0]  # 第一个资源
```

### 步骤 3：构建并执行查询

查询最近 7 天的搜索查询：

```python
# 构造查询
query = webproperty.query.range('today', days=-7).dimension('query')

# 执行并获取结果
report = query.get()

# 打印结果
print(f"Found {len(report)} rows")

for row in report:
    print(f"Query: {row.query}")
    print(f"  Clicks: {row.clicks}")
    print(f"  Impressions: {row.impressions}")
    print(f"  CTR: {row.ctr:.2%}")
    print(f"  Position: {row.position:.1f}")
    print()
```

### 完整示例

可运行的完整示例：

```python
import searchconsole

# 认证
account = searchconsole.authenticate(client_config='client_secrets.json', credentials='credentials.json')

# 选择资源
webproperty = account['https://www.example.com/']

# 查询最近 7 天点击量最高的查询词
report = (
    webproperty.query
    .range('today', days=-7)
    .dimension('query')
    .limit(10)
    .get()
)

# 展示结果
print(f"Top 10 queries from the last 7 days:\n")
for i, row in enumerate(report, 1):
    print(f"{i}. {row.query}")
    print(f"   Clicks: {row.clicks}, Impressions: {row.impressions}, "
          f"CTR: {row.ctr:.2%}, Avg Position: {row.position:.1f}\n")
```

预期输出示例：

```
Top 10 queries from the last 7 days:

1. python tutorial
   Clicks: 150, Impressions: 2500, CTR: 6.00%, Avg Position: 5.2

2. learn python
   Clicks: 120, Impressions: 1800, CTR: 6.67%, Avg Position: 4.8

3. python for beginners
   Clicks: 95, Impressions: 1200, CTR: 7.92%, Avg Position: 3.5
...
```

## 导出数据

### 导出为 Pandas DataFrame

```python
# 导出到 pandas（需已安装 pandas）
df = report.to_dataframe()

print(df.head())
print(f"\nTotal clicks: {df['clicks'].sum()}")
print(f"Total impressions: {df['impressions'].sum()}")

# 保存为 CSV
df.to_csv('search_data.csv', index=False)
```

## 常见查询模式

### 日期范围

```python
# 最近 7 天
query = webproperty.query.range('today', days=-7)

# 最近 30 天
query = webproperty.query.range('today', days=-30)

# 指定日期区间
query = webproperty.query.range('2024-01-01', '2024-01-31')

# 仅昨天
query = webproperty.query.range('yesterday')

# 最近 3 个月
query = webproperty.query.range('today', months=-3)
```

### 多维度

```python
# 按页面拆分的查询词
report = (
    webproperty.query
    .range('today', days=-7)
    .dimension('query', 'page')
    .get()
)

# 按日期拆分的查询词
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('query', 'date')
    .get()
)

# 按国家与设备拆分的页面
report = (
    webproperty.query
    .range('today', days=-7)
    .dimension('page', 'country', 'device')
    .get()
)
```

### 过滤

```python
# 查询词包含 "python"
report = (
    webproperty.query
    .range('today', days=-7)
    .dimension('query')
    .filter('query', 'python', 'contains')
    .get()
)

# 指定页面 URL
report = (
    webproperty.query
    .range('today', days=-7)
    .dimension('query')
    .filter('page', 'https://www.example.com/blog/', 'equals')
    .get()
)

# 多个过滤条件
report = (
    webproperty.query
    .range('today', days=-7)
    .dimension('query')
    .filter('page', '/blog/', 'contains')
    .filter('country', 'usa', 'equals')
    .get()
)
```

## 故障排除

### OAuth 流程中浏览器未打开

若浏览器未打开或出现错误：

```
OSError: [Errno 98] Address already in use
```

可改用控制台流程：

```python
account = searchconsole.authenticate(client_config='client_secrets.json', flow='console')
```

控制台会打印可复制的 URL。

### 没有返回数据

请检查：

- 日期范围是否有效（API 有约 16 个月的数据保留限制）  
- 过滤条件是否过严  
- 该资源在所选时间段内是否有数据  
- 是否使用了正确的 `search_type`（web、image 等）  

## 下一步

掌握基础后可以继续阅读：

- **[身份验证指南](authentication.md)** — 不同认证方式详解  
- **[API 参考](api-reference.md)** — 所有方法的详细文档  
- **[示例](examples.md)** — 更复杂的查询与场景  

## 速查表

### 常用维度

| Dimension | Description | Example Values |
|-----------|-------------|----------------|
| `query` | Search query | "python tutorial", "learn coding" |
| `page` | Landing page URL | "https://example.com/blog/post" |
| `date` | Date of search | "2024-01-15" |
| `country` | Country code | "usa", "gbr", "can" |
| `device` | Device type | "desktop", "mobile", "tablet" |
| `searchAppearance` | Search feature | "RICH_RESULT", "AMP_BLUE_LINK" |

### 常用过滤运算符

| Operator | Description | Example |
|----------|-------------|---------|
| `equals` | Exact match | `.filter('country', 'usa', 'equals')` |
| `contains` | Contains substring | `.filter('query', 'python', 'contains')` |
| `notEquals` | Not equal to | `.filter('device', 'tablet', 'notEquals')` |
| `notContains` | Doesn't contain | `.filter('page', '/admin/', 'notContains')` |
| `includingRegex` | Matches regex | `.filter('page', r'/blog/\d+/', 'includingRegex')` |
| `excludingRegex` | Doesn't match regex | `.filter('query', r'^test', 'excludingRegex')` |

### 搜索类型

| Type | Description |
|------|-------------|
| `web` | Regular web search (default) |
| `image` | Image search |
| `video` | Video search |
| `news` | News search |
| `discover` | Google Discover (no position data) |
| `googleNews` | Google News (no position data) |
