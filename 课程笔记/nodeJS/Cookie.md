<span style="color:rgb(255, 0, 0)">前端修改cookie：</span>
通过document.cookie查看或者设置cookie字符串
<span style="color:rgb(255, 0, 0)">后端设置cookie:</span>
`res.setHeader('Set-Cookie','username=?;path=/;httponly;expires=?')`
username：记住当前登录的用户名，方便校验
path：指定cookie生效的路由路径，如果不设置会默认为当前的path；这样当路由不等于设置的path,cookie会不生效
httponly：不允许前端命令读取cookie，或者修改cookie,只能由服务端设置
expires：设置失效时间
<span style="color:rgb(255, 0, 0)">后端获取cookie:</span>
`req.headers.cookie`
