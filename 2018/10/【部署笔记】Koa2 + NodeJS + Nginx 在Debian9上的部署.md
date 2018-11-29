#【部署笔记】Koa2 + NodeJS + Nginx 在Debian9上的部署

### 背景

前段时间写完了[Bing壁纸](http://bing.creepersan.com/)的预览版 :P ，想看看究竟跑起来如何，遂想把其部署到服务器上去看看，当然了~这篇文章写出来的时候已经部署成功了哈，当时找问题折腾了老半天、甚至还小小的熬了个夜也没搞出来，最后才发现是个低级错误。。。想哭😢，于是在这里记录下这次悲惨的排错

### 情景再现

1. 写完预览版啦！想赶紧试试部署起来看看是怎么样的，首先是把文件都搬到服务器上啦。我先在本地把先关文件打成压缩包，然后使用FileZila这款软件把文件上传到服务器中。软件是免费的，官网可以[戳我](https://filezilla-project.org/)访问。

2. 文件上传到vps中后，赶紧的解压然后安装NodeJS让它跑起来，VPS安装的是Debian9操作系统，解压zip文件可以使用`zip`这个来安装相应工具，命令如下

   ```shell
   sudo apt-get update
   sudo apt-get install zip unzip
   ```

   对应的解压命令也很简单，我们先cd到压缩文件所在的目录，比如我们的文件叫`bing-image.zip`，然后键入如下命令

   ```
   unzip ./bing-image.zip 
   ```

   这样就可以把文件的内容解压到当前目录下，如果存在同名文件，则会进一步询问操作（跳过还是覆盖还是取消）

3. 由于我的VPS上已经存在了其他网站了，所以使用了Nginx作为反向代理服务器，代理80端口到Koa2默认的3000端口，经过一番上网搜索，最后写出的Nginx配置大概是这样子的

   ```nginx
   upstream 你的应用名称 {
       server 127.0.0.1:你的NodeJS应用端口
   }
   
   server {
       listen 80;
       server_name 你的域名地址;
       
       location / {
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header Host $http_host;
           proxy_set_header X-NginX-Proxy true;
           
           proxy_redirect off;
           proxy_pass http://你的应用名称;
       }
   }
   ```

   顺带一提，我的配置文件在这里 `/etc/nginx/sites-enabled` 文件夹里面

4. 根据应用配置好数据库和其他东西

5. 配置好防火墙，开放相应端口

6. 差不多好啦，跑起来吧！cd到应用目录下，安装需要的东西，测试一下

   ```shell
   npm install
   npm start
   ```

   好，看到终端上看到相关信息了，用IP地址+端口访问看看

   嗯，Nice，那么就让它在后台跑把

   关闭刚刚跑起来的测试的，然后键入以下指令

   ```shell
   nohup npm start &
   ```

   让程序在后台一直跑，哪怕是当前用户退出后也能继续运行。如果不用nohup的话，当当前用户退出后，NodeJS进程就会被杀死，网站自然就挂了。所以用这条指令让其保持在后台运行，相关日志将会保存在当前目录的`nohup`这个文件中，可以直接vim查看相关内容

7. 赶紧用于域名访问看看

   ？？？

   傻眼？？？怎么无法访问？？？

   哇？

   刚刚不是好好的吗？

   哪里错了？？？

   ？？？

   赶紧查错！！

   。。。

   没错啊？

   重新写一遍看看？？

   陷入循环。。

   哇，2点了！

   唉，先睡觉吧。。。

8. 次日去到公司，是真的困😴，熬夜真的不舒服

   特别还解决不了问题。。

   真让人崩溃。

   好像。。。

   我好像漏了什么？

   我好想还没有加A记录……

   赶紧的去添加A记录

   加完重新访问！

   哇。

   假的吧？还是不行？

   哦对还没生效。。。等半小时后再看看吧

9. 哎，好了😌

   能访问了

   问题终结！

