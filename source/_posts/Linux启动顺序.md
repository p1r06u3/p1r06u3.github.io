---
title: Linux启动顺序
date: 2017-02-07 23:51:42
categories: 安全运维
tags: Linux
---

[一、运行/sbin/init](#id1)

[二、读取/etc/inittab文件](#id2)

[三、初始化系统/etc/rc.d/rc.sysinit](#id3)
<!--more-->

[四、启动对应运行级别的守护进程(/etc/rc.d/rc--->/etc/rc.d/rc*.d/)](#id4)

[五、用户自定义引导程序（/etc/rc.d/rc.local）](#id5)

[六、启动终端/sbin/mingetty和X-Window界面](#id6)

[七、登录系统/bin/login](#id7)

[八、加载/bin/bash，设置环境变量](#id8)


## <a name="id1">1. 运行/sbin/init</a>
init的进程号是1，init进程是系统所有进程的起点，Linux在完成核内引导以后，就开始运行init程序，Linux启动的初始化程序，所有进程的父进程。
## <a name="id2">2. 读取/etc/inittab文件</a>
init程序接着需要读取配置文件/etc/inittab。inittab是一个不可执行的文本文件，它有若干指令所组成。

在Rethat系统中，inittab内容如下所示：
```
[root@localhost etc]# vi inittab    

#
# inittab       This file describes how the INIT process should set up
#               the system in a certain run-level.
#
# Author:       Miquel van Smoorenburg, <miquels@drinkel.nl.mugnet.org>
#
# inittab       This file describes how the INIT process should set up
#               the system in a certain run-level.
#
# Author:       Miquel van Smoorenburg, <miquels@drinkel.nl.mugnet.org>
#               Modified for RHS Linux by Marc Ewing and Donnie Barnes
#

# Default runlevel. The runlevels used by RHS are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode
#   4 - unused
#   5 - X11
#   6 - reboot (Do NOT set initdefault to this)
#
id:5:initdefault:

# System initialization.
si::sysinit:/etc/rc.d/rc.sysinit

l0:0:wait:/etc/rc.d/rc 0
l1:1:wait:/etc/rc.d/rc 1
l2:2:wait:/etc/rc.d/rc 2
l3:3:wait:/etc/rc.d/rc 3
l4:4:wait:/etc/rc.d/rc 4
l5:5:wait:/etc/rc.d/rc 5
l6:6:wait:/etc/rc.d/rc 6

# Trap CTRL-ALT-DELETE
ca::ctrlaltdel:/sbin/shutdown -t3 -r now

# When our UPS tells us power has failed, assume we have a few minutes

l0:0:wait:/etc/rc.d/rc 0
l1:1:wait:/etc/rc.d/rc 1
l2:2:wait:/etc/rc.d/rc 2
l3:3:wait:/etc/rc.d/rc 3
l4:4:wait:/etc/rc.d/rc 4
l5:5:wait:/etc/rc.d/rc 5
l6:6:wait:/etc/rc.d/rc 6

# Trap CTRL-ALT-DELETE
ca::ctrlaltdel:/sbin/shutdown -t3 -r now

# When our UPS tells us power has failed, assume we have a few minutes
# of power left.  Schedule a shutdown for 2 minutes from now.
# This does, of course, assume you have powerd installed and your
# UPS connected and working correctly.
pf::powerfail:/sbin/shutdown -f -h +2 "Power Failure; System Shutting Down"

# If power was restored before the shutdown kicked in, cancel it.
pr:12345:powerokwait:/sbin/shutdown -c "Power Restored; Shutdown Cancelled"


# Run gettys in standard runlevels
1:2345:respawn:/sbin/mingetty tty1
2:2345:respawn:/sbin/mingetty tty2
3:2345:respawn:/sbin/mingetty tty3
4:2345:respawn:/sbin/mingetty tty4
5:2345:respawn:/sbin/mingetty tty5
6:2345:respawn:/sbin/mingetty tty6

# Run xdm in runlevel 5
x:5:respawn:/etc/X11/prefdm -nodaemon

```
> Runlevel 0 系统停机状态，系统默认运行级别不能设为0，否则不能正常启动（关机）

> Runlevel 1 单用户工作状态，root权限，用于系统维护，禁止远程登陆

> Runlevel 2 多用户状态(没有NFS)

> Runlevel 3 完全的多用户状态(有NFS)，登陆后进入控制台命令行模式（服务器默认模式）

> Runlevel 4 系统未使用，保留

> Runlevel 5 X11控制台，登陆后进入图形GUI模式

> Runlevel 6 系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动（重启）

## <a name="id3">3. 初始化系统/etc/rc.d/rc.sysinit</a>
开始启动运行等级的服务前，检测与初始化系统环境，在/etc/inittab文件中有写。

它是一个bash shell脚本，是每一个运行级别都要首先运行的重要脚本。

它主要完成的工作有：**激活交换分区** ，**检查磁盘**，**加载硬件模块**以及其它一些需要优先执行任务。

>（1）获取网络环境与主机类型。首先会读取网络环境设置文件"/etc/sysconfig/network"，获取主机名称与默认网关等网络环境。

>（2）测试与载入内存设备/proc及usb设备/sys。除了/proc外，系统会主动检测是否有usb设备，并主动加载usb驱动，尝试载入usb文件系统。

>（3）决定是否启动SELinux。

>（4）接口设备的检测与即插即用（pnp）参数的测试。

>（5）用户自定义模块的加载。用户可以再"/etc/sysconfig/modules/*.modules"加入自定义的模块，此时会加载到系统中。

>（6）加载核心的相关设置。按"/etc/sysctl.conf"这个文件的设置值配置功能。

>（7）设置系统时间（clock）。

>（8）设置终端的控制台的字形。

>（9）设置raid及LVM等硬盘功能。

>（10）以方式查看检验磁盘文件系统。

>（11）进行磁盘配额quota的转换。

>（12）重新以读取模式载入系统磁盘。

>（13）启动quota功能。

>（14）启动系统随机数设备（产生随机数功能）。

>（15）清除启动过程中的临时文件。

>（16）将启动信息加载到"/var/log/dmesg"文件中。

## <a name="id4">4. 启动对应运行级别的守护进程(/etc/rc.d/rc--->/etc/rc.d/rc*.d/)</a>
在rc.sysinit执行后，将返回init继续其它的动作，通常接下来会执行到/etc/rc.d/rc程序。以运行级别3为例，init将执行配置文件inittab中的以下这行：
```
l5:5:wait:/etc/rc.d/rc 5
```
这一行表示以5为参数运行/etc/rc.d/rc，/etc/rc.d/rc是一个Shell脚本，它接受5作为参数，去执行/etc/rc.d /rc5.d/目录下的所有的rc启动脚本，/etc/rc.d/rc5.d/目录中的这些启动脚本实际上都是一些链接文件，而不是真正的rc启动脚本，真正的rc启动脚本实际上都是放在/etc/rc.d/init.d/目录下。而这些rc启动脚本有着类似的用法，它们一般能接受start、stop、 restart、status等参数。

/etc/rc.d/rc5.d/中的rc启动脚本通常是K或S开头的链接文件，对于以以S开头的启动脚本，将以start参数来运行。而如果发现存在相应的脚本也存在K打头的链接，而且已经处于运行态了(以/var/lock/subsys/下的文件作为标志)，则将首先以stop为参数停止这些已经启动了的守护进程，然后再重新运行。这样做是为了保证是当init改变运行级别时，所有相关的守护进程都将重启。
```
[root@localhost rc0.d]# ls -al S01halt 
lrwxrwxrwx 1 root root 14 Jun 17  2016 S01halt -> ../init.d/halt
[root@localhost rc0.d]# runlevel
N 5
[root@localhost rc0.d]# vi /etc/rc.d/rc.sysinit 
[root@localhost rc0.d]# ls
K01dnsmasq         K20nfs           K74haldaemon       K89dund
K01smartd          K20rwhod         K74lvm2-monitor    K89hidd
K02avahi-daemon    K24irda          K74nscd            K89iscsi
K02avahi-dnsconfd  K25phpstudy      K74ntpd            K89iscsid
K02NetworkManager  K25squid         K74rsyslog         K89netplugd
K02oddjobd         K25sshd          K75netfs           K89pand
K03yum-updatesd    K30sendmail      K85mdmonitor       K89rdisc
K05anacron         K30spamassassin  K85mdmpd           K90bluetooth
K05atd             K35smb           K85messagebus      K90network
K05conman          K35vncserver     K85rpcgssd         K91capi
K05innd            K35winbind       K85rpcidmapd       K91isdn
K05jexec           K44rawdevices    K86nfslock         K92ip6tables
K05saslauthd       K50netconsole    K87irqbalance      K92iptables
K05wdaemon         K50tux           K87mcstrans        K95firstboot
K10cups            K50vsftpd        K87multipathd      K95kudzu
K10dc_server       K50xinetd        K87named           K99cpuspeed
K10hplip           K60crond         K87portmap         K99microcode_ctl
K10psacct          K69rpcsvcgssd    K87restorecond     K99readahead_early
K10tcsd            K72autofs        K88auditd          K99readahead_later
K10xfs             K73ipmi          K88pcscd           S00killall
K12dc_client       K73ypbind        K88syslog          S01halt
K15gpm             K74acpid         K88wpa_supplicant

```
至于在每个运行级中将运行哪些守护进程，用户可以通过chkconfig或setup中的"System Services"来自行设定。

如何增加一个服务：

>1.服务脚本必须存放在/etc/ini.d/目录下；

>2.chkconfig --add servicename

> 在chkconfig工具服务列表中增加此服务，此时服务会被在/etc/rc.d/rcN.d中赋予K/S入口了；

>3.chkconfig --level 345 mysqld on
    修改服务的默认启动等级。

其他使用范例：
>chkconfig --list        #列出所有的系统服务

>chkconfig --del httpd        #删除httpd服务

>chkconfig --list mysqld        #列出mysqld服务设置情况

>chkconfig --level 35 mysqld on        #设定mysqld在等级3和5为开机运行服务，--level 35表示操作只在等级3和5执行，on表示启动，off表示关闭

>chkconfig mysqld on        #设定mysqld在各等级为on，“各等级”包括2、3、4、5等级


常见的守护进程有：
> amd：自动安装NFS守护进程

> apmd:高级电源管理守护进程

> arpwatch：记录日志并构建一个在LAN接口上看到的以太网地址和IP地址对数据库

> autofs：自动安装管理进程automount，与NFS相关，依赖于NIS

> crond：Linux下的计划任务的守护进程

> named：DNS服务器

> netfs：安装NFS、Samba和NetWare网络文件系统

> network：激活已配置网络接口的脚本程序

> nfs：打开NFS服务

> portmap：RPC portmap管理器，它管理基于RPC服务的连接

> sendmail：邮件服务器sendmail

> smb：Samba文件共享/打印服务

> syslog：一个让系统引导时起动syslog和klogd系统日志守候进程的脚本

> xfs：X Window字型服务器，为本地和远程X服务器提供字型集

> Xinetd：支持多种网络服务的核心守护进程，可以管理wtp、sshd、telnet等服务

这些守护进程也启动完成了，rc程序也就执行完了，然后又将返回init继续下一步。

## <a name="id5">5. 用户自定义引导程序（/etc/rc.d/rc.local）</a>
一般来说，自定义的程序不需要执行上面所说的繁琐的建立shell增加链接文件的步骤，只需要将命令放在rc.local里面就可以了，这个shell脚本就是保留给用户自定义启动内容的。

```
[root@localhost ~]# ls /etc/rc.d/
init.d  rc  rc0.d  rc1.d  rc2.d  rc3.d  rc4.d  rc5.d  rc6.d  rc.local  rc.sysinit
```
## <a name="id6">6. 启动终端/sbin/mingetty和X-Window界面</a>
rc执行完毕后，返回init。这时基本系统环境已经设置好了，各种守护进程也已经启动了。init接下来会打开6个终端，以便用户登录系统。通过按Alt+Fn(n对应1-6)可以在这6个终端中切换。在inittab中的以下6行就是定义了6个终端：


1:2345:respawn:/sbin/mingetty tty1

2:2345:respawn:/sbin/mingetty tty2

3:2345:respawn:/sbin/mingetty tty3

4:2345:respawn:/sbin/mingetty tty4

5:2345:respawn:/sbin/mingetty tty5

6:2345:respawn:/sbin/mingetty tty6

从上面可以看出在2、3、4、5的运行级别中都将以respawn方式运行mingetty程序，mingetty程序能打开终端、设置模式。同时它会显示一个文本登录界面，这个界面就是我们经常看到的登录界面，在这个登录界面中会提示用户输入用户名，而用户输入的用户将作为参数传给login程序来验证用户的身份。

除了这6个之外还会执行"/etc/X11/prefdm -nodaemon"，这个主要启动X-Window，显示一个文本登录界面，这个界面就是我们经常看到的登录界面，在这个登录界面中会提示用户输入用户名，而用户输入的用户将作为参数传给login程序来验 证用户的身份。

## <a name="id7">7. 登录系统/bin/login</a>
Runlevel 为5的图形方式用户登录，通过图形化的登录界面输入用户名和密码登录。登录后可以直接进入KDE、Gnome等窗口管理器。

文本方式登录：

当我们看到mingetty的登录界面时，我们就可以输入用户名和密码来登录系统了。

Linux的账号验证程序是login，login接受mingetty传来的用户名作为用户名参数。然后login会对用户名进行分析：如果用户名 不是root，且存在/etc/nologin文件，login将输出nologin文件的内容，然后退出。这通常用来系统维护时防止非root用户登 录。只有/etc/securetty中登记了的终端才允许root用户登录，如果不存在这个文件，则root可以在任何终端上登录。/etc/usertty文件用于对用户作出附加访问限制，如果不存在这个文件，则没有其他限制。

在分析完用户名后，login将搜索/etc/passwd以及/etc/shadow来验证密码以及设置账户的其它信息，比如：主目录是什么、使用何种shell。如果没有指定主目录，将默认为根目录；如果没有指定shell，将默认为/bin/bash。

login程序登录成功后，会向对应的终端再输出最近一次登录的信息（在/var/log/lastlog中有记录），并检查用户是否有新邮件(在/usr/spool/mail/的对应用户名目录下)。

## <a name="id8">8. 加载/bin/bash，设置环境变量</a>
>/bin/bash是人与Linux进行终端交互的程序

/bin/login开始设置各种环境变量：对于/bin/bash来说，系统首先寻找/etc/profile脚本文件，并执行它；然后如果用户的主目录中存在.bash_profile文件，就执行它，在这些文件中又可能调用了其它配置文件，所有的配置文件执行后后，各种环境变量也设好了，这时会出现大家熟悉的命令行提示符，到此整个启动过程就结束了。

当系统登录成功后，系统会加载环境变量用来做实例、函数、应用支持。Linux中环境变量包括系统级和用户级，系统级的环境变量是每个登录到系统的用户都要读取的系统变量，而用户级的环境变量则是该用户使用系统时加载的环境变量。

### 1）/etc/profile
> bash首先执行/etc/profile脚本

### 2）/etc/profile.d/*.sh
> /etc/profile脚本先依次执行/etc/profile.d/*.sh

```
for i in /etc/profile.d/*.sh ; do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . $i
        else
            . $i >/dev/null 2>&1
        fi
    fi
done

```

### 3）~/.bash_profile
> 随后bash会执行~/.bash_profile脚本

```
[root@localhost init.d]# cat ~/.bash_profile 
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH
unset USERNAME

```

### 4）~/.bashrc 
> ～/.bash_profile脚本会执行～/.bashrc脚本

```
[root@localhost init.d]# cat ~/.bash_profile 
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin

export PATH
unset USERNAME
[root@localhost init.d]# cat ~/.bashrc 
# .bashrc

# User specific aliases and functions

alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi

```

### 5）/etc/bashrc

> ～/.bashrc脚本会执行/etc/bashrc脚本

```
[root@localhost init.d]# cat /etc/bashrc 
# /etc/bashrc

# System wide functions and aliases
# Environment stuff goes in /etc/profile

# are we an interactive shell?
if [ "$PS1" ]; then
  if [ -z "$PROMPT_COMMAND" ]; then
    case $TERM in
	xterm*)
		if [ -e /etc/sysconfig/bash-prompt-xterm ]; then
			PROMPT_COMMAND=/etc/sysconfig/bash-prompt-xterm
		else
            PROMPT_COMMAND='printf "\033]0;%s@%s:%s\007" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}"'
		fi
		;;
	screen)
		if [ -e /etc/sysconfig/bash-prompt-screen ]; then
			PROMPT_COMMAND=/etc/sysconfig/bash-prompt-screen
		else
            PROMPT_COMMAND='printf "\033]0;%s@%s:%s\033\\" "${USER}" "${HOSTNAME%%.*}" "${PWD/#$HOME/~}"'
		fi
		;;
	*)
		[ -e /etc/sysconfig/bash-prompt-default ] && PROMPT_COMMAND=/etc/sysconfig/bash-prompt-default
	    ;;
    esac
  fi
  # Turn on checkwinsize
  shopt -s checkwinsize
  [ "$PS1" = "\\s-\\v\\\$ " ] && PS1="[\u@\h \W]\\$ "
fi

if ! shopt -q login_shell ; then # We're not a login shell
	# Need to redefine pathmunge, it get's undefined at the end of /etc/profile
    pathmunge () {
		if ! echo $PATH | /bin/egrep -q "(^|:)$1($|:)" ; then
			if [ "$2" = "after" ] ; then
				PATH=$PATH:$1
			else
				PATH=$1:$PATH
			fi
		fi
	}

    # By default, we want umask to get set. This sets it for non-login shell.
    # You could check uidgid reservation validity in
    # /usr/share/doc/setup-*/uidgid file
    if [ $UID -gt 99 ] && [ "`id -gn`" = "`id -un`" ]; then
       umask 002
    else
       umask 022
    fi

	# Only display echos from profile.d scripts if we are no login shell
    # and interactive - otherwise just process them to set envvars
    for i in /etc/profile.d/*.sh; do
        if [ -r "$i" ]; then
            if [ "$PS1" ]; then
                . $i
            else
                . $i >/dev/null 2>&1
            fi
        fi
    done

	unset i
	unset pathmunge
fi
# vim:ts=4:sw=4

```

实际上bash只执行了/etc/profile脚本，其他的脚本都是一个包含一个进行模块调用执行

/etc/profile是全局用户登录后的初始化环境变量的脚本（所有用户，需要重启生效）

/etc/bashrc是每一个用户bash的全局初始化脚本

在/sbin/mingetty终端启动成功后，/bin/login进行验证登录，登录成功后使用bash或sh或其他的shell取决于/etc/passwd中的配置

~/.bash_profile（需重启生效）和~/.bashrc是当前用户终端会话的环境变量（当前用户）

>~/.bash_profile 是交互式、login 方式进入bash 运行的；

>~/.bashrc 是交互式 non-login 方式进入bash 运行的；

>通常二者设置大致相同，所以通常前者会调用后者。


***
