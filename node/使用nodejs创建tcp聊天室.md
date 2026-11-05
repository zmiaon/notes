基础内容：
1. 创建服务器：
		1. **导入net模块**： `require('net')`
		2. **创建服务器**：`net.createServer()`
		3. **监听本机的端口**：`server.listen(端口号，()=>{})`
		4. **on事件**
			1) '<mark style="background: #FFB8EBA6;">connection</mark>'：去，触发对应函数**
				函数参数为连接对象clientSocket
				1)) 连接对象clientSocket的on事件：
					<mark style="background: #FFB86CA6;">data</mark>: **可以监听客户端发送数据的接收情况**，进行操作
					<mark style="background: #FFB86CA6;">end</mark>:**客户端断开 连接**
					<mark style="background: #FFB86CA6;">error</mark>:**客户端连接出错**	
					clientSocket.write() --- 向客户端发送信息
			2）'<mark style="background: #FFB8EBA6;">error'</mark>: 服务器出错	
			
2.  创建客户端
	1. **导入net模块**： `require('net')`
	2. **创建客户端连接**: `net.createConnection({ host:'127.0.0.1',port:9000 })`
	3. **监听on事件**  
		1. <mark style="background: #FFB8EBA6;">connect</mark>: **监听连接事件**，连接成功之后触发对应函数
			1. `process.stdout.write()` ----- 进程中输出对应的内容
			2. `process.stdin.on('data',chunk=>{  })` ----- 监听用户的输入内容
		2. <mark style="background: #FFB8EBA6;">data</mark>: **接收服务器发送过来的数据** 
		3. <mark style="background: #FFB8EBA6;">end</mark>: **连接结束**
		4. <mark style="background: #FFB8EBA6;">error</mark>:**连接出错**


