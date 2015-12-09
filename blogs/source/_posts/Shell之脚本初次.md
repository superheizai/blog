title: Shell之脚本初次
date: 2015-12-04 16:52:47
tags: 
- Shell
- 脚本
categories: Shell
---


## 背景  
由于自己与服务器有很多台，可能由于需要重启服务等原因，每次都要到ssh登陆到远程机器，进行操作，这样子每次登陆都要输入命令，很麻烦。所以，我想通过脚本形式来解决ssh登陆的问题，这次脚本碰到的问题，也是由于这个产生的。  

## 大纲  
1. 变量
2. 环境
2. 命令执行与输出
3. 管道与子shell
4. 疑惑点  
5. 脚本例子

<!--more-->
### 变量  

关于变量的操作，主要碰到两个问题，一个是expect下面，用set方式来定义，而bash下面直接用“=”来赋值。其实“=”赋值的时候，左右两边都是不能有空格的，如果值域上面有特殊字符或者空格之类的，最好用“”括起来。  

### 环境
  
   1. 环境配置  
   由于我的开发是在iterm2，安装了oh-my-zsh之后进行的开发，所以在配置的时候要小心点。现在的更新，应该在~/.zshrc上面进行修改，完成后需要source配置文件.  
   2. 新脚本授权  
   在新脚本建立完毕后，要给它可执行权限，以便已“./filename.sh"方式来执行  
   
   		chmod +x filename.sh

### 命令执行与输出  

1. awk的使用
   在使用脚本的时候，希望某些值以配置文件的形式定义在文件系统中，运行的时候从文件中读取相应的值来做判断，这些值的读取与解析就是通过awk来完成的。执行过程中，碰到了如下一些问题：  
    + 读取文件内容并执行
    + 分隔符的使用
    + 返回数组  
    
    以例子来说明这几个问题及其解决方案：	
    	
		// line是读取文件内容的当前行，此行代码即读取当前行并通过awk进行解析  
		// -F"="是指当前的分隔符是“=“，把每行文本以”=“为分隔符，分成$1,$2  
		name=`echo "$line" | awk -F"=" '{print $1,$2}'`  
		//name返回的两个逗号分隔字符串，正好通过“数组=(逗号分隔字符串）”拼成数组  
		name=($name)  
		
2. 不同域下面的脚本执行  
   由于我们写ssh自动登陆脚本是需要用到expect的，而expect是在bash下面的，并不在bash里面。所以，在普通的bash脚本里面，由于执行环境定义，一般只定义为/bin/bash，是不能够直接运行expect相关的命令的，所以，我们需要把脚本分成两个部分。一个部分是bash脚本部分，一个是expect脚本部分,在bash脚本里面，通过如下命令行去执行expect脚本  
   
		expect sub_shell $param  
		
### 管道与子shell  

 这里先给出一份之前写的代码，之前没有这个概念：管道其实是子shell，子shell的修改，父shell其实是拿不到的,所以下面这段代码中，对server变量的修改，在这段程序结束后，就丢失了  
 
	cat /Users/superheizai/shells/config | while read line
		do
		name=`echo "$line" | awk -F"=" '{print $1,$2}'`
		name=($name)
		if [[ ${name[0]} = $param ]];then
			server=${name[1]}
		    break
		fi
		done


  修改的点也比较简单，就是不用管道，利用"<"操作符，直接从文件输出到while循环中。  
  
  	while read line
	 do
       name=`echo "$line" | awk -F"=" '{print $1,$2}'`
       name=($name)
       if [[ ${name[0]} = $param ]];then
          server=${name[1]}
          echo $server
          break
        fi
     done < /Users/superheizai/shells/config
		
### 疑惑点
   最主要的疑惑点是，当前bash下面和expect里面，语法使用不一样，比如变量定义，一个用set，一个用“=”；比如if语句，一个判断用if{}{}，一个用if[];then fi	
### 脚本例子
   这里把自己的写的脚本和相关文件贴出来，分为bash脚本和expect脚本，bash脚本会在内部调用expect脚本  
   
   + 配置文件config
   		
			a=10.199.XXX.xx 
			b=10.199.xxx.xx  
			c=10.199.xxx.xx  
			d=10.199.xxx.xx  
			  
   +  login_expect.sh  
   
   			#!/usr/bin/expect

			set user [lindex $argv 0]
			set pwd [lindex $argv 1]
			set server [lindex $argv 2]
			send_user "$user--$pwd--$server"
			spawn ssh $user@$server -p 22
			send_user "start ssh\n"
			expect  {
   			"*password:" {send $pwd\r}
  			 "yes/no" {send "yes\r"}
			}
			expect "]#"
			send "echo ssh started-------\r"
			interact  
			
			
+ login.sh  

	
				#!/bin/bash
			
				user=user
				pwd=pwd
				server="10.199.XXX.XX"
			
				if [[ $# -eq 1 ]]; then
				param=$1
				cat /Users/superheizai/shells/config | while read line
				do
				      name=`echo "$line" | awk -F"=" '{print $1,$2}'`
				      name=($name)
				     if [[ ${name[0]} = $param ]];then
				       server=${name[1]}
				       break
				      fi
				done
			
				fi
			
				if [[ $# -eq 2 ]];then
				param=$1
				cat /Users/superheizai/shells/config | while read line
				do
				      name=`echo "$line" | awk -F"=" '{print $1,$2}'`
				      name=($name)
				     if [[ ${name[0]} = $param ]];then
				       server=${name[1]}
				       break
				       fi
			
				done	
			
				 pwd=$2
				fi
				expect login_expect $user  $pwd $server
