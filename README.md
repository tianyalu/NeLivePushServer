# NeLivePushServer 直播推流服务器搭建

## 一、环境准备
* 下载 [Nginx](http://nginx.org/en/download.html)： `wget http://nginx.org/download/nginx-1.16.0.tar.gz`  
* 解压Nginx：`tar -zxvf nginx-1.16.0.tar.gz`  
* 下载 [Nginx RTMP](https://github.com/arut/nginx-rtmp-module) 模块：`wget https://github.com/arut/nginx-rtmp-module/archive/v1.2.1.tar.gz`  
* 解压Nginx RTMP模块：`tar -zxvf v1.2.1.tar.gz`  

## 二、编译安装
### 2.1 正常步骤
 * 进入Nginx解压目录：`cd nginx-1.16.0`  
 * 将help输出text方便查看：`./configure --help > nginx_configure_help.txt`  
 * 执行configure：`./configure --prefix=./bin --add-module=../nginx-rtmp-module-1.2.1`  
 * 执行完毕后生成Makefile文件，编译：`make install`  
 * 编译完生成bin目录  
 ```bash
[root@iZwz9ci7skvj0jj2sfdmqgZ nginx-1.16.0]# ls
auto  bin  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  Makefile  man  nginx_configure_help.txt  objs  README  src
[root@iZwz9ci7skvj0jj2sfdmqgZ nginx-1.16.0]# cd bin
[root@iZwz9ci7skvj0jj2sfdmqgZ bin]# ls
conf  html  logs  sbin
[root@iZwz9ci7skvj0jj2sfdmqgZ bin]# 
 ```

> Conf：配置相关  
> html：欢迎页面、错误页面  
> logs：日志存放区  
> sbin：可执行文件存放区  

### 2.2 异常
> nginx中gzip模块需要zlib库，rewrite模块需要pcre库，ssl功能需要openssl库。所以如果服务器为安装这三个依赖库的话会报错，需要先安装着三个依赖库。  

本文所用的是阿里云服务器，初始环境，出现如下异常：
#### 2.2.1 缺少`pcre`库
安装 [pcre](https://www.pcre.org/) 步骤：  
* 下载：`wget https://netix.dl.sourceforge.net/project/pcre/pcre/8.40/pcre-8.40.tar.gz`  
* 解压缩：`tar -zxvf pcre-8.40.tar.gz`  
* 进入目录：`cd pcre-8.40  
* 运行configure：`./configure --enable-utf8`  
* 执行make命令：`make && make install`   

**继续报错：** 
```bash
configure: error: You need a C++ compiler for C++ support.
```
**解决方案:**  
没有安装`gcc-cc++`库，安装即可：`yum -y install gcc-c++`   

* 检查`pcre`是否安装：`rpm -qa | grep pcre` 

  ```bash
  [root@iZwz9ci7skvj0jj2sfdmqgZ pcre-8.40]# rpm -qa | grep pcre
  pcre2-10.32-1.el8.x86_64
  pcre-8.42-4.el8.x86_64
  ```

此时表明安装成功。

#### 2.2.2 缺少`ssl module`
报错信息如下：  
```bash
./configure: error: SSL modules require the OpenSSL library.
You can either do not enable the modules, or install the OpenSSL library
into the system, or build the OpenSSL library statically from the source
with nginx by using --with-openssl=<path> option.
```
安装ssl module步骤：  
* 安装命令：`yum -y install openssl openssl-devel`
* 查看版本号：`openssl version -a`  

#### 2.2.3  Nginx将警告当做错误处理
报错信息如下：  
```bash
../nginx-rtmp-module-1.2.1/ngx_rtmp_eval.c: In function ‘ngx_rtmp_eval’:
../nginx-rtmp-module-1.2.1/ngx_rtmp_eval.c:160:17: error: this statement may fall through [-Werror=implicit-fallthrough=]
                 switch (c) {
                 ^~~~~~
../nginx-rtmp-module-1.2.1/ngx_rtmp_eval.c:170:13: note: here
             case ESCAPE:
```
解决方法：  
* 进入Nginx目录的objs目录下：  
```bash
cd nginx-1.16.0
cd objs
```
* 编辑`Makefile`文件：`vim Makefile`    
* 找到如下一行内容并去掉`-Werror`  
```bash
CFLAGS =  -pipe  -O -W -Wall -Wpointer-arith -Wno-unused-parameter -Werror -g　
```
**注意：** 此时无需再执行Nginx `configure` 步骤，直接执行 Nginx `make install`步骤即可。

## 三、修改配置
Nginx默认不支持rtmp，需要修改配置文件。  
进入`bin/conf`目录，找到`nginx.conf`文件，修改如下：  
```bash
worker_processes  1;
error_log  logs/error.log debug;
events {
    worker_connections  1024;
}
#rtmp标签
rtmp {
    #服务标签，一个rtmp服务中可以有多个server标签，每个标签可监听不同端口号
    server {
        #注意端口占用，1935为默认端口
        listen 1935;
        #应用标签，一个服务标签中可以有多个应用标签
        application myapp {
            live on;
            #丢弃闲置5s的连接
            drop_idle_publisher 5s;
        }
    }
}
http {
    server {
        #注意端口占用
        listen      8080;
        #数据统计模块，将流媒体的状态记录到 stat.xsl 中
        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }
        #将stat.xsl 访问目录指定到nginx-rtmp-module中
        location /stat.xsl {
        #注意目录
            root /root/software/nginx-rtmp-module-1.2.1;
        }
        #控制器模块，可录制直播视频、踢出推流/拉流用户、重定向推流/拉流用户
        location /control {
            rtmp_control all;
        }
        location /rtmp-publisher {
            #注意目录
            root /root/software/nginx-rtmp-module-1.2.1/test;
        }
        location / {
            #注意目录
            root /root/software/nginx-rtmp-module-1.2.1/test/www;
        }
    }
}
```



## 四、启动服务

进入`sbin`目录尝试执行`nginx`：  `./nginx -t`
```bash
[root@iZwz9ci7skvj0jj2sfdmqgZ bin]# cd sbin
[root@iZwz9ci7skvj0jj2sfdmqgZ sbin]# ./nginx -t
```
> -t 表示测试  

**报错信息如下：**    

```bash
nginx: [alert] could not open error log file: open() "./bin/logs/error.log" failed (2: No such file or directory)
2020/03/25 16:23:13 [emerg] 22997#0: open() "./bin/conf/nginx.conf" failed (2: No such file or directory)
nginx: configuration file ./bin/conf/nginx.conf test failed
```
仔细看错误说明，`./bin/logs/error.log` 找不到，也就是当前目录下早不到`bin/logs/error.log`。  因为我们执行的当前目录是sbin，里面只有可执行文件`nginx`，所以找不到。  

**解决方法：**  
到`Nginx`根目录下执行：  
```bash
[root@iZwz9ci7skvj0jj2sfdmqgZ sbin]# cd ../../
[root@iZwz9ci7skvj0jj2sfdmqgZ nginx-1.16.0]# ls
auto  bin  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  Makefile  man  nginx_configure_help.txt  objs  README  src
[root@iZwz9ci7skvj0jj2sfdmqgZ nginx-1.16.0]# ./bin/sbin/nginx -t
nginx: the configuration file ./bin/conf/nginx.conf syntax is ok
nginx: configuration file ./bin/conf/nginx.conf test is successful
```

此时表明测试成功，可以正式执行，启动服务了：  
```bash
[root@iZwz9ci7skvj0jj2sfdmqgZ nginx-1.16.0]# ./bin/sbin/nginx
```

查看Nginx服务进程：  
```bash
[root@iZwz9ci7skvj0jj2sfdmqgZ nginx-1.16.0]# ps aux | grep nginx
root     18451  0.0  0.2  48200  4992 ?        Ss   18:31   0:00 nginx: master process ./bin/sbin/nginx
nobody   18452  0.0  0.4  81276  8624 ?        S    18:31   0:00 nginx: worker process
root     18487  0.0  0.0  12112   964 pts/0    R+   18:31   0:00 grep --color=auto nginx
```
此时表明服务启动成功。  

## 五、测试服务
在浏览器中通过HTTP来访问：`http://47.115.6.127:8080`  
> 报错：403 Forbidden 
> 其实前面我们查看nginx进程的时候，可以发现master process和worker process的用户不一致，一个是root ，而另一个是nobody  

重新修改`bin/conf/nginx.conf`文件，添加`root`用户：  
```bash
#设置未root用户
user root;
worker_processes 1;
error_log logs/error.log debug;
```

配置文件改好了，需要重新加载配置文件：  
```bash
./bin/sbin/nginx -s reload
```
刷新浏览器就正常了。  

## 六、停止服务
* 从容停止服务
```bash 
./bin/sbin/nginx -s quit
```
这种方式较`stop`相比就比较温和一些了，需要进程完成当前工作后再停止。  
* 立即停止服务
```bash 
./bin/sbin/nginx -s stop
```
这种方式较强硬，无论进程是否在工作，都直接停止进程。  
* 杀死进程
```bash
pkill -9 nginx
```

## 七、直播推流测试
* 推流可以使用EV录屏    
> 设置串流地址： `rtmp://xxx.xxx.xxx/mapp`  

* 播放可以使用EV播放器  
> 播放网络流地址：`rtmp://xxx.xxx.xxx/myapp`  


参考：[直播推流服务器端搭建](https://www.jianshu.com/p/cf7f0552ffe9)

