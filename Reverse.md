### 逆向

逆向大佬出的题目, 一堆坑。

* re 1

  ```
  用 Resouce Hacker 打开即可，flag 影藏在资源里面, notepad ++ 好像也是可以的。。。。
  ```

*  re 2 

  先看下加密算法，然后你就可以拿起的 IDA, OD 慢慢逆吧. balabala 然后就出来了

  ```python
  flag = 'flag{x0r_#_simPle}'

  ret = ''
  ret2 = 0

  for c in flag:
      ret += chr((~(ord(c)^ret2^0x22^0x33))&0xff)
      ret2 = ord(ret[-1]) & 0xff
      print(hex(ret2))

  for c in ret:
      print('0'+str(hex(ord(c)))[2:]+'h,',end='')
  print()
  ```

* re 3

  反调试的，原本担心 * 号查看器可以看到 flag ,  re大佬临时改了，直接硬编码输出 ```*```

  你们比我懂得 -.-