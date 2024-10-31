# ChatGPT-on-WeChat 智谱 AI 配置指南

本指南将帮助您在 **ChatGPT-on-WeChat** 项目中配置和使用 **智谱 AI（Zhipu AI）** 的服务。通过遵循以下步骤，您将能够成功地将智谱 AI 的模型集成到您的微信聊天机器人中。

---

## 目录

- [简介](#简介)
- [环境要求](#环境要求)
- [配置步骤](#配置步骤)
  - [1. 获取智谱 AI API 密钥](#1-获取智谱-ai-api-密钥)
  - [2. 修改 `docker-compose.yml` 文件](#2-修改-docker-composeyml-文件)
  - [3. 更新环境变量](#3-更新环境变量)
  - [4. 重启 Docker 容器](#4-重启-docker-容器)
- [验证配置](#验证配置)
  - [1. 检查环境变量](#1-检查环境变量)
  - [2. 查看日志信息](#2-查看日志信息)
  - [3. 测试聊天功能](#3-测试聊天功能)
- [注意事项](#注意事项)
- [常见问题](#常见问题)
- [参考资料](#参考资料)
- [许可证](#许可证)

---

## 简介

**ChatGPT-on-WeChat** 是一个开源项目，旨在将 OpenAI 的 ChatGPT 模型集成到微信平台，实现智能聊天机器人的功能。为了满足不同的需求，项目支持多种模型和服务提供商，包括 OpenAI、智谱 AI、MoonShot 等。

本指南将指导您如何配置项目以使用 **智谱 AI** 的服务，从而在微信中体验智谱 AI 模型的强大功能。

---

## 环境要求

- **操作系统**：Linux、macOS 或 Windows（需支持 Docker）。
- **Docker**：确保已安装 Docker 和 Docker Compose。
- **Git**：用于克隆项目代码。
- **微信账号**：用于登录和测试微信机器人。

---

## 配置步骤

### 1. 获取智谱 AI API 密钥

- **注册账号**：访问 [智谱 AI 官网](https://www.zhipu.ai/)，注册并登录账户。
- **获取 API 密钥**：在用户中心或开发者中心，生成并获取您的 **API 密钥**。
- **注意**：妥善保管您的 API 密钥，避免泄露。

### 2. 修改 `docker-compose.yml` 文件

在项目根目录下，找到或创建 `docker-compose.yml` 文件，编辑以下内容：

```yaml
version: '2.0'
services:
  chatgpt-on-wechat:
    image: zhayujie/chatgpt-on-wechat
    container_name: chatgpt-on-wechat
    security_opt:
      - seccomp:unconfined
    ports:
      - "3001:3000"  # 本地端口3001映射到容器端口3000
    environment:
      # 启用智谱 AI
      USE_ZHIPU_AI: 'True'
      ZHIPU_AI_API_KEY: 'YOUR_ZHIPU_AI_API_KEY'  # 请替换为您的实际密钥
      ZHIPU_AI_API_BASE: 'https://open.bigmodel.cn/api/paas/v4/'
      MODEL: 'chatglm_std'  # 智谱 AI 提供的模型名称，请根据需要选择
      # 禁用其他模型
      USE_OPENAI: 'False'
      USE_MOONSHOT: 'False'
      USE_KIMI: 'False'
      # 其他配置
      CHANNEL_TYPE: 'wx'
      PROXY: ''  # 如果需要代理，请在此填写，例如 'http://127.0.0.1:7890'
      HOT_RELOAD: 'False'
      SINGLE_CHAT_PREFIX: '[""]'
      SINGLE_CHAT_REPLY_PREFIX: '" "'
      GROUP_CHAT_PREFIX: '["@bot","bot"]'
      GROUP_NAME_WHITE_LIST: '["ALL_GROUP"]'
      IMAGE_CREATE_PREFIX: '["画", "看", "找"]'
      CONVERSATION_MAX_TOKENS: 4000
      SPEECH_RECOGNITION: 'True'
      CHARACTER_DESC: '你是一个智慧的助手，能够回答各种问题。'
      SUBSCRIBE_MSG: '感谢您的关注！这里是智能助理，可以自由对话。'
      EXPIRES_IN_SECONDS: 3600
      USE_GLOBAL_PLUGIN_CONFIG: 'True'
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

**说明：**

- **`USE_ZHIPU_AI`**：设置为 `'True'`，启用智谱 AI 服务。
- **`ZHIPU_AI_API_KEY`**：替换为您的实际 API 密钥。
- **`ZHIPU_AI_API_BASE`**：智谱 AI 的 API 基础地址，确保填写正确。
- **`MODEL`**：指定使用的模型名称，如 `'chatglm_std'`。请根据智谱 AI 提供的模型列表选择合适的模型。
- **禁用其他模型**：将 `USE_OPENAI`、`USE_MOONSHOT`、`USE_KIMI` 设置为 `'False'`。
- **其他配置项**：根据您的需求进行调整。

### 3. 更新环境变量

确保所有环境变量名称和值正确无误，尤其要注意大小写和拼写。

### 4. 重启 Docker 容器

在终端中执行以下命令，使更改生效：

```bash
# 停止并删除现有容器
docker-compose down

# 根据新的配置启动容器
docker-compose up -d
```

---

## 验证配置

### 1. 检查环境变量

进入容器内部，确认环境变量是否正确加载：

```bash
# 进入容器
docker exec -it chatgpt-on-wechat /bin/bash

# 查看环境变量
env | grep ZHIPU_AI
```

应看到类似以下的输出：

```bash
USE_ZHIPU_AI=True
ZHIPU_AI_API_KEY=YOUR_ZHIPU_AI_API_KEY
ZHIPU_AI_API_BASE=https://open.bigmodel.cn/api/paas/v4/
```

### 2. 查看日志信息

查看容器日志，检查是否有错误信息：

```bash
docker logs chatgpt-on-wechat
```

- **正常情况下**，日志应显示程序启动成功的信息。
- **若有错误**，根据日志提示进行排查和调整。

### 3. 测试聊天功能

- **在微信中与机器人对话**，发送消息，查看是否能够正常收到回复。
- **验证功能**：
  - 文本回复是否正常。
  - 图片生成、语音识别等功能是否正常（需根据实际配置）。

---

## 注意事项

- **API 密钥安全**：
  - 请勿将您的 API 密钥公开分享，避免泄露。
  - 如果怀疑密钥泄露，建议立即在智谱 AI 平台上重置密钥。

- **模型名称**：
  - 确保 `MODEL` 的值与智谱 AI 提供的模型名称一致。
  - 可访问智谱 AI 官网，查看可用的模型列表。

- **代理设置**：
  - 如果服务器无法直接访问外部网络，需要在 `PROXY` 中配置代理地址。
  - 代理格式示例：`http://127.0.0.1:7890`

- **配置生效**：
  - 每次修改 `docker-compose.yml` 文件后，都需要重启容器以使更改生效。

---

## 常见问题

### 1. 程序提示未提供 `api_key`

**解决方案**：

- 确认 `ZHIPU_AI_API_KEY` 已正确设置，并在 `docker-compose.yml` 中指定。
- 检查环境变量名称是否有拼写错误，名称需完全匹配，包括大小写。

### 2. 程序仍然尝试连接其他 API

**解决方案**：

- 确保已将 `USE_OPENAI`、`USE_MOONSHOT`、`USE_KIMI` 设置为 `'False'`。
- 检查是否有其他配置文件（如 `config.json`）中启用了其他服务。

### 3. 无法访问智谱 AI 的 API 端点

**解决方案**：

- 检查服务器的网络连接，确保能够访问 `https://open.bigmodel.cn/`。
- 如果需要代理，正确配置 `PROXY`。

### 4. 收到错误回复或功能异常

**解决方案**：

- 查看容器日志，查找错误信息。
- 确认模型名称和参数设置是否正确。
- 检查项目是否需要更新到最新版本。

---

## 参考资料

- **ChatGPT-on-WeChat 项目 GitHub**：  
  [https://github.com/zhayujie/chatgpt-on-wechat](https://github.com/zhayujie/chatgpt-on-wechat)

- **智谱 AI 官网**：  
  [https://www.zhipu.ai/](https://www.zhipu.ai/)

- **智谱 AI API 文档**：  
  [https://www.zhipu.ai/documentation](https://www.zhipu.ai/documentation)

- **Docker 官方文档**：  
  [https://docs.docker.com/](https://docs.docker.com/)

---

## 许可证

本项目遵循 [MIT 许可证](LICENSE)。您可以自由地使用、修改和分发本指南，但需保留原作者信息。

---

希望本指南能帮助您成功配置 **ChatGPT-on-WeChat** 项目，并充分利用 **智谱 AI** 的强大功能。如有任何问题，建议您：

- **提交 Issue**：在项目的 GitHub 页面提交问题，提供详细的错误日志和配置信息（注意隐藏敏感信息）。
- **联系智谱 AI 支持**：通过官方渠道获取帮助。
