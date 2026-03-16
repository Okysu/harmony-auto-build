# HarmonyOS Auto Build

这是一个用于 HarmonyOS 应用自动化构建的 GitHub Action 工作流模板，支持在 GitHub Actions 中自动编译 HAP、HAR、HSP、APP 等鸿蒙应用产物。

## 简介

本项目提供了一套开箱即用的 GitHub Actions 工作流配置，帮助开发者实现 HarmonyOS 应用的持续集成与自动化构建。通过配置好的工作流，您可以在代码推送、PR 合并或手动触发时自动完成应用构建，并上传构建产物供下载使用。

### 主要特性

- 🚀 **自动化构建**：支持代码推送、PR、手动触发等多种触发方式
- 📦 **多种产物支持**：支持构建 HAP、HAR、HSP、APP 等多种鸿蒙应用格式
- ⚡ **智能缓存**：自动缓存命令行工具，大幅缩短构建时间
- 🔧 **灵活配置**：可自定义触发分支、构建类型等参数

## 快速开始

### 1. 获取工作流文件

将 [`demo.yml`](demo.yml) 文件复制到您的 HarmonyOS 项目仓库的 `.github/workflows/` 目录下，可根据实际需求重命名文件。

### 2. 配置触发分支

默认监听以下分支的推送和 PR 事件：

```yaml
branches: [ "main", "master", "dev", "develop" ]
```

您可以根据实际项目需求修改分支列表。

### 3. 配置命令行工具下载地址

工作流中默认使用的命令行工具下载地址为：

```
https://github.com/Okysu/harmony-auto-build/releases/download/commandline-tools-linux-x64-6.1.0.816/commandline-tools-linux-x64-6.1.0.816.zip
```

> ⚠️ **重要提示**：此链接为本项目 Release 发布的静态资源，可能会发生版本更新或变动。如需稳定构建环境，建议您自行托管命令行工具包，可使用以下方案：
> - AWS S3
> - Cloudflare R2
> - 阿里云 OSS
> - 其他对象存储服务

修改 [`demo.yml`](demo.yml:43) 中的下载地址：

```yaml
run: |
  echo "Cache miss, downloading tools..."
  curl -L -o tools.zip YOUR_CUSTOM_DOWNLOAD_URL
  unzip tools.zip
  rm tools.zip
```

### 4. 选择构建类型

根据您的业务需求，修改构建命令。默认配置为编译 **HAP 包**：

```yaml
- name: Build HAP (Debug Mode)
  run: |
    hvigorw assembleHap --mode module -p product=default -p buildMode=debug --no-daemon
```

## 构建命令参考

根据业务情况，执行相应的构建命令：

### 清理工程

```bash
hvigorw clean --no-daemon
```

### 构建 HAP

生成产物：`${PROJECT_PATH}/{moduleName}/build/{productName}/outputs/{targetName}/xxx.hap`

```bash
hvigorw assembleHap --mode module -p product=default -p buildMode=debug --no-daemon
```

### 构建 HSP

生成产物：`${PROJECT_PATH}/{moduleName}/build/{productName}/outputs/{targetName}/(xxx.har | xxx.hsp)`

```bash
hvigorw assembleHsp --mode module -p module=library@default -p product=default --no-daemon
```

### 构建 HAR

生成产物：`${PROJECT_PATH}/{moduleName}/build/{productName}/outputs/{targetName}/outputs/xxx.har`

```bash
hvigorw assembleHar --mode module -p module=library1@default -p product=default --no-daemon
```

### 构建 APP

生成产物：`${PROJECT_PATH}/build/outputs/{productName}/xxx.app`

```bash
hvigorw assembleApp --mode project -p product=default -p buildMode=debug --no-daemon
```

## 工作流说明

### 触发条件

| 触发方式 | 说明 |
|---------|------|
| `push` | 监听指定分支的代码推送事件 |
| `pull_request` | 向指定分支提交 PR 时触发构建校验 |
| `workflow_dispatch` | 支持在 Actions 页面手动触发构建 |

### 构建步骤

1. **Checkout code** - 拉取仓库代码
2. **Setup Node.js** - 安装 Node.js 18（HarmonyOS 工具链依赖）
3. **Cache Command Line Tools** - 缓存命令行工具，加速后续构建
4. **Download Command Line Tools** - 缓存未命中时下载工具链
5. **Setup Build Environment** - 配置环境变量与工具权限
6. **Install Dependencies** - 通过 ohpm 安装项目依赖
7. **Build HAP** - 执行构建命令
8. **Upload Build Artifacts** - 上传构建产物

### 产物下载

构建完成后，可在 GitHub Actions 页面的 Artifacts 区域下载构建产物，包括：

- `.hap` 文件
- `.app` 文件
- `.hsp` 文件
- `.har` 文件

## 高级配置

### 修改构建模式

将 `buildMode=debug` 修改为 `buildMode=release` 可构建正式版本：

```bash
hvigorw assembleHap --mode module -p product=default -p buildMode=release --no-daemon
```

> 注意：正式版本构建需要配置签名证书。

### 自定义缓存版本

如需更新命令行工具版本，修改缓存键值：

```yaml
key: ${{ runner.os }}-harmony-tools-YOUR_VERSION
```

## 常见问题

**Q: 构建失败提示找不到工具？**

A: 请检查命令行工具下载地址是否有效，或考虑使用对象存储服务托管工具包。

**Q: 如何构建正式版本？**

A: 需要配置签名证书，并在构建命令中指定 `buildMode=release`。

**Q: 缓存如何清理？**

A: 在 Actions 页面手动清除缓存，或修改缓存键值强制更新。

## 贡献

欢迎提交 Issue 和 Pull Request 来完善本项目！

## 许可证

[MIT License](LICENSE)
