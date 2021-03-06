# Sockd
dante-server / sockd ? The private sockd service 

## Option 1 Compile&Install  

### Environment:  
```  
$ cat /etc/redhat-release   
CentOS Linux release 7.3.1611 (Core)   
$ uname -a  
Linux xxx 3.10.0-514.6.2.el7.x86_64 #1 SMP Thu Feb 23 03:04:39 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux  
```  

###  Start to build:  
* Download Source code  
`wget https://www.inet.no/dante/files/dante-1.4.2.tar.gz`  

* Install dependence package  
```  
yum install -y autoconf automake binutils gcc make rpm-build rpmdevtools  
yum install -y bison flex glibc-devel libtool pam-devel  
yum install -y gcc make rpm-build bison flex pam-devel  
```  

* Build package  

`rpmbuild -ta dante-1.4.2.tar.gz`  

###  Install the RPM  
```  
rpm -ivh rpmbuild/RPMS/x86_64/dante-1.4.2-1.el7.centos.x86_64.rpm  
rpm -ivh rpmbuild/RPMS/x86_64/dante-server-1.4.2-1.el7.centos.x86_64.rpm  
```  

## Option 2 Using RPMBuild  
```
cd  
git clone -b master https://github.com/stardock/Sockd  
cd Sockd/x86_64/
ll
rpm -ivh dante-1.4.2-1.el7.x86_64.rpm
rpm -ivh dante-server-1.4.2-1.el7.x86_64.rpm
```

##  Setup configure  
```  
vim /etc/sockd.conf  
```  

Configure for reference  

```
# /etc/sockd.conf

logoutput: syslog
user.privileged: root
user.unprivileged: nobody

# The listening network interface or address.
internal: 0.0.0.0 port=1080

# The proxying network interface or address.
external: eth0

# socks-rules determine what is proxied through the external interface.
# The default of "none" permits anonymous access.
socksmethod: username

# client-rules determine who can connect to the internal interface.
# The default of "none" permits anonymous access.
clientmethod: none

client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    #log: connect disconnect error
}

socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    #log: connect disconnect error
}
```

将上述配置文件中的external修改为VPS实际的流量出口interface，比如我的openVZ机子是venet0:0  
创建一个用户并修改密码，用于，赋予最低的权限。  
```
useradd -r -s /bin/false USERNAME
passwd USERNAME
```

## Test  

使用内置的-V命令测试sockd.conf文件是否正确，若正确则无任何输出  
```
sockd -V
```
直接以daemon模式运行即可  
```
sockd -D
```
使用Telegram设置该地址为socks代理，填入刚才新建的用户名即可。  

## Access Control  

需求如下：socks5服务器仅代理Telegram的IP段（类似于Shadowsocks-libev的ACL功能），对其它代理请求一律拒绝  

只需把sockd.conf最后的  
```
socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
    #log: connect disconnect error
}
```
替换为  
```
socks pass {
    from: 0.0.0.0/0 to: 91.108.4.0/22
}

socks pass {
    from: 0.0.0.0/0 to: 91.108.8.0/21
}

socks pass {
    from: 0.0.0.0/0 to: 91.108.16.0/21
}

socks pass {
    from: 0.0.0.0/0 to: 91.108.36.0/23
}

socks pass {
    from: 0.0.0.0/0 to: 91.108.38.0/23
}

socks pass {
    from: 0.0.0.0/0 to: 91.108.56.0/22
}

socks pass {
    from: 0.0.0.0/0 to: 149.154.160.0/2
}

socks block {
    from: 0.0.0.0/0 to: 0.0.0.0/0
}
```

## Setup firewall to allowed  

firewall-cmd --zone=public --add-port=1080/tcp --permanent  
firewall-cmd --reload  

##  Start your service  

`systemctl restart sockd`  



Special thanks for the reference:  
https://kagasu.hatenablog.com/entry/2017/08/26/173052     (thumb)  
http://lixingcong.github.io/2018/05/25/dante-socks5/  
https://lvii.gitbooks.io/outman/content/dante.html  
