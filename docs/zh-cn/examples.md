---
icon: lucide/lightbulb
---

# 示例

Google Search Console 库的实用示例与常见用法。

## 基础示例

### 表现最好的查询

获取最近 30 天内点击量最高的 10 个查询：

```python
import searchconsole

account = searchconsole.authenticate(
    client_config='client_secrets.json',
    credentials='credentials.json'
)

webproperty = account['https://www.example.com/']

# 查询热门搜索词
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('query')
    .limit(10)
    .get()
)

print("Top 10 Queries (Last 30 Days)\n" + "="*50)
for i, row in enumerate(report, 1):
    print(f"{i:2d}. {row.query}")
    print(f"    Clicks: {row.clicks:>6,} | Impressions: {row.impressions:>8,}")
    print(f"    CTR: {row.ctr:>6.2%} | Avg Position: {row.position:>5.1f}\n")
```

### 按流量排序的热门页面

查看哪些页面带来最多流量：

```python
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('page')
    .limit(20)
    .get()
)

# 转为 DataFrame 便于分析
df = report.to_dataframe()

# 按点击量排序
top_pages = df.nlargest(10, 'clicks')

print("Top 10 Pages by Clicks\n")
for idx, row in top_pages.iterrows():
    print(f"{row['page']}")
    print(f"  {row['clicks']:,} clicks, {row['impressions']:,} impressions\n")
```

### 跟踪特定关键词

监控若干关键词随时间的表现：

```python
keywords = ['python tutorial', 'learn python', 'python for beginners']

for keyword in keywords:
    report = (
        webproperty.query
        .range('today', days=-90)
        .dimension('date')
        .filter('query', keyword, 'equals')
        .get()
    )
    
    df = report.to_dataframe()
    
    print(f"\n{keyword.upper()}")
    print(f"Total clicks: {df['clicks'].sum():,}")
    print(f"Total impressions: {df['impressions'].sum():,}")
    print(f"Average position: {df['position'].mean():.1f}")
```

## 分析类示例

### 点击率（CTR）分析

按排名区间分析 CTR：

```python
import pandas as pd

# 获取查询粒度数据
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('query')
    .get()
)

df = report.to_dataframe()

# 按排名分段
df['position_range'] = pd.cut(
    df['position'],
    bins=[0, 3, 5, 10, 20, 100],
    labels=['1-3', '4-5', '6-10', '11-20', '20+']
)

# 按排名区间汇总
analysis = df.groupby('position_range').agg({
    'clicks': 'sum',
    'impressions': 'sum',
    'ctr': 'mean',
    'position': 'mean'
}).round(4)

print(analysis)
```

### 地理维度表现

对比不同国家/地区的表现：

```python
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('country')
    .get()
)

df = report.to_dataframe()

# 按点击量排序
df = df.sort_values('clicks', ascending=False)

print("Performance by Country\n")
print(df.head(10).to_string(index=False))

# 占总量百分比
df['click_share'] = (df['clicks'] / df['clicks'].sum() * 100).round(2)

top_5 = df.head(5)
print(f"\nTop 5 Countries Account for {top_5['click_share'].sum():.1f}% of Clicks")
```

### 设备分布

按设备类型分析流量：

```python
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('device')
    .get()
)

df = report.to_dataframe()

print("Performance by Device\n")
for _, row in df.iterrows():
    print(f"{row['device'].upper()}")
    print(f"  Clicks: {row['clicks']:,}")
    print(f"  Impressions: {row['impressions']:,}")
    print(f"  CTR: {row['ctr']:.2%}")
    print(f"  Avg Position: {row['position']:.1f}\n")

# 各设备点击占比
df['device_share'] = (df['clicks'] / df['clicks'].sum() * 100)

print("Click Share by Device:")
for _, row in df.iterrows():
    print(f"  {row['device']}: {row['device_share']:.1f}%")
```

### 趋势分析

跟踪指标随时间的变化：

```python
import matplotlib.pyplot as plt

# 最近 90 天的按日数据
report = (
    webproperty.query
    .range('today', days=-90)
    .dimension('date')
    .get()
)

df = report.to_dataframe()
df['date'] = pd.to_datetime(df['date'])
df = df.sort_values('date')

# 绘制趋势图
fig, axes = plt.subplots(2, 2, figsize=(15, 10))

# 点击趋势
axes[0, 0].plot(df['date'], df['clicks'], marker='o')
axes[0, 0].set_title('Clicks Over Time')
axes[0, 0].set_xlabel('Date')
axes[0, 0].set_ylabel('Clicks')

# 展示趋势
axes[0, 1].plot(df['date'], df['impressions'], marker='o', color='orange')
axes[0, 1].set_title('Impressions Over Time')
axes[0, 1].set_xlabel('Date')
axes[0, 1].set_ylabel('Impressions')

# CTR 趋势
axes[1, 0].plot(df['date'], df['ctr'] * 100, marker='o', color='green')
axes[1, 0].set_title('CTR Over Time')
axes[1, 0].set_xlabel('Date')
axes[1, 0].set_ylabel('CTR (%)')

# 平均排名趋势
axes[1, 1].plot(df['date'], df['position'], marker='o', color='red')
axes[1, 1].set_title('Average Position Over Time')
axes[1, 1].set_xlabel('Date')
axes[1, 1].set_ylabel('Position')
axes[1, 1].invert_yaxis()  # 排名数值越小越好，倒置 Y 轴

plt.tight_layout()
plt.savefig('search_console_trends.png')
print("Trend chart saved to search_console_trends.png")
```

## 内容分析示例

### 博客文章表现

分析所有博客路径下的页面：

```python
# 路径包含 '/blog/' 的页面
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('page')
    .filter('page', '/blog/', 'contains')
    .get()
)

df = report.to_dataframe()

print(f"Total blog posts: {len(df)}")
print(f"Total blog clicks: {df['clicks'].sum():,}")
print(f"Total blog impressions: {df['impressions'].sum():,}")

# 表现最好的博文
top_blogs = df.nlargest(10, 'clicks')

print("\nTop 10 Blog Posts:\n")
for idx, row in top_blogs.iterrows():
    # 从 URL 提取标题（可按需自定义）
    title = row['page'].split('/')[-2].replace('-', ' ').title()
    print(f"{title}")
    print(f"  URL: {row['page']}")
    print(f"  Clicks: {row['clicks']:,} | Impressions: {row['impressions']:,}")
    print(f"  CTR: {row['ctr']:.2%} | Position: {row['position']:.1f}\n")
```

### 找出表现偏弱的页面

识别展示量高但点击偏少的页面：

```python
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('page')
    .get()
)

df = report.to_dataframe()

# 筛选展示量较高的页面
significant_pages = df[df['impressions'] >= 100].copy()

# 按 CTR 升序，找出偏低页面
underperformers = significant_pages.nsmallest(10, 'ctr')

print("Top 10 Underperforming Pages (High Impressions, Low CTR)\n")
for idx, row in underperformers.iterrows():
    print(f"{row['page']}")
    print(f"  Impressions: {row['impressions']:,} | Clicks: {row['clicks']}")
    print(f"  CTR: {row['ctr']:.2%} | Position: {row['position']:.1f}")
    print(f"  💡 Opportunity: Improve meta description or title tag\n")
```

### 内容缺口分析

查找有排名但点击偏低的查询：

```python
# 排名前 10 但 CTR 偏低的查询
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('query')
    .get()
)

df = report.to_dataframe()

# 排名前 10 且展示不少于 50
top_10 = df[(df['position'] <= 10) & (df['impressions'] >= 50)].copy()

# 按 CTR 排序找优化机会
opportunities = top_10.nsmallest(20, 'ctr')

print("Ranking Opportunities (Top 10 Position, Low CTR)\n")
for idx, row in opportunities.iterrows():
    print(f"Query: {row['query']}")
    print(f"  Position: {row['position']:.1f} | CTR: {row['ctr']:.2%}")
    print(f"  Clicks: {row['clicks']} | Impressions: {row['impressions']}")
    print(f"  💡 Consider: Optimizing for featured snippets or improving title\n")
```

## 对比类示例

### 环比（月对月）

对比本月与上月：

```python
import datetime

# 本月数据
this_month_start = datetime.date.today().replace(day=1)
this_month_report = (
    webproperty.query
    .range(this_month_start, 'today')
    .dimension('query')
    .get()
)

# 上月数据
last_month_end = this_month_start - datetime.timedelta(days=1)
last_month_start = last_month_end.replace(day=1)
last_month_report = (
    webproperty.query
    .range(last_month_start, last_month_end)
    .dimension('query')
    .get()
)

# 转为 DataFrame
this_month_df = this_month_report.to_dataframe()
last_month_df = last_month_report.to_dataframe()

# 对比汇总指标
this_month_clicks = this_month_df['clicks'].sum()
last_month_clicks = last_month_df['clicks'].sum()
change = ((this_month_clicks - last_month_clicks) / last_month_clicks * 100)

print(f"Month-over-Month Performance\n{'='*50}")
print(f"\nThis Month:")
print(f"  Clicks: {this_month_clicks:,}")
print(f"  Impressions: {this_month_df['impressions'].sum():,}")

print(f"\nLast Month:")
print(f"  Clicks: {last_month_clicks:,}")
print(f"  Impressions: {last_month_df['impressions'].sum():,}")

print(f"\nChange: {change:+.1f}%")
```

### 同比（年对年）

与去年同期对比：

```python
import datetime

# 今年（最近 30 天）
this_year_report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('date')
    .get()
)

# 去年同期（同样 30 天）
days_ago_365 = datetime.date.today() - datetime.timedelta(days=365)
days_ago_395 = days_ago_365 - datetime.timedelta(days=30)

last_year_report = (
    webproperty.query
    .range(days_ago_395, days_ago_365)
    .dimension('date')
    .get()
)

# 对比
this_year_df = this_year_report.to_dataframe()
last_year_df = last_year_report.to_dataframe()

metrics = {
    'Clicks': ('clicks', this_year_df['clicks'].sum(), last_year_df['clicks'].sum()),
    'Impressions': ('impressions', this_year_df['impressions'].sum(), last_year_df['impressions'].sum()),
    'CTR': ('ctr', this_year_df['ctr'].mean(), last_year_df['ctr'].mean()),
    'Position': ('position', this_year_df['position'].mean(), last_year_df['position'].mean()),
}

print("Year-over-Year Comparison (Last 30 Days)\n")
for metric_name, (col, this_year, last_year) in metrics.items():
    change = ((this_year - last_year) / last_year * 100) if last_year > 0 else 0
    print(f"{metric_name}:")
    print(f"  This Year: {this_year:,.2f}")
    print(f"  Last Year: {last_year:,.2f}")
    print(f"  Change: {change:+.1f}%\n")
```

## 搜索类型示例

### 图片搜索分析

分析图片搜索表现：

```python
# 图片搜索数据
image_report = (
    webproperty.query
    .search_type('image')
    .range('today', days=-30)
    .dimension('page')
    .get()
)

df = image_report.to_dataframe()

print("Image Search Performance\n")
print(f"Total clicks: {df['clicks'].sum():,}")
print(f"Total impressions: {df['impressions'].sum():,}")
print(f"Average CTR: {df['ctr'].mean():.2%}")

# 图片搜索下表现最好的页面
top_images = df.nlargest(10, 'clicks')
print("\nTop 10 Pages for Image Search:\n")
for idx, row in top_images.iterrows():
    print(f"{row['page']}")
    print(f"  Clicks: {row['clicks']:,} | Impressions: {row['impressions']:,}\n")
```

### 对比多种搜索类型

对比不同搜索类型的表现：

```python
search_types = ['web', 'image', 'video', 'news']

results = []
for search_type in search_types:
    try:
        report = (
            webproperty.query
            .search_type(search_type)
            .range('today', days=-30)
            .dimension('page')
            .get()
        )
        
        df = report.to_dataframe()
        
        results.append({
            'search_type': search_type,
            'clicks': df['clicks'].sum(),
            'impressions': df['impressions'].sum(),
            'ctr': df['ctr'].mean()
        })
    except Exception as e:
        print(f"No data for {search_type}: {e}")

# 打印对比表
import pandas as pd
comparison_df = pd.DataFrame(results)

print("Search Type Comparison\n")
print(comparison_df.to_string(index=False))

# 可视化
comparison_df.plot(
    x='search_type',
    y=['clicks', 'impressions'],
    kind='bar',
    title='Clicks and Impressions by Search Type'
)
```

### Google Discover 表现

分析 Google Discover 流量：

```python
# 说明：Discover 不提供排名数据
discover_report = (
    webproperty.query
    .search_type('discover')
    .range('today', days=-30)
    .dimension('page')
    .get()
)

df = discover_report.to_dataframe()

print("Google Discover Performance (Last 30 Days)\n")
print(f"Total clicks: {df['clicks'].sum():,}")
print(f"Total impressions: {df['impressions'].sum():,}")
print(f"Average CTR: {df['ctr'].mean():.2%}")

# Discover 中表现最好的页面
top_discover = df.nlargest(10, 'clicks')
print("\nTop 10 Pages in Discover:\n")
for idx, row in top_discover.iterrows():
    print(f"{row['page']}")
    print(f"  Clicks: {row['clicks']:,} | CTR: {row['ctr']:.2%}\n")
```

## 高级过滤示例

### 多条件过滤分析

分析美国移动端访问博客的流量：

```python
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('query', 'page')
    .filter('page', '/blog/', 'contains')
    .filter('country', 'usa', 'equals')
    .filter('device', 'mobile', 'equals')
    .get()
)

df = report.to_dataframe()

print("Mobile Traffic (USA) for Blog Posts\n")
print(f"Total clicks: {df['clicks'].sum():,}")
print(f"Unique queries: {df['query'].nunique()}")
print(f"Unique pages: {df['page'].nunique()}")

# 该细分下热门查询
top_queries = df.groupby('query').agg({
    'clicks': 'sum',
    'impressions': 'sum'
}).nlargest(10, 'clicks')

print("\nTop 10 Mobile Queries (USA, Blog):\n")
print(top_queries)
```

### 正则过滤

查找符合日期路径模式的 URL：

```python
# 匹配 /blog/年/月/ 形式的 URL
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('page')
    .filter('page', r'/blog/\d{4}/\d{2}/', 'includingRegex')
    .get()
)

df = report.to_dataframe()

print(f"Found {len(df)} blog posts with date-based URLs\n")

# 提取年、月
import re

def extract_date(url):
    match = re.search(r'/blog/(\d{4})/(\d{2})/', url)
    if match:
        return f"{match.group(1)}-{match.group(2)}"
    return None

df['year_month'] = df['page'].apply(extract_date)

# 按年月汇总
monthly_performance = df.groupby('year_month').agg({
    'clicks': 'sum',
    'impressions': 'sum',
    'page': 'count'
}).rename(columns={'page': 'num_posts'})

print("Performance by Publication Month:\n")
print(monthly_performance.sort_index(ascending=False))
```

### 排除内部页面

过滤掉后台、预发或测试路径：

```python
# 排除后台、预发、测试路径后的页面
report = (
    webproperty.query
    .range('today', days=-30)
    .dimension('page')
    .filter('page', '/admin/', 'notContains')
    .filter('page', 'staging.', 'notContains')
    .filter('page', '/test/', 'notContains')
    .get()
)

df = report.to_dataframe()

print(f"Public pages: {len(df)}")
print(f"Total clicks: {df['clicks'].sum():,}")
```

## 下一步

- **[API 参考](api-reference.md)** — 完整 API 文档  
- **[贡献指南](contributing.md)** — 参与改进本库  
