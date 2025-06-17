---
title: 浅尝Ollama
date: 2025-01-08 15:00:00
tags:
 - Ollama
 - Dify
---
# Ollama
## Ollama简介
Ollama是一个开源的LLM服务工具，可私有化部署。支持Llama3、Phi3、Mistral、Gemma、qwen2.5等开源的大语言模型。

Ollama支持的大语言模型：[https://ollama.com/library](https://ollama.com/library)

## Ollama下载&安装
官网下载：[https://ollama.com/](https://ollama.com/)

安装成功后，你就得到了一只小羊驼，双击运行即可。

## Ollama使用
```bash
# 拉取llama3.2并运行
ollama run llama3.2

# 拉取其他模型
ollama run qwen2.5

# 查看本地拉取的大模型
ollama list

# 查看正在运行的大模型信息
ollama show

# 查看服务状态
ollama ps

# 查看完整的命令列表
ollama --help
```

# Dify
## Dify简介
Dify 是一款开源的大语言模型(LLM) 应用开发平台。它融合了后端即服务（Backend as Service）和 LLMOps 的理念，使开发者可以快速搭建生产级的生成式 AI 应用。

Dify官网：[https://dify.ai/](https://dify.ai/)
## Dify安装
- 安装Docker，并配置镜像源
```json
{
  "builder": {
    "gc": {
      "defaultKeepStorage": "20GB",
      "enabled": true
    }
  },
  "experimental": false,
  "registry-mirrors": [ // 配置镜像源
    "https://docker.m.daocloud.io",
    "https://docker.1panel.live"
  ]
}
```
- 安装docker-compose
- git拉取源码：git clone https://github.com/langgenius/dify.git
- 运行dify
```bash
# 进入 dify 源代码的 docker 目录
cd dify/docker

# 复制环境配置文件
cp .env.example .env

# 查看docker-compose版本
docker-compose --version

# 启动docker容器
docker compose up -d # docker-compose v2
docker-compose up -d # docker-compose v1

# 运行效果如下，运行起来后，可以即可访问：http://localhost
xiaoming@MacAir docker % docker compose up -d
[+] Running 11/11
 ✔ Network docker_ssrf_proxy_network  Created                              0.0s 
 ✔ Network docker_default             Created                              0.0s 
 ✔ Container docker-weaviate-1        Sta...                               0.6s 
 ✔ Container docker-ssrf_proxy-1      S...                                 0.9s 
 ✔ Container docker-sandbox-1         Star...                              0.8s 
 ✔ Container docker-redis-1           Starte...                            0.7s 
 ✔ Container docker-db-1              Started                              0.8s 
 ✔ Container docker-web-1             Started                              0.7s 
 ✔ Container docker-api-1             Started                              1.2s 
 ✔ Container docker-worker-1          Start...                             1.2s 
 ✔ Container docker-nginx-1           Starte...                            1.4s 

# 查看服务状态
docker compose ps

# 停止服务
docker compose down
```

## 接入Ollama部署的本地模型
文档：[接入 Ollama 部署的本地模型](https://docs.dify.ai/zh-hans/development/models-integration/ollama)