### Misc

Misc 大佬出的题目..... emmm....

* 1
```flag\{\w+\}```

* 2
``` binwalk分析压缩包本身，得到flag的二维码```

* 3
```
曼切斯特转二维码，扫描得到另一张大小53k的指向同一个链接的二维码，在线生成一张同样内容的二维码，两张做盲水印
```
* 4 
``` 
压缩包弱口令12345，解压的key经jsfuck解码得到残缺的key，生成字典暴破wifi握手包得到正确的key，去掉前八位做md5解密得到flag
```

* 5
```
发封邮件到[zhouyuanlang@foxmail.com](mailto:zhouyuanlang@foxmail.com)收到回复电话13005220522和姓名zhouyuanlang。构造社工字典，写个py调用MP3Stego逐个尝试解密得到flag
```

* 6

```txt文件伪加密，提取出频次统计，得到密码解出step1。

step1文件尾-.换1和0转 ascii码提示XOR 30，异或30后解出step2， 

crc32碰撞step2内1字节文件内容，文件名键盘解密排序得到密码提出step3，

step3内图片提示408和google，F5隐写提出隐藏文本

文本内一串rsa私钥和密文，解密得到flag
```
