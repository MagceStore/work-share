# SELinux
  SELinux是一个安全体系结构，它通过LSM(Linux Security Modules)框架被集成到Linux Kernel 2.6.x中。它是NSA (United States National Security Agency)和SELinux社区的联合项目,是 Linux历史上最杰出的新安全子系统。

操作系统有两类访问控制：自主访问控制（DAC）和强制访问控制（MAC）。标准Linux安全是一种DAC，SELinux为Linux增加了一个灵活的和可配置的的MAC。

只有同时满足了【标准Linux访问控制】和【SELinux访问控制】时，主体才能访问客体。

Linux系统是使用自主访问控制的，用户可以自己请求更高的权限，由此恶意软件几乎可以访问任何它想访问的文件，而如果你授予其root权限，那它就无所不能了。

在SELinux中没有root这个概念，安全策略是由管理员来定义的，任何软件都无法取代它。这意味着那些潜在的恶意软件所能造成的损害可以被控制在最小。一般情况下只有非常注重数据安全的企业级用户才会使用SELinux。


/etc/sysconfig/selinux配置文件
  /etc/sysconfig/selinux是一个软链，真正的配置文件为：/etc/selinux/config 
    SELINUX=enforcing|permissive|disabled
    SELINUXTYPE=targeted|minimum|mls
    
SELINUXTYPE(安全策略)
   getsebool -a： 列出SELinux的所有布尔值
   setsebool： 设置SELinux布尔值，如：setsebool -P dhcpd_disable_trans=0，-P表示即使用reboot之后，仍然有效。


/etc/selinux/semanage.conf

日志文件
/var/log/audit/audit.log
type=AVC msg=audit(1378974214.610:465): avc:  denied  { open } for pid=2359 comm="httpd" path="/var/www/html/index.html" dev="sda1"ino=1317685 scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0 tclass=file


 SELinux工具

 getenforce - 查看当前SELinux运行模式 enforcing|permissive|disabled
 setenforce — 修改SELinux运行模式，例子如下：

         • setenforce 1 — SELinux以强制(enforcing)模式运行
         • setenforce 0 — SELinux以警告(permissive)模式运行

    关闭SELinux，修改配置文件：/etc/selinux/config或/etc/sysconfig/selinux
    
    /usr/sbin/sestatus -v — 显示系统的详细状态
    /sbin/restorecon — 通过为适当的文件或安全环境标记扩展属性，设置一个或多个文件的安全环境
    假设CentOS安装了apache，网页默认的主目录是/var/www/html，我们经常遇到这样的问题，在其他目录中创建了一个网页文件，然后用mv移动到网页默认目录/var/www/html中，但是在浏览器中却打不开这个文件，这很可能是因为这个文件的SELinux配置信息是继承原来那个目录的，与/var/www/html目录不同，使用mv移动的时候，这个SELinux配置信息也一起移动过来了，从而导致无法打开页面
    
    
    
     /sbin/fixfiles — 检查或校正文件系统中的安全环境数据库
      fixfiles
         一般是对整个文件系统的， 后面一般跟 relabel，对整个系统 relabel后，一般我们都重新启动。如果，在根目录下有.autorelabel空文件的话，每次重新启动时都调用 fixfiles relabel
         
         setfiles / chcon
      chcon 修改文件、目录的安全上下文
      chcon –u[user]
      chcon –r[role]
      chcon –t[type] 
      chcon –R  递归
      
      
      进程domain的确认

      程序现在在那个domain里运行，我们可以在ps 命令后加 －Z进行查看：
      ROLE的确认和变更

     命令id能用来确认自己的 security context。
      
      
context
        • rpm包安装的：会根据rpm包内记录来生成安全上下文；

        • 手动创建的文件：会根据policy中规定的来设置安全上下文；

        • cp：会重新生成安全上下文；

        • mv：安全上下文则不变。
        
         USER：ROLE：TYPE[LEVEL[：CATEGORY]]
         [用户user]:[角色role]:[类型type（SELinux默认预设规则）]:[MLS、MCS]
         unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 
         
USER

         1) user identity：类似Linux系统中的UID，提供身份识别，用来记录身份；安全上下文的一部分；
         2) 三种常见的 user:

              • user_u ：普通用户登录系统后的预设；
             • system_u ：开机过程中系统进程的预设；
             • root ：root 登录后的预设；

        4) 在targeted policy 中比较重要，所有预设的 SELinux Users 都是以 “_u” 结尾的，root 除外。
        
ROLE

        1) 文件、目录和设备的role：通常是 object_r；
        2) 程序的role：通常是 system_r；
        3) 用户的role：targeted policy为system_r； strict policy为sysadm_r、staff_r、user_r；用户的role，类似系统中的GID，不同角色具备不同的的权限；用户可以具备多个role；但是同一时间内只能使用一个role；        

        4) 使用基于RBAC(Roles Based Access Control) 的strict和mls策略中，用来存储角色信息
        
TYPE

        1) type：用来将主体(subject)和客体(object)划分为不同的组，给每个主体和系统中的客体定义了一个类型；为进程运行提供最低的权限环境；
        2) 当一个类型与执行中的进程相关联时，其type也称为domain；
        3) type是SElinux security context 中最重要的部位，是 SELinux Type Enforcement 的心脏，预设值以_t结尾；

        LEVEL和CATEGORY：定义层次和分类，只用于mls策略中
             • LEVEL：代表安全等级,目前已经定义的安全等级为s0-s15,等级越来越高
             • CATEGORY：代表分类，目前已经定义的分类为c0-c1023



命令修改 
 cp

        可以跟 -Z,--context=CONTEXT 在拷贝的时候指定目的地文件的security context
 ls ps
 find
        可以跟 –context 查特定的type的文件


http://selinuxproject.org/page/Main_Page
