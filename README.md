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
