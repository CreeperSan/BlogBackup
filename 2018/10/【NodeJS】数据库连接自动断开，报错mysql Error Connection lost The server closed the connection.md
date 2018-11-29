# 【NodeJS】数据库连接自动断开，报错mysql Error: Connection lost The server closed the connection

### 前言

在部署完Bing壁纸以后观察了一段时间发现每天都有记录直接挂掉🤦‍♀️，然后SSH到VPS上查看nuhup文件，发现都是有这样的报差信息`Error: Connection lost The server closed the connection`

### 原因

经过一番查找原因，发现是npm里面的mysql数据库连接断开了，导致error，而服务器这边没有捕捉异常导致程序直接挂掉，网站自然就挂了。具体原因跟数据库连接的默认设置有关，默认情况下如果Mysql一段时间（默认是8个小时）内没有请求的话会断开连接，导致异常发生。而相应的解决方法也很简单，只需要捕捉异常，当事件发生的时候重新连接数据库就可以了。

### 解决方法

解决的代码如下

```javascript
function openConnection() {
    if (db) {
        db.end((err) => {
            console.log('已关闭数据库连接')
            process.exit();
        })
    }
    db = mysql.createConnection({
        host : 数据库地址,
        user : 连接数据库用户名,
        password : 连接数据库的密码,
        database : 数据库名称
    });
    db.on('error', (err) => {
        db = null;
        if (err.code === 'PROTOCOL_CONNECTION_LOST') {
            openConnection();
        }else{
            throw err;
        }}
    )
}

openConnection();
```



### 参考

[StackOverflow - Node.js MySQL Error Handling](https://stackoverflow.com/questions/40141332/node-js-mysql-error-handling)

[MaiTian丶 - 解决nodejs mysql Error: Connection lost The server closed the connection的两种方法](https://blog.csdn.net/wb_001/article/details/79000522)

