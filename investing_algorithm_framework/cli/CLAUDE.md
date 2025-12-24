# CLI 模块

[根目录](../../CLAUDE.md) > [investing_algorithm_framework](../) > **cli**

## 模块职责

命令行界面模块，提供项目初始化和云部署功能：

- **项目初始化**: 创建交易机器人项目骨架
- **AWS Lambda 部署**: 一键部署到 AWS Lambda
- **Azure Functions 部署**: 一键部署到 Azure Functions
- **模板管理**: 支持多种项目模板（default, web, aws_lambda, azure_function）

## 入口与启动

### 命令行入口

```bash
# 查看帮助
investing-algorithm-framework --help

# 初始化项目
investing-algorithm-framework init

# 部署到 AWS Lambda
investing-algorithm-framework deploy-aws-lambda --help

# 部署到 Azure Functions
investing-algorithm-framework deploy-azure-function --help
```

### 主要入口点

| 文件 | 描述 |
|------|------|
| `cli.py` | Click CLI 主入口 |
| `initialize_app.py` | 项目初始化逻辑 |
| `deploy_to_aws_lambda.py` | AWS Lambda 部署 |
| `deploy_to_azure_function.py` | Azure Functions 部署 |

## 对外接口

### init 命令

```bash
investing-algorithm-framework init [OPTIONS]
```

**选项：**
- `--type`: 项目类型
  - `default`: 标准交易机器人（默认）
  - `default_web`: 带 Web API 的交易机器人
  - `aws_lambda`: AWS Lambda 就绪项目
  - `azure_function`: Azure Function 就绪项目
- `--path`: 目标目录路径（默认当前目录）
- `--replace`: 是否覆盖已存在的文件

**示例：**

```bash
# 创建默认项目
investing-algorithm-framework init

# 创建 AWS Lambda 项目
investing-algorithm-framework init --type aws_lambda --path my-bot

# 创建项目并覆盖现有文件
investing-algorithm-framework init --type azure_function --replace
```

### deploy-aws-lambda 命令

```bash
investing-algorithm-framework deploy-aws-lambda [OPTIONS]
```

**必填选项：**
- `--lambda_function_name`: Lambda 函数名称
- `--region`: AWS 区域

**可选选项：**
- `--project_dir`: 项目目录路径（默认当前目录）
- `--memory_size`: 内存大小，MB（默认 3000）
- `--env -e KEY VALUE`: 环境变量（可多次使用）

**示例：**

```bash
# 部署到 AWS Lambda
investing-algorithm-framework deploy-aws-lambda \
    --lambda_function_name my-trading-bot \
    --region us-east-1 \
    --memory_size 2048 \
    -e BINANCE_API_KEY key123 \
    -e BINANCE_API_SECRET secret456
```

### deploy-azure-function 命令

```bash
investing-algorithm-framework deploy-azure-function [OPTIONS]
```

**必填选项：**
- `--resource_group`: 资源组名称
- `--deployment_name`: 部署名称（将用作 Function App 名称）
- `--region`: Azure 区域

**可选选项：**
- `--subscription_id`: 订阅 ID
- `--storage_account_name`: 存储账户名称
- `--container_name`: Blob 容器名称（默认 iafcontainer）
- `--create_resource_group_if_not_exists`: 不存在时创建资源组
- `--skip_login`: 跳过登录（用于 CI/CD）

**示例：**

```bash
# 部署到 Azure Functions
investing-algorithm-framework deploy-azure-function \
    --resource_group my-rg \
    --deployment_name my-trading-bot \
    --region eastus \
    --create_resource_group_if_not_exists
```

## 项目模板

### default 模板

标准交易机器人项目结构：

```
project/
├── app.py              # 应用入口（保持不变）
├── strategy.py         # 策略文件（自定义）
├── .env                # 环境变量
└── requirements.txt    # 依赖
```

### aws_lambda 模板

AWS Lambda 就绪项目：

```
project/
├── lambda_function.py  # Lambda 处理函数
├── strategy.py         # 策略文件
├── .env                # 环境变量
└── requirements.txt    # 依赖
```

### azure_function 模板

Azure Function 就绪项目：

```
project/
├── function_app.py     # Function App 入口
├── strategy.py         # 策略文件
├── host.json           # Azure Function 配置
├── local.settings.json # 本地设置
└── requirements.txt    # 依赖
```

## 关键依赖与配置

### 依赖项

- `click`: CLI 框架
- `boto3`: AWS SDK
- `azure-identity`: Azure 身份验证
- `azure-mgmt-web`: Azure Web 应用管理
- `azure-mgmt-resource`: Azure 资源管理
- `azure-mgmt-storage`: Azure 存储管理
- `azure-storage-blob`: Azure Blob 存储

### 环境变量

AWS Lambda 部署：
- `AWS_ACCESS_KEY_ID`: AWS 访问密钥
- `AWS_SECRET_ACCESS_KEY`: AWS 密钥
- `AWS_REGION`: AWS 区域

Azure Functions 部署：
- `AZURE_CLIENT_ID`: Azure 客户端 ID
- `AZURE_CLIENT_SECRET`: Azure 客户端密钥
- `AZURE_TENANT_ID`: Azure 租户 ID

## 数据模型

### 部署配置

AWS Lambda：
```python
{
    "function_name": "my-bot",
    "region": "us-east-1",
    "memory_size": 3000,
    "timeout": 900,
    "environment_variables": {...}
}
```

Azure Function：
```python
{
    "resource_group": "my-rg",
    "deployment_name": "my-bot",
    "region": "eastus",
    "storage_account": "mystorageaccount",
    "container_name": "iafcontainer"
}
```

## 测试与质量

### 测试目录

```
tests/cli/
└── test_initialize.py    # 初始化测试
```

### 测试覆盖

- 项目初始化
- 文件生成
- 模板选择

## 常见问题 (FAQ)

### Q: 如何更新现有项目而不覆盖自定义文件？

A: 不使用 `--replace` 标志，已存在的文件将被跳过：

```bash
investing-algorithm-framework init --path my-project
# 只添加不存在的文件
```

### Q: AWS Lambda 部署失败怎么办？

A: 检查以下几点：
1. AWS 凭证是否正确配置
2. Lambda 函数名称是否唯一
3. 内存大小是否在允许范围内（128-3008 MB）
4. 超时时间是否足够（最大 900 秒）

### Q: Azure Functions 部署需要什么权限？

A: 需要以下 Azure 权限：
- 创建资源组
- 创建存储账户
- 创建 Function App
- 部署代码

### Q: 如何在 CI/CD 中使用部署命令？

A: 使用 `--skip_login` 标志并预先配置认证：

```bash
# AWS
aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY

investing-algorithm-framework deploy-aws-lambda \
    --lambda_function_name my-bot \
    --region us-east-1

# Azure
az login --service-principal \
    -u $AZURE_CLIENT_ID \
    -p $AZURE_CLIENT_SECRET \
    --tenant $AZURE_TENANT_ID

investing-algorithm-framework deploy-azure-function \
    --resource_group my-rg \
    --deployment_name my-bot \
    --region eastus \
    --skip_login
```

## 相关文件清单

### CLI 目录

```
cli/
├── __init__.py
├── cli.py                           # Click CLI 主入口
├── initialize_app.py                # 项目初始化实现
├── deploy_to_aws_lambda.py          # AWS Lambda 部署
└── deploy_to_azure_function.py      # Azure Functions 部署
```

### 模板文件（在代码中定义）

模板文件在 `initialize_app.py` 中定义为字符串常量：

- `DEFAULT_TEMPLATE`: 标准项目模板
- `DEFAULT_WEB_TEMPLATE`: Web 项目模板
- `AWS_LAMBDA_TEMPLATE`: AWS Lambda 模板
- `AZURE_FUNCTION_TEMPLATE`: Azure Function 模板

## 变更记录 (Changelog)

### 2025-12-24 10:25:42 - 模块文档创建
- 创建 cli 模块详细文档
- 整理命令接口和选项说明
- 添加文件清单和常见问题
