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
$ ls
auto  bin  CHANGES  CHANGES.ru  conf  configure  contrib  html  LICENSE  Makefile  man  objs  README  src
$ cd bin/
$ ls
conf  html  logs  sbin
 ```

> Conf：配置相关  
> html：欢迎页面、错误页面  
>  logs：日志存放区  
> sbin：可执行文件存放区  

### 2.2 异常
> nginx中gzip模块需要zlib库，rewrite模块需要pcre库，ssl功能需要openssl库。所以如果服务器为安装这三个依赖库的话会报错，需要先安装着三个依赖库。  

本文所用的是阿里云服务器，初始环境，出现如下异常：
#### 2.2.1 缺少`pcre`库
安装[pcre](https://www.pcre.org/)步骤：  
* 下载：`wget https://ftp.pcre.org/pub/pcre/pcre-8.13.tar.gz`  
* 解压缩：`tar -zxvf pcre-8.13.tar.gz`  
* 进入目录：`cd pcre-8.13.tar.gz`  
* 运行configure：`./configure --enable-utf8`  
* 执行make命令：`make && make install`  

**问题** 
继续报错：
```bash
make[1]: *** [Makefile:888: pcrecpp.lo] Error 1
make[1]: Leaving directory '/root/software/pcre-8.13'
```

搜索说是没有安装`gcc-cc++`库，安装即可：`yum -y install gcc-c++`  

#### 2.2.2 缺少`ssl module`

```bas
./configure: error: SSL modules require the OpenSSL library.
```

安装ssl module步骤：  






参考：[直播推流服务器端搭建](https://www.jianshu.com/p/cf7f0552ffe9)

