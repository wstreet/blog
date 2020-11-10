---
title: Http缓存
date: 2020-11-10 21:25:53
tags:
categories:
---


<a name="riul2"></a>
## 为什么需要缓存
在前后端通过Http通信过程中，有些资源的内容是没有发生改变的，这些资源如果不缓存的话，就会造成资源浪费，以及资源加影响性能。
<a name="F5E3x"></a>
## 缓存分类
Http缓存分为两类，强缓存和协商缓存，检测顺序是：先检测强缓存，强缓存有并且未过期，则走强缓存，否则检测协商缓存，协商缓存有并且未过期，则走协商缓存，否则从服务器端重新加载资源。
<a name="Enmsp"></a>
### 强缓存
请求命中强缓存时，不会将请求发送给服务端，状态码是200，size显示from cache，强缓存是利用响应头中的Expires或者Cache-Control来控制的
<a name="Gt3Fr"></a>
#### Expires
Expires是HTTP/1.0中的，它的值是一个GMT格林威治标准时间，代表该资源过期的时间。但是存在 一个问题，如果把客户端 时间往后设置，那资源就不会过期，所以在HTTP/1.1新增了Cache-Cotrol<br />

<a name="aoJ9q"></a>
#### Cache-Cotrol
缓存请求指令

| 指令 | 参数 | 说明 |
| --- | --- | --- |
| no-cache | 无 | 强制向源服务器再次验证 |
| no-store | 无 | 不缓存请求或者响应的任何内容 |
| max-age=[秒] | 必须 | 响应的最大Age值 |
| max-stale(=[秒]) | 可省略 | 接收已过期的响应 |
| min-fresh=[秒] | 必须 | 期望在指定时间内的响应任然有效 |
| no-transform | 无 | 代理不可更改媒体类型 |
| only-if-cached | 无 | 从缓存获取资源 |
| cache-extension | - | 新指令标记 |

缓存响应指令

| 指令 | 参数 | 说明 |
| --- | --- | --- |
| public | 无 | 可向任意方提供响应的缓存 |
| private | 可省略 | 仅向特定用户返回响应 |
| no-cache | 可省略 | 缓存前必须确认其有效性 |
| no-store | 无 | 不缓存请求或者响应的任何内容 |
| no-transform | 无 | 代理不可更改媒体类型 |
| must-revalidate | 无 | 可缓存但必须再向源服务器进行确认 |
| proxy-revalidate | 无 | 要求中间缓存服务器对缓存的响应有效性再进行确认 |
| max-age=[秒] | 必须 | 响应的最大Age值 |
| s-maxage=[秒] | 必须 | 公共缓存服务器响应的最大Age值 |
| cache-extension | - | 新指令标记 |

<a name="y6Si2"></a>
### 协商缓存
当强缓存过期后，客户端发起的请求会去和服务器交互协商（请求头中存在以下两组信息），确认协商缓存资源是否有效，如果有效，则返回状态码304，客户端从缓存池加载缓存资源；如果资源过期，则服务端重新返回资源并根据请求头判断是否将资源加入缓存池，状态码200。控制协商缓存的header字段有两组
<a name="Tlrrg"></a>
#### Last-Modified/If-Modified-Since
Last-Modified和If-Modified-Since都是标准的HTTP头标签，代表资源最后修改时间，Last-Modified由服务器发送给客户端，存在于Response Headers中；If-Modified-Since是由客户端发送给服务器，存在Request Headers中，服务器会拿If-Modified-Since的时间去对比资源最后修改时间，如果If-Modified-Since的时间和资源最后修改时间一致，说明资源没有过期；如果不一致，说明过期了，这时候服务端会重新给客户端返回一份最新的资源，并在Response Headers中返回最后Last-Modified：修改时间

但是这种协商缓存策略有一些缺点，比如：标识修改时间只能到秒，如果在1秒内多次修改某一文件，则在客户端不会更新该文件，因为检测不出来。

<a name="PJ0HE"></a>
#### Etag/If-None-Match
Last-Modified/If-Modified-Since是根据最后修改时间来判断缓存资源是否有效的，而Etag/If-None-Match是根据文件内容判断的，首先根据文件内容生成一个唯一标识，比如：ETag: "50b1c1d4f775c61:df3"，向客户端返回，客户端将资源缓存到缓存池，当再次发起请求时，请求头中会携带If-None-Match: "50b1c1d4f775c61:df3"，服务器接收到请求时根据这个标识判断资源是否被修改。

