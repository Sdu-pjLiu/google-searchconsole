---
icon: lucide/key
---

# 身份验证

本库支持多种身份验证方式，以适应不同场景。本文档详细说明各选项。

## 身份验证方式

主要有两种方式：

1. **OAuth2 流程** — 个人使用，或需要用户自行登录授权的应用  
2. **服务账号** — 服务端应用或自动化脚本  

## OAuth2 身份验证

对大多数用户推荐使用 OAuth2。你可以「以本人身份」登录，并访问你拥有或有权限的资源。

### 交互式 OAuth2 流程

最简单的入门方式：

```python
import searchconsole

# 首次认证
account = searchconsole.authenticate(client_config='client_secrets.json')
```

该流程会：

1. 打开系统默认浏览器  
2. 提示你登录 Google  
3. 请求你授予权限  
4. 重定向回来表示成功  
5. 返回已认证的 `Account` 对象  

!!! tip "一次性设置"
    首次认证成功后，请保存凭据，以便之后跳过浏览器流程。

### 保存凭据

避免每次运行都重复 OAuth 流程：

```python
# 首次认证（会打开浏览器）
account = searchconsole.authenticate(client_config='client_secrets.json')

# 保存凭据供以后使用
account.serialize_credentials('credentials.json')
```

之后的会话中：

```python
# 加载已保存凭据（无需再开浏览器）
account = searchconsole.authenticate(
    client_config='client_secrets.json',
    credentials='credentials.json',
)
```

!!! warning "安全提示"
    `credentials.json` 包含你账号的访问令牌。请妥善保管，并加入 `.gitignore`，避免提交到版本库。

### 控制台流程（远程开发环境）

若在无法使用网页回调的远程环境（例如 Google Colab）中运行，请改用控制台流程：

```python
account = searchconsole.authenticate(client_config='client_secrets.json', flow='console')
```

流程说明：

1. 在控制台打印授权 URL  
2. 你在本机浏览器中手动打开该 URL  
3. 授权后，Google 会重定向到 localhost URL  
4. 从浏览器地址栏复制完整的重定向 URL  
5. 粘贴回控制台  

控制台输出示例：

```
Please visit this URL to authorize this application:
https://accounts.google.com/o/oauth2/auth?client_id=...

Enter the authorization code: 
```

授权后，复制完整 URL：

```
http://localhost:8080/?code=4/0AX4XfWh...&scope=https://www.googleapis.com/auth/webmasters.readonly
```

并粘贴到控制台。

### 使用配置字典

除文件路径外，也可传入字典形式的配置：

```python
client_config = {
    "installed": {
        "client_id": "your_client_id.apps.googleusercontent.com",
        "client_secret": "your_client_secret",
        "auth_uri": "https://accounts.google.com/o/oauth2/auth",
        "token_uri": "https://oauth2.googleapis.com/token",
        "redirect_uris": ["http://localhost"]
    }
}

credentials = {
    "token": "ya29.a0AfH6SMBx...",
    "refresh_token": "1//0gZ9K8W...",
    "token_uri": "https://oauth2.googleapis.com/token",
    "client_id": "your_client_id.apps.googleusercontent.com",
    "client_secret": "your_client_secret",
    "scopes": ["https://www.googleapis.com/auth/webmasters.readonly"]
}

account = searchconsole.authenticate(client_config=client_config, credentials=credentials)
```

适用于将凭据放在环境变量或密钥管理系统中的场景。

## 服务账号身份验证

服务账号适用于：

- 服务端应用  
- 自动化脚本与定时任务  
- CI/CD 流水线  
- 无需人工交互的应用  

### 创建服务账号

1. **创建服务账号**：  
    - 打开 [Google Cloud Console](https://console.cloud.google.com/)  
    - 进入 **IAM & Admin** > **Service Accounts**  
    - 点击 **Create Service Account**  
    - 填写名称与说明  
    - 点击 **Create and Continue**  

2. **创建密钥**：  
    - 点击刚创建的服务账号  
    - 打开 **Keys** 标签页  
    - 点击 **Add Key** > **Create new key**  
    - 选择 **JSON** 格式  
    - 点击 **Create**  
    - 将下载的 JSON 保存为 `service_account.json`  

3. **在 Search Console 中授权**：  
    - 打开 [Google Search Console](https://search.google.com/search-console/)  
    - 选择你的资源  
    - 点击 **Settings** > **Users and permissions**  
    - 点击 **Add user**  
    - 输入服务账号邮箱（JSON 中的 `client_email`）  
    - 设置权限级别（Owner 或 Full）  
    - 点击 **Add**  

### 使用服务账号认证

```python
import searchconsole

# 使用服务账号认证
account = searchconsole.authenticate(service_account='service_account.json')

# 与 OAuth 用法相同
webproperty = account['https://www.example.com/']
report = webproperty.query.range('today', days=-7).get()
```

也支持传入字典。

!!! warning "服务账号限制"
    - 不能同时使用 OAuth2 与服务账号凭据  
    - 服务账号凭据不能像 OAuth2 那样通过本库序列化保存  
    - 必须在 Search Console 中为每个资源显式添加该服务账号  

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

控制台会打印可复制的 URL，在浏览器中打开即可。

## 安全最佳实践

1. **切勿将凭据提交到版本库**  

2. **生产环境优先使用环境变量**  
    ```python
    credentials = os.environ.get('GOOGLE_CREDENTIALS')
    ```

3. **定期轮换服务账号密钥**  
    - 周期性创建新密钥  
    - 轮换完成后删除旧密钥  

4. **遵循最小权限原则**  
    - 只授予必要权限  
    - 不需要完全权限时可使用「受限用户」等较低角色  

5. **关注访问日志**  
    - 定期查看 [Google 账号活动](https://myaccount.google.com/permissions)  
    - 在 Cloud Console 中审查服务账号使用情况  

## 下一步

- **[入门指南](getting-started.md)** — 运行第一次查询  
- **[API 参考](api-reference.md)** — 详细 API 说明  
- **[示例](examples.md)** — 实际用法示例  
