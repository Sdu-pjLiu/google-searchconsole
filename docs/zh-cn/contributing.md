---
icon: lucide/users
---

# 参与贡献

感谢你有意为本 Google Search Console Python 库做贡献！本文说明如何开始。

## 贡献方式

你可以通过多种方式参与：

- **报告缺陷** — 发现问题请告诉我们  
- **功能建议** — 有改进想法欢迎提出  
- **分享示例** — 展示你如何使用本库  
- **完善文档** — 帮助文档更清晰、更全面  
- **提交代码** — 修复缺陷或实现新功能  

## 开始之前

### 1. Fork 与克隆

1. 在 GitHub 上 Fork 本仓库  
2. 克隆你的 Fork：  

```bash
git clone https://github.com/YOUR_USERNAME/google-searchconsole.git
cd google-searchconsole
```

### 2. 搭建开发环境

推荐使用 [uv](https://docs.astral.sh/uv/) 管理依赖：

```bash
# 安装依赖
uv sync

# 或使用 pip
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
```

### 3. 创建分支

为你的工作创建分支：

```bash
git checkout -b feature/your-feature-name
# 或
git checkout -b fix/your-bug-fix
```

建议使用清晰的分支名，例如：

- `feature/add-bulk-query-support`  
- `fix/pagination-edge-case`  
- `docs/improve-authentication-guide`  

## 开发流程

### 运行测试

项目使用 Python 标准库中的 `unittest`。

#### 前置条件

测试需要将认证信息配置为环境变量：

```bash
export SEARCHCONSOLE_CLIENT_CONFIG='{"installed": {...}}'
export SEARCHCONSOLE_CREDENTIALS='{"token": "...", ...}'
export SEARCHCONSOLE_WEBPROPERTY_URI='https://www.example.com/'
```

或使用 `python-dotenv` 的 `.env` 文件：

```bash
# .env
SEARCHCONSOLE_CLIENT_CONFIG={"installed": {...}}
SEARCHCONSOLE_CREDENTIALS={"token": "...", ...}
SEARCHCONSOLE_WEBPROPERTY_URI=https://www.example.com/
```

#### 执行测试

```bash
# 使用 uv
uv run python -m unittest tests

# 使用 pip
python -m unittest tests

# 使用 tox（多 Python 版本）
tox

# 运行指定测试文件
python -m unittest tests.TestQuery
```

### 代码质量

#### 静态检查与格式化

项目使用 [Ruff](https://docs.astral.sh/ruff/) 进行 lint 与格式化：

```bash
# 检查代码质量
tox -e lint

# 自动格式化
tox -e format

# 或直接运行 ruff
ruff check src/searchconsole tests.py
ruff format src/searchconsole tests.py
```

#### 代码风格

- 遵循 [PEP 8](https://pep8.org/)  

示例：

```python
def filter(dimension=expression, operator="equals", group_type="and"):
    """
    按维度取值过滤结果。

    Args:
        dimension: 要过滤的维度（query, page, date, country, device, searchAppearance）
        expression: 过滤所用的值
        operator: 过滤运算符（equals、contains、notContains 等）
        group_type: 多个过滤条件的组合方式（默认 "and"）

    Returns:
        应用过滤后的新 Query 对象

    Example:
        >>> query.filter('country', 'usa', 'equals')
        >>> query.filter('query', 'python', 'contains')
    """
    # 具体实现
    pass
```

### 测试编写指引

#### 编写测试

为新功能或缺陷修复补充测试：

```python
import unittest
from searchconsole import Query

class TestNewFeature(unittest.TestCase):
    def test_feature_works(self):
        """验证新功能按预期工作。"""
        # 准备数据
        query = Query(...)
        
        # 执行
        result = query.new_feature()
        
        # 断言
        self.assertEqual(result.something, expected_value)
    
    def test_feature_edge_case(self):
        """验证边界情况处理。"""
        query = Query(...)
        
        with self.assertRaises(ValueError):
            query.new_feature(invalid_input)
```

#### 测试覆盖率

带覆盖率报告运行测试：

```bash
# 使用 tox
tox -e coverage

# 或直接运行
coverage run -m unittest tests
coverage report -m
coverage html  # 在 htmlcov/ 生成 HTML 报告
```

## 文档

### 编写说明

文档使用 [Zensical](https://zensical.org/) 构建，源文件位于 `docs/`：

```
docs/
├── index.md           # 首页
├── getting-started.md # 入门指南
├── authentication.md  # 身份验证指南
├── api-reference.md   # API 参考
├── examples.md        # 示例与菜谱
└── contributing.md    # 本文件（贡献指南）
```

#### 本地预览文档

```bash
zensical serve
```

在浏览器中访问 `http://localhost:8000` 预览。

### Docstring

所有公开 API 均应有 docstring：

```python
def range(self, start=None, stop=None, months=0, days=0):
    """
    定义查询的日期范围。

    Args:
        start: 起始日期（str、datetime.date 或 None）。支持 "today"、"yesterday"、
               "2024-01-01" 等日期串或 datetime.date。默认：昨天。
        stop: 结束日期，格式同 start。默认：与 start 相同。
        months: 相对起始日期的月份偏移，负数为过去，正数为未来。
        days: 相对起始日期的天数偏移，负数为过去，正数为未来。

    Returns:
        已设置日期范围的新 Query 对象。

    Examples:
        >>> query.range('today', days=-7)  # 最近 7 天
        >>> query.range('2024-01-01', '2024-01-31')  # 指定区间
        >>> query.range('today', months=-3)  # 最近 3 个月

    Note:
        API 数据保留约 16 个月。
    """
```

## 提交变更

### 1. 提交代码

请编写清晰、可读的提交说明：

```bash
git add .
git commit -m "为批量查询执行增加支持

- 实现 BulkQuery 类
- 补充并发执行相关测试
- 更新文档与示例"
```

推荐的提交说明格式：

```
简短摘要（建议不超过 50 字符）

必要时写更长说明：改了什么、为什么这样改。

- 可用列表条目
- 关联 issue：Fixes #123
```

### 2. 推送到你的 Fork

```bash
git push origin feature/your-feature-name
```

### 3. 发起 Pull Request

1. 打开 GitHub 上的[本仓库](https://github.com/joshcarty/google-searchconsole)  
2. 点击「New Pull Request」  
3. 选择你的 Fork 与分支  

### 功能建议

提出功能建议时，请尽量包含：

1. **使用场景**：为什么需要该功能？  
2. **方案设想**：期望如何工作？  
3. **备选方案**：你还考虑过哪些做法？  
4. **示例**：可能的调用方式示例代码  

### 代码目录结构

```
google-searchconsole/
├── src/
│   └── searchconsole/
│       ├── __init__.py      # 对外导出的公开 API
│       ├── auth.py          # 身份验证
│       ├── account.py       # Account 与 WebProperty
│       ├── query.py         # Query 与 Report
│       └── utils.py         # 工具函数
├── tests.py                 # 测试套件
├── docs/                    # 文档
├── pyproject.toml          # 项目元数据
└── README.md               # 快速开始说明
```

## 交流与求助

### 获取帮助

- **文档**：请先查阅[在线文档](https://joshcarty.github.io/google-searchconsole/)  
- **Issues**：搜索[已有 issue](https://github.com/joshcarty/google-searchconsole/issues)  
- **讨论**：在 GitHub [Discussions](https://github.com/joshcarty/google-searchconsole/discussions) 发起讨论  

### 提问建议

提问前请：

1. 先搜索已有 issue / 讨论  
2. 提供最小可复现示例  
3. 附上相关报错信息  
4. 说明你已尝试过的步骤  

## 许可证

参与贡献即表示你同意将贡献内容以 MIT 许可证授权。

## 还有疑问？

若对贡献流程有疑问，可以：

- 新建带 `question` 标签的 issue  
- 在 GitHub 上发起讨论  
- 查阅文档获取更多说明  

再次感谢你对 Google Search Console Python 库的贡献！
