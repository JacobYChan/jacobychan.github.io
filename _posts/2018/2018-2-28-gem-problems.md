---
layout: post
title:  gem安装过程中遇到的问题
date:   2018-02-28 17:42:00 +0800
categories: document
tag: 问题
#大类配置
categories: 技术文章
---

* content
{:toc}

关于gem
====================================
+ ##### gem是ruby的包管理器，使用gem之前必须要安装ruby
+ ##### gems是一个用于对rails组建近些年个打包的ruby打包系统，它提供了一个分发ruby程序喝库的标准格式，还提供了一个管理程序包的工具。Rubyems的功能类似于linux下的apt-get，是个包管理器，可以从远程下载所需的包
+ ##### 你可以这样理解，gem是一系列文件和包的总称，是一些rails项目依赖的软件或者环境，或者是依赖的关系库，当你的项目中缺少的时候，你可以用gem install 来进行安装，这种安装是通过RubyGems这个包管理工具来安装的，当然你也可以通过bundleer来安装。说到这两种安装方法，区别在于: 
+ ##### gem install xxx.gem是通过Rubyems工具来进行安装的，将所需要的gem都安装到/usr/local/ruby/lib/ruby/gems/1.8（你的ruby的安装目录）。这其中包括了Cache、doc、gems、specifications 4个目录，cache下放置下载的原生gem包，gems下则放置的是解压过的gem包。当安装过程中遇到问题时，可以进入这些目录，把有问题的gem删掉，重新 gem install 即可 
+ ##### bundle install 默认情况下也是将所需要的gem安装到这个位置，但是在一些情况，可能你当前的用户权限对那个目录没有可写权限，这个时候bundler将会在一个临时目录里来升级所需的一切gem，然后管你要sudo的密码，这样的话，才有权限copy这些gems到系统的目录去。其实你应该永远也不要用sudo bundle install，因为在bundle install的时候，有些步骤是必须要用你现在的用户角色来进行的。 
+ ##### Rails 3中如果需要 require 某个 gem 必须通过 Gemfile 来管理。 

gem安装jekyll时遇到的问题
====================================

1、墙墙墙
------------------------------------
通过 gem source 查看你的当前的gem资源库位置，如果你的当前资源库的位置为： https://rubygems.org/，不好意思，你是无法安装成功的，因为这个资源库在国外，所以你需要安装的githug是无法下载成功的，那我们怎么办呢，其实方法很简单，修改当前资源库的位置：
{% highlight bash %}
1- gem sources -r https://rubygems.org/ 删除原来的资源库位置 
2- gem sources -a https://ruby.taobao.org/ 添加新的资源库位置 
3- gem sources -u 更新资源库
{% endhighlight%}

2、目录权限不够 
------------------------------------
{% highlight bash %}
报错： 
ERROR: While executing gem … (Gem::FilePermissionError) You don’t have write permissions for the /Library/Ruby/Gems/2.0.0 directory. 
{% endhighlight %}

解决： 
`sudo chmod 777 /Library/Ruby/Gems/2.0.0 修改权限`

3、gem需要更新 
-------------------------------------
{% highlight bash %}
报错： 
ERROR: While executing gem … (Errno::EACCES) 
Permission denied - /Library/Ruby/Gems/2.0.0/cache/i18n-0.7.0.gem 
{% endhighlight %}
解决： 
gem update –system

4、没有/usr/bin权限
--------------------------------------
{% highlight bash %}
报错：
ERROR:  While executing gem ... (Gem::FilePermissionError)
    You don't have write permissions for the /usr/bin directory.
{% endhighlight %}
解决：
`sudo gem install jekyll -u /usr/local/bin`

5、如果还是不行，又想用gem安装怎么办？sudo gem install….
-------------------------------------------
