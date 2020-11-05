SSH密钥对是一种安全便捷的登录认证方式。

##前提条件
- 已生成SSH密钥，如何生成我的SSH密钥？可参考：[如何生成我的SSH密钥](
https://app.didiyun.com/static/pages/sshkey.html)

- 已创建云服务器。

- 已选择以”SSH密钥”为登录方式。

- 已为云服务器分配固定公网EIP。

- 使云服务器进入运行中状态。

- 确定云服务器所在的安全组已开放22端口的安全组规则。

##访问

- 登录帐号是dc2-user，通过支持ssh的工具来访问您的云服务器
   
        ssh dc2-user@116.85.30.254
        
- 如果需要root权限，通过dc2-user可以切换到root:
 
        dc2-user@localhost:~$ sudo su
        root@localhost:~#

