title: Finder里面Go2Shell修改默认iterm2为打开shell
date: 2015-12-05 19:41:53
tags: 
- MAC
- iterm2
- go2shell
categories: 
- MAC
---
Go2Shell是个可以与Finder结合使用的，从目录浏览切换到命令行的工具。  
安装完Go2Shell后，可以把app拖到Finder的工具栏上，从而可以从Finder直接进入命令行。但是，Go2Shell默认打开的是MAC自带的默认terminal，而我们常用的经常是自定义的，比如我用的iterm2，这个时候，我们就需要修改这个默认打开方式了。  
怎么做？一个命令就好  

		open -a Go2Shell --args config  
		
在下面这个图中，勾选相应的命令行终端即可  
>![](http://7xoxy0.com1.z0.glb.clouddn.com/go2shell.png)