---
icon: lucide/home
---

# Google Search Console for Python

[![Tests](https://github.com/joshcarty/google-searchconsole/actions/workflows/tests.yml/badge.svg)](https://github.com/joshcarty/google-searchconsole/actions/workflows/tests.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**google-searchconsole** 是一个 Python 库，用于更简单、更顺手地调用 Google Search Console API。

直接对接 Google Search Console API 往往比较繁琐。本库帮助你轻松完成：

- **身份验证** — 通过 OAuth2 或服务账号访问你的 Google 账号  
- **查询** — 用直观的方式组合复杂查询  
- **浏览** — 查看账号下所有网站资源  
- **导出** — 将数据导出为 Pandas DataFrame 或 JSON，便于分享或进一步分析  

本库基于 [Google 官方 API 客户端](https://developers.google.com/webmaster-tools)，设计思路参考了 [@debrouwere](https://github.com/debrouwere) 的优秀项目 [google-analytics](https://github.com/debrouwere/google-analytics)。

## 快速开始

请先安装本包：

```bash
pip install git+https://github.com/joshcarty/google-searchconsole
```

然后：

```python
import searchconsole

# 在浏览器中完成认证
account = searchconsole.authenticate(client_config='client_secrets.json')

# 选择网站资源
webproperty = account['https://www.example.com/']

# 查询并分析
report = webproperty.query.range('today', days=-7).dimension('query').get()

# 导出为 pandas 以便分析
df = report.to_dataframe()
print(df.head())

```

## 快速链接
<div class="grid cards" markdown>

- :material-rocket-launch: **[入门指南](getting-started.md)**

    配置凭据并运行第一次查询

- :material-key: **[身份验证指南](authentication.md)**

    了解 OAuth2 与服务账号认证

- :material-book-open-variant: **[API 参考](api-reference.md)**

    所有类与方法的完整说明

- :material-lightbulb: **[示例](examples.md)**

    实用示例与常见场景

</div>


## 主要特性

### 简单的身份验证

```python
# 交互式 OAuth2 流程
account = searchconsole.authenticate(client_config='client_secrets.json')

# 保存凭据供下次使用
account.serialize_credentials('credentials.json')

# 使用已保存凭据再次认证
account = searchconsole.authenticate(
    client_config='client_secrets.json',
    credentials='credentials.json'
)
```

### 流式查询 API

通过方法链构建复杂查询：

```python
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('query', 'page')
    .filter('query', 'python', 'contains')
    .filter('page', '/blog/', 'contains')
    .limit(1000)
    .get()
)
```

### 多种搜索类型

除常规网页搜索外，还可查询其他搜索类型：

```python
# 图片搜索
image_data = webproperty.query.search_type('image').range('today', days=-7).get()

# Google Discover
discover_data = webproperty.query.search_type('discover').range('today', days=-7).get()

# 视频、新闻等
video_data = webproperty.query.search_type('video').range('today', days=-7).get()
```

### 过滤

为查询应用更细粒度的过滤条件：

```python
# 包含（contains）过滤
report = query.filter('query', 'python', 'contains').get()

# 正则过滤（RE2 语法）
report = query.filter('page', r'/blog/\d{4}/\d{2}/', 'includingRegex').get()

# 多个过滤条件
report = (
    query
    .filter('country', 'usa', 'equals')
    .filter('device', 'mobile', 'equals')
    .get()
)
```

### 便捷的数据导出

```python
# 导出为 pandas DataFrame
df = report.to_dataframe()
df.to_csv('search_data.csv', index=False)
```

## 入门步骤

1. **[配置 Google API 凭据](getting-started.md#setup-credentials)** — 在 Google Developers Console 中创建项目  
2. **[身份验证](authentication.md)** — 选择 OAuth2 或服务账号  
3. **[构建第一次查询](getting-started.md#your-first-query)** — 开始探索你的数据  
4. **[查看示例](examples.md)** — 从实际用例中学习  

## 可用维度（Dimensions）

可在多个维度上查询数据：

- **`query`** — 触发你网页展示的搜索查询  
- **`page`** — 着陆页 URL  
- **`date`** — 搜索发生的日期  
- **`country`** — 国家/地区代码（ISO 3166-1 alpha-2）  
- **`device`** — 设备类型（desktop、mobile、tablet）  
- **`searchAppearance`** — 在搜索结果中的展示形态  

## 可用指标（Metrics）

所有查询都会返回以下指标：

- **`clicks`** — 来自搜索结果的点击次数  
- **`impressions`** — 页面在搜索中出现的次数  
- **`ctr`** — 点击率（点击 ÷ 展示）  
- **`position`** — 搜索结果中的平均排名（Discover / Google 新闻不提供）  

## 支持的搜索类型

- `web` — 常规网页搜索（默认）  
- `image` — 图片搜索  
- `video` — 视频搜索  
- `news` — 新闻搜索  
- `discover` — Google Discover  
- `googleNews` — Google 新闻  

## 许可证

本项目采用 MIT 许可证。详情请参阅 [LICENSE](https://github.com/joshcarty/google-searchconsole/blob/master/LICENSE)。

## 参与贡献

欢迎贡献代码与文档！请参阅[贡献指南](contributing.md)。

## 支持

- :fontawesome-brands-github: [报告问题](https://github.com/joshcarty/google-searchconsole/issues)  
- :fontawesome-brands-github: [查看源码](https://github.com/joshcarty/google-searchconsole)  
