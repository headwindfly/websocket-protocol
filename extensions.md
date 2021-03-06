WebSocket客户端可以请求本规范的扩展，且WebSocket服务器可以接受一些或所有客户端请求的扩展。服务器不必响应不是客户端请求的任何扩展。如果扩展参数包含在客户端和服务器之间的协商中，这些参数必须按照参数应用到的扩展规范来选择。

##9.1.协商扩展

客户端通过包含一个|Sec-WebSocket-Extensions|头字段请求扩展，其按照正常的HTTP头字段规则（参考[[RFC2616](http://www.faqs.org/rfcs/rfc2616.html)], 4.2节）并且头字段的值按照以下ABNF定义[[RFC2616](http://www.faqs.org/rfcs/rfc2616.html)]。注意本章使用的ABNF语法/规则来源于[[RFC2616](http://www.faqs.org/rfcs/rfc2616.html)]，包括“隐式的 *LWS规范”。如果客户端或服务器在协商阶段接收到的值不符合下边的ABNF，这种畸形数据的接收人必须立即_失败WebSocket连接_。

         Sec-WebSocket-Extensions = extension-list
         extension-list = 1#extension
         extension = extension-token *( ";" extension-param )
         extension-token = registered-token
         registered-token = token
         extension-param = token [ "=" (token | quoted-string) ]
             ;当使用引用字符串的语法变种时，引用字符串之后的值必须
             ;符合’token’ABNF

注意，像其他HTTP头字段，这个头字段可以跨多个行分割或组合，因此，以下是等价的：

         Sec-WebSocket-Extensions: foo
         Sec-WebSocket-Extensions: bar; baz=2

完全等价于

         Sec-WebSocket-Extensions: foo, bar; baz=2

所有使用的extension-token必须是一个egistered token（参考11.4节）。任何给定扩展提供的参数必须被扩展定义。注意，客户端只需提供使用任何公布的扩展，除非服务器表示它希望使用使用扩展，否则必须使用它们。

注意，扩展的顺序是重要的。在多个扩展间的相互作用可以定义在定义扩展的文档中。在没有这样定义的情况下，解释是它请求中的客户端列出的头字段表示一个它希望使用的头字段的偏好，第一个列出的选项是最优选的。服务器在响应中列出的扩展表示扩展是实际正在用于连接的扩展。扩展应该修改数据和/或组帧，数据的操作顺序应该假定是与打开阶段握手期间服务器响应中列出的扩展顺序是一样的。

例如，如果有两个扩展“foo”和“bar”，且如果服务器发送的头字段|Sec-WebSocket-Extensions|有值“foo”、“bar”，那么数据上的操作将变为bar(foo(data))，是更改数据本身（如压缩）或更改可能“堆叠（stack）”的组帧。

可接受的扩展头字段（注意：为了可读性，将折叠较长行）的非规范化例子：

         Sec-WebSocket-Extensions: deflate-stream
         Sec-WebSocket-Extensions: mux; max-channels=4; flow-control,
          deflate-stream
         Sec-WebSocket-Extensions: private-extension

服务器通过包含一个容纳了一个或多个扩展的客户端请求的|Sec-WebSocket-Extensions|头字段来接受一个或多个扩展。所有扩展参数的解释，和什么构成一个有效的到客户请求的参数集的服务器响应，将由各个扩展定义。

##9.2.已知扩展

扩展提供了一种机制来实现选择性加入的附加协议特性。本文档没有定义任何扩展，但实现可以使用单独定义的扩展。