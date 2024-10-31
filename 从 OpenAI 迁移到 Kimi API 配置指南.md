# 从 OpenAI 迁移到 Kimi API 配置指南

本指南将帮助您将 **ChatGPT-on-WeChat** 项目从使用 **OpenAI** 的服务迁移到使用 **Kimi API**。通过遵循以下步骤，您可以在不大幅修改代码的情况下，成功地将应用和服务迁移到 Kimi 大模型。

---

## 目录

- [简介](#简介)
- [环境要求](#环境要求)
- [迁移步骤](#迁移步骤)
  - [1. 获取 Kimi API 密钥](#1-获取-kimi-api-密钥)
  - [2. 修改配置文件](#2-修改配置文件)
    - [2.1 更新 `docker-compose.yml` 文件](#21-更新-docker-composeyml-文件)
    - [2.2 修改代码中的 API 调用](#22-修改代码中的-api-调用)
  - [3. 处理 API 差异](#3-处理-api-差异)
- [验证配置](#验证配置)
  - [1. 检查环境变量](#1-检查环境变量)
  - [2. 测试聊天功能](#2-测试聊天功能)
- [注意事项](#注意事项)
- [常见问题](#常见问题)
- [参考资料](#参考资料)
- [许可证](#许可证)

---

## 简介

**Kimi API** 兼容了 OpenAI 的接口规范，您可以使用 OpenAI 提供的 Python 或 Node.js SDK 来调用和使用 Kimi 大模型。这意味着，如果您的应用和服务基于 OpenAI 的模型进行开发，那么只需要将 `base_url` 和 `api_key` 替换成 Kimi 大模型的配置，即可无缝地将您的应用和服务迁移至使用 Kimi 大模型。

本指南将详细阐述从 OpenAI 迁移到 Kimi API 的步骤，并提出可行的解决方案，帮助开发者顺利完成迁移工作。

---

## 环境要求

- **操作系统**：Linux、macOS 或 Windows（需支持 Docker）
- **Docker**：确保已安装 Docker 和 Docker Compose
- **Git**：用于克隆项目代码
- **微信账号**：用于登录和测试微信机器人

---

## 迁移步骤

### 1. 获取 Kimi API 密钥

- **注册账号**：访问 [Kimi 开放平台](https://kimi.com/)，注册并登录账户。
- **获取 API 密钥**：在开发者中心，生成并获取您的 **API 密钥**。
- **注意**：妥善保管您的 API 密钥，避免泄露。

### 2. 修改配置文件

#### 2.1 更新 `docker-compose.yml` 文件

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
      # 启用 Kimi API
      USE_KIMI: 'True'
      KIMI_API_KEY: 'YOUR_KIMI_API_KEY'  # 请替换为您的实际密钥
      API_BASE_URL: 'https://api.moonshot.cn/v1'  # Kimi API 的基础地址
      MODEL: 'kimi-v1'  # Kimi 提供的模型名称，请根据需要选择
      # 禁用其他模型
      USE_OPENAI: 'False'
      USE_ZHIPU_AI: 'False'
      USE_MOONSHOT: 'False'
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

- **`USE_KIMI`**：设置为 `'True'`，启用 Kimi API 服务。
- **`KIMI_API_KEY`**：替换为您的实际 API 密钥。
- **`API_BASE_URL`**：设置为 Kimi API 的基础地址。
- **`MODEL`**：指定使用的模型名称，如 `'kimi-v1'`。请根据 Kimi 提供的模型列表选择合适的模型。
- **禁用其他模型**：将 `USE_OPENAI`、`USE_ZHIPU_AI`、`USE_MOONSHOT` 设置为 `'False'`。
- **其他配置项**：根据您的需求进行调整。

#### 2.2 修改代码中的 API 调用

如果您的代码中直接使用了 OpenAI 的 SDK，需要进行以下修改：

**示例代码：**

```python
import openai

# 设置 API 密钥和 Base URL
openai.api_key = 'YOUR_KIMI_API_KEY'  # 替换为您的 Kimi API 密钥
openai.api_base = 'https://api.moonshot.cn/v1'  # 将 base_url 替换为 Kimi 的 API 基础地址

# 调用接口
response = openai.ChatCompletion.create(
    model='kimi-v1',  # 确认使用的模型名称
    messages=[
        {"role": "system", "content": "你是一个智慧的助手"},
        {"role": "user", "content": "你好，请介绍一下Kimi模型。"}
    ],
    temperature=0.7,
)

# 打印结果
print(response.choices[0].message['content'])
```

**注意事项：**

- **替换 `api_key` 和 `api_base`**：使用您的 Kimi API 配置替换 OpenAI 的配置。
- **模型名称**：确保 `model` 参数使用 Kimi 提供的模型名称。
- **保持其他参数不变**：Kimi API 兼容 OpenAI 的接口规范，大部分参数和调用方式相同。

### 3. 处理 API 差异

虽然 Kimi API 尽力保证与 OpenAI API 的兼容性，但在某些特殊场合，可能存在一些差异。请注意以下几点：

- **接口兼容性**：以下接口与 OpenAI 兼容，可直接使用：

  - `/v1/chat/completions`
  - `/v1/files`
  - `/v1/files/{file_id}`
  - `/v1/files/{file_id}/content`

- **参数支持**：部分参数可能在 Kimi API 中未被支持或有不同的默认值，请参考 [Kimi API 文档](https://kimi.com/documentation) 进行确认。

- **错误处理**：Kimi API 的错误代码和消息可能与 OpenAI 不同，建议在代码中添加异常处理，以捕获并处理可能的错误。

---

## 验证配置

### 1. 检查环境变量

进入容器内部，确认环境变量是否正确加载：

```bash
# 进入容器
docker exec -it chatgpt-on-wechat /bin/bash

# 查看环境变量
env | grep KIMI
```

应看到类似以下的输出：

```bash
USE_KIMI=True
KIMI_API_KEY=YOUR_KIMI_API_KEY
API_BASE_URL=https://api.moonshot.cn/v1
```

### 2. 测试聊天功能

- **在微信中与机器人对话**，发送消息，查看是否能够正常收到回复。
- **验证功能**：
  - 文本回复是否正常。
  - 其他功能（如图片生成、语音识别）是否正常。

---

## 注意事项

- **API 密钥安全**：
  - 请勿将您的 API 密钥公开分享，避免泄露。
  - 如果怀疑密钥泄露，建议立即在 Kimi 开放平台上重置密钥。

- **模型名称**：
  - 确保 `MODEL` 的值与 Kimi 提供的模型名称一致。
  - 可访问 Kimi 开放平台，查看可用的模型列表。

- **代理设置**：
  - 如果服务器无法直接访问外部网络，需要在 `PROXY` 中配置代理地址。
  - 代理格式示例：`http://127.0.0.1:7890`

- **配置生效**：
  - 每次修改 `docker-compose.yml` 文件或代码后，都需要重启容器以使更改生效。

---

## 常见问题

### 1. 程序提示未提供 `api_key`

**解决方案**：

- 确认 `KIMI_API_KEY` 已正确设置，并在 `docker-compose.yml` 中指定。
- 检查环境变量名称是否有拼写错误，名称需完全匹配，包括大小写。

### 2. 程序仍然尝试连接 OpenAI 的 API

**解决方案**：

- 确保已将 `USE_OPENAI` 设置为 `'False'`。
- 检查代码中是否仍有硬编码的 OpenAI API 地址，需替换为 Kimi 的 `API_BASE_URL`。

### 3. 无法访问 Kimi API 端点

**解决方案**：

- 检查服务器的网络连接，确保能够访问 `https://api.moonshot.cn/`。
- 如果需要代理，正确配置 `PROXY`。

### 4. 收到错误回复或功能异常

**解决方案**：

- 查看容器日志或程序输出，查找错误信息。
- 确认模型名称和参数设置是否正确。
- 检查 Kimi API 文档，了解可能的参数差异。

---

## 参考资料

- **ChatGPT-on-WeChat 项目 GitHub**：  
  [https://github.com/zhayujie/chatgpt-on-wechat](https://github.com/zhayujie/chatgpt-on-wechat)

- **Kimi 开放平台**：  
  [https://kimi.com/](https://kimi.com/)

- **Kimi API 文档**：  
  [Kimi API 使用指南](https://kimi.com/documentation)

- **OpenAI Python 库**：  
  [https://github.com/openai/openai-python](https://github.com/openai/openai-python)

- **Docker 官方文档**：  
  [https://docs.docker.com/](https://docs.docker.com/)

---

## 许可证

本项目遵循 [MIT 许可证](LICENSE)。您可以自由地使用、修改和分发本指南，但需保留原作者信息。

---

希望本指南能帮助您成功将 **ChatGPT-on-WeChat** 项目从 OpenAI 迁移到 **Kimi API**，并充分利用 Kimi 大模型的强大功能。如有任何问题，建议您：

- **提交 Issue**：在项目的 GitHub 页面提交问题，提供详细的错误日志和配置信息（注意隐藏敏感信息）。
- **联系 Kimi 支持**：通过官方渠道获取帮助。
