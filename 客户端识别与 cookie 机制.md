#### 1. 个性化接触

HTTP  最初是一个匿名、无状态的请求/响应协议。服务器处理来自客户端的请求，然后向客户端回送一条响应。Web 服务器几乎没有什么信息用来判定是哪个用户发送的请求，也无法记录来访用户的请求序列。

用户识别机制

- 承载用户身份信息的 HTTP 首部
- 客户端 IP 地址跟踪，通过用户的 IP 地址对其进行识别。
- 用户登录，用认证方式来识别用户。
- 胖 URL，一种在 URL 中嵌入识别信息的技术
- cookie，一种功能强大且高效的持久身份识别技术

##### HTTP 首部

| 首部名称 | 首部类型 | 描述 |
| :---: | :---: | :---: |
| From | 请求 | 用户的 E-mail 地址|
| User-Agent | 请求 | 用户的浏览器软件 |
| Referer | 请求 | 用户是从这个页面上依照链接跳转过来的 |
| Authorization | 请求 | 用户名和密码 |
| Client-IP | 扩展（请求） | 客户端的 IP 地址 |
X-Forwarded-For | 扩展（请求） | 客户端的 IP 地址 |
| Cookie | 扩展（请求） | 服务器产生的 ID 标签 |


**from** 首部包含了用户的 E-mail 地址。每个用户都有不同的 E-mail 地址，所以在理想情况下，可以将这个地址作为可行的源端来识别用户。但由于担心哪些不讲道德的服务器会手机这些 Email 地址，用户垃圾邮件的散发，所以很少有浏览器会发送 **From** 首部。实际上 **From** 首部是由自动化的机器人或蜘蛛发送的，这样在出现问题时，网管还有个地方可以发送愤怒的投诉的邮件。

**User-Agent** 首部可以将用户所用浏览器的相关信息告知服务器，包括程序的名称和版本，通常还包含操作系统的相关信息。

**Referer** 首部提供了用户的来源页面的 URL。Referer 首部自身并不能完全标识用户，但它确实说明了用户之前访问过那个页面。

**From**、**User-Agent** 和 **Referer** 首部都不足以实现可靠的识别

----

##### 客户端 IP 地址

通常在 HTTP 首部并不提供客户端 IP 地址，但 web 服务器可以找到承载 HTTP 请求的 TCP 连接的另一端的 IP 地址。

在 Unix 系统中，函数调用 getpeername 就可以返回发送端机器的客户端 IP 地址：

`status = getpeername(tcp_connection_socket, ...)` 

使用客户端 IP 地址来识别用户存在着很多缺点，限制了将其作为用户识别技术的效能。

- 客户端 IP 地址描述的是所用的机器，而不是用户。
- 很多英特网服务提供商都会在用户登录时为其动态分配 IP 地址。
- 为了提供安全性，并对稀缺的地址资源进行管理，很多用户都是通过网络地址转换（NAT）防火墙来浏览网络内容的。这些 NAT 设备隐藏了防火墙后面哪些实际客户端的 IP 地址，将实际的用户 IP 地址转换成了一个共享的防火墙 IP 地址（和不同的端口号）。
- HTTP 代理和网关通常会代开一些新的、到原始服务器的 TCP 连接。Web 服务器看到的将是代理服务器的 IP 地址，而不是客户端的。有些代理为了绕过这个问题会添加特殊的 Client-IP 或 X-Forwarded-For 扩展首部来保存原始的 IP 地址。但并不是所有的代理都支持这种行为。

少数站点甚至将客户端 IP 地址作为一种安全特性使用，他们只向来自特定 IP 地址的用户提供文档。在内部网络中可能可以这么做，但是在因特网中就不行了，主要是应为因特网上 IP 地址太容易被欺骗（伪造）了。路径上如果有拦截代理优惠破坏此方案。

---

####  胖 URL 
 有些 web 站点会为每个用户生成特定版本的 URL 来追踪用户的身份。通常会对真正的 URL 进行扩展，在 URL 路径开始或结束的地方添加一些状态信息。。用户浏览站点时，Web 服务器会动态生成一些超链，继续维护 URL 中的状态信息。

可以通过 胖 URL 将 Web 服务器上若干个独立的 HTTP 事务捆绑成一个 “会话” 或 “访问” 。用户首次访问这个 Web 站点时，会生成一个唯一的 ID，用服务器可以识别的方法将这个 ID 添加到 URL 中去，然后服务器就会将客户端重新导向这个胖 URL。无论什么时候，只要服务器收到了对胖 URL 的请求，就可以去查找与那个用户 ID 相关的所有增量状态，然后重写所有的输出超链，使其成为胖 URL，已维护用户的 ID。

这种技术存在几个严重的问题：

- 丑陋的 URL   
    浏览器中显示的胖 URL 会给新用户带来困扰

- 无法共享 URL     
    胖 URL 中包含了与特定用户和会话相关的状态信息。

- 破坏缓存  
    为每个 URL 生成用户特有的版本就意味着不再有可供公共访问的 URL 需要缓存了

- 额外的服务器负荷  
    服务器需要重写 HTML 页面使 URL 变胖

- 逃逸口   
    用户跳转到其他站点或者请求一个特定的 URL 时，就很容易在无意中 “逃离” 胖 URL 会话。只有用户严格地追随预先修改过的链接时，胖 URL 才能工作。

- 在会话间是非持久的     
    除非用户收藏了特定的胖 URL，否则用户退出登录时，所有信息都会丢失。

---

#### cookie

cookie 是当前识别用户，实现持久会话的最好方式。cookie 是由网景公司开发的。cookie 的存在也影响了缓存，大多数缓存和浏览器都不允许对任何 cookie 的内容进行缓存。

1. cookie 的类型   

    - 会话 cookie: 记录了用户访问站点时的设置和偏好，用户退出浏览器时，会话 cookie 就被删除了。
    - 持久 cookie: 生存时间更长一些，存储在硬盘上，浏览器退出，计算机重启时他们仍然存在。通常会用持久 cookie 维护某个用户会周期性访问的站点的配置文件或登录名。

    两者的区别：它们的过期时间。如果设置了 Discard 参数，或者没有设置 Expires 或 Max-Age 参数来说明扩展的过期时间，这个 cookie 就是一个会话 cookie。

2. cookie 是如何工作的

    Web 服务器对首次访问的用户一无所知，希望这个用户会再次回来，所以就在这个用户 “拍上” 一个独有的 cookie，这样以后他就可以识别出这个用户了。cookie 中包含了一个由名字 = 值（name=value）这样的信息构成的任意列表，并通过 Set-Cookie 或 Set-Cookie2 HTTP 响应（扩展）首部将其贴到用户身上去。

    cookie 中可以包含任意信息，但它们通常都只包含一个服务器为了进行追踪而产生的独特的识别码。

3. cookie 罐：客户端的状态
    cookie 的基本思想就是让浏览器积累一组服务器特有的信息，每次访问服务器时都将这些信息提供给他。因为浏览器要负责存储 cookie 信息，所以此系统就被称为客户端测状态（client-side state）。不同的浏览器会以不同的方式存储 cookie。

##### 不同站点使用不同的 cookie 

浏览器不会将每个 cookie 都发送给所有的站点。实际上，通常只向每个站点发送 2~3 个 cookie。原因：

- 对所有这些 cookie 字节进行传输会严重降低性能。浏览器实际传输的 cookie 字节数要比实际的内容字节多
- cookie 中包含的是服务器特有的明值对，所以对大部分站点来说，大多数 cookie 都只是无法识别的无用数据。
- 将所有的 cookie 发送给所有站点只会引发潜在的隐私问题。

总之，浏览器只向服务器发送服务器产生的那些 cookie。

1. cookie 的域属性  
    产生 cookie 的服务器可以向 Set-Cookie 响应首部添加一个 Domain 属性来控制哪些站点可以看到那个 cookie。比如，下面的 HTTP 响应首部就是在告诉浏览器将 cookie user="mary17" 发送给域 ".airtravelgains.com" 中所有的站点：

    ``Set-cookie: user="mary17"; domain="airtravelbargains.com"``

    用户访问的是 www.airtravelbargains.com 或者 specials.airtravelbargains.com 或任意已 .airtravelbargains.com 结尾的站点。下面 Cookie首部都会被发布出去：

    ``Cookie: user="mary17``

2. cookie路径属性   
    cookie 规范甚至允许用户将 cookie 与部分 Web 站点关联起来。可以通过 Path 属性来实现这一功能，在这个属性列出的 URL 路径前缀下所有的 cookie 都是有效的。

    某个 Web 服务可能是由两个组织共享的，每个组织都有独立的 cookie，站点 www.airtravelbargains.com 可能会将部分 Web 站点用于汽车租赁--比如，https://www.airtravelbargains.com/autos/ 用一个独立的 cookie 来记录用户喜欢的汽车尺寸。可能会生成一个如下所示的特殊汽车租赁 cookie:

    ``Set-cookie: pre=compact; domain="airtravelbargains.com"; path=/autos/``

    用户访问 http://www.airtravelbargains.com/special.html 就只会得到这个 cookie: 
    
    ``Cookie: user="mary17"``

    但如果访问 http://www.airtravelbargains.com/autos/cheapo.index.html，就会得到这两个 cookie：

```
    Cookie: user="mary17"    
    Cookie: pre=compact
```

---

####cookies 版本0
定义了 Set-Cookie 响应首部、cookie 请求首部以及用于控制 cookie 的手段。

```
Set-Cookie: name=value[; expires=date] [; path=path] [; domain=domain] [; secure]

Cookie: name1=value1 [; name2=value2] ...
```

| Set-Cookie 属性 | 描述及实例 |
| :---: | : --- |
| NAME=VALUE | 强制的，NAME 和 VALUE 都是字符序列，除非包含在双引号内，否则不包含分号、逗号、等号和空格。Web 服务器可以创建任意的 NAME=VALUE 关联，在后继站点的访问中会将其送回 Web 服务器: <br>Set-Cookie: customer=Mary |
| Expires | 可选的，这个属性会指定一个日期字符串，用来定义 cookie 的实际生存期，一旦到了过期日期，就不再存储或发布这个 cookie 了。日期的格式为： Weekday, DD-Mon-YY HH:MM:SS GMT <br> 唯一合法的时区为 GMT，日期元素之间的分隔符一定要是长划线。如果没有指定 Expires,cookie 就会在用户会话结束时过期：<br>Set-Cookie: foo-bar; expires=Wednesday, 09-Nov-99 23:12:40 GMT |
| Domain | 可选的，浏览器只向指定域中的服务器主机名发送 cookie，这样的服务器就将 cookie 限制在了特定的域中。acme.com 域就与 anvil.acme.com 和 shipping.crate.acme.com 相匹配，但与 www.cnn.com 就不匹配了。<br>只有指定域中主机才能为一个域设置 cookie，这些域中至少有两个或三个句号，以防止出现 .com、.edu 和 va.us 等形式的域。<br> 如果没有指定域，就默认为产生 Set-Cookie 响应的服务器的主机名: <br> Set-Cookie: SHIPPING=FEDEX; domain="joes-hardware.com" 
|  Path  | 可选的，通过这个属性可以为服务器上特定的文档分配 cookie，如果 Path 属性是一个 URL 路径前缀，就可以为附加一个 cookie。路径 /foo 与 /foorbar 和 /foo/bar.html 相匹配。路径 "/" 与域名中所有内容都匹配。<br> 如果没有指定路径，就将其设置为产生 Set-Cookie 响应的 URL 的路径:<br>Set-Cookie: lastorder=00183; path=/orders |
| Secure | 可选的。如果包含了这个属性，只有在 http 使用 SSL 安全连接时才会发送 cookie: <br> Set-Cookie: prevate_id=519; secure


---

#### cookie与缓存

缓存那些与
