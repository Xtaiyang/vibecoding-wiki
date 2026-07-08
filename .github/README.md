# GitHub 通用模板

这是一个通用的 `.github` 目录模板，可以在任何新项目中直接使用。

## 使用方法

1. 将整个 `.github` 目录复制到你的新项目中
2. 替换以下占位符：

### 占位符列表

| 占位符 | 说明 | 示例 |
|--------|------|------|
| `{{PROJECT_NAME}}` | 项目名称（用于 Docker 镜像名） | `my-project` |
| `{{OWNER}}` | GitHub 用户名/组织名 | `username` |
| `{{REPO}}` | GitHub 仓库名 | `my-project` |

### Secrets 配置

在项目的 **Settings → Secrets and variables → Actions** 中配置以下 secrets：

| Secret 名称 | 说明 | 是否必需 |
|-------------|------|----------|
| `GHCR_TOKEN` | GitHub Container Registry 访问令牌 | ✅ 必需 |
| `DOCKERHUB_USERNAME` | Docker Hub 用户名 | ❌ 可选 |
| `DOCKERHUB_TOKEN` | Docker Hub 访问令牌 | ❌ 可选 |

> **注意：** 如果不需要 Docker Hub 镜像推送，只需配置 `GHCR_TOKEN` 即可。

### GitHub Token 权限

确保 `GHCR_TOKEN` 具有以下权限：
- `read:packages` - 读取包
- `write:packages` - 写入包

## 文件说明

```
.github/
├── CONTRIBUTING.md          # 贡献指南模板
├── pull_request_template.md # PR 模板
├── ISSUE_TEMPLATE/
│   ├── config.yml           # Issue 配置
│   ├── bug.yml              # Bug 报告模板
│   ├── feature-request.yml  # 功能请求模板
│   └── question.yml         # 问题咨询模板（含部署问题）
└── workflows/
    ├── Build_Docker.yml     # 推送时构建 Docker（仅构建不推送）
    └── Release_docker.yml   # 发布时构建并推送 Docker
```

## 自定义

### 调整分支名

如果你的主分支不是 `main`，请在 `workflows/*.yml` 文件中修改：

```yaml
on:
  push:
    branches:
      - "main"  # 修改为你的分支名
```

### 调整构建平台

默认构建 `linux/amd64` 和 `linux/arm64`，如需调整请修改：

```yaml
platforms: linux/amd64,linux/arm64
```

### 禁用 Docker Hub

如果不需要推送到 Docker Hub，删除 `Release_docker.yml` 中的以下部分：

```yaml
- name: Login to Docker Hub
  if: env.DOCKERHUB_CONFIGURED == 'true'
  uses: docker/login-action@v3
  with:
    username: ${{ secrets.DOCKERHUB_USERNAME }}
    password: ${{ secrets.DOCKERHUB_TOKEN }}
```

以及 `tags` 中的 Docker Hub 标签。
