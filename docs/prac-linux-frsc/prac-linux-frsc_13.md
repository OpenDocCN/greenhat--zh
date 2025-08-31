## 第十三章：文件/目录列表供数字调查员使用

![Image](img/common01.jpg)

本附录包含在流行 Linux 系统中常见的文件和目录列表，以及数字取证调查员的描述。

在大多数 Linux 系统中找到的文件和目录在两个 man 页面中有描述：hier(7) 和 file-hierarchy(7)。根据 Linux 发行版、当地自定义配置和已安装的软件包，本文档中列出的一些文件可能在你分析的取证镜像中不存在。如果你知道其他从调查或取证角度有趣的文件，请通过电子邮件联系我 *nikkel@digitalforensics.ch*，我会考虑将它们添加到本文档中。

本文档的最新版本已发布在我的网站 *[`digitalforensics.ch/linux/`](https://digitalforensics.ch/linux/)* 上。

### /

***/*** 系统的顶级或 *根* 目录；所有附加的文件系统或伪文件系统都挂载在此树中的子目录下。

***./*** 每个目录都有一个点子目录，指向它本身。

***../*** 每个目录都有一个双点子目录，指向其父目录。

***/bin/*** 包含可执行文件；通常符号链接到 */usr/bin/*。

***/boot/*** 包含引导加载器文件（如 grub 等）以及可能的 EFI 挂载点的目录。

***/cdrom/*** 传统的通用挂载点，用于临时挂载可移动介质，如 CD 或 DVD 光盘；在取证镜像中可能为空。

***/desktopfs-pkgs.txt*****,** ***/rootfs-pkgs.txt*** Manjaro 初始软件包安装列表。

***/dev/*** 设备文件的位置，通常由 udev 守护进程动态创建（和删除）；在取证镜像中可能为空。

***/etc/*** 存储系统配置数据的目录；有助于重建系统的配置方式。

***/home/*** 系统中普通用户的主目录；包含最多的用户活动证据。

***/initrd.img*** 指向初始 RAM 磁盘映像的符号链接（通常来自 */boot/*）；如果 initrd 已更新，也可能有 *initrd.img.old*。

***/lib32/*** 包含 32 位兼容库和可执行文件；可能符号链接到 */usr/lib32/*。

***/lib64/*** 包含 64 位兼容库；可能符号链接到 */usr/lib64/*。

***/lib/*** 包含库和可执行文件；通常符号链接到 */usr/lib/*。

***/libx32/*** 包含适用于 x32 ABI（64 位指令，32 位指针）的兼容库和可执行文件；可能符号链接到 */usr/libx32/*。

***/lost+found/*** 存放在文件系统修复过程中发现的孤立文件（没有父目录的文件）的目录。它可能存在于任何已挂载文件系统的根目录下。

***/media/*** 用于动态创建可移动介质（如 USB 存储棒、SD 卡、CD/DVD 光盘等）挂载点的目录；在取证镜像中可能为空。

***/mnt/*** 传统的通用挂载点，用于临时挂载的文件系统；在取证镜像中可能为空。

***/opt/*** 包含“可选”或附加软件的目录。

***/proc/*** 用于获取有关运行进程的信息的伪文件系统挂载点；在法医镜像中可能为空。

***/root/*** 根用户的主目录（故意位于*/home/*之外）。

***/run/*** 用于运行时数据的 tmpfs 文件系统挂载点；可能会符号链接到*/var/run/*；在法医镜像中可能为空。

***/sbin/*** 包含可执行文件；通常是符号链接到*/usr/sbin/*或*/usr/bin*（如果*bin*和*sbin*已合并）。

***/snap/*** 用于 Snap 软件包符号链接和挂载点的目录；可能会符号链接到*/var/lib/snapd/snap*。

***/srv/*** 用于存储提供的内容（HTTP、FTP、TFTP 等）的目录。

***/swapfile*** 基于文件的交换分区替代方案；可能包含上次系统运行时的内存碎片或休眠内存镜像。

***/sys/*** 用于运行内核的伪文件系统接口的挂载点；在法医镜像中可能为空。

***/tmp/*** 用于临时文件的 tmpfs 文件系统挂载点（重启时丢失）；在法医镜像中可能为空。

***/usr/*** 旨在成为多个系统共享的只读文件目录；如今主要包含来自已安装包的静态文件。

***/var/*** 用于存储可变系统和应用程序数据的目录；通常在重启时保持持久，并包含存储在日志文件中的证据。

***/vmlinuz*** 内核镜像的符号链接（通常位于*/boot/*目录）；如果内核被更新，也可能有*vmlinuz.old*。

### /boot/

***/boot/amd-ucode.img*** AMD CPU 微码更新（包含文件的存档）。

***/boot/cmdline.txt*** 树莓派上的内核参数。

***/boot/config-**** 内核配置。

***/boot/initramfs.**** 初始 RAM 磁盘（包含文件的存档）。

***/boot/initrd.**** 初始 RAM 磁盘（包含文件的存档）。

***/boot/intel-ucode.img*** Intel CPU 微码更新（包含文件的存档）。

***/boot/System.map-**** 内核符号表。

***/boot/vmlinuz-**** Linux 内核镜像文件。

#### */boot/grub/*

***/boot/grub/custom.cfg*** 额外的 GRUB 自定义配置。

***/boot/grub/grub.cfg*** GRUB 配置文件（也可以位于*EFI/*目录中）。

***/boot/grub/grubenv*** GRUB 环境块，1024 字节，固定大小。

***/boot/grub/i386-pc/*** 32 位 GRUB 模块。

***/boot/grub/, /boot/grub2/*** 用于引导加载程序文件的 GRUB 目录。

***/boot/grub/x86_64-efi/*** 64 位 GRUB 模块。

#### */boot/loader/*

***/boot/loader/*** Systemd 的引导加载程序（`systemd-boot`，以前是`gummiboot`）。

***/boot/loader/loader.conf*** 整体`systemd-boot`配置。

***/boot/loader/entries/*.conf*** 引导条目配置文件。

#### *EFI/*

***EFI/*** EFI 系统分区（ESP），FAT 文件系统；通常挂载在*/boot/efi/*或*/efi/*。

***EFI/BOOT/BOOT64.EFI, EFI/BOOT/BOOTX64.EFI*** 常见的默认 64 位 EFI 引导加载程序。

***EFI/BOOT/BOOTIA32.EFI*** 常见的默认 32 位 EFI 引导加载程序。

***EFI/fedora/, EFI/ubuntu/, EFI/debian/*** 发行版特定的 EFI 目录示例。

***EFI/*/grubx64.efi*** GRUB 的 EFI 引导加载程序。

***EFI/*/shim.efi, EFI/*/shimx64.efi, EFI/*/shimx64-fedora.efi*** 用于安全启动的签名二进制文件。

### /etc/

***/etc/.updated*** Systemd 可能在更新时创建此文件，包含时间戳。

***/etc/lsb-release, /etc/machine-info, /etc/release, /etc/version*** 已安装的 Linux 发行版信息。

***/etc/*.release, /etc/*-release, /etc/*_version*** 已安装的 Linux 发行版信息。

***/etc/abrt/*** 自动化的错误报告工具配置。

***/etc/acpi/*** ACPI 事件和处理脚本。

***/etc/adduser.conf*** `adduser` 和 `addgroup` 命令的配置文件。

***/etc/adjtime*** 硬件时钟及漂移信息。

***/etc/aliases, /etc/aliases.d/*** 邮件地址别名文件。

***/etc/alternatives*** 替代命令的配置。

***/etc/anaconda/*** Fedora 安装程序配置。

***/etc/apache2/*** Apache web 服务器配置。

***/etc/apparmor/, /etc/apparmor.d/*** AppArmor 配置和配置文件。

***/etc/apport/*** Ubuntu 崩溃报告器配置。

***/etc/appstream.conf*** AppStream 通用包管理器配置。

***/etc/apt/*** Debian APT 配置。

***/etc/audit/audit.rules, /etc/audit/rules.d/*.rules*** Linux 审计系统规则。

***/etc/authselect/*** Fedora `authselect` 配置。

***/etc/autofs/, /etc/autofs.**** 配置按需自动挂载文件系统。

***/etc/avahi/*** Avahi（零配置）守护进程配置。

***/etc/bash.bash_logout*** Bash shell 全局注销脚本。

***/etc/bashrc, /etc/bash.bashrc*** Bash shell 全局登录脚本。

***/etc/binfmt.d/*.conf*** 配置启动时可执行文件的附加二进制格式。

***/etc/bluetooth/*.conf*** 蓝牙配置文件。

***/etc/ca-certificates/, /etc/ca-certificates.conf*** 系统范围的证书颁发机构（信任和阻止的）。

***/etc/casper.conf*** `initramfs-tools` 引导 live 系统的配置文件。

***/etc/chrony**** Chrony 时间同步守护进程的配置。

***/etc/conf.d/*** Arch Linux 配置文件。

***/etc/cron**** Cron 调度配置。

***/etc/crontab, /etc/anacrontab, /etc/cron.**** 定时的 cron 作业。

***/etc/crypttab*** 指定如何挂载加密文件系统。

***/etc/ctdb/*** Manjaro 的崩溃处理程序配置。

***/etc/cups/*** CUPS 打印机配置文件。

***/etc/dbus-1/*** D-Bus 配置（系统和会话）。

***/etc/dconf/*** dconf 配置数据库。

***/etc/debconf.conf*** Debian 配置系统。

***/etc/default/*** 各种守护进程和子系统的默认配置文件。

***/etc/defaultdomain*** 默认 NIS 域名。

***/etc/deluser.conf*** `deluser` 和 `delgroup` 命令的配置文件。

***/etc/dhclient*.conf, /etc/dhcp**** DHCP 配置。

***/etc/dnf/*** Fedora DNF 包管理配置。

***/etc/dnsmasq.conf*****,** ***/etc/dnsmasq.d/*** DNSMasq、DNS 和 DHCP 服务器的设置。

***/etc/dpkg/*** Debian 配置设置。

***/etc/dracut.conf, /etc/dracut.conf.d/*** 用于创建 initramfs 镜像的 Dracut 配置。

***/etc/environment, /etc/environment.d/*** 设置 systemd 用户实例的环境变量。

***/etc/ethertypes*** 以太网帧类型。

***/etc/exports*** NFS 文件系统导出。

***/etc/fake-hwclock.data*** 包含没有时钟的系统（如 Raspberry Pi）的最近时间戳。

***/etc/firewalld/*** firewalld 守护进程的配置文件。

***/etc/flatpak/*** Flatpak 配置和仓库。

***/etc/fscrypt.conf*** 引导时挂载的加密文件系统。

***/etc/fstab*** 引导时挂载的文件系统。

***/etc/ftpusers*** 禁止的 FTP 用户列表。

***/etc/fuse3.conf, /etc/fuse.conf*** 配置用户空间文件系统。

***/etc/fwupd/*.conf*** 配置固件更新守护进程。

***/etc/gconf/*** GNOME 2 配置数据库。

***/etc/gdm/, /etc/gdm3/*** GNOME 显示管理器 (GDM) 配置。

***/etc/geoclue/geoclue.conf*** GeoClue 地理位置服务的配置。

***/etc/gnupg/gpgconf.conf*** GnuPG/GPG 的默认配置。

***/etc/group, /etc/group-*** 群组信息文件。

***/etc/gshadow*** 群组影子文件（包含哈希密码）。

***/etc/hostapd/*** 用作 Wi-Fi 接入点的 Linux 配置。

***/etc/hostid*** 系统的唯一标识符。

***/etc/hostname*** 系统定义的主机名（此主机名不唯一）。

***/etc/hosts*** 主机列表及其匹配的 IP 地址。

***/etc/hosts.allow, /etc/hosts.deny*** TCP 包装器访问控制文件。

***/etc/init.d/*** 传统的 System V 初始化脚本。

***/etc/init/*, /etc/rc*.d/*** 传统初始化系统。

***/etc/initcpio/, /etc/mkinitcpio.conf, /etc/mkinitcpio.d/, /etc/initramfs-tools/**** 用于 initramfs 创建的配置和文件。

***/etc/inittab*** 传统的 System V 初始化和运行级别配置。

***/etc/issue, /etc/issue.d/, /etc/issue.net*** 网络登录时显示的横幅。

***/etc/iwd/*** iNet 无线守护进程配置。

***/etc/linuxmint/info, /etc/mintSystem.conf*** 特定于 Linux Mint 的信息。

***/etc/locale.conf*** 包含定义语言环境设置的变量。

***/etc/locale.gen*** 包含要包含的语言环境列表。

***/etc/localtime*** 指向 */usr/share/zoneinfo/** 中时区文件的符号链接。

***/etc/login.defs*** 登录程序的系统范围配置。

***/etc/logrotate.conf, /etc/logrotate.d/*** 日志轮换配置。

***/etc/lvm/**** Linux 卷管理器配置和配置文件。

***/etc/machine-id*** 系统的唯一标识符。

***/etc/magic, /etc/magic.mime, /etc/mime.types, /etc/mailcap*** 用于识别和关联内容与程序的文件。

***/etc/mail.rc*** BSD mail 或 mailx 程序运行的命令。

***/etc/mdadm.conf, /etc/mdadm.conf.d/*** Linux 软件 RAID 配置。

***/etc/modprobe.d/, /modules, /etc/modules-load.d/*** 启动时加载的内核模块。

***/etc/motd*** 传统 Unix 日常信息，在登录时显示。

***/etc/netconfig*** 网络协议定义。

***/etc/netctl/*** `netctl` 网络管理器配置文件。

***/etc/netgroup*** NIS 网络组文件。

***/etc/netplan/*** Ubuntu netplan 网络配置文件。

***/etc/network/*** Debian 网络配置目录。

***/etc/NetworkManager/system-connections/*** 网络连接，包括 Wi-Fi 和 VPN。

***/etc/networks*** 将名称与 IP 网络关联。

***/etc/nftables.conf*** 用于指定 nftables 规则的常见文件。

***/etc/nscd.conf*** 名称服务缓存守护进程配置文件。

***/etc/nsswitch.conf*** 名称服务切换配置文件。

***/etc/ntp.conf*** 网络时间协议（NTP）配置文件。

***/etc/openvpn/*** OpenVPN 客户端和服务器配置。

***/etc/ostree/*, /etc/ostree-mkinitcpio.conf*** OSTree 版本化文件系统树配置。

***/etc/PackageKit/**** PackageKit 配置文件。

***/etc/pacman.conf, /etc/pacman.d/*** Arch Linux Pacman 包管理器配置。

***/etc/pam.conf, /etc/pam.d/*** 可插拔认证模块（PAM）。

***/etc/pamac.conf*** Arch Linux 图形包管理器配置。

***/etc/papersize, /etc/paperspecs*** 默认纸张大小和规格。

***/etc/passwd, /etc/passwd-, /etc/passwd.YaST2save*** 包含用户帐户信息的文件。

***/etc/polkit-1/*** 策略工具规则和配置。

***/etc/products.d/*** SUSE Zypper 产品信息。

***/etc/profile, /etc/profile.d/*** 登录 Shell 的启动文件。

***/etc/protocols*** 协议号列表。

***/etc/resolv.conf, /etc/resolvconf.conf*** DNS 解析器配置文件。

***/etc/rpm/*** Red Hat 包管理器（RPM）配置。

***/etc/rsyslog.conf, /etc/rsyslog.d/*.conf*** rsyslog 守护进程配置。

***/etc/sane.d/*.conf*** SANE 扫描器配置文件。

***/etc/securetty*** 允许 root 登录的终端。

***/etc/security/*** 存储安全配置的目录。

***/etc/services*** TCP 和 UDP 端口号及其关联名称的列表。

***/etc/shadow, /etc/shadow-, /etc/shadow.YaST2save*** 隐藏密码文件（包含加密密码）。

***/etc/shells*** 有效登录 Shell 的列表。

***/etc/skel/*** 新用户的默认文件（包括“.”文件）。

***/etc/ssh/*** 安全外壳（SSH）服务器和默认客户端配置。

***/etc/ssl/*** SSL/TLS 配置和密钥。

***/etc/sssd/*** 系统安全服务守护进程（sssd）配置。

***/etc/sudoers, /etc/sudoers.d/, /etc/sudo.conf*** `sudo` 配置文件。

***/etc/swid/*** 软件标识标签。

***/etc/sysconfig/*** 系统配置文件；通常用于 Red Hat 或 SUSE。

***/etc/sysctl.conf, /etc/sysctl.d/*** `sysctl` 在启动时或通过命令读取的值。

***/etc/syslog-ng.conf, /etc/syslog.conf*** syslog-ng 和传统 syslog 配置文件。

***/etc/systemd/*.conf*** systemd 守护进程的配置文件。

***/etc/systemd/network/*** systemd 链接、netdev 和网络（ini 风格）配置文件。

***/etc/systemd/system/, /usr/lib/systemd/system/*** 系统实例的 systemd 单元文件。

***/etc/systemd/user/, /usr/lib/systemd/user/,*** ***~******/.config/systemd/user/*** 用户实例的 systemd 单元文件。

***/etc/tcsd.conf*** TrouSerS 可信计算守护进程配置文件（TPM 模块）。

***/etc/tlp.conf, /etc/tlp.d/*** 笔记本电源工具的配置。

***/etc/trusted-key.key*** DNSSEC 信任锚密钥。

***/etc/ts.conf*** 触摸屏库的配置。

***/etc/udev/*** `systemd-udev` 规则和配置。

***/etc/udisks2/modules.conf.d/, /etc/udisks2.conf*** udisks 磁盘管理配置。

***/etc/ufw/*** 简单防火墙规则和配置。

***/etc/update-manager/*** `update-manager` 图形工具的配置。

***/etc/updatedb.conf*** `mlocate` 数据库的配置文件。

***/etc/vconsole.conf*** 虚拟控制台的配置文件。

***/etc/wgetrc*** `wget` 工具的下载文件配置。

***/etc/wicked/*** SUSE Wicked 网络管理器的配置文件。

***/etc/wireguard/*** WireGuard VPN 的配置文件。

***/etc/wpa_supplicant.conf*** WPA supplicant 守护进程配置文件。

***/etc/X11/*** Xorg 配置（*xinitrc*、*xserverrc*、*Xsession* 等）。

***/etc/xattr.conf*** 由 *attr* 拥有，用于 XFS 扩展属性。

***/etc/xdg/*** XDG 系统范围的桌面配置文件（包括 *autostart* 和 *user-dirs.defaults*）。

***/etc/YaST2/**** SUSE YaST 系统范围配置。

***/etc/yum.repos.d/*** Fedora YUM 仓库配置数据。

***/etc/zsh/, /etc/zshrc, /etc/zprofile, /etc/zlogin, /etc/zlogout*** Z shell 登录和注销文件。

***/etc/zypp/*** SUSE Zypper 包管理配置。

### /home/*/

本部分的文件指向已配置的用户（通常是人）。其中一些文件也可能存在于 */root/*，即 root 用户的主目录中。

#### *XDG 和 freedesktop 目录*

***.cache/*** 非必需的持久化用户缓存数据 (*$XDG_CACHE_HOME*)。

***.config/*** 持久化用户配置数据 (*$XDG_CONFIG_HOME*)。

***.local/share/*** 持久化的用户应用数据 (*$XDG_DATA_HOME*)。

***Documents/*** 办公文档。

***Downloads/*** 下载内容的默认位置。

***Desktop/*** 常规文件和 **.desktop* 定义文件，出现在桌面上。

***Music/*** 音乐和音频文件。

***Pictures/*** 照片和图片。

***Templates/*** 应用模板（办公文档等）。

***Videos/*** 视频文件。

#### *.cache/*

***.cache/clipboard-indicator@tudmotu.com/registry.txt*** GNOME 剪贴板历史。

***.cache/flatpak/*** 用户缓存的 Flatpak 数据。

***.cache/gnome-software/shell-extensions/*** 用户安装的 GNOME 扩展。

***.cache/libvirt/qemu/log/linux.log*** QEMU 虚拟机活动日志。

***.cache/sessions/*** 桌面会话状态数据。

***.cache/simple-scan/simple-scan.log*** 扫描应用程序日志（可能包含保存扫描文件的文件名）。

***.cache/thumbnails/, .cache/thumbs-*/*** 缓存的缩略图图像。

***.cache/tracker/, .cache/tracker3/*** GNOME 搜索索引文件。

***.cache/xfce4/clipman/textsrc*** Xfce 剪贴板历史记录。

***.cache/*/*** 任何可能缓存持久数据以提高性能或效率的应用程序。

#### *.config/* 

***.config/autostart/*** 自动启动的 **.desktop** 程序和插件。

***.config/baloofilerc*** Baloo 桌面搜索配置。

***.config/dconf/user*** dconf 用户配置数据库。

***.config/goa-1.0/accounts.conf*** GNOME 在线账户配置。

***.config/g*rc*** 以 *g* 开头、以 *rc* 结尾的 GNOME 重写配置文件。

***.config/Jitsi Meet/*** Jitsi 视频通话的缓存、状态、偏好设置、日志等。

***.config/kdeglobals*** KDE 全局重写设置。

***.config/k*rc, .config/plasma*rc*** KDE/Plasma 重写配置文件，以 *k* 开头、以 *rc* 结尾。

***.config/libaccounts-glib/accounts.db*** KDE 配置的云账户数据。

***.config/mimeapps.list*** 用户文件类型的默认应用程序。

***.config/Qlipper/qlipper.ini*** 剪贴板数据（Lubuntu）。

***.config/session/, gnome-session/*** 保存的桌面和应用程序状态。

***.config/systemd/user/*** 用户的 systemd 单元文件。

***.config/user-dirs.dirs*** 用户定义的默认 freedesktop 目录。

***.config/xsettingsd/xsettingsd.conf*** X11 设置配置。

***.config/*/*** 任何可能保存用户配置数据的应用程序。

#### *.local/* 

***.local/lib/python/site-packages*** 用户安装的 Python 模块。

***.local/share/akonadi/*** KDE/Plasma Akonadi 个人信息管理器搜索数据库。

***.local/share/baloo/*** KDE/Plasma Baloo 文件搜索数据库。

***.local/share/dbus-1/*** 用户配置的 D-Bus 会话服务。

***.local/share/flatpak/*** 用户安装的 Flatpak 软件包。

***.local/share/gvfs-metadata/*** GNOME 虚拟文件系统的元数据。

***.local/share/kactivitymanagerd/*** KDE KActivities 管理器。

***.local/share/keyrings/*** GNOME 密钥环文件。

***.local/share/klipper/history2.lst*** KDE 剪贴板历史记录。

***.local/share/kwalletd/*** KDE 钱包文件。

***.local/share/modem-manager-gui/*** 移动网络（短信）应用程序。

***.local/share/RecentDocuments/*** **.desktop** 文件，包含最近文档信息。

***.local/share/recently-used.xbel*** GTK 应用程序的最近使用文件。

***.local/share/Trash/*** 来自 *[freedesktop.org](http://freedesktop.org)* 规范的垃圾桶目录。

***.local/share/xorg/Xorg.0.log*** Xorg 启动日志。

***.local/user-places.xbel*** GTK 应用程序的最近访问位置。

***.local/cache/*/*** 其他任何可能保存数据的应用程序。  

#### *其他点文件和目录*  

***.bash_history*** Bash shell 历史记录文件。  

***.bash_logout*** Bash shell 注销脚本。  

***.bash_profile, .profile, .bashrc*** Bash shell 登录脚本。  

***.ecryptfs/*** 加密 Ecryptfs 树的常见默认目录。  

***.gnome2/keyrings/*** 旧版 GNOME 2 密钥环。  

***.gnupg/*** GnuPG/GPG 目录，包含配置和密钥。  

***.john/*** John the Ripper 密码破解程序。  

***.mozilla/*** Firefox 浏览器目录；包括配置文件、配置等。  

***.ssh/*** SSH 目录，包含配置、密钥和已知主机。  

***.thumbnails/*** 旧版缩略图图像目录。  

***.thunderbird/*** Thunderbird 邮件客户端目录；包括配置文件、配置、缓存的邮件等。  

***.Xauthority*** X11 MIT Magic Cookie 文件。  

***.xinitrc*** 用户自定义的 X11 会话启动脚本。  

***.xsession-errors, .xsession-errors.old*** X11 当前和上一会话的错误日志。  

### /usr/  

***/usr/bin/, /usr/sbin/*** 包含可执行文件；如果*bin*和*sbin*已合并，则为符号链接。  

***/usr/games/*** 游戏程序目录。  

***/usr/include/*** 系统 C 头文件（**.h*）。  

***/usr/lib/, /usr/lib64/, /usr/lib32/, /usr/libx32/*** 包含库和可执行文件；架构相关的库存放在不同的目录中。  

***/usr/local/, /usr/local/opt/*** 可选附加软件包的目录。  

***/usr/opt/*** 附加软件包的替代位置。  

***/usr/src/*** 系统源代码。  

#### */usr/lib/*  

***/usr/lib/*** 系统范围使用的静态和动态库及支持文件。  

***/usr/libexec/*** 守护进程和系统组件的可执行文件（非管理员使用）。  

***/usr/lib/locale/locale-archive*** 使用配置的区域设置构建的二进制文件。  

***/usr/lib/modules/, /usr/lib/modprobe.d/, /usr/lib/modules-load.d/*** 内核模块和配置文件。  

***/usr/lib/os-release*** 包含已安装发行版信息的文件。  

***/usr/lib/python*/*** 系统范围内的 Python 模块和支持文件。  

***/usr/lib/sysctl.d/*** 默认的`sysctl`配置文件。  

***/usr/lib/udev/*** udev 支持文件和规则（*rules.d/*）。  

***/usr/lib/tmpfiles.d/*** 临时文件和目录的配置。

#### */usr/lib/systemd/*  

***/lib/systemd/system/*** 默认的系统单元文件。  

***/lib/systemd/user/*** 默认的用户单元文件。  

***/usr/lib/systemd/*generators*/*** 用于创建单元文件的生成程序。  

***/usr/lib/systemd/network/*** 默认的网络、链接和网卡设备文件。  

***/usr/lib/systemd/systemd**** Systemd 可执行文件。  

#### */usr/local/, /usr/opt/*  

***/usr/local/*** 目录，传统 Unix 系统中用于本地安装二进制文件的位置，而非网络挂载的目录。Linux 系统可能会将其用于附加软件包。  

***/usr/local/bin/, /usr/local/sbin/*** 本地二进制文件。  

***/usr/local/etc/*** 本地配置。  

***/usr/local/doc/, /usr/local/man/*** 本地文档和手册页。  

***/usr/local/games/*** 本地游戏。

***/usr/local/lib/, /usr/local/lib64/, /usr/local/libexec/*** 相关的本地文件。

***/usr/local/include/, /usr/local/src/*** 头文件和源代码。

***/usr/local/share/*** 与架构无关的文件。

#### */usr/share/*

***/usr/share/*** 不同软件包或架构之间共享的文件。

***/usr/share/dbus-1/*** 默认的系统和会话 D-Bus 配置数据。

***/usr/share/factory/etc/*** 一些*/etc/*文件的初始安装默认值。

***/usr/share/hwdata/pci.ids*** PCI 供应商、设备和子系统的列表。

***/usr/share/hwdata/usb.ids*** USB 供应商、设备和接口的列表。

***/usr/share/hwdata/pnp.ids*** 产品供应商名称缩写的列表。

***/usr/share/i18n/, /usr/share/locale/*** 国际化数据。

***/usr/share/metainfo/*** 带有 AppStream 元数据的 XML 文件。

***/usr/share/polkit-1/*** PolicyKit 规则和操作。

***/usr/share/zoneinfo/*** 不同区域的时区数据文件。

***/usr/share/accounts/*** KDE 在线账户的服务和提供者文件。

***/usr/share/doc/*** 软件包提供的文档。

***/usr/share/help/*** GNOME 帮助文件及其翻译。

***/usr/share/man/*** 带翻译的手册页。

***/usr/share/src/, /usr/share/include/*** 源代码；C 语言头文件（**.h*）文件。

### /var/

***/var/backups/*** Debian 软件包、替代项以及 passwd/group 文件的备份数据。

***/var/games/*** 安装游戏的可变数据；可能包含带有名称和日期的高分文件。

***/var/local/*** 安装在*/usr/local/*中的软件的可变数据。

***/var/opt/*** 安装在*/usr/opt/*中的软件的可变数据。

***/var/run/*** 运行时数据；通常在取证镜像中为空。

***/var/tmp/*** 临时文件；在重启后仍然存在。

***/var/crash/*** 崩溃转储、堆栈跟踪和报告。

***/var/mail/*** 本地缓存的邮件（一些发行版如 Ubuntu 和 Fedora 不再默认设置邮件子系统）。

***/var/www/*** 存储 HTML 页面的位置。

***/var/db/sudo/lectured/*** 空文件，表示用户第一次使用`sudo`时已接受相关提示。

#### */var/cache/*

***/var/cache/*** 持久化的系统范围缓存数据。

***/var/cache/apt/*** Debian 包的缓存下载。

***/var/cache/cups/*** CUPS 打印系统。

***/var/cache/cups/job.cache*** 打印作业缓存，包含文件名、时间戳和打印机名称。

***/var/cache/cups/job.cache.**** *job.cache* 的轮换版本。

***/var/cache/debconf/*** 系统范围的 Debian 缓存数据。

***/var/cache/debconf/passwords.dat*** 包含系统生成的密码。

***/var/cache/dnf/*** 系统范围的 Fedora DNF 包缓存数据。

***/var/cache/PackageKit/*** 与发行版无关的系统范围 PackageKit 包缓存数据。

***/var/cache/pacman/*** 系统范围的 Arch Linux Pacman 包缓存数据。

***/var/cache/snapd/*** 系统范围的 Ubuntu Snap 包缓存数据。

***/var/cache/zypp/*** 系统范围内的缓存 SUSE Zypper 包数据。

#### */var/log/*

***/var/log/alternatives.log*** Debian 替代命令名称系统日志。

***/var/log/anaconda/*** Fedora Anaconda 初始安装程序日志。

***/var/log/apache2/*** 默认的 Apache 网络服务器日志。

***/var/log/apport.log*** Ubuntu 崩溃处理系统日志。

***/var/log/apt/*** Debian Apt 包管理器日志。

***/var/log/aptitude*** Debian Aptitude 执行动作日志。

***/var/log/archinstall/install.log*** Arch Linux 初始安装日志。

***/var/log/audit/*** Linux 审计系统日志。

***/var/log/boot.log*** Plymouth 启动画面控制台输出。

***/var/log/btmp*** 登录失败（错误）尝试的日志。

***/var/log/Calamares.log*** Calamares 初始安装日志。

***/var/log/cups/*** CUPS 打印系统的访问、错误和页面日志。

***/var/log/daemon.log*** 与守护进程相关的常见 syslog 文件。

***/var/log/*** 系统范围日志文件的默认位置。

***/var/log/dmesg*** 内核环形缓冲区日志。

***/var/log/dnf.log*** Fedora DNF 包管理器日志。

***/var/log/dpkg.log*** Debian `dpkg` 包管理器日志。

***/var/log/firewalld*** firewalld 守护进程日志。

***/var/log/hawkey.log*** Fedora Anaconda 日志。

***/var/log/installer/*** Debian 初始安装程序日志。

***/var/log/journal/*** Systemd 日志文件（系统和用户）。

***/var/log/kern.log*** 与内核相关的常见 syslog 文件（环形缓冲区）。

***/var/log/lastlog*** 最后登录日志及来源信息。

***/var/log/lightdm/*** Lightdm 显示管理器日志。

***/var/log/mail.err*** 与邮件相关的常见 syslog 错误日志。

***/var/log/messages*** 传统的 Unix 日志文件，包含 syslog 消息。

***/var/log/mintsystem.log, mintsystem.timestamps*** Linux Mint 特定的日志。

***/var/log/openvpn/*** OpenVPN 系统日志。

***/var/log/pacman.log*** Arch Linux Pacman 包管理器日志。

***/var/log/sddm.log*** SDDM 显示管理器日志。

***/var/log/tallylog*** PAM 计数状态文件，记录失败的登录尝试。

***/var/log/ufw.log*** 简单防火墙（UFW）日志。

***/var/log/updateTestcase-*/*** SUSE 错误报告数据。

***/var/log/wtmp*** 传统的系统登录记录。

***/var/log/Xorg.0.log*** Xorg 启动日志。

***/var/log/YaST2*** SUSE YaST 日志。

***/var/log/zypper.log*** SUSE Zypper 包管理器日志。

***/var/log/zypp/history*** SUSE Zypper 包管理器历史记录。

***/var/log/**** 其他由应用程序或系统组件创建的日志。

#### */var/lib/*

***/var/lib/*** 已安装软件的持久性变量数据。

***/var/lib/abrt/*** 自动化错误报告工具数据。

***/var/lib/AccountsService/icons/**** 用户选择的登录图标。

***/var/lib/AccountsService/users/**** 用户的默认或最后会话登录设置。

***/var/lib/alternatives/*** 替代命令名称的符号链接。

***/var/lib/bluetooth/*** 蓝牙适配器和配对的蓝牙设备。

***/var/lib/ca-certificates/*** 系统范围内的 CA 证书存储库。

***/var/lib/dnf/*** Fedora DNF 安装包信息。

***/var/lib/dpkg/, /var/lib/apt/*** Debian 安装的包信息。

***/var/lib/flatpak/*** Flatpak 安装的包信息。

***/var/lib/fprint/*** 指纹识别器数据，包括已注册的用户指纹。

***/var/lib/gdm3/*** GNOME 3 显示管理器设置和数据。

***/var/lib/iwd/*** iNet 无线守护进程，包括接入点信息、密码。

***/var/lib/lightdm/*** Lightdm 显示管理器设置和数据。

***/var/lib/linuxmint/mintsystem/*** Linux Mint 系统范围的设置。

***/var/lib/mlocate/mlocate.db*** `locate` 搜索命令的文件数据库。

***/var/lib/NetworkManager/*** 网络管理器数据，包括租约、bssids 等。

***/var/lib/PackageKit/*** PackageKit *transactions.db*。

***/var/lib/pacman/*** Arch Linux Pacman 数据。

***/var/lib/polkit-1/*** PolicyKit 数据。

***/var/lib/rpm/*** RPM SQLite 包数据库。

***/var/lib/sddm/*** SDDM 显示管理器数据。

***/var/lib/selinux/*** SELinux 模块、锁和数据。

***/var/lib/snapd/*** Ubuntu 安装的 Snap 包信息。

***/var/lib/systemd/*** 系统范围的 systemd 数据。

***/var/lib/systemd/coredump/*** Systemd 核心转储数据。

***/var/lib/systemd/pstore/*** `pstore` 保存的崩溃转储数据。

***/var/lib/systemd/timers/*** Systemd 定时器单元文件。

***/var/lib/systemd/timesync/clock*** 空文件；`mtime` 可用于在没有硬件时钟的系统上设置大致时间。

***/var/lib/ucf*** 更新配置文件数据。

***/var/lib/upower/*** 电源历史文件（笔记本电脑的充电/放电情况）。

***/var/lib/whoopsie/whoopsie-id*** 用于发送到 Ubuntu/Canonical 服务器的崩溃数据的唯一标识符。

***/var/lib/wicked/*** Wicked 网络管理器数据。

***/var/lib/YaST2/*** SUSE YaST 配置数据。

***/var/lib/zypp/AnonymousUniqueId*** 用于联系 SUSE 服务器的唯一标识符。

***/var/lib/zypp/*** SUSE Zypper 包管理器数据。

#### */var/spool/* 

***/var/spool/*** 用于使用队列目录进行作业的守护进程的位置。

***/var/spool/abrt/, /var/tmp/abrt*** 发送到 Fedora 的崩溃报告数据。

***/var/spool/at/*** 计划的 `at` 作业。

***/var/spool/cron/, /var/spool/anacron/*** 计划的 `cron` 作业。

***/var/spool/cups/*** CUPS 打印队列目录。

***/var/spool/lpd/*** 传统的行打印机守护进程队列目录。

***/var/spool/mail/*** 参见 */var/mail/*。
