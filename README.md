# GLM API 服务

支持高速流式输出、支持多轮对话、支持智能体对话、支持AI绘图、支持联网搜索、支持长文档解读、支持图像解析，多路token支持，自动清理会话痕迹。

与ChatGPT接口完全兼容。


## 目录

* [免责声明](#免责声明)
* [接入准备](#接入准备)
  * [多账号接入](#多账号接入)
* [Docker部署](#Docker部署)
* [使用说明](#使用说明)
* [注意事项](#注意事项)
  * [Nginx反代优化](#Nginx反代优化)
  
## 免责声明

**逆向API是不稳定的，建议前往智谱AI官方 https://open.bigmodel.cn/ 付费使用API，避免封禁的风险。**

**仅限自用，禁止对外提供服务或商用，避免对官方造成服务压力，否则风险自担！**

## 接入准备

从 [智谱清言](https://chatglm.cn/) 获取refresh_token

建议使用Chrome浏览器

注册或者登录智谱清言后发起一个对话，然后F12打开开发者工具，从Application > Cookies中找到`chatglm_refresh_token`的值，copy保存下来，这将作为API KEY使用。

### 多账号接入

目前智谱清言限制同个账号同时只能有*一路*输出，你可以通过提供多个账号的chatglm_refresh_token并使用`,`拼接提供：

`Authorization: Bearer TOKEN1,TOKEN2,TOKEN3`

每次请求服务会从中挑选一个。

## Docker部署

请准备一台具有公网IP的服务器并将8000端口开放。

拉取镜像并启动服务

```shell
docker run -it -d --init --restart=unless-stopped --name glm-api -p xxxx:8000 -e TZ=Asia/Shanghai snakeying/glm-api:V1
```
***xxxx为你选用来映射的端口，不可以被占用***


查看服务实时日志

```shell
docker logs -f glm-api
```

重启服务

```shell
docker restart glm-api
```

停止服务

```shell
docker stop glm-api
```

## 使用说明

支持与openai兼容的 `/v1/chat/completions` 接口，自行使用与openai或其他兼容的客户端接入接口。

## 注意事项

### Nginx反代优化

如果您正在使用Nginx反向代理glm-api，请添加以下配置项优化流的输出效果，优化体验感。

```nginx
# 关闭代理缓冲。当设置为off时，Nginx会立即将客户端请求发送到后端服务器，并立即将从后端服务器接收到的响应发送回客户端。
proxy_buffering off;
# 启用分块传输编码。分块传输编码允许服务器为动态生成的内容分块发送数据，而不需要预先知道内容的大小。
chunked_transfer_encoding on;
# 开启TCP_NOPUSH，这告诉Nginx在数据包发送到客户端之前，尽可能地发送数据。这通常在sendfile使用时配合使用，可以提高网络效率。
tcp_nopush on;
# 开启TCP_NODELAY，这告诉Nginx不延迟发送数据，立即发送小数据包。在某些情况下，这可以减少网络的延迟。
tcp_nodelay on;
# 设置保持连接的超时时间，这里设置为120秒。如果在这段时间内，客户端和服务器之间没有进一步的通信，连接将被关闭。
keepalive_timeout 120;
```

