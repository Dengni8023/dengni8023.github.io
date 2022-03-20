<!--
 * @Author: 梅继高
 * @Date: 2020-02-09 23:22:22
 * @LastEditTime: 2022-03-20 10:26:35
 * @LastEditors: 梅继高
 * @Description: 
 * @FilePath: /dengni8023.github.io/README.md
 * Copyright © 2021 MeiJigao. All rights reserved.
-->
# Dengni8023博客

> 博客的搭建参考 [快速搭建个人博客](http://qiubaiying.github.io/2017/02/06/快速搭建个人博客/) 
> 
> [我的博客在这里 &rarr;](https://dengni8023.github.io)

## 博客本地环境搭建

1. 拉取博客模板（`GitHub Pages + Jekyll`）
	
	```
	$ git clone https://github.com/Dengni8023/dengni8023.github.io.git
	或者
	$ git clone git@github.com:Dengni8023/dengni8023.github.io.git
	```

2. 博客本地网站结构

	`Jekyll`网站的基础结构如下：
	
	```
	|--CNAME
	|--_config.yml
	|--_drafts
	|  |--begin-with-the-crazy-ideas.textile
	|  |--on-simplicity-in-technology.markdown
	|--_includes
	|  |--footer.html
	|  |--head.html
	|  |--nav.html
	|--_layouts
	|  |--default.html
	|  |--post.html
	|--_posts
	|--_site
	|--img
	|--index.html
	|--LICENSE
	
	_config.yml：全局配置文件
	_posts：放置博客文章的文件夹
	img：存放图片的文件夹，建议此处仅存放少量博客框架相关图片，博客文章图片建议使用对象存储等云存储服务
	```	
	
	[Jekyll详情研究请查看这里](https://www.jekyll.com.cn/docs/structure/)

3. 修改博客配置
	
	进入仓库，找到网站的全局配置文件`_config.yml`，编辑该文件进行网站配置
	
	1. 基础设置
	
		```
		# Site settings
		title: Dengni8023 # 博客的标题
		SEOTitle: Dengni8023的博客 | Dengni8023 Blog # 显示在浏览器上搜索的时候显示的标题
		header-img: img/default-header.jpg # 显示在首页的背景图片
		email: 945835664@qq.com # 联系邮箱地址
		description: "Dengni8023个人学习总结博客。" # 网站介绍
		keyword: "等你8023, Dengni8023, Jigao, 继高, iOS, Apple, iPhone" # 关键词
		url: "http://dengni8023.github.io" # 博客地址
		baseurl: ""      # 一般无需配置 for example, '/blog' if your blog hosted on 'host/blog'
		github_repo: "https://github.com/dengni8023/dengni8023.github.io.git" # 博客仓库地址
		```
	
	2. 侧边栏
	
		```
		# Sidebar settings
		sidebar: true # 是否开启侧边栏
		sidebar-about-description: "Not implemented, everything is nothing.<br /><br />没有执行，一切为零。" # 侧边栏格言，装逼的话...
		sidebar-avatar: /img/avatar-dengni8023.jpg # 你的个人头像，可以改成img文件夹中的照片
		```
	
	3. 社交账号

		```
		# SNS settings
		github_username:    Dengni8023
		```
	
	其他配置请参考 [快速搭建个人博客](http://qiubaiying.github.io/2017/02/06/快速搭建个人博客/) 

## 写博客文章

文章统一放在网站根目录下的`_posts`的文件夹中

1. 每一篇文章文件命名采用的是`2021-01-01-Hello-2021.md`时间+标题的形式，空格用“-”替换连接。
2. 文件格式是`.md`的**MarkDown**文件。
3. 博客文章格式采用是`MarkDown + YAML`的方式。

文章大概结构如下：

```
---
layout:     post
title:      iOS推送机制和流程
date:       2018-02-27
author:     dengni8023
catalog:    true
tags:
    - iOS
    - 推送
    - APNS
---
	
# APNS概念
	
APNS：Apple Push Notification Service，苹果推送通知服务。

Provider：推送服务提供者，即接受推送Client APP的后台服务器。
```

按格式创建文章，修改提交，进入你的博客主页，新的文章将会出现在主页上。

## 自定义博客域名

博客默认域名为仓库名称，如：`dengni8023.github.io`，如不想使用这种长域名，可以修改自定义域名。

1. 购买注册域名，`GitHub Pages`无法处理中文域名，请购买非中文域名

	购买注册流程此处不做描述

2. 解析域名

	注册好域名后，需要将域名解析到你的博客上
	
	管理控制台 → 域名与网站（万网） → 域名
	
	添加解析，分别添加两个 <font color="red">`A`</font>记录类型
	
	```
	一个主机记录为 www，代表可以解析 www.dengni8023.top的域名
	另一个为 @, 代表 dengni8023.top
	```
		
	记录值就是我们博客的IP地址，是 GitHub Pagas 的服务器的地址 185.199.108.153，可以通过 [网站查询](http://ip.chinaz.com/dengni8023.github.io) 或者终端 <font color="red">`ping 你的地址`</font> 命令查看博客的IP地址 
	
	```
	$ ping dengni8023.github.io
	```
	
	<font color="red">注意：不同时间获取的IP可能不一样</font>

3. 修改`CNAME `

	找到仓库下的`CNAME`文件，使用购买注册的域名进行替换

<font color="red">
至此，博客本地环境搭建、文章发布、域名访问配置大功告成！
	
你可以通过[博客地址](http://dengni8023.github.io)或者[购买的域名](http://dengni8023.top)访问博客啦!
</font>

## 进阶

如需对博客进行其他美化、个性化配置等，请继续学习`Jekyll `的[开发文档](https://www.jekyll.com.cn)

## Mac本地调试博客（macOS Big Sur, 11.6）

`Jekyll`使用说明详见：[https://jekyllrb.com/docs/](https://jekyllrb.com/docs/)

新版本Mac系统，及`Jekyll`不再适用`gem install`方式

### `gem install`方式踩坑

1. 安装 `jekyll` 和 `bundler`

	```
	# gem 安装，避免文件权限报错
	$ sudo gem install -n /usr/local/bin jekyll bundler
	```
	
2. 进入博客代码仓库根目录，然后启动本地服务器

	```
	# 启动本地调试服务
	$ jekyll s
	```
			
	控制台显示如下：
	
	```
	Configuration file: /Users/meijigao/Desktop/Git•GitHub/Dengni8023/dengni8023.github.io/_config.yml
	  Dependency Error: Yikes! It looks like you don't have jekyll-paginate or one of its dependencies installed. In order to use Jekyll as currently configured, you'll need to install this gem. If you've run Jekyll with `bundle exec`, ensure that you have included the jekyll-paginate gem in your Gemfile as well. The full error message from Ruby is: 'cannot load such file -- jekyll-paginate' If you run into trouble, you can find helpful resources at https://jekyllrb.com/help/! 
	                    ------------------------------------------------
	      Jekyll 4.2.1   Please append `--trace` to the `serve` command 
	                     for any additional information or backtrace. 
	                    ------------------------------------------------
	```

	以上报错是因为博客配置文件`_config.yml`中使用了`jekyll-paginate`，需要安装对应的插件，启动之前需执行以下命令安装必须插件：
	
	```
	# 安装 jekyll-paginate 插件
	sudo gem install -n /usr/local/bin jekyll-paginate
	```
	
	安装成功后继续执行启动命令
	
	```
	# 启动本地调试服务
	$ jekyll s
	```
	
	出现如下错误：
	
	```
	Configuration file: /Users/meijigao/Desktop/Git•GitHub/Dengni8023/dengni8023.github.io/_config.yml
            Source: /Users/meijigao/Desktop/Git•GitHub/Dengni8023/dengni8023.github.io
       Destination: /Users/meijigao/Desktop/Git•GitHub/Dengni8023/dengni8023.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 0.035 seconds.
 Auto-regeneration: enabled for '/Users/meijigao/Desktop/Git•GitHub/Dengni8023/dengni8023.github.io'
                    ------------------------------------------------
      Jekyll 4.2.1   Please append `--trace` to the `serve` command 
                     for any additional information or backtrace. 
                    ------------------------------------------------
<internal:/opt/homebrew/Cellar/ruby/3.0.2_1/lib/ruby/3.0.0/rubygems/core_ext/kernel_require.rb>:85:in `require': cannot load such file -- webrick (LoadError)
        from <internal:/opt/homebrew/Cellar/ruby/3.0.2_1/lib/ruby/3.0.0/rubygems/core_ext/kernel_require.rb>:85:in `require'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/commands/serve/servlet.rb:3:in `<top (required)>'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/commands/serve.rb:179:in `require_relative'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/commands/serve.rb:179:in `setup'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/commands/serve.rb:100:in `process'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/command.rb:91:in `block in process_with_graceful_fail'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/command.rb:91:in `each'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/command.rb:91:in `process_with_graceful_fail'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/lib/jekyll/commands/serve.rb:86:in `block (2 levels) in init_with_program'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/mercenary-0.4.0/lib/mercenary/command.rb:221:in `block in execute'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/mercenary-0.4.0/lib/mercenary/command.rb:221:in `each'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/mercenary-0.4.0/lib/mercenary/command.rb:221:in `execute'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/mercenary-0.4.0/lib/mercenary/program.rb:44:in `go'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/mercenary-0.4.0/lib/mercenary.rb:21:in `program'
        from /opt/homebrew/lib/ruby/gems/3.0.0/gems/jekyll-4.2.1/exe/jekyll:15:in `<top (required)>'
        from /usr/local/bin/jekyll:23:in `load'
        from /usr/local/bin/jekyll:23:in `<main>'
```
    
    报错显示缺少`webrick`，最新Ruby中不再包含`webrick`，通过各种尝试及查看[Jekyll官方文档](https://jekyllrb.com/docs/)，确认`gem install `不再适用，需要通过bundle方式
    
    ### bundle方式启动本地调试
    
	结合官方4.2.1版本介绍的快速启动步骤，本博客启动步骤如下：
	
	1. Install all [prerequisites](https://jekyllrb.com/docs/installation/).
	2. Install the jekyll and bundler gems

		```
		$ gem install jekyll bundler
		# 为避免文件权限报错，个人使用的安装命令如下
		$ sudo gem install -n /usr/local/bin jekyll bundler
		```
		
	3. 进入博客根目录，执行如下命令，启动本地调试服务

		```
		$ bundle exec jekyll serve
		```
	
	4. 如根目录不存在`Gemfile`文件，则需要执行如下命令

		```
		# 初始化Gemfile文件
		$ bundle init
		# 添加依赖
		$ bundle add jekyll
		$ bundle add jekyll-paginate
		$ bundle add webrick
		```
	
	5. 全新克隆的博客项目，如存在`Gemfile`文件，则需要执行以下命令以启动本地调试服务

		```
		# 安装依赖
		$ bundle install
		$ bundle exec jekyll serve
		```
	
	启动成功会显示：
	
	```
	 Configuration file: /Users/meijigao/Desktop/Git•GitHub/Dengni8023/dengni8023.github.io/_config.yml
            Source: /Users/meijigao/Desktop/Git•GitHub/Dengni8023/dengni8023.github.io
       Destination: /Users/meijigao/Desktop/Git•GitHub/Dengni8023/dengni8023.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 0.052 seconds.
 Auto-regeneration: enabled for '/Users/meijigao/Desktop/Git•GitHub/Dengni8023/dengni8023.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
	```
	
	至此，`jekyll`本地服务器搭建、启动成功。你可以在`http://127.0.0.1:4000/`看到你的博客，你对本地博客的修改都会在这个地址进行显示，这大大提高了对博客的配置效率。
	
	使用`ctrl+c`可以停止本地调试服务。

## 致谢

1. 博客搭建感谢作者[快速搭建个人博客](http://qiubaiying.github.io/2017/02/06/快速搭建个人博客/)作者[柏荧(BY)](https://github.com/qiubaiying)

2. 感谢 Jekyll、Github Pages 和 Bootstrap!

## 参考资料

1. [快速搭建个人博客](http://qiubaiying.vip/2017/02/06/快速搭建个人博客/)
2. [Jekyll官方文档](https://jekyllrb.com/docs/)
3. [Jekyll github page博客本地调](https://www.jianshu.com/p/20ea66b43e21)

## License

遵循 MIT 许可证。有关详细,请参阅 [LICENSE](https://github.com/dengni8023/dengni8023.github.io/blob/master/LICENSE)。

