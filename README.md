# NeLivePushServer 直播推流服务器搭建

## 一、环境准备
### 1.1 shell命令
shell可执行文件必须以`#!/bin/bash`开头
```bash
# 该方式不需要权限
/bin/bash demo.sh
# 该方式需要权限
chmod 777 demo.sh
./demo.sh 
```
实操：  
![image](https://github.com/tianyalu/NeShellVariable/blob/master/show/bin_bash.png)



<img src="/Users/tian/Library/Application Support/typora-user-images/image-20200316154829274.png" alt="image-20200316154829274" style="zoom:50%;" />

1.2

 



### 1.2 Android.mk基本格式
以下是一个简单的Android.mk文件的内容:
```linux
# 定义模块当前路径（必须定义在文件开头，只需要定义一次）
LOCAL_PATH := $(call my-dir)

# 清空当前环境变量（LOCAL_PATH除外）
include $(CLEAR_VARS)

# 当前模块名（这里会生成libhello-jni.so）
LOCAL_MODULE := hello-jni.c

# 当前模块包含的源代码文件
LOCAL_SRC_FILES := hello-jni.c

# 表示当前模块将被编译成一个共享库
include $(BUILD_SHARED_LIBRARY)
```

### 1.3 编译多个共享库
一个Android.mk可能编译产生多个共享库模块。这里会产生libmodule1.so和libmodule2.so两个动态库
```linux
LOCAL_PATH := $(call my-dir)

# 模块1
include $(CLEAR_VARS)
LOCAL_MODULE := module1
LOCAL_SRC_FILES := module1.c
include $(BUILD_SHARED_LIBRARY)

# 模块2
include $(CLEAR_VARS)
LOCAL_MODULE := module2
LOCAL_SRC_FILES := module2.c
include $(BUILD_SHARDE_LIBRARY)
```

### 1.4 编译静态库
虽然Android应用程序不能直接使用静态库，静态库可以用来编译动态库。比如将第三方代码添加到原生项目中时，可以不用直接将第三方源码包括在原生项目中，而是将第三方源码编译成静态库，然后并入共享库。
```linux
LOCAL_PATH := $(call my-dir)

# 第三方AVI库
include $(CLEAR_VARS)
LOCAL_MODULE := avilib
LOCAL_SRC_FILES := avilib.c platform_posix.c
include $(BUILD_STATIC_LIBRARY)

# 原生模块
include $(CLEAR_VARS)
LOCAL_MODULE := module
LOCAL_SRC_FILES := module.c
# 将静态库模块名添加到LOCAL_STATIC_LIBRARIES变量
LOCAL_STATIC_LIBRARIES := avilib
include $(BUILD_SHARED_LIBRARY)
```

### 1.5 使用共享库共享通用模块
静态库可以保证源代码模块化，但是当静态库与共享库相连时，它就变成了共享库的一部分。在多个共享库的情况下，多个共享库与静态库连接时，需要将通用模块的多个副本与不同的共享库重复相连，这样就增加了APP的大小。这种情况，可以将通用模块作为共享库。
```linux
LOCAL_PATH := $(call my-dir)

# 第三方AVI库
include $(CLEAR_VARS)
LOCAL_MODULE := avilib
LOCAL_SRC_FILES := avilib.c platform_posix.c
include $(BUILD_SHARED_LIBRARY)

# 原生模块1
include $(CLEAR_VARS)
LOCAL_MODULE := module1
LOCAL_SRC_FILES := module1.c
LOCAL_SHARED_LIBRARIES := avilib
include $(BUILD_SHARED_LIBRARY)

# 原生模块2
include $(CLEAR_VARS)
LOCAL_MODULE := module2
LOCAL_SRC_FILES := module2.c
LOCAL_SHARED_LIBRARIES := avilib
include $(BUILD_SHARED_LIBRARY)
```

### 1.6 在多个NDK项目间共享模块
> 首先将avilib作为
>
>

![image](https://github.com/tianyalu/NeMakefile/blob/master/show/make_file_fun_param.png)  

