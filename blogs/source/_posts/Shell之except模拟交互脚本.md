title: Shell之except模拟交互脚本
date: 2015-12-04 16:55:15
tags: expect
categories: Shell
---
## 背景
   
 由于需要写个脚本自动SSH到云主机，启动某个服务，所以写了这个脚本。中间有段时间一直登陆失败，后来查看下来，原来是因为开启了VPN。  
## 代码及其分析  

 先来一份代码示例  
      //1
 		#!/usr/bin/expect 

   		//2
		set user user 
		set pwd pwd
		set server 10.199.XXX.XX


		//3
		spawn ssh $user@$server -p 22
		//9 
		send_user "start ssh\n"
		//4
		expect  {
  		 "*password:" {send $pwd\r}
  		 "yes/no" {send "yes\r"}
		}
		expect "]#"
		//10
		log_file log_result
		//5
		send "ps -ef\r"
		//6  
		send "ret=`echo $?`\r"
		send "exit \$ret\r"
		//7
		#interact
		//8
		#expect eof
		
		<!--more-->
		
这里简单介绍下我的这段代码：  
 1. 定义此脚本是通过/usr/bin/expect来执行。如果没有这行直接执行，会告诉你spawn，send等命令找不到  
 2. 此处的代码是设置shell里面的变量  
 3. spawn命令会启动一个进程，用来给ssh运行进程加个壳，传递交互指令。再强调一次，spawn是进入expect之后才可以看到的指令，用which spawn是看不到spawn命令的  
 4. expect可以有一个或者多个返回，多个返回的时候可以用{}括起来对不同情况的处理，一个返回的时候，直接就省略掉{}.expect多个返回时候，"{"必须和expect同一行    
 5. 发送执行命令指令  
 6. 这里是希望看到执行结果，所以我用了$?这个上次执行结果输出到变量  
 7. 这个指令是指继续留在SSH登陆状态,这里以内执行了exit，直接退出了,interact不起作用  
 8. 执行结束标示，在spawn进程结束后会给expect发送eof  
 9. 给用户提示信息，只作纯文本的提示，像echo一样  
 10. log_file filename会把日志打到filename文本里面，默认是append，所以会有文件大小问题

 一些参考资料  
 [ expect学习笔记及实例详解 ](http://blog.chinaunix.net/uid-15007890-id-273604.html)  
 [通过wait获取返回值](http://zhidao.baidu.com/link?url=NTNequSIXB5Hq38HGD1lxCTIXxaC8kKZQUZdNj-EGhu6JZmWMLFyOQmiPdZ4DW-lp0p_rJjzF8MU5tBkVwuaPq)  
 [expect-return-code](http://stackoverflow.com/questions/23614039/how-to-get-the-exit-code-of-spawned-process-in-expect-shell-script)
 