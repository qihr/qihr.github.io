---
title: Git使用手册
tags: [Git]
categories: Program
---



### 1.连接GitHub

 配置信息

```shell
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub注册邮箱"
```

 生成密钥文件

```shell
ssh-keygen -t rsa -C "你的GitHub注册邮箱"
```

后面提示连按三个回车即可。

找到C:\Users\Sayi1\.ssh文件夹中的id_rsa.pub，复制全部内容。

打开[GitHub-Settings-SSH And GPGkeys](https://github.com/settings/keys) ，选择New SSH Key，粘贴.pub的内容。

检测GitHub公钥设置是否成功，输入：

```shell
ssh git@github.com
```

出现 Hi ! “你的github名字”即设置成功。

之所以设置GitHub密钥原因是，通过非对称加密的公钥与私钥来完成加密，公钥放置在GitHub上，私钥放置在自己的电脑里。GitHub要求每次推送代码都是合法用户，所以每次推送都需要输入账号密码验证推送用户是否是合法用户，为了省去每次输入密码的步骤，采用了ssh，当你推送的时候，git就会匹配你的私钥跟GitHub上面的公钥是否是配对的，若是匹配就认为你是合法用户，则允许推送。这样可以保证每次的推送都是正确合法的。

### 2.配置环境

1.检测Node.is和Npm是否安装成功：

```shell
node -v
npm -v
```

如果Bash上显示了Node和npm的版本号即安装成功

2.安装Hexo

```shell
npm install -g hexo-cli 
```

完成上步后，输入：

```shell
hexo init blog
```

检测网站是否可以运行：

```shell
hexo new test_my_site

hexo g

hexo s
```

完成后，在浏览器中输入：localhost:4000可以看到已经有一个简陋的网站了。

在bash中输入hexo可以看有哪些常用指令：

```shell
npm install hexo -g #安装Hexo
npm update hexo -g #升级
hexo init #初始化博客

hexo n "我的博客" == hexo new "我的博客" #新建文章
hexo g == hexo generate #生成
hexo s == hexo server #启动服务预览
hexo d == hexo deploy #部署

hexo server #Hexo会监视文件变动并自动更新，无须重启服务器
hexo server -s #静态模式
hexo server -p 5000 #更改端口
hexo server -i 192.168.1.1 #自定义 IP
hexo clean #清除缓存，若是网页正常情况下可以忽略这条命令
```

### 3.网站推送

找到网站目录，打开`_config.yml`。`_config.yml`是网站的配置文件。themes目录下也有一个同名文件，那个是主题配置文件。

打开文件，找到最后的deploy，修改为：

```yaml
deploy:
  type: git
  repository: git@github.com:qihr/qihr.github.io.git
  branch: master
```

修改这是为了给hexo d配置对应参数，此时生成的静态页面会推送至master分支。

