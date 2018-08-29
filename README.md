
## 博客框架
[Jekyll](https://www.jekyll.com.cn/)

## 博客主题
[nexT](http://theme-next.simpleyyt.com/)

## 主题配置
* [基本配置](http://theme-next.simpleyyt.com/getting-started.html)
* [高级配置](http://theme-next.simpleyyt.com/theme-settings.html)

---
# Window 搭建过程
（虽然最后使用的是linux）
* 本地环境：Windows 7 64位
* Ruby版本：2.4.4
* 笔记中碰到的问题见文末

## 环境准备
### 一、安装Ruby
Ruby官方网站：http://www.ruby-lang.org/zh_cn/

在Windows下，可以使用**RubyInstall**来安装Ruby环境

下载包含**Devkit**的RubyInstall_64位版本，并安装

是否安装完成？`ruby -v`
### 二、安装Git
本步略过

### 三、安装Jekyll
Jekyll官方网站：https://www.jekyll.com.cn/
```
gem install jekyll
```
是否安装成功？`jekyll -v`
### 四、在GitHub创建仓库
仓库名格式为：`username@github.io`

### 五、Clone仓库到本地
```
git clone https://github.com/licoba/licoba.github.io.git
```
### 六、下载并使用主题
主题官网：http://jekyllthemes.org/

### 七、Push到GitHub
```
git add .
git commit -m"My Push"
git push
```


## 一些问题
---
### gem下载慢or无法下载？
替换gem源：https://gems.ruby-china.com/
```
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
```

###  git push 太慢？
针对SSR，首先打开“允许来自局域网的连接”
- 设置代理
```
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
git config --global http.proxy 'socks5://127.0.0.1:1080' 
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

- 取消代理
```
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 提示缺少相应的包？

查看提示，缺少什么然后使用命令`gem install`来安装相应的包

比如说缺少`jekyll-paginate`，则对应的命令为;
```
gem install jekyll-paginate
```



