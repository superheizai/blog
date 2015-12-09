title: Shell之定时任务
date: 2015-12-04 16:54:45
tags: 
- cron
- 定时任务
categories: Shell
---

## crond服务
crond是linux下面的定时任务框架，是可以重复执行的定时任务。还有一种@，at定时任务，是一次性的定时任务。  

定时任务服务的操作：

	/sbin/service crond start           //启动服务
	/sbin/service crond stop            //关闭服务
	/sbin/service crond restart        //重启服务
	/sbin/service crond reload         //重新载入配置
	/sbin/service crond status        //查看服务状态  
	
## crond介绍	

crontab在/etc目录下面，存在以下目录及文件  

 + cron.hourly
 + cron.daily
 + cron.weekly
 + cron.monthly
 + cron.d
 + crontab
 + cron.deny
 + cron.allow
 
 
<!--more-->

### 目录
前五个是目录，前四种顾名思义，不用解释。cron.d是系统自动定期需要做的任务,但是又不是按小时,按天,按星期,按月来执行的,那么就放在这个目录下面.  


### 文件
crontab文件一般如下:  

	01 * * * * root run-parts /etc/cron.hourly
	02 4 * * * root run-parts /etc/cron.daily
	22 4 * * 0 root run-parts /etc/cron.weekly
	42 4 1 * * root run-parts /etc/cron.monthly
	
我们可在此文件中添加自己需要的cron job.  


#### crontab的限制
　　cron.allow：将可以使用 crontab 的帐号写入其中，若不在这个文件内的使用者则不可使用 crontab；

　　cron.deny：将不可以使用 crontab 的帐号写入其中，若未记录到这个文件当中的使用者，就可以使用 crontab 。

以优先顺序来说， /etc/cron.allow 比 /etc/cron.deny 要优先， 而判断上面，这两个文件只选择一个来限制而已，因此，建议你只要保留一个即可， 免得影响自己在配置上面的判断！一般来说，系统默认是保留 /etc/cron.deny ， 你可以将不想让他运行 crontab 的那个使用者写入 /etc/cron.deny 当中，一个帐号一行！

## 定时任务定制  

	crontab [-u username] [-l|-e|-r]
	选项与参数：
	-u  ：只有 root 才能进行这个任务，亦即帮其他使用者创建/移除 crontab 工作排程；
	-e  ：编辑 crontab 的工作内容
	-l  ：查阅 crontab 的工作内容
	-r  ：移除所有的 crontab 的工作内容，若仅要移除一项，请用 -e 去编辑

我们一般使用crontab -e 来进行编辑任务，输入这个命令后，会以vi打开当前任务，退出保存的时候，也会提示当前定时任务已经更新。
  个人的定时任务全部保存在/var/spool/cron目录下面，会生成以当前用户名的名称的个人任务列表。
  cron运行的每一项工作都会被记录到/var/log/cron里面，可以在里面查看运行的定时任务记录。

## 定时任务语法  
在启动定时任务
crontab的格式讲解
每项工作 (每行) 的格式都是具有六个栏位，这六个栏位的意义为：  
代表意义	分钟	小时	日期	月份	周	
数字范围	0-59	0-23	1-31	1-12	0-7	
 
注意两点：  
 + 周的数字为 0 或 7 时，都代表『星期天』的意思  
 + 周几与月日不可同时并存，否则会月日和周几都去执行  

还有一些辅助的字符，大概有底下这些：

+  *: 星号代表任何时刻都接受    
+  ,: 逗号代表分隔时段的意思，下达的工作是 3:00 与 6:00 时，就会是：
0 3,6 * * * command  
+  -: 减号代表一段时间范围内，比如20 8-12 * * * command，代表8点20到12点20 
+  /n: n 代表数字，每隔n单位间隔的意思，*/5 * * * * command，每隔5分钟

## 注意事项

+ 环境变量，在要执行的脚本里面，要去source下环境变量，否则很多命令看不到  
+ 尽量使用绝对路径，不用相对路径，避免不必要的麻烦   
+ ``执行命令
+ 当程序在你所指定的时间执行后，系统会寄一封信给你，显示该程序执行的内容，若是你不希望收到这样的信，请在每一行空一格之后加上 > /dev/null 2>&1 即可
+ run-parts是系统命令，全名/usr/bin/run-parts，后跟目录名，执行某一路径下全部命令

	
	
## 自己脚本例子


	#!/bin/bash
	# source环境变量，让命令和变量在cron脚本可见
	source /root/.bashrc
	source /etc/profile
	
	echo "cron tab job  `date -d today +"%Y-%m-%d %T"`" >> /opt/scan_log
	
	# 9000端口较多，但是监听的应该只有一个，所以要grep下”（LISTEN）“
	cnt=`lsof -i :9000| grep "(LISTEN)" | grep java | wc -l`
	if [ $cnt -eq 1 ];then
	echo "alreay Running `date -d today +"%Y-%m-%d %T"`" >> /opt/scan_log
	exit 0
	else
	echo "start penguin `date -d today +"%Y-%m-%d %T"`" >> /opt/scan_log
	`cd /apps/penguin-penguin4sdk;rm -rf RUNNING_PID;cd bin;chmod 777 	penguin;./penguin -Denv=stage -Dhttp.port=9000  -Dconfig.file=../	conf/penguin.conf -Dlogger.file=conf/penguin-logger.xml > /tmp/	penguinForleo_nohup.out &`
	fi

## 参考资料
[cron实用手册](http://www.cnblogs.com/ggjucheng/archive/2012/08/19/2646763.html)  
[cron定时任务配置](http://www.cnblogs.com/kerrycode/p/3238346.html)
[]()
[]()
