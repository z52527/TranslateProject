
如何在Linux/Unix中的Web服务根文档目录(DocumentRoot)上设置只读文件权限
======
我是如何对存放在/var/www/html/目录中的所有我的文件设置只读权限的？


你可以使用`chmod`命令对Linux / Unix / macOS / Apple OS X / *BSD操作系统上的所有文件来设置只读权限。这篇文章介绍如何在Linux/Unix的web服务器（如 Nginx, Lighttpd, Apache等）上来设置只读文件权限。
[![Proper read-only permissions for Linux/Unix Nginx/Apache web server's directory][1]][1]

### 如何设置文件为只读模式


语法为:

```
### 仅针对文件 ##
chmod 0444 /var/www/html/*
chmod 0444 /var/www/html/*.php
```

### 如何设置目录为只读模式

语法为：
```
### 仅针对目录##
chmod 0444 /var/www/html/
chmod 0444 /path/to/your/dir/
# ***************************************************************************
# Say webserver user/group is www-data, and file-owned by ftp-data user/group
# 假如webserver用户/用户组是www-data ,文件拥有者是ftp-data用户/用户组
# ***************************************************************************
# 设置目录所有文件为只读
chmod -R 0444 /var/www/html/
#设置文件/目录拥有者为ftp-data
chown -R ftp-data:ftp-data /var/www/html/
# 所有目录和子目录的权限为0445 (这样webserver用户或用户组就可以读取我们的文件)
find /var/www/html/ -type d -print0 | xargs -0 -I {} chmod 0445 "{}"
```
找到所有/var/www/html下的所有文件（包括子目录），键入：

```
### 仅对文件有效##
find /var/www/html -type f -iname "*" -print0 | xargs -I {} -0 chmod 0444 {}
```
然而，你需要在/var/www/html目录及其子目录上设置只读和执行权限，如此才能让web server能够访问根目录，键入：

```
### 仅对目录有效 ##
find /var/www/html -type d -iname "*" -print0 | xargs -I {} -0 chmod 0544 {}
```

### 警惕写权限


请注意在/var/www/html/目录上的写权限会允许任何人删除文件或添加新文件。也就是说，你可能需要设置一个只读权限给/var/www/html/目录本身。
```
### web根目录只读##
chmod 0555 /var/www/html
```
在某些情况下，根据你的设置要求，你可以改变文件的属主和属组来设置严格的权限。
```
### 如果/var/www/html目录的拥有人是普通用户，你可以设置拥有人为： root:root 或 httpd:httpd (推荐) ###
chown -R root:root /var/www/html/
 
### 确保apache拥有/var/www/html/ ##
chown -R apache:apache /var/www/html/
```

### 关于NFS导出目录

你可以在/etc/exports文件中指定哪个目录应该拥有[只读或者读写权限 ][2] . 这个文件定义各种各样的共享在NFS服务器和他们的权限。如：

```
# 对任何人只读权限
/var/www/html *(ro,sync) 
 
# 对192.168.1.10(upload.example.com)客户端读写权限访问
/var/www/html 192.168.1.10(rw,sync)
```


### 关于Samba (CIFS)只读共享对MS-Windows客户端


共享sales为只读，更新smb.conf，如下：
```
[sales]
comment = Sales Data
path = /export/cifs/sales
read only = Yes
guest ok = Yes
```

### 关于文件系统表（file systems table）

你可以在Unix/Linux上的/etc/fstab文件中配置挂载某些文件为只读模式。
你需要有专用分区，不要设置其他系统分区为只读模式。

如下在/etc/fstab文件中设置/srv/html为只读模式。
```
/dev/sda6 /srv/html ext4 ro 1 1
```
你可以使用`mount`命令[重新挂载分区为只读模式][3]（使用root用户）

```
# mount -o remount,ro /dev/sda6 /srv/html
```
或者
```
# mount -o remount,ro /srv/html
```


上面的命令会尝试重新挂载已挂载的文件系统在/srv/html上。这是改变文件系统挂载标志的常用方法，特别是让只读文件改为可写的。这种方式不会改变设备或者挂载点。让文件变得再次可写,键入：
```
# mount -o remount,rw /dev/sda6 /srv/html
```
或
```
# mount -o remount,rw /srv/html
```

### Linux: chattr 命令


你可以在Linux文件系统上使用`chattr`命令[改变文件属性为只读][4],如：
```
chattr +i /path/to/file.php
chattr +i /var/www/html/
 
# 查找任何在/var/www/html下的文件并设置为只读#
find /var/www/html -iname "*" -print0 | xargs -I {} -0 chattr +i {}
```


通过提供`-i`选项可删除只读属性
```
chattr -i /path/to/file.php
```

FreeBSD, Mac OS X 和其他BSD Unix用户可使用[`chflags`命令][5]:

```
### 设置只读 ##
chflags schg /path/to/file.php
 
### 删除只读 ##
chflags noschg /path/to/file.php
```


--------------------------------------------------------------------------------

来源: https://www.cyberciti.biz/faq/howto-set-readonly-file-permission-in-linux-unix/

作者：[Vivek Gite][a]
译者：[yizhuoyan](https://github.com/yizhuoyan)
校对：[校对者ID](https://github.com/校对者ID)

本文由 [LCTT](https://github.com/LCTT/TranslateProject) 原创编译，[Linux中国](https://linux.cn/) 荣誉推出

[a]:https://www.cyberciti.biz
[1]:https://www.cyberciti.biz/media/new/faq/2012/04/linux-unix-set-read-only-file-system-permission-for-apache-nginx.jpg
[2]:https://www.cyberciti.biz//www.cyberciti.biz/faq/centos-fedora-rhel-nfs-v4-configuration/
[3]:https://www.cyberciti.biz/faq/howto-freebsd-remount-partition/
[4]:https://www.cyberciti.biz/tips/linux-password-trick.html
[5]:https://www.cyberciti.biz/tips/howto-write-protect-file-with-immutable-bit.html