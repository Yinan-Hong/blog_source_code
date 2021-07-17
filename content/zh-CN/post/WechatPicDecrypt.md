+++
author = "兔楠"
title = "微信图片dat格式解码"
date = "2021-05-25"
description = "testing..."
tags = [
    "图片解码"
]
categories = [
    "小工具"
]
toc = true

+++

PC端微信会把图片保存为.dat格式，实际上是对图片的16进制编码进行了简单的异或加密。本文简单分析一下加密原理，然后提供一个常见图片格式解码的python实现。

<!--more-->

## 问题背景

事情是这样的...

有一天想找一个**远古聊天记录里的照片**，因为我换了手机，所以只能在pc端慢慢往上翻到去年的记录... 于是我就想，图片肯定会存在本地的呀，打开文件发现微信保存的格式是没见过的.dat，于是我打开了百度...

其实下面讲的原理和方法都是参考了网上dalao们发过的，我就是整合了一下，代码优化了下，加了点细节，~~然后又可以水一篇文章哈哈哈。~~



## 太长不看版

对于没有编程能力或者对原理不感兴趣的同学，我贴心地封装了可执行文件，下载下来就可以直接运行了。

[`Yinan-Hong/WechatPicDecrypt (github.com)`](https://github.com/Yinan-Hong/WechatPicDecrypt)

GG，因为调用了os库，所以封装的可执行文件会被windows当成木马查杀。要怎么解决我有空再搞吧我要睡觉了.... QwQ

// todo


## 微信的文件存储



<img src="../WechatPicDecryptPic/Snipaste_2021-05-26_01-51-40.png" alt="Snipaste_2021-05-26_01-51-40" style="zoom: 25%;" />

PC端的微信，点开后的图片会被下载到本地，没点开的只会存高糊的预览图。微信的文件会存储在你设定的文件夹里，默认是在C盘安装微信的位置，可以在微信左下角【设置】→【文件管理】查看文件存储目录。

<img src="../WechatPicDecryptPic/Snipaste_2021-05-26_02-06-12.png" alt="Snipaste_2021-05-26_02-06-12" style="zoom:50%;" />

打开目录下名为【WeChat Files】的目录，里面会有以【微信号】命名的目录，里面存的就是这个微信号的聊天文件，包括文本，发送的文件，图片，表情包之类的。进入【FileStorage】→【Image】，里面是按月份分的聊天记录内下载过的图片。文件是后缀是.dat格式，只有在客户端用微信自带的图片查看器才能正常打开。



## 加密原理



用16进制编辑器打开dat文件，发现每个文件的前几个字节都是一样的。因为jpg、png文件开头都是固定的，比如jpg文件第一个字节是0xFF, 0xD8，所以猜测这个加密方式是对每个字节用密码进行异或。

<img src="../WechatPicDecryptPic/Snipaste_2021-05-26_02-15-08.png" alt="Snipaste_2021-05-26_02-15-08" style="zoom: 50%;" />

<img src="../WechatPicDecryptPic/Snipaste_2021-05-26_02-18-23.png" alt="Snipaste_2021-05-26_02-18-23" style="zoom:50%;" />

将文件前两个字节0x07, 0x20与jpg文件的前两个字节0xFF, 0xD8进行异或

<img src="../WechatPicDecryptPic/Snipaste_2021-05-26_02-33-00.png" alt="Snipaste_2021-05-26_02-33-00" style="zoom:60%;" />

<img src="../WechatPicDecryptPic/Snipaste_2021-05-26_02-33-56.png" alt="Snipaste_2021-05-26_02-33-56" style="zoom:60%;" />

运算结果都为0xF8，所以猜测加密密码是0xF8。也就是说这个是将文件的每个字节与0xF8异或，然后将文件保存为.dat后缀的文件。实际上这个文件就是这样加密的，只是每个用户的加密密码是不一样的。



## 解码程序思路



首先要输入文件夹的绝对地址，然后遍历里面的每一个文件。

因为发现目录下可能有些其他格式的配置文件，所以判断一下文件后缀不是.dat的跳过。

对dat文件，分别取jpg，png，gif（这三个格式比较常见，所以只做了这三个的）的前两个字节，【文件的第一个字节】与【jpg的第一个字节】异或得到【十六进制数1】，【文件的第二个字节】与【jpg的第二个字节】异或得到两位【十六进制数2】。如果得到的【十六进制数1】等于【十六进制数2】，那就能判断原文件的格式是jpg，并且得到了加密密码就是【十六进制数1】。

同样的道理判断文件出的格式，得到密码后，将文件解码，保存为.jpg文件，然后输出到目录下名为output的文件夹。

## 代码

```python
import os

'''
jpg文件的第一个字节为 pic_head[0], pic_head[1]
png为[2],[3]
gif为[4],[5]
'''
pic_head = [0xff, 0xd8, 0x89, 0x50, 0x47, 0x49]
decode_code = 0 # 解密码

#判断文件类型，并获取dat文件解密码
def get_code(file_path):    # file_path: dat文件路径

    dat_file = open(file_path, "rb")
    dat_read = dat_file.read(2)
    head_index = 0

    while head_index < len(pic_head):
        #使用第一个头信息字节来计算加密码
        code = dat_read[0] ^ pic_head[head_index]
        idf_code = dat_read[1] ^ code
        head_index = head_index + 1

        #第二个字节来验证解密码是否正确
        if idf_code == pic_head[head_index]:
            dat_file.close()
            return code
        head_index = head_index + 1

    #如果解码成功，则返回解密码，如果文件不是这三种格式则返回0
    print("not jpg, png, gif")
    return 0


def decrypt(file_name, file_path, output_file): #dat文件路径，生成文件路径
    #获取密码
    decode_code = get_code(file_path)
    dat_file = open(file_path, "rb")
    pic_name = output_file + ".jpg"
    pic_write = open(pic_name, "wb")
    #解码
    for dat_data in dat_file:
        for dat_byte in dat_data:
            pic_data = dat_byte ^ decode_code
            pic_write.write(bytes([pic_data]))
    print(file_name + "---解码成功")
    dat_file.close()
    pic_write.close()


def mkdir(path):
    path=path.strip()
    # 去除尾部 \ 符号
    path=path.rstrip("\\")
    # 判断路径是否存在
    isExists=os.path.exists(path)
    if not isExists:
        # 如果不存在则创建目录
        # 创建目录操作函数
        os.makedirs(path)
        print (path + ' 目录创建成功')
        return True
    else:
        # 如果目录存在则不创建，并提示目录已存在
        print (path + ' 目录已存在')
        return False


def find_datfile(dir_path):
    #获取dat文件目录下所有的文件
    files_list = os.listdir(dir_path)
    output_path = dir_path + "\output"
    #在原目录下新建一个output目录
    mkdir(output_path)

    cnt = 0
    succeed = 0
    #对每个文件获取文件名然后解码
    for file_name in files_list:
        #判断文件后缀是否为.dat
        cnt = cnt + 1
        file_type = file_name[-4:]
        if(file_type!=".dat"):
            print(file_name + '---文件不是.dat格式')
            continue
        file_path = dir_path + "\\" + file_name
        output_file = output_path + "\\" +file_name
        decrypt(file_name, file_path, output_file)
        succeed = succeed + 1

    print("==================")
    print("All done!")
    print ("目录下文件共", cnt, "个，成功解码", succeed, "个。")
    print("==================")


path = input("请输入微信dat文件的目录（绝对路径）:")
find_datfile(path)
```

## 运行示例

运行前的文件夹，有三个.dat文件，有一个不是

<img src="../WechatPicDecryptPic/Snipaste_2021-05-26_02-54-31.png" alt="Snipaste_2021-05-26_02-54-31" style="zoom: 33%;" />

运行程序

<img src="../WechatPicDecryptPic/Snipaste_2021-05-26_02-55-20.png" alt="Snipaste_2021-05-26_02-55-20" style="zoom:60%;" />

输入目录路径

<img src="../WechatPicDecryptPic/Snipaste_2021-05-26_02-59-09.png" alt="Snipaste_2021-05-26_02-59-09" style="zoom:60%;" />

运行后的文件夹

<img src="../WechatPicDecryptPic/Snipaste_2021-05-26_03-00-23.png" alt="Snipaste_2021-05-26_03-00-23" style="zoom: 33%;" />



## 写在最后

其实这个是大半年前遇到的问题，当时网上搜到了比较方便的在线解码器。但是那时候想想，都学计算机了，要不试试**像程序员一样去解决问题**，于是就搜了更多方法。

本文的内容参考了一些博客，只要搜微信dat文件之类的关键词就能搜到很多，不少人很早之前就遇到并解决了这个问题。不过这是我第一次用编程的方法解决我遇到的实际问题，感觉非常有纪念意义，值得写这么一篇东西记录一下哈哈。

