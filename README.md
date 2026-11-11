# 使用 cURL 发送 HTTP 标头

[![Bright Data 推广](https://github.com/bright-cn/Awesome-Web-Scraping/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner%20CN.png)](https://www.bright.cn/)

本指南将向你展示如何有效地结合 cURL 使用 HTTP 标头，以提升数据采集和服务器通信能力：

- [了解 HTTP 标头](#了解-http-标头)
- [开始使用 cURL 标头](#开始使用-curl-标头)
- [查看默认 cURL 标头](#查看默认-curl-标头)
- [使用 -H 修改默认标头](#使用--h-修改默认标头)
- [创建自定义标头](#创建自定义标头)
- [使用空标头](#使用空标头)
- [删除标头](#删除标头)
- [一次发送多个标头](#一次发送多个标头)
- [总结](#总结)

## 了解 HTTP 标头

超文本传输协议（HTTP）是一种遵循客户端-服务器架构的[无状态协议](https://byjus.com/gate/difference-between-stateless-and-stateful-protocol/)，客户端发出请求并等待服务器响应。这些请求包含 HTTP 方法、服务器位置、路径、查询参数和标头等重要元素。

HTTP 标头本质上是在客户端与服务器之间传输元数据和指令的键值对。它们在指定内容类型、缓存规则和身份验证方法等参数方面发挥着关键作用，确保客户端与服务器顺畅、安全地交互。在[网页爬虫](https://www.bright.cn/blog/how-tos/what-is-web-scraping)操作中，HTTP 标头使你能够根据网站要求和协议，通过模拟不同的 User-Agent、管理内容协商和处理身份验证来自定义请求。

HTTP 标头在网页爬虫中的常见应用包括更改 User-Agent（UA）、指定响应格式、执行条件请求，以及通过应用程序编程接口（API）进行身份验证。

## 开始使用 cURL 标头

在继续本教程之前，请在终端中执行以下命令，确保你的系统已安装 curl：

```sh
curl --version
```

如果安装正确，你将收到类似以下内容的版本信息：

```
curl 7.55.1 (Windows) libcurl/7.55.1 WinSSL
Release-Date: [unreleased]
Protocols: dict file ftp ftps http https imap imaps pop3 pop3s smtp smtps telnet tftp
Features: AsynchDNS IPv6 Largefile SSPI Kerberos SPNEGO NTLM SSL
```

如果遇到 `curl is not recognized as an internal or external command, operable program or batch file` 或 `command not found` 等错误，则需要[安装 curl](https://curl.se/download.html)。

你还需要一个用于检查标头的服务，例如 [httpbin.org](http://httpbin.org/)，它提供简单易用的 HTTP 请求和响应服务。

如果你熟悉 curl，就会知道它的基本语法遵循以下模式：

```sh
curl [options] [url]
```

这意味着，要从 `mywebpage.com` 获取内容，你需要运行：

```sh
curl www.mywebpage.com
```

## 查看默认 cURL 标头

要使用 httpbin.org 检查 curl 默认发送的标头，请执行以下命令：

```sh
curl http://httpbin.org/headers
```

响应将显示已发送的标头：

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd2eb0-0617353714d52f3777c9c267"
  }
```

curl 请求默认包含 `Accept`、`Host` 和 `User-Agent` 标头。

`Accept` 标头会告知服务器客户端可以处理哪些媒体类型。它传达客户端接受的内容类型，从而帮助客户端与服务器进行内容协商。

表示客户端偏好 JSON 的 `Accept` 标头如下所示：

```
Accept: application/json
```

`User-Agent` 字段包含你的客户端信息，在本例中是 curl 应用程序及其版本号（与你安装的版本一致）。

`Host` 标头用于标识 HTTP 请求对应的具体 Web 域名（主机）和端口号。如果未指定端口，则使用默认端口（HTTP 使用端口 80，HTTPS 使用端口 443）。

`X-Amzn-Trace-Id` 不是 curl 的默认标头，但它表示你的请求经过了 Amazon Web Services（AWS）服务（例如 AWS 负载均衡器）的路由，并且可用于追踪 HTTP 请求。

要确认 curl 默认发送哪些标头，可以使用带有 `-v` 或 `--verbose` 标志的详细模式，该模式会显示详细的请求和响应信息，包括标头。

运行以下命令以查看 curl 的默认标头：

```sh
curl -v http://httpbin.org/headers
```

你的输出将类似于：

```
- Trying 50.16.63.240...
* TCP_NODELAY set
* Connected to httpbin.org (50.16.63.240) port 80 (#0)
> GET /headers HTTP/1.1
> Host: httpbin.org
> User-Agent: curl/7.55.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Fri, 22 Mar 2024 07:18:00 GMT
< Content-Type: application/json
< Content-Length: 173
< Connection: keep-alive
< Server: gunicorn/19.9.0
< Access-Control-Allow-Origin: *
< Access-Control-Allow-Credentials: true
<
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd30a8-624365ad52781957578cd5b1"
  }
}
* Connection #0 to host httpbin.org left intact
```

以大于号（>）开头的行表示客户端（curl）发送到端点的内容，并确认传输了以下标头：

- 发送到端点 /headers 的 `GET`（HTTP 方法）
- 值为 `httpbin.org` 的 `Host`
- 值为 `curl/7.55.1` 的 `User-Agent`
- 值为 `*/*` 的 `Accept`

在输出中，以小于号（<）开头的行（例如 `< Content-Type: application/json`）表示响应标头。

## 使用 -H 修改默认标头

`-H` 或 `--header` 标志允许你向服务器发送自定义标头，可用于测试。

例如，要将 `User-Agent` 从 `curl/7.55.1` 更改为 `Your-New-User-Agent`，请使用：

```sh
curl -H "User-Agent: Your-New-User-Agent" http://httpbin.org/headers
```

响应将显示：

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "User-Agent": "Your-New-User-Agent",
    "X-Amzn-Trace-Id": "Root=1-65fd5123-3ebe566a4681427c6996c72c"
  }
}
```

如果你想将 `Accept` 标头从 `*/*`（接受任意内容类型）更改为 `application/json`（仅接受 JSON 内容），请运行：

```sh
curl --header "Accept: application/json" http://httpbin.org/headers
```

输出将为：

```json
{
  "headers": {
    "Accept": "application/json",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd55c3-05c21f81770c1c5e6343b1fc"
  }
}
```

> **注意：**
>
> 在此示例中，使用的是 `--header`，而不是 `-H`。这些标志等效，执行相同的功能。

从 curl 7.55.0 版开始，你还可以使用包含标头的文件。如果标头文件名为 `header_file`，可以使用：

```sh
curl -H @header_file
```

## 创建自定义标头

自定义标头是由开发者定义的字段，用于提供标准 HTTP 标头之外的额外信息。

要使用 curl 发送自定义标头，请使用 `-H` 标志。例如，要发送名为 `My-Custom-Header`、值为 `Value of custom header` 的自定义标头，请执行：

```sh
curl -H "My-Custom-Header: Value of custom header" http://httpbin.org/headers
```

响应将为：

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "My-Custom-Header": "Value of custom header",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd7d2a-3b683be160ff2965023b3a31"
  }
}
```

## 使用空标头

有时需要发送空标头，例如为了满足特定 API 的要求，即使没有内容，也必须包含某些标头。例如，[HTTP 严格传输安全（HSTS）标头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)会强制网站使用安全的 HTTPS 连接。虽然此标头通常包含有关 HSTS 持续时间和行为的指令，但发送空值可以确保立即实施 HSTS。

空标头还可用于清除之前设置的标头。要重置或清除默认设置的标头，发送空标头即可有效移除其值。

要使用 curl 发送空标头，请在标头名称后添加分号，以表示空值。以下命令展示了如何发送名为 `My-Custom-Header` 的空自定义标头：

```sh
curl -H "My-Custom-Header;" http://httpbin.org/headers
```

输出显示 `My-Custom-Header` 的值为空：

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "My-Custom-Header": "",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd84e2-7a42d9d62a42741e448c426f"
  }
}
```

## 删除标头

要使用 curl 完全删除某个标头，请在标头名称后添加冒号，且不要在冒号后指定任何值。

例如，要移除默认的 `User-Agent` 标头，请使用：

```sh
curl -H "User-Agent:" http://httpbin.org/headers
```

响应中不会包含 `User-Agent` 标头，确认它已被移除：

```json
{
  "headers": {
    "Accept": "*/*",
    "Host": "httpbin.org",
    "X-Amzn-Trace-Id": "Root=1-65fd862d-13b181583501ae11046374a1"
  }
}
```

## 一次发送多个标头

到目前为止，我们查看的都是单个标头示例，但 curl 支持同时发送多个标头。只需在命令中添加多个 `-H` 标志即可。

例如，要发送两个标头（`Custom-Header-1` 和 `Custom-Header-2`），其值分别为 `one` 和 `two`，请运行：

```sh
curl -H "Custom-Header-1: one" -H "Custom-Header-2: two" http://httpbin.org/headers
```

输出将显示：

```json
{
  "headers": {
    "Accept": "*/*",
    "Custom-Header-1": "one",
    "Custom-Header-2": "two",
    "Host": "httpbin.org",
    "User-Agent": "curl/7.55.1",
    "X-Amzn-Trace-Id": "Root=1-65fd8781-143be3502c559bc5605fc6f1"
  }
}
```

## 总结

本文介绍了 HTTP 标头的基础知识，并演示了如何使用 curl 对其进行有效管理。

如果你需要全面的网页爬虫解决方案，可以考虑 Bright Data。Bright Data 提供专业工具和服务，包括可增强匿名性并防止 IP 被封禁的[代理服务](https://www.bright.cn/proxy-types)，以及可帮助访问受地理位置限制的内容且无需处理验证码的[网络解锁器](https://www.bright.cn/products/web-unlocker)。

立即开始免费试用！