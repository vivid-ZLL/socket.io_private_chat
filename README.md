- git clone  https://github.com/umlink/socket.io_private_chat.git
- cd socket.io_private_chat
- npm install
- node serve / npm start
- 打开客户端 http://localhost:8084
- 注册后可给指定client发送消息
index.html
~~~
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>socket.io演示</title>
    <script src="https://cdn.socket.io/socket.io-1.4.5.js"></script>
</head>
<body>
<h1>socket.io演示客户端</h1><br>
<label for="">连接数：</label><span id="connectNum"></span>
<ul id="messages" style="width:100%;border-bottom: 1px solid #dedede">

</ul>
<label for="">我是：</label><input type="text" id="register">
<button onclick="register()">注册</button>
<br><br>
<label for="">发送目标：</label>
<select id="to">

</select>
<label for="">内容：</label><input type="text" id="content">
<button id="send" onclick="send()">发送</button>
</body>
<script>
    var socket = io.connect('127.0.0.1:8084');
    var messages = document.getElementById("messages");
    var selectTO = document.getElementById("to");
    socket.on('message', function (data) {
        var newLi = document.createElement("li");
        newLi.innerHTML = data.from + ":" + data.msg;
        messages.appendChild(newLi);
    });
    socket.on('connectNum', function (num) {
        console.log("连接数：" + num);
        document.getElementById('connectNum').innerHTML = num;
    });
    socket.on('users', function (users) {
        var newOp = null;
        document.getElementById("to").innerHTML = "";
        Object.keys(users).map(x => {
            newOp = document.createElement("option");
            newOp.innerHTML = x;
            selectTO.appendChild(newOp)
        });
    });


    function register() {
        var from = document.getElementById('register').value;
        socket.emit('setName', from);
        sessionStorage.setItem('name', from);
    };

    function send() {
        var from = document.getElementById('register').value;
        var to = document.getElementById('to').value;
        var content = document.getElementById('content').value;
        socket.emit('sayTo', {
            from: from,
            to: to,
            msg: content,
        })
    }
    if(sessionStorage.getItem('name')){
        document.getElementById('register').value = sessionStorage.getItem('name');
    }
</script>
</html>
~~~
server.js
~~~
var app = require('http').createServer(handler);
var io = require('socket.io')(app);
var fs = require('fs');
var _ = require('underscore');
app.listen(8084);

var hashName = {};

function handler (req, res) {
    fs.readFile(__dirname + '/index.html',
        function (err, data) {
            res.writeHead(200);
            res.end(data);
        });
}
function broadcast() {
    io.sockets.emit("connectNum",Object.keys(hashName).length);
    io.sockets.emit("users",hashName);
}
io.on('connection', function(socket){
    console.log('connection succed!');
    broadcast();
    socket.on('setName',function (data) {
        var name = data;
        hashName[name] = socket.id;
        console.log(hashName);
        broadcast();
    });
    socket.on('sayTo',function (data) {
        var toName = data.to;
        var toId;
        console.log(toName);
        if(toId = hashName[toName]){
            var toSocket = _.findWhere(io.sockets.sockets,{id:toId});
            toSocket.emit('message',data);
        }
    })
    socket.on('disconnect', function(){
        console.log('connection is disconnect!');
    });
});
~~~
效果：
![image](http://note.youdao.com/favicon.ico)
