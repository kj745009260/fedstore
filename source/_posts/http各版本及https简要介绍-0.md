---
title: http各版本及https简要介绍
date: 2021-03-01 17:53:40
tags: http协议
categories:
  - http
---

## __HTTP优化：__

影响一个 HTTP 网络请求的因素主要有两个方面：带宽和延迟。

随着网络基础建设的完善，带宽因素已经不需要再考虑，仅需要考虑的就是延迟。延迟主要受三个方面影响：浏览器阻塞（HOL blocking）, DNS查询（DNS Lookup）,建立连接（Initial connection）.

## __HTTP1.1__

> + 支持长连接。
> + 在HTTP1.0的基础上引入了更多的缓存控制策略。
> + 引入了请求范围设置，优化了带宽。
> + 在错误通知管理中新增了错误状态响应码。
> + 增加了Host头处理，可以传递主机名（hostname）。

缺点: 传输内容是明文，不够安全

## __HTTPS__

> + HTTPS运行在安全套接字协议(Secure Sockets Layer，SSL )或传输层安全协议（Transport Layer Security，TLS）之上，所有在TCP中传输的内容都需要经过加密。
> + 连接方式不同，HTTP的端口是80，HTTPS的端口是443.
> + HTTPS可以有效防止运营商劫持。

注: SSL运行在TCP之上

## __HTTP2.0（SPDY的升级版）__

> + HTTP2.0支持明文传输，而HTTP 1.X强制使用SSL/TLS加密传输。
> + 和HTTP 1.x使用的header压缩方法不同。
> + HTTP2.0 基于二进制格式进行解析，而HTTP 1.x基于文本格式进行解析。
> + 多路复用，HTTP1.1是多个请求串行化单线程处理，HTTP 2.0是并行执行，一个请求超时并不会影响其他请求。

### __HTTP2.0的多路复用提升了网页性能：__

> + 在 HTTP1 中浏览器限制了同一个域名下的请求数量（Chrome下一般是六个），当在请求很多资源的时候，由于队头阻塞，当浏览器达到最大请求数量时，剩余的资源需等待当前的六个请求完成后才能发起请求。
> + HTTP2 中引入了多路复用的技术，这个技术可以只通过一个 TCP连接就可以传输所有的请求数据。多路复用可以绕过浏览器限制同一个域名下的请求数量的问题，进而提高了网页的性能。

注意: 
> + 主流浏览器只支持基于TLS部署的HTTP2.0协议，所以要将网站升级为HTTP 2.0，就需要先升级为HTTPS。
> + HTTP 2.0完全兼容HTTP 1.x,所以对于部署了HTTP 2.0的网站可以自动向下兼容HTTP 1.X.

## __HTTP 3.0 (QUIC)__

QUIC (Quick UDP Internet Connections), 快速 UDP 互联网连接。

QUIC是基于UDP协议的。

两个主要特性：

### __1.线头阻塞(HOL)问题的解决更为彻底__

> 基于TCP的HTTP/2，尽管从逻辑上来说，不同的流之间相互独立，不会相互影响，但在实际传输方面，数据还是要一帧一帧的发送和接收，一旦某一个流的数据有丢包，则同样会阻塞在它之后传输的流数据传输。而基于UDP的QUIC协议则可以更为彻底地解决这样的问题，让不同的流之间真正的实现相互独立传输，互不干扰。

### __2.切换网络时的连接保持__

> 当前移动端的应用环境，用户的网络可能会经常切换，比如从办公室或家里出门，WiFi断开，网络切换为3G或4G。基于TCP的协议，由于切换网络之后，IP会改变，因而之前的连接不可能继续保持。而基于UDP的QUIC协议，则可以内建与TCP中不同的连接标识方法，从而在网络完成切换之后，恢复之前与服务器的连接。
