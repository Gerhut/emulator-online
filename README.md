emulator-online
===============

客户端分玩家控制网页模拟器。

总体结构
--------

-   浏览器：[WebSocket][]连接业务层。
    消息格式为DOMString时为JSON；格式为ArrayBuffer时为画面。
-   业务层：node。[ws][]库连接浏览器，[net.Socket][]连接模拟器。
-   模拟器采用BizHawk。嵌入Lua脚本，用[LuaSocket][]连接业务层。

[WebSocket]: https://developer.mozilla.org/en-US/docs/Web/API/WebSocket
[ws]: https://github.com/einaros/ws
[net.Socket]: http://nodejs.org/api/net.html#net_class_net_socket
[LuaSocket]: http://w3.impa.br/~diego/software/luasocket/

用户登录
--------

-   浏览器向node发送连接请求。
-   在超时时间内浏览器主动发送名字。
-   名字由浏览器提示用户主动输入。
-   收到名字之后视为已登录，node向该浏览器发送当前在线名单。
-   登录后node向所有已登录的其他浏览器发送上线通知。
-   断线时node向所有已登录的浏览器发送下线通知。

-   `{user: "<username>"}` 登录发送用户名。
-   `{onlines: ["<username>", "<username>", ...]}` 当前在线用户。
-   `{user: "<username>", online: true}` 用户上线。
-   `{user: "<username>", offline: true}' 用户下线。

交流传输
--------

-   浏览器向node发送交流消息。
-   node向所有已登录的其他浏览器发送交流消息。

-   `{chat: "<content>"}`
-   `{user: "<username>", chat: "<content>"}`

画面传输
--------

-   Lua建立TCP服务器监听来自本地客户端的连接。
-   node建立TCP客户端，连接到Lua服务器。
-   node向Lua发送空消息表示等待接收当前画面。
-   Lua接收到客户端的消息后向该客户端发送模拟器当前画面。
-   node收到画面后，进行JPEG编码。
-   node向所有已登录的浏览器发送编码后的画面。
-   浏览器收到ArrayBuffer格式的消息。
-   浏览器将消息解析成图像渲染在画布上。

操作传输
--------

-   Lua建立TCP服务器监听来自本地客户端的连接。
-   node建立TCP客户端，连接到Lua服务器。
-   node向Lua发送`joypad <player> <key> <down>`表示发送操作信息。
-   Lua接收到操作信息后实现在模拟器中。
-   当操作位有空闲时，包含两种情况：
    -   新用户登录，操作位有空闲。
    -   接受操作的浏览器下线。
    -   node向所有已登录的浏览器发送接受操作的浏览器及操作位编号，
        并向该浏览器增加一个属性。
-   浏览器向node发送操作消息。

-   `{user: "<username>", player: <player index>, you: true/false}`
-   `{key: "<keyname>", down: true/false}`

