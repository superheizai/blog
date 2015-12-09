title: Git 多Repo配置
date: 2015-12-04 13:08:24
tags: 
- git 
- 多repo 
- config
categories: 
- git
---
在需要连接多个repo的场景下，可以通过添加config文件来解决这个问题，步骤如下：

## 生成ssh key
   
     ssh-keygen -t rsa -C "superheizai" -f ~/.ssh/github
     
     -C ：指的是目标GIT repo的用户名
     -f ：指的是生成密钥的本地地址
   
   运行命令后，会在~/.ssh下面生成github私钥和github.pub公钥，把github公钥写到对应的GIT repo配置
   
## 定义config

  在.ssh下面生成一个config文件，里面配置如下信息：    
  
    host github.com #GIT REPO域名，比如git@github.com中的github.com
	 user git  #用户，一般都是这个,比如git@github.com中的git
	 hostname github.com  #可选，指向域名，比如git@github.com中的github.com
	 port 22  #端口，一般不变
	 identityfile ~/.ssh/github #ssh key生成的时候，文件路径
	
	
## 测试连通性
 ssh -T host # host是config里面定义的host
 
 如果提示连通，说明生效了  