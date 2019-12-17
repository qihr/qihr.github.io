---
title: Git使用手册
date: 2019-12-15 20:40:21
tags: [Git]
categories: Program
---



hexo和git的基本操作。

<!-- more -->

### 1.连接GitHub

 配置信息 

```bash
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub注册邮箱"
```

 生成密钥文件

```bash
$ ssh-keygen -t rsa -C "你的GitHub注册邮箱"
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

修改这是为了给hexo d配置对应参数，生成的静态页面会推送至master分支。

安装Git部署插件：

```powershell
npm install hexo-deployer-git --save
```

然后推送至master：

```shell
hexo clean 
hexo g 
hexo d
```

### 4.主题下载

```shell
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

打开站点目录下的`_config.yml`文件，修改主题为下载的：

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next  
#theme: landscape

```

### 5.保存文章和配置文件到分支

master分支保存的只是渲染出的静态网页，新建一个分支用于保存文章和网站配置文件比较靠谱。

新建一个分支，名如hexo，然后设置为默认分支。

执行：

```shell
git add .
git commit -m "..."
git push origin hexo
```

这样网站文件会推送至hexo分支，而静态网页（hexo d）生成的文件会在master分支中。



### 6.更换环境后的重新部署

0.执行章节2的git部分操作。

1.将hexo分支的网站文件clone至本地：

```shell
git clone git@github.com:......
```

因为刚刚设置了默认分支为hexo，所以会直接download网站文件。

2.然后执行章节2的hexo部分操作。



###  7.Bash生成hexo文章

md手动去打标签和日期很不方便。



### 9.其他问题

#### 关于git commit -m 卡在vim编辑器无法退出

vim对新人真的太不又好了，退出都不会。

> 按下小写字母i，会进入编辑模式。可以在此模式下输入你想要的commit message。输入结束以后，按下esc退出编辑模式，这时按下英文输入法下的冒号，再输入wq，就可以保存退出了。w是write，q是quit。也可以在按esc退出编辑模式以后，切换到大写模式，连按两下Z。
>
> 如果你用不惯这个编辑器的话，可以通过配置git config --global core.editor '其它文本编辑器的执行文件的路径'，这样需要调用文本编辑器时，就不会用默认的vi了。譬如设置成notepad++，sublimetext等等。
>
> > [core]
> >         editor = 'D:/Program Files/Notepad/notepad++.exe' -multiImst -nosession
>
> 作者：Elpie Kay
>链接：https://www.zhihu.com/question/61913534/answer/192748096
> 来源：知乎
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。



#### 站点文章并没有按时间排序

记得写date,不然会按文章标题排序。

```markdown
title: Mac OS X 10.10.3下android-5.1.1_r9 源码下载与编译
date: 2015-08-18 10:36:21
categories: Android
tags: [Tech,Android]
```



### 0.附录

[^1]: [GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)
[^2]: [使用hexo，如果换了电脑怎么更新博客](https://www.zhihu.com/question/21193762/answer/79109280)





























