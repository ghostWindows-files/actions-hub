# Docker 镜像同步工具

[![GitHub Actions](https://img.shields.io/badge/CI-GitHub%20Actions-2088FF?logo=github-actions)](https://github.com/features/actions)
[![Docker](https://img.shields.io/badge/Registry-Docker%20Hub-2496ED?logo=docker)](https://hub.docker.com)

基于 GitHub Actions 的 Docker 镜像同步解决方案，支持多架构镜像同步与智能同步策略。

## 功能特性

🚀 **核心功能**
- **灵活同步配置**：支持手动输入或环境变量配置同步映射
- **格式验证**：自动校验输入格式有效性
- **智能镜像检查**：自动跳过已存在且内容相同的镜像
- **多架构支持**：完整保留源镜像的架构信息（amd64/arm64 等）
- **安全认证**：通过 Docker Hub 令牌进行安全认证
- **同步结果实时通知**（支持WxPusher）
- **智能跳过已同步镜像**（Digest校验）

⚙️ **技术特性**
- 使用 skopeo copy 处理多架构镜像
- 基于 Skopeo 的镜像元数据分析
- 自动修正镜像命名规范
- 完善的错误处理机制

## 使用指南

### 准备工作

1. **Docker Hub 认证配置**
    - 在仓库 Settings → Secrets 中添加：
        - `DOCKERHUB_USERNAME`: Docker Hub 用户名
        - `DOCKERHUB_TOKEN`: Docker Hub Access Token
    - （可选）配置 `WPUSH_KEY` 用于发送通知

2. **配置同步规则**
    - 方式一：环境变量配置（推荐）
      在仓库 Settings → Variables 添加：
      - `DOCKER_SYNC_MAPPINGS`: `nginx=myrepo/nginx,alpine:latest=myrepo/alpine:stable`
    - 方式二：手动触发时输入
      - `nginx=myrepo/nginx,alpine:latest=myrepo/alpine:stable`
### 触发同步

**手动触发：**
1. 进入仓库 Actions 页面
2. 选择 "Docker 镜像同步"
3. 点击 "Run workflow"
4. 输入同步映射（格式：`源镜像=目标镜像`，多个用逗号分隔）**[可选]**

**自动触发：**
```yaml
on:
schedule:
- cron: '0 0 * * *'  # 每天UTC时间0点执行
```

## 参数说明

### 同步映射格式
`sync_mappings` 参数格式：
```bash
<源镜像>[:<标签>]=<目标镜像>[:<标签>]
```
- 当源镜像未指定标签时，默认使用 `latest`
- 当目标镜像未指定标签时，继承源镜像标签

**示例：**
```bash
# 同步 ubuntu:latest → myrepo/ubuntu:latest
ubuntu=myrepo/ubuntu  
# 同步 alpine:3.18 → myrepo/alpine:stable
alpine:3.18=myrepo/alpine:stable  
```

## 同步逻辑

🔄 **同步流程**
1. 镜像规范化处理（自动补全docker.io前缀）
2. 架构分析（单架构/多架构）
3. Digest比对校验
4. 使用`skopeo copy --all`进行镜像同步

⏭️ **跳过条件**
- 目标镜像已存在且 Digest 一致

## 通知示例

📨 **同步结果通知**
```
镜像同步结果

🎉 同步成功：
✅ `docker.io/library/ubuntu:latest` → `docker.io/myrepo/ubuntu:latest`
✅ `docker.io/library/alpine:3.18` → `docker.io/myrepo/alpine:stable`

⏩ 跳过同步：
🔜 `docker.io/library/centos:latest` → `docker.io/myrepo/centos:latest`

💥 失败任务：
❌ `docker.io/library/invalid-image:latest` → `docker.io/myrepo/invalid-image:latest`
```

## 注意事项

1. **权限要求**
    - Docker Hub Token 需要至少 `read, write` 权限
    - 目标仓库需要提前创建或具有自动创建权限

2. **镜像命名规范**
    - 官方镜像可简写（如 `docker.io/library/ubuntu` → `ubuntu`）
    - 自定义镜像需要完整路径（如 `myrepo/nginx`）

