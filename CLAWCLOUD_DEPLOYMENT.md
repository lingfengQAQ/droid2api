# ClawCloud 部署指南

本文档详细说明如何在 ClawCloud Run 平台上部署 droid2api 项目。

## 目录
- [前置准备](#前置准备)
- [方法一：使用 GitHub Actions 自动构建](#方法一使用-github-actions-自动构建推荐)
- [方法二：使用现有镜像直接部署](#方法二使用现有镜像直接部署)
- [部署配置详解](#部署配置详解)
- [常见问题](#常见问题)

---

## 前置准备

### 1. 账号准备
- ✅ GitHub 账号
- ✅ Docker Hub 账号 (免费注册: https://hub.docker.com/signup)
- ✅ ClawCloud Run 账号 (注册: https://run.claw.cloud)

### 2. Fork 本项目
1. 访问 https://github.com/1e0n/droid2api
2. 点击右上角 **Fork** 按钮
3. 将项目 Fork 到你的 GitHub 账号

---

## 方法一：使用 GitHub Actions 自动构建(推荐)

这种方法可以自动构建 Docker 镜像并推送到 Docker Hub，无需本地 Docker 环境。

### 步骤 1: 配置 GitHub Actions

#### 1.1 修改工作流文件
在你 Fork 的仓库中，找到或创建 `.github/workflows/docker-image.yml` 文件，内容如下:

```yaml
name: Docker Image CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:  # 允许手动触发

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3

    - name: Login to Docker Hub
      uses: docker/login-action@v3
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v5
      with:
        context: .
        file: ./Dockerfile
        push: true
        tags: |
          ${{ secrets.DOCKER_USERNAME }}/droid2api:latest
          ${{ secrets.DOCKER_USERNAME }}/droid2api:${{ github.sha }}
```

#### 1.2 获取 Docker Hub Access Token
1. 登录 Docker Hub
2. 点击右上角头像 → **Account Settings**
3. 左侧菜单选择 **Security**
4. 点击 **New Access Token**
5. 描述填写: `GitHub Actions`
6. 点击 **Generate**
7. **立即复制 Token**(只显示一次!)

#### 1.3 配置 GitHub Secrets
1. 进入你 Fork 的仓库
2. 点击 **Settings** 标签
3. 左侧菜单: **Secrets and variables** → **Actions**
4. 点击 **New repository secret**，添加两个密钥:

**密钥 1:**
- Name: `DOCKER_USERNAME`
- Value: 你的 Docker Hub 用户名

**密钥 2:**
- Name: `DOCKER_PASSWORD`
- Value: 刚才复制的 Access Token

### 步骤 2: 触发构建

#### 自动触发
- 每次 push 代码到 `main` 分支会自动构建

#### 手动触发
1. 进入仓库的 **Actions** 标签
2. 点击左侧 **Docker Image CI**
3. 点击右侧 **Run workflow** → **Run workflow**
4. 等待构建完成(约 3-5 分钟)

### 步骤 3: 验证镜像
构建成功后，访问 Docker Hub:
```
https://hub.docker.com/r/你的用户名/droid2api
```

你会看到镜像已成功推送，标签为 `latest`。

---

## 方法二：使用现有镜像直接部署

如果已经有现成的 Docker 镜像，可以直接跳到 [部署配置详解](#部署配置详解)。

---

## 部署配置详解

### 1. 创建应用

1. 登录 ClawCloud Run: https://run.claw.cloud
2. 点击 **App Launchpad** → **Create App**

### 2. 基础配置

#### Application Name
```
droid2api
```
或任何你喜欢的名称。

#### Image 配置
- **Public/Private**: 选择 `Public`
- **Image Name**:
  ```
  你的Docker用户名/droid2api:latest
  ```

**示例:**
- 如果你的 Docker Hub 用户名是 `john`
- 填写: `john/droid2api:latest`

### 3. 资源配置 (Usage)

#### 固定实例模式 (Fixed)
- **Replicas**: `1`
- **CPU**: `0.5` - `1` (建议从 0.5 开始)
- **Memory**: `512M` - `1G` (建议从 512M 开始)

> 💡 **提示**: 可以根据实际使用情况调整资源配置

### 4. 网络配置 (Network)

#### Container Port 配置

**Port 1** (如果有 80 端口需求):
- Container Port: `80`
- Public Access: 关闭 ❌

**Port 2** (主要服务端口):
- Container Port: `3000`
- Public Access: **开启** ✅

> ⚠️ **重要**: 必须开启 Public Access 才能从外网访问

### 5. 高级配置 (Advanced Configuration)

#### Command
```
留空
```

#### Arguments
```
留空
```

> 📝 **说明**: Dockerfile 中已定义启动命令 `CMD ["npm", "start"]`，无需额外配置

#### Environment Variables (环境变量)

添加以下环境变量:

| Key | Value |
|-----|-------|
| `PORT` | `3000` |
| `NODE_ENV` | `production` |

**添加步骤:**
1. 点击 **Add** 按钮
2. Key 填写: `PORT`
3. Value 填写: `3000`
4. 重复以上步骤添加 `NODE_ENV`

### 6. 部署应用

1. 检查所有配置无误
2. 点击页面底部的 **Deploy** 或 **Create** 按钮
3. 等待部署完成(约 2-5 分钟)

### 7. 访问应用

部署成功后，ClawCloud 会提供一个公网访问地址:
```
https://your-app-name.run.claw.cloud
```

或通过配置的自定义域名访问。

---

## 配置总览

### 快速配置表

| 配置项 | 值 |
|--------|-----|
| Application Name | `droid2api` |
| Image Name | `你的用户名/droid2api:latest` |
| CPU | `0.5` - `1` |
| Memory | `512M` - `1G` |
| Container Port | `3000` |
| Public Access | ✅ 开启 |
| Environment - PORT | `3000` |
| Environment - NODE_ENV | `production` |
| Command | 留空 |
| Arguments | 留空 |

---

## 常见问题

### Q1: 部署失败，提示 "镜像拉取失败"
**A:** 检查以下几点:
1. 确认 Docker Hub 镜像已成功推送
2. 确认镜像名称格式正确: `用户名/droid2api:latest`
3. 确认镜像是 Public (公开的)，不是 Private

### Q2: 应用启动后无法访问
**A:** 检查:
1. Container Port 是否配置为 `3000`
2. Public Access 是否已开启
3. 查看应用日志，检查是否有启动错误

### Q3: 如何查看应用日志?
**A:**
1. 进入 ClawCloud 控制台
2. 找到你的应用
3. 点击应用名称进入详情页
4. 查看 **Logs** 或 **Terminal** 标签

### Q4: 如何更新应用?
**A:**
1. 推送新代码到 GitHub
2. GitHub Actions 自动构建新镜像
3. 在 ClawCloud 重新部署或重启应用
4. 新版本会自动拉取 `latest` 标签的镜像

### Q5: 端口配置错误怎么办?
**A:**
确保:
- Dockerfile 中 `EXPOSE 3000`
- ClawCloud Container Port 配置为 `3000`
- 环境变量 `PORT=3000`
- 应用代码监听的端口也是 `3000`

### Q6: GitHub Actions 构建失败
**A:** 常见原因:
1. **Secrets 未配置**: 检查 `DOCKER_USERNAME` 和 `DOCKER_PASSWORD` 是否正确设置
2. **Token 过期**: 重新生成 Docker Hub Access Token
3. **Dockerfile 错误**: 检查 Dockerfile 语法是否正确

### Q7: 内存不足 (OOM)
**A:**
- 增加 Memory 配置到 `1G` 或更高
- 检查应用是否有内存泄漏
- 优化应用代码

### Q8: 如何配置自定义域名?
**A:**
1. 在 ClawCloud 应用设置中找到 **Custom Domain**
2. 添加你的域名
3. 按照提示配置 DNS 记录 (CNAME)
4. 等待 DNS 生效(可能需要几分钟到几小时)

---

## 完整部署流程图

```
┌─────────────────────────────────────────────────────────────┐
│ 1. Fork 项目到 GitHub                                        │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│ 2. 配置 GitHub Actions                                       │
│    - 修改 .github/workflows/docker-image.yml                │
│    - 添加 Secrets (DOCKER_USERNAME, DOCKER_PASSWORD)        │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│ 3. 触发构建                                                  │
│    - 自动触发: Push 代码                                     │
│    - 手动触发: Actions → Run workflow                       │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│ 4. 验证镜像                                                  │
│    - 访问 Docker Hub                                        │
│    - 检查镜像是否存在                                        │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│ 5. 在 ClawCloud 创建应用                                     │
│    - 填写应用名称                                            │
│    - 配置镜像: 用户名/droid2api:latest                       │
│    - 配置资源: CPU, Memory                                  │
│    - 配置端口: 3000 (Public Access)                         │
│    - 配置环境变量: PORT, NODE_ENV                           │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│ 6. 部署并访问                                                │
│    - 等待部署完成                                            │
│    - 获取访问 URL                                           │
│    - 测试应用功能                                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 技术支持

- **ClawCloud 文档**: https://docs.run.claw.cloud
- **项目 Issues**: https://github.com/1e0n/droid2api/issues
- **Docker Hub**: https://hub.docker.com

---

## 许可证

请遵循原项目的许可证要求。

---

**最后更新**: 2025-10-10
