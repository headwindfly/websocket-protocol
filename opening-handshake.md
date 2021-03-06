##4.1.客户端要求
要_建立WebSocket连接_，客户端打开一个连接并发送一个握手，就像本节中定义那样。一个连接最初被定义为一个CONNECTING状态。客户端将需要提供一个/host/、/port/、/resource name/、和/secure/标记，它们都是在第三章讨论的WebSocket URI的组件，连同一起使用的一个/protocols/和/extensions/列表。此外，如果客户端是一个web浏览器，它提供/origin/。客户端运行在一个受控环境，例如绑定到特定运营商的手机上的浏览器，可以下移（offload）连接管理到网络上的另一个代理。在这种情况下，用于本规范目的的客户端被认为包括手机软件和任何这样的代理。

当客户端要_建立一个WebSocket连接_，给定一组（/host/、/port/、/resource name/、和/secure/标记）、连同一起使用的一个/protocols/和/extensions/列表、和在web浏览器情况下的一个/origin/，它必须打开一个连接、发送一个打开阶段握手、并读取服务器响应中的握手。应如何打开连接的确切要求、在打开阶段握手应发送什么、以及应如何解释服务器响应，在本节如下所述。在下面的文本中，我们将使用第三章的术语，如定义在那章的“/host” 、和“/sucure/标记” 。

1. 传入该算法的WebSocket URI组件(/host/、/port/、/resource name/、和/secure/ 标记) 根据指定在第3章的WebSocket URI规范，必须是有效的。如果任何组件是无效的，客户端必须_失败WebSocket连接_并终止这些步骤。

2. 如果客户端已经有一个到通过主机/host/和端口/port/对标识的远程主机（IP地址）的WebSocket连接，即使远程主机是已知的另一个名字，客户端必须等待直到连接已建立或由于连接已失败。必须不超过一个连接处于CONNECTING 状态。如果同时多个连接到同一个IP地址，客户端必须 序列化它们，以致一次不多于一个连接在以下步骤中运行。

    如果客户端不能决定远程主机的IP地址（例如，因为所有通信是通过代理服务器本身进行DNS查询），那么客户端必须假定这步的目的是每一个主机名引用一个不同远程主机，且相反，客户端应该限制同时挂起的连接总数为一个适当低的数（例如，客户端可能允许到a.example.com和b.example.com同时挂起连接，但如果30个同时连接到同一个请求的主机，那可能是不允许的）。例如，在一个web浏览器上下文中，客户端需要考虑用户已经打开的标签数量，在设置同时挂起的连接数量的限制时。

    注意：这使得它很难仅通过打开大量的WebSocket连接到远程主机为脚本执行一个拒绝服务攻击。当攻击在关闭连接之前被暂停时，服务器可以进一步降低自身的负载，因为这将降低客户端重新连接的速度。

    注意：没有限制一个客户端可以与单个远程主机有的已建立的WebSocket连接数量。服务器可以拒绝接受来自具有大量的现有连接的主机/IP地址的连接或当遭受高负载时断开占用资源的连接。

3.  _使用代理_：当有WebSocket协议连接主机/host/和端口/port/时，如果客户端配置使用代理，那么客户端应该连接到代理并要求它打开一个到由/host/给定主机和/port/给定端口的TCP连接。
    例子：例如，如果客户端为所有信息传输使用一个HTTP代理，那么如果它试图连接到服务器example.com的80端口，它可能会发送以下行到代理服务器：

        CONNECT example.com:80 HTTP/1.1
        Host: example.com

    如果还有密码，连接可能看起来像：

        CONNECT example.com:80 HTTP/1.1
        Host: example.com
        Proxy-authorization: Basic ZWRuYW1vZGU6bm9jYXBlcyE=

    如果客户端没有配置使用一个代理，那么应该打开一个直接TCP连接到由/host/给定的主机和/port/给定的端口。

    注意：不暴露明确的UI来为WebSocket连接选择一个独立于其他代理的代理实现，鼓励使用SOCKS5[[RFC1928](http://tools.ietf.org/html/rfc1928)]代理用于WebSocket连接，如果有的话，或做不到这一点，选择为HTTPS连接配置代理胜过为HTTP连接配置代理。

    为了代理自动配置脚本，传给函数的URI必须从/host/、/port/、/resource name/、和/secure/标记来构造，使用第三章给定的WebSocket URI定义。

    注意：WebSocket协议可以在代理自动配置脚本中从模式中识别（“ws”用于未加密的连接和“wss”用于加密的连接）。

4. 如果连接无法打开，或者因为直接连接失败或者因为任何使用的代理返回一个错误，那么客户端必须_失败WebSocket连接_并终止连接尝试。

5. 如果/secure/是true，客户端必须在连接之上执行一个TLS握手在打开连接之后和发生握手数据之前[[RFC2818](http://tools.ietf.org/html/rfc2818)]。如果这个失败了（例如，服务器的证书不能被验证），那么客户端必须_失败WebSocket连接_并终止连接。否则，所有在该通道上的进一步的通信必须通过加密隧道[[RFC5246](http://tools.ietf.org/html/rfc5246)]。

客户端必须在TLS握手中使用服务器命名指示（Server Name Indication）扩展[[RFC6066](http://tools.ietf.org/html/rfc6066)]。

一旦一个到服务器的连接连接（包括通过代理或在TLS加密隧道之上的连接），客户端必须发送一个打开阶段握手到服务器。该握手包括一个HTTP Upgrade请求，连同一个必需的和可选的头字段列表。该握手的要求如下所示。

1. 握手必须是像[[RFC2616](http://tools.ietf.org/html/rfc6066)]指定的那样的有效的HTTP请求。

2. 请求方法必须是GET、且HTTP版本必须是至少1.1。

    例如，如果WebSocket URI是“ws://example.com/chat”，发送的第一行应该是“GET /chat HTTP/1.1”。

3. 请求的“Request-URI”部分必须匹配定义在第三章的（一个相对URI）/resource name/或是一个绝对的http/https URI，当解析时，有一个/resource name/、/host/、和/port/匹配相应的ws/wss URI。

4. 请求必须包含一个|Host|头字段，其值包含/host/加上可选的“:”后跟/port/（当没用默认端口时）。

5. 请求必须包含一个|Upgrade|头字段，其值必须包含“websocket”关键字。

6. 请求必须包含一个|Connection|头字段，其值必须包含“Upgrade”标记。

7. 请求必须包含一个名字为| Sec-WebSocket-Key |的头字段，这个头字段的值必须是临时（nonce）组成的一个随机选择的已经base64编码的（参见[[RFC4648](http://tools.ietf.org/html/rfc4648#section-4)]第4章）16位的值。临时必须是为每个连接随机选择的。

    注意：作为一个例子，如果随机选择的值是字节序列0x01 0x02 0x03 0x04 0x05 0x06 0x07 0x08 0x09 0x0a 0x0b 0x0c 0x0d 0x0e 0x0f 0x10，头字段的值必须是“AQIDBAUGBwgJCgsMDQ4PEC==”

8. 如果请求来自一个浏览器客户端，请求必须包含一个名字为|Origin|[RFC6454]的头字段。如果连接是来自非浏览器客户端，如果该客户端的语义匹配描述在这的用于浏览器的使用情况时，请求可以包含这个字段。该头字段的值是在建立正运行的连接代码中的环境的origin 的ASCII序列化。参考[[RFC6454](http://tools.ietf.org/html/rfc6454)]获取如果构造该头字段的值的详细信息。

    作为一个例子，如果从www.example.com下载的代码试图建立到ww2.example.com的连接，该头字段的值将是“http://www.example.com”。

9. 请求必须包含一个名字为|Sec-WebSocket-Version|的头字段。该头字段的值必须是13。

    注意：尽管本文档的草案版本（-09、-10、-11、和-12）发布了（它们多不是编辑上的修改和澄清而不是改变电报协议[wire protocol]），值9、10、11、和12不被用作有效的Sec-WebSocket-Version。这些值被保留在IANA注册中心，但并将不会被使用。

10. 请求可以包含一个名字为|Sec-WebSocket-Protocol|的头字段。如果存在，该值表示一个或多个逗号分割的客户端想要表达的子协议，按优先顺序排列。包含该值的元素必须是非空字符串，且字符在U+0021到U+007E范围内但不包含定义在[[RFC2616](http://tools.ietf.org/html/rfc2616)]中的分割字符且必须所有是唯一的字符串。用于该头字段值的ABNF是1#token，其构造和规则定义在[[RFC2616](http://tools.ietf.org/html/rfc2616)]给出。

11. 请求可以包含一个名字为|Sec-WebSocket-Extensions|的头字段。如果存在，该值表示客户端想要表达的协议级的扩展。该头字段的解释和格式描述在第9.1节。

12. 请求可以包含任意其他头字段，例如，cookie[[RFC6265](http://tools.ietf.org/html/rfc6265)]和/或验证相关的头字段例如|Authorization|头字段[[RFC2616](http://tools.ietf.org/html/rfc2616)]，其根据定义它们的文档处理。


一旦客户端的打开阶段握手已经发送，客户端在发送任何进一步数据之前必须等待自服务器的一个响应。客户端必须验证服务器的响应，如下所示：

1. 如果收到的服务器的状态码不是101，客户端处理每个HTTP[[RFC2616](http://tools.ietf.org/html/rfc2616)]程序的响应。尤其是，如果收到一个401状态码客户端可能执行身份验证；服务器可能使用一个3xx状态码重定向客户端（但客户端不需要跟随他们），等等。否则，按以下步骤处理。

2. 如果响应缺少一个|Upgrade|头字段或|Upgrade|头字段包含的值不是一个不区分大小写的ASCII匹配值“websocket”，客户端必须_失败WebSocket连接_。

3. 如果想要缺少一个|Connection|头字段或|Connection|头字段不包含一个不区分大小的ASCII匹配值“Upgrade”符号，客户端必须_失败WebSocket连接_。

4. 如果想要缺少一个|Sec-WebSocket-Accept|头字段或|Sec-WebSocket-Accept|包含一个不是|Sec-WebSocket-Key|（一个字符串，不是base64编码的）与字符串“258EAFA5-   E914-47DA-95CA-C5AB0DC85B11”但忽略任何前导和结尾空格相关联的base64编码的SHA-1值，客户端必须_失败WebSocket连接_。

5. 如果响应包含一个|Sec-WebSocket-Extensions|头字段且此头字段表示使用一个扩展但没有出现在客户端握手中（服务器表示的一个扩展，不是客户端请求的），客户端必须_失败WebSocket连接_。解析该头字段以确定请求了哪些扩展在9.1节讨论。

6. 如果响应包含一个|Sec-WebSocket-Protocol|头字段且该头字段表示使用一个子协议但没出现在客户端握手中（服务器表示的一个子协议，不是客户端请求的），客户端必须_失败WebSocket连接_。

如果服务器响应不符合定义在本节和4.2.2节中的服务器握手的要求，客户端必须_失败WebSocket连接_。

请注意，根据[[RFC2616](http://tools.ietf.org/html/rfc2616)]，所有命名在HTTP请求和HTTP响应中的头字段是不区分大小写的。

如果服务器响应验证了以上提供的，这是说，_WebSocket连接建立了_且WebSocket连接处于OPEN状态。_使用中的扩展_被定义为一个（可能空的）字符串，其值等于服务器握手中提供的|Sec-WebSocket-Extensions|头字段的值或如果在服务器握手中没有该头字段则为null值。_使用中的子协议_被定义为服务器握手中的|Sec-WebSocket-Protocol|头字段的值或如果在服务器握手中没有该头字段则为null值。另外，如果在服务器握手中表示cookie应该被设置（定义在[[RFC6265](http://tools.ietf.org/html/rfc6265)]）的任何头字段，这些cookie被称为_在服务器打开阶段握手期间的Cookie设置_。

##4.2.服务器端要求
服务器可以下移（offload）连接管理到网络上的其他代理，例如，负载均衡和反向代理。在这样的情况下，用于本规范的目的的服务器被认为是包括服务器端基础设施的所有部分，从开始的设备到终止TCP连接，处理请求和发送响应的服务器的所有方式。

例如：一个数据中心可能有一个用适当的握手来响应WebSocket请求的服务器，并接着传递连接到另一个服务器来真正处理数据帧。对于本规范的目的，“服务器”是结合了两种计算机。

###4.2.1.读取客户端的打开阶段握手
当客户端开始一个WebSocket连接，它发送它的打开阶段握手部分。服务器必须至少解析这个握手为了获取必要的信息来生成服务器握手部分。

客户端打开阶段握手包括以下部分。如果服务器，当读取握手时，发现客户端没有发送一个匹配下面描述的握手（注意，按照[[RFC2616](http://tools.ietf.org/html/rfc2616)]，头字段顺序是不重要的），包括但不限制任何违反ABNF语法指定的握手组件，服务器必须停止处理客户端握手并返回一个具有一个适当错误码的（例如400错误的请求）HTTP响应。

1. 一个HTTP/1.1或更高版本的GET请求，包括一个“Request-URI” [[RFC2616](http://tools.ietf.org/html/rfc2616)]应该被解释为定义在第3章的/resource name/（或一个包含/resource name/的绝对HTTP/HTTPS URI）。

2. 一个|Host|头字段包含服务器的权限。

3. 一个|Upgrade|头字段包含值“websocket”，视为一个不区分大小写的ASCII值。

4. 一个|Connection|头字段包含符号“Upgrade”，视为一个不区分大小写的ASCII值。

5. 一个|Sec-WebSocket-Key|头字段，带有一个base64编码的值（参见[[RFC4648](http://tools.ietf.org/html/rfc4648#section-4)]第4章），当解码时，长度是16字节。

6. 一个|Sec-WebSocket-Version|头字段，带有值13。

7. 可选的，一个|Origin|头字段。该头字段由所有浏览器客户端发送。一个试图缺失此头字段的连接不应该被解释为来自浏览器客户端。

8. 可选的，一个|Sec-WebSocket-Protocol|头字段，带有表示客户端想要表达的协议的值列表，按优先顺序排列。

9. 可选的，一个|Sec-WebSocket-Extensions|头字段，带有表示客户端想要表达的扩展的值列表。此头字段的解释在9.1节讨论。

10. 可选的，其他头字段，例如这些用于发送cookie或请求服务器身份验证的。未知的头字段被忽略，按照[[RFC2616](http://tools.ietf.org/html/rfc2616)]。

###4.2.2.发送服务器的打开阶段握手

当客户端建议一个到服务器的WebSocket连接，服务器必须完成以下步骤来接受该连接并发送服务器的打开阶段握手。

1. 如果连接发送在一个HTTPS (HTTP-over-TLS)端口上，在连接之上执行一个TLS握手。如果失败了（例如，在扩展的客户端hello“server_name”扩展中的客户端指示的一个主机名，服务器对主机不可用），则关闭该连接；否则，用于该连接的所有进一步的通信必须贯穿加密的隧道[[RFC5246](http://tools.ietf.org/html/rfc5246)]。

2. 服务器可以执行额外的客户端身份认证，例如，返回401状态码与描述在[[RFC2616](http://tools.ietf.org/html/rfc2616)]中的相关的|WWW-Authenticate|头字段。

3. 服务器可以使用3xx状态码[[RFC2616](http://tools.ietf.org/html/rfc2616)]重定向客户端。注意，此步骤可能连同，之前，或之后的可选的上面描述的身份验证步骤一起发生。

4. 建立如下信息：

/origin/

客户端握手中的|Origin|头字段表示建立连接的脚本的来源。Origin是序列化为ASCII并转换为小写。服务器可以使用这个信息作为决定是否接受传入连接的一部分。如果服务器没有验证origin，它将接受来自任何地方的连接。如果服务器不想接受这个连接，它必须返回一个适当的HTTP错误码（例如，403 Forbidden）并中断描述在本章中的WebSocket握手。更多详细信息，请参阅第10章。

/key/

在客户端握手中的|Sec-WebSocket-Key|头字段包括一个base64编码的值，如果解码，长度是16字节。这个（编码的）值用在创建服务器握手时来表示接受连接。服务器没必要使用base64解码|Sec-WebSocket-Key|值。

/version/

客户端握手中的|Sec-WebSocket-Version|头字段包括客户端试图通信的WebSocket协议的版本。如果该版本没有匹配服务器理解的一个版本，服务器必须中断描述在本节的WebSocket握手并替代返回一个适当的HTTP错误码（例如，426 Upgrade Required）且一个|Sec-WebSocket-Version|头字段表示服务器能理解的版本。

/resource name/

由服务器提供的服务的标识符。如果服务器提供多个服务，那么该值应该源自客户端我手中的GET方法的“Request-URI”中给定的资源名。如果请求的服务器不可用，服务器必须发生一个适当的HTTP错误码（例如404 NotFound）并中断WebSocket握手。

/subprotocol/

或者一个代表服务器准备使用的子协议的单个值或者null。选择的值必须源自客户端握手，从|Sec-WebSocket-Protocol|字段具体地选择一个值，服务器将使用它用于这个连接（如果有）。如果客户端握手不包含这样一个头字段或如果服务器不同意任何客户端请求的子协议，仅接受的值为null。这个字段不存在等价于null值（意思是如果服务器不想同意任何建议的子协议，它必须在它的响应中不发送回一个|Sec-WebSocket-Protocol|头字段）。用于这些目的，空字符串与null值是不一样的，且它不是这个字段合法的值。用于该头字段的值ABNF是（符号），构造定义和规则在[RFC2616]中给出。

/extensions/

表示服务器准备使用的协议级别扩展的一个列表（可能为空）。如果服务器支持多个扩展，那么该值必须源自客户端握手，通过从|Sec-WebSocket-Extensions|字段具体地选择一个或多个值。这个字段不存在等价于null值。用于这些目的，空字符串与null值是不一样的。客户未列出的扩展必须不被列出。那些值应该被选择和解释的方法在9.1节讨论。

5. 如果服务器选择接受传入的连接，它必须以一个有效的表示以下的HTTP响应应答。

1. 一个按照RFC2616[RFC2616]带有101响应码的Status-Line。这样的响应可能看起来像“HTTP/1.1 101 Switching Protocols”。

2. 一个按照RFC2616[RFC2616]带有值“websocket”的|Upgrade|头字段。

3. 一个带有“Upgrade”的| Connection |头字段。

4. 一个|Sec-WebSocket-Accept|头字段。该头字段的值通过连接/key/构造，它定义在4.2.2节第4步，带有字符串“258EAFA5- E914-47DA-95CA-C5AB0DC85B11”，采用SHA-1散列这个连接的值来获取一个20字节的值并base64编码（参考[RFC4648]第4章）这个20字节的散列。

该头字段的ABNF[RFC2616]定义如下：

           Sec-WebSocket-Accept     = base64-value-non-empty
           base64-value-non-empty = (1*base64-data [ base64-padding ]) |
                                    base64-padding
           base64-data      = 4base64-character
           base64-padding   = (2base64-character "==") |
                              (3base64-character "=")
           base64-character = ALPHA | DIGIT | "+" | "/"

注意：例如，如果客户端握手中的|Sec-WebSocket-Key|头字段的值是“dGhlIHNhbXBsZSBub25jZQ==”，服务器将追加字符串“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”为字符串dGhlIHNhbXBsZSBub25jZQ==258EAFA5-E914-47DA-95CA-C5AB0DC85B11形式。

服务器将采取SHA-1散列这个字符串，并给出值0xb3 0x7a 0x4f 0x2c 0xc0 0x62 0x4f 0x16 0x90 0xf6 0x46 0x06 0xcf 0x38 0x59 0x45 0xb2 0xbe 0xc4 0xea。这个值接着base64编码，给出值“s3pPLMBiTxaQ9kYGzzhZRbK+xOo=”，这将在|Sec-WebSocket-Accept|头字段中被返回。

5. 可选的，一个|Sec-WebSocket-Protocol|头字段，带有一个定义在4.2.2节第4步的值/subprotocol/。

6. 可选的，一个|Sec-WebSocket-Extensions|头字段，带有一个定义在4.2.2节第4步的值/ extensions /。如果使用多个扩展，那么可以把所有都列在一个|Sec-WebSocket-Extensions|头字段中或者分配到|Sec-WebSocket-Extensions|头字段的多个实例之间。
这就完成了服务器握手。如果服务器完整这些步骤且没有中断WebSocket握手，服务器认为WebSocket连接已建立且WebSocket连接处于OPEN状态。此时，服务器可以开始发送（和接收）数据了。
###4.3.为握手中使用的新的头字段整理的ABNF
本节使用的ABNF语法/规范来自[RFC2616]第2.1节，包括“隐式*LWS规则”。   

注意，以下ABNF约定用于本节中。一些规则的名相当于相应的头字段名字。这样的规则表示相应的头字段值，例如Sec-WebSocket-Key ABNF规则描述了|Sec-WebSocket-Key| 头字段值的语法。在名字中带有“-Client”后缀的ABNF规则仅用在由客户端到服务器发送请求的情况；在名字中带有“-Server”后缀的ABNF规则仅用在由服务器到客户端发生响应的情况。例如，ABNF规则Sec-WebSocket-Protocol-Client描述了客户端到服务器发送的|Sec-WebSocket-Protocol|头字段值的语法。

以下新的头字段可以在从客户端到服务器握手期间被发送：

      Sec-WebSocket-Key = base64-value-non-empty
      Sec-WebSocket-Extensions = extension-list
      Sec-WebSocket-Protocol-Client = 1#token
      Sec-WebSocket-Version-Client = version

      base64-value-non-empty = (1*base64-data [ base64-padding ]) |
                                base64-padding
      base64-data      = 4base64-character
      base64-padding   = (2base64-character "==") |
                         (3base64-character "=")
      base64-character = ALPHA | DIGIT | "+" | "/"
      extension-list = 1#extension
      extension = extension-token *( ";" extension-param )
      extension-token = registered-token
      registered-token = token

      extension-param = token [ "=" (token | quoted-string) ]
           ; When using the quoted-string syntax variant, the value
           ; after quoted-string unescaping MUST conform to the
           ; 'token' ABNF.
      NZDIGIT       =  "1" | "2" | "3" | "4" | "5" | "6" |
                       "7" | "8" | "9"
      version = DIGIT | (NZDIGIT DIGIT) |
                ("1" DIGIT DIGIT) | ("2" DIGIT DIGIT)
                ; Limited to 0-255 range, with no leading zeros

以下新的头字段可以在服务器到客户端握手期间被发送：

      Sec-WebSocket-Extensions = extension-list
      Sec-WebSocket-Accept     = base64-value-non-empty
      Sec-WebSocket-Protocol-Server = token
      Sec-WebSocket-Version-Server = 1#version

##4.4.支持多个版本的WebSocket协议

本节提供了在客户端和服务器中支持多个版本的WebSocket协议的一些指导。

使用WebSocket版本通知能力（|Sec-WebSocket-Version|头字段），客户端可以初始请求它选择的WebSocket协议的版本（这并不一定必须是客户端支持的最新的）。如果服务器支持请求的版本且握手消息是本来有效的，服务器将接受该版本。如果服务器不支持请求的版本，它必须以一个包含所有它将使用的版本的|Sec-WebSocket-Version|头字段（或多个|Sec-WebSocket-Version|头字段）来响应。此时，如果客户端支持一个通知的版本，它可以使用新的版本值重做WebSocket握手。

以下示例演示了上述的版本协商。

      GET /chat HTTP/1.1
      Host: server.example.com
      Upgrade: websocket
      Connection: Upgrade
      ...
      Sec-WebSocket-Version: 25

服务器的响应可能看起来像如下：

    HTTP/1.1 400 Bad Request
    ...
    Sec-WebSocket-Version: 13, 8, 7

注意服务器最后的响应可能也看起来像：

    HTTP/1.1 400 Bad Request
    ...
    Sec-WebSocket-Version: 13
    Sec-WebSocket-Version: 8, 7

客户端现在可以重做符合版本13的握手： 

    GET /chat HTTP/1.1
    Host: server.example.com
    Upgrade: websocket
    Connection: Upgrade
    ...
    Sec-WebSocket-Version: 13
