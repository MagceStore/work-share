# SELinux
  SELinux是一个安全体系结构，它通过LSM(Linux Security Modules)框架被集成到Linux Kernel 2.6.x中。它是NSA (United States National Security Agency)和SELinux社区的联合项目,是 Linux历史上最杰出的新安全子系统。

操作系统有两类访问控制：自主访问控制（DAC）和强制访问控制（MAC）。<br/>
标准Linux安全是一种DAC，SELinux为Linux增加了一个灵活的和可配置的的MAC。<br/>
只有同时满足了【标准Linux访问控制】和【SELinux访问控制】时，进程才能访问文件。<br/>

## Linux安全问题
  * 管理员不能控制用户：用户可以把谁都可读入的权限赋予敏感文件，例如 ssh key及惯常用来放置这些key的目录，~/.ssh/。
  * 进程可以更改安全性属性：每位用户的邮件文件应该只供该用户读入，但邮件客户端软件有能力将它们改为谁都可读入。
  
## SELinux的优势
  在SELinux中没有root这个概念，安全策略是由管理员来定义的，任何软件都无法取代它。这意味着那些潜在的恶意软件所能造成的损害可以被控制在最小。一般情况
  下只有非常注重数据安全的企业级用户才会使用SELinux。

## 配置文件
  * /etc/sysconfig/selinux是一个软链，真正的配置文件为：/etc/selinux/config 
    * SELINUX=enforcing|permissive|disabled
    * SELINUXTYPE=targeted|minimum|mls
    
## 日志文件
<table>
  <tr>
    <td>auditd on</td>
    <td>/var/log/audit/audit.log</td>
  </tr>
  <tr>
    <td>auditd off; rsyslogd on</td>
    <td>/var/log/messages</td>
  </tr>
  <tr>
    <td>rsyslogd and auditd on</td>
    <td>/var/log/audit/audit.log & /var/log/messages</td>
  </tr>
</table>
### 例子
  /var/log/audit/audit.log
  * type=AVC msg=audit(1378974214.610:465): avc:  denied  { open } for pid=2359 comm="httpd" path="/var/www/html/index.html"
  dev="sda1"ino=1317685 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file

## context
    cp：会重新生成context；
    mv：context不变。
    
  USER：ROLE：TYPE[LEVEL[：CATEGORY]]
  [用户user]:[角色role]:[类型type（SELinux默认预设规则）]:[MLS、MCS]
  unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 
  
  * USER
    * user identity：类似Linux系统中的UID，提供身份识别，用来记录身份
    * 三种常见的 user:
      user_u ：普通用户登录系统后的预设；
      system_u ：开机过程中系统进程的预设；
      root ：root 登录后的预设；
    * 所有预设的 SELinux Users 都是以 “_u” 结尾的，root 除外。
        
  * ROLE
    * 文件、目录和设备的role：通常是 object_r；
    * 程序的role：通常是 system_r；    
    * 使用基于RBAC(Roles Based Access Control) 的strict和mls策略中，用来存储角色信息
        
  * TYPE
    * type：用来将进程和文件划分为不同的组，给每个主体和系统中的文件定义了一个类型；为进程运行提供最低的权限环境；
    * 当一个类型与执行中的进程相关联时，其type也称为domain；
    * type是SElinux security context 中最重要的部分，预设值以_t结尾；

  * LEVEL和CATEGORY：定义层次和分类，只用于mls策略中
    * LEVEL：代表安全等级,目前已经定义的安全等级为s0-s15 （/etc/selinux/targeted/setrans.conf）
    * CATEGORY：代表分类，目前已经定义的分类为c0-c1023

## SELinux相关命令

  * getenforce - 查看当前SELinux运行模式 enforcing|permissive|disabled
  * setenforce — 修改SELinux运行模式，例子如下：
    * setenforce 1 — SELinux以强制(enforcing)模式运行
    * setenforce 0 — SELinux以警告(permissive)模式运行
    * 关闭SELinux，修改配置文件：/etc/selinux/config或/etc/sysconfig/selinux
    * SELinux 拥有三种访问控制方法：
      * 强制类型（TE）：TE 是针对型政策所采用的主要访问控制机制
      * 基于⻆色的访问控制（RBAC）：它以 SELinux 用户（未必等同 Linux 用户）为基础，但缺省的针对型政策并未采用它
      * 多层保障（MLS）：普遍不获采用，而且经常隐藏在缺省的针对型政策内。
  
  * getsebool -a： 列出SELinux的所有布尔值 （安全策略）
  * setsebool： 设置SELinux布尔值，如：setsebool -P dhcpd_disable_trans=0，-P表示即使用reboot之后，仍然有效。
    
  * sestatus -v — 显示系统的详细状态
  * restorecon — 通过为适当的文件或安全环境标记扩展属性，设置一个或多个文件的安全环境 （restorecon -v 查看修改的部分）
  ### 例子
    假设CentOS安装了apache，网页默认的主目录是/var/www/html，我们经常遇到这样的问题，在其他目录中创建了一个网页文件，然后用mv移动到网页默认目录
    /var/www/html中，但是在浏览器中却打不开这个文件，这很可能是因为这个文件的SELinux配置信息是继承原来那个目录的，与/var/www/html目录不同，使
    用mv移动的时候，这个SELinux配置信息也一起移动过来了，从而导致无法打开页面
  
  * fixfiles
    一般是对整个文件系统的， 后面一般跟 relabel，对整个系统 relabel后，一般我们都重新启动。如果，在根目录下有.autorelabel空文件的话，每次重新启动时都调用 fixfiles relabel
    
  * chcon 修改文件、目录的安全上下文 （chcon -t unconfined_exec_t /usr/sbin/httpd）
    * chcon –u[user]
    * chcon –r[role]
    * chcon –t[type] 
    * chcon –R  递归
  
  * semanage fcontext -a -t httpd_sys_content_t "/var/www/html(/.*)?" <br/>
    semanage port -a -t httpd_port_t -p tcp 81 (允许81能够被监听)

  * secon－p 进程domain的确认
  
  * sealert -a /var/log/audit/audit.log > /path/to/mylogfile.txt 
  
  * id 能用来确认自己的 security context。
      
## 其他命令 －
  ls ps -Z
 find –context 查特定的type的文件

## 自定义安全策略
  * SELinux 切换至 Permissive 模式并运行一段时间，便可以在允许访问的情况下记录 SELinux 的问题。
  ```bash
    type=AVC msg=audit(1218128130.653:334): avc:  denied  { connectto } for  pid=9111 comm="smtpd" 
    path="/var/spool/postfix/postgrey/socket"
    scontext=system_u:system_r:postfix_smtpd_t:s0 tcontext=system_u:system_r:initrc_t:s0 tclass=unix_stream_socket
    type=AVC msg=audit(1218128130.653:334): avc:  denied  { write } for  pid=9111 comm="smtpd" name="socket" dev=sda6 
    ino=39977017 scontext=system_u:system_r:postfix_smtpd_t:s0 tcontext=system_u:object_r:postfix_spool_t:s0 
    tclass=sock_file 
  ```
  
  * 通过audit2allow 来生成 安全策略文件
  ```bash
    grep smtpd_t /var/log/audit/audit.log | audit2allow -m postgreylocal > postgreylocal.te
    cat postgreylocal.te
    module postgreylocal 1.0;
    require {
            type postfix_smtpd_t;
            type postfix_spool_t;
            type initrc_t;
            class sock_file write;
            class unix_stream_socket connectto;
    }
    #============= postfix_smtpd_t ==============
    allow postfix_smtpd_t initrc_t:unix_stream_socket connectto;
    allow postfix_smtpd_t postfix_spool_t:sock_file write; 
  ```
  
  * 续继用 audit2allow 创建一个自定的政策模块
  ```bash
    grep smtpd_t /var/log/audit/audit.log | audit2allow -M postgreylocal 
  ```
  
  * 利用 semodule 这个指令将 postgrey 政策模块装入现有的 SELinux 政策内
  ```bash
    semodule -i postgreylocal.pp 
  ```

### 参考资料
 * selinux Project : http://selinuxproject.org/page/Main_Page
 * centos SELinux : https://wiki.centos.org/HowTos/SELinux (https://wiki.centos.org/zh/HowTos/SELinux)
 * fedora SELinux : https://docs.fedoraproject.org/en-US/Fedora/19/html/Security_Guide/ch09.html
