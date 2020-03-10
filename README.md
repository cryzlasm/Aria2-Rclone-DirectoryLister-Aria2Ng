# 利用 VPS 打造 无限空间在线播放离线网盘（Debian/Ubuntu） - 就是爱生活 - 发现生活的另一面

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [https://www.94ish.me/1676.html](https://www.94ish.me/1676.html)


[千影](#) • 2018 年 02 月 04 日

近两天一直都在折腾 VPS 挂载**谷歌云盘**打造**无限**容量的。虽然说网上各类教程非常多，但是要做到全套开机自启，自动上传，自动更新还是有些麻烦的。

本次教程全套组件为：**Lnmp+Aria2+Rclone+DirectoryLister+Aria2Ng**

本篇博文是利用`Vultr Ubuntu 16 X64`做的搭建实例，首先感谢`Rats`提供 VPS 作为示范。由于 Linux 系统的多样性，不保证按照此教程搭建一定成功。

<a name="88210852"></a>
## 准备工作

- 
一个域名：本篇教程使用的域名为`aria2down.tk`<br />
使用时请自行将`aria2down.tk`更换为**自己的域名**

- 
一个 VPS：要求 KVM 1 核 512M 内存以上，硬盘空间建议 50G 以上（由于是先下载到 VPS **再上传到谷歌云盘**里，所以硬盘限制了单次能下载文件的**最大**大小）。

- 
谷歌云盘账号：需要一个无限容量的谷歌云盘账号。如果没有请自寻购买（价格大概 15-20 元）。


需要的东西并不多，其中_域名_为可选项。

<a name="dbcc5dc1"></a>
## 搭建网站环境

由于这次教程中 Aria2Ng 和 DirectoryLister 需要 nginx 和 php 的支持，所以我们先来安装 Lnmp。这里推荐我一直使用的一键包 [Oneinstack](https://www.94ish.me/go/aHR0cHM6Ly9vbmVpbnN0YWNrLmNvbS8=)

```
apt-get update
apt-get -y install wget screen curl python zip unzip sudo vim
#由于shell的局限性，请先复制上方内容执行完再复制下方执行
wget http://mirrors.linuxeye.com/oneinstack.tar.gz
tar xzf oneinstack.tar.gz && rm -f oneinstack.tar.gz
cd oneinstack
screen -S oneinstack
./install.sh
```

**交互记录**

```
Please input SSH port(Default: 22): #回车

Do you want to enable iptables? [y/n]: n

Do you want to install Web server? [y/n]: y

Please select Nginx server:
        1. Install Nginx
        2. Install Tengine
        3. Install OpenResty
        4. Do not install
Please input a number:(Default 1 press Enter) #回车

Please select Apache server:
        1. Install Apache-2.4
        2. Install Apache-2.2
        3. Do not install
Please input a number:(Default 3 press Enter) #回车

Please select tomcat server:
        1. Install Tomcat-8
        2. Install Tomcat-7
        3. Install Tomcat-6
        4. Do not install
Please input a number:(Default 4 press Enter) #回车

Do you want to install Database? [y/n]: n

Do you want to install PHP? [y/n]: y

Please select a version of the PHP:
        1. Install php-5.3
        2. Install php-5.4
        3. Install php-5.5
        4. Install php-5.6
        5. Install php-7.0
        6. Install php-7.1
        7. Install php-7.2
Please input a number:(Default 5 press Enter) #回车

Do you want to install opcode cache of the PHP? [y/n]: y
Please select a opcode cache of the PHP:
        1. Install Zend OPcache
        3. Install APCU
Please input a number:(Default 1 press Enter) #回车

Do you want to install ionCube? [y/n]: n

Do you want to install ImageMagick or GraphicsMagick? [y/n]: n

Do you want to install Pure-FTPd? [y/n]: n

Do you want to install phpMyAdmin? [y/n]: n

Do you want to install redis? [y/n]: n

Do you want to install memcached? [y/n]: n

Do you want to install HHVM? [y/n]: n
```

然后脚本就开始自动安装 **nginx** 和 **php** 环境了

这段时间我们可以用来设置域名的绑定，本篇教程是将 **Aria2Ng** 的域名定为`aria2down.com`，**DirectoryLister** 设置为`www.aria2down.com`。

Oneinstack 安装速度很快，大概一顿饭的功夫就能安装完毕。

```
Please restart the server and see if the services start up fine.
Do you want to restart OS ? [y/n]: y
```

看到如下字样，输入`y`重启即可。

<a name="0111016e"></a>
## 添加站点程序

我们先不慌部署 aria2，先把网站程序上传上去。使用 oneinstack 创建站点。<br />
首先创建的是 Aria2Ng 的站点

```
cd /root/oneinstack
./vhost.sh
```

**交互记录**

```
#######################################################################
#       OneinStack for CentOS/RadHat 6+ Debian 7+ and Ubuntu 12+      #
#       For more information please visit https://oneinstack.com      #
#######################################################################

What Are You Doing?
    1. Use HTTP Only
    2. Use your own SSL Certificate and Key
    3. Use Let's Encrypt to Create SSL Certificate and Key
    q. Exit
Please input the correct option: 1

Please input domain(example: www.example.com): aria2down.tk          
domain=aria2down.tk

Please input the directory for the domain:aria2down.tk :#回车
(Default directory: /data/wwwroot/aria2down.tk): 
Virtual Host Directory=/data/wwwroot/aria2down.tk

Create Virtul Host directory......
set permissions of Virtual Host directory......

Do you want to add more domain name? [y/n]: n

Do you want to add hotlink protection? [y/n]: n

Allow Rewrite rule? [y/n]: n

Allow Nginx/Tengine/OpenResty access_log? [y/n]: n

nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
Reload Nginx......

#######################################################################
#       OneinStack for CentOS/RadHat 6+ Debian 7+ and Ubuntu 12+      #
#       For more information please visit https://oneinstack.com      #
#######################################################################
Your domain:                  aria2down.tk
Virtualhost conf:             /usr/local/nginx/conf/vhost/aria2down.tk.conf
Directory of:                 /data/wwwroot/aria2down.tk
```

大家将`aria2down.tk`更换成自己的域名即可，接下来是上传站点程序。

```
cd /data/wwwroot/aria2down.tk
wget "https://github.com/chiakge/Aria2-Rclone-DirectoryLister-Aria2Ng/raw/master/website/Aria2Ng.zip"
unzip Aria2Ng.zip && rm -f Aria2Ng.zip
```

现在我们就可以尝试打开自己域名看看能不能访问 Aria2Ng<br />
![](https://www.94ish.me/usr/uploads/2018/02/1773291397.png#alt=)

接下来部署 **DirectoryLister**

```
cd /root/oneinstack
./vhost.sh
```

**交互记录**

```
#######################################################################
#       OneinStack for CentOS/RadHat 6+ Debian 7+ and Ubuntu 12+      #
#       For more information please visit https://oneinstack.com      #
#######################################################################

What Are You Doing?
    1. Use HTTP Only
    2. Use your own SSL Certificate and Key
    3. Use Let's Encrypt to Create SSL Certificate and Key
    q. Exit
Please input the correct option: 1

Please input domain(example: www.example.com): www.aria2down.tk
domain=www.aria2down.tk

Please input the directory for the domain:www.aria2down.tk :
(Default directory: /data/wwwroot/www.aria2down.tk): 
Virtual Host Directory=/data/wwwroot/www.aria2down.tk

Create Virtul Host directory......
set permissions of Virtual Host directory......

Do you want to add more domain name? [y/n]: n

Do you want to add hotlink protection? [y/n]: n

Allow Rewrite rule? [y/n]: n

Allow Nginx/Tengine/OpenResty access_log? [y/n]: n

nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
Reload Nginx......

#######################################################################
#       OneinStack for CentOS/RadHat 6+ Debian 7+ and Ubuntu 12+      #
#       For more information please visit https://oneinstack.com      #
#######################################################################
Your domain:                  www.aria2down.tk
Virtualhost conf:             /usr/local/nginx/conf/vhost/www.aria2down.tk.conf
Directory of:                 /data/wwwroot/www.aria2down.tk
```

大家将`www.aria2down.tk`更换成自己的域名即可，接下来是上传站点程序。

```
cd /data/wwwroot/www.aria2down.tk
wget "https://github.com/chiakge/Aria2-Rclone-DirectoryLister-Aria2Ng/raw/master/website/DirectoryLister.zip"
unzip DirectoryLister.zip && rm -f DirectoryLister.zip
mkdir Download
mkdir Cloud
```

现在我们就可以尝试打开自己域名看看能不能访问 DirectoryLister<br />
![](https://www.94ish.me/usr/uploads/2018/02/3889155156.png#alt=)<br />
如果需要更改版权请去 resources/themes/bootstrap 更改。

教程到这里，**前端**已经部署完毕，接下来是**后端**的部署。

<a name="d6f9e575"></a>
## 编译安装 Aria2

在这里我推荐大家使用编译来安装`aria2`，虽然耗时比较久，但是不容易出错。

> 在编译之前，先要安装`GCC-4.9`。由于安装 BBR 魔改版自动把 GCC4.9 安装上了，**BBR 加速**也很有用。这里大家就通过 [Linux 网络优化加速一键脚本](https://www.94ish.me/1635.html)来安装吧。（先安装内核）


**编译安装**

```
wget https://github.com/aria2/aria2/releases/download/release-1.33.1/aria2-1.33.1.tar.gz
tar xzvf aria2-1.33.1.tar.gz
cd aria2-1.33.1
./configure
make
make install
```

编译过程有些慢，大概需要 **10-15** 分钟。大家可以喝口水等等。编译完成后我们可以输入`aria2c`测试是否可以运行。<br />
![](https://www.94ish.me/usr/uploads/2018/02/1180041521.png#alt=)

接下来开始配置 aria2 的配置文件

```
mkdir "/root/.aria2" && cd "/root/.aria2"
wget "https://raw.githubusercontent.com/chiakge/Aria2-Rclone-DirectoryLister-Aria2Ng/master/sh/aria2.conf"
wget "https://raw.githubusercontent.com/chiakge/Aria2-Rclone-DirectoryLister-Aria2Ng/master/sh/autoupload.sh"
wget "https://raw.githubusercontent.com/chiakge/Aria2-Rclone-DirectoryLister-Aria2Ng/master/sh/dht.dat"
wget "https://raw.githubusercontent.com/chiakge/Aria2-Rclone-DirectoryLister-Aria2Ng/master/sh/trackers-list-aria2.sh"
echo '' > /root/.aria2/aria2.session
chmod +x /root/.aria2/trackers-list-aria2.sh
chmod +x /root/.aria2/autoupload.sh
chmod 777 /root/.aria2/aria2.session
wget --no-check-certificate https://raw.githubusercontent.com/chiakge/Aria2-Rclone-DirectoryLister-Aria2Ng/master/sh/aria2 -O /etc/init.d/aria2
chmod +x /etc/init.d/aria2
update-rc.d -f aria2 defaults
```

有两个配置文件需要修改，我们先修改 **aria2.conf**。aria2.conf 为 aria2 的配置文件。<br />
输入命令`vim aria2.conf`，按`i`进行编辑

```
## 用户必改项 ##
# 文件的保存路径(可使用绝对路径或相对路径), 默认: 当前启动位置
dir=
# 设置的RPC授权令牌, v1.18.4新增功能, 取代 --rpc-user 和 --rpc-passwd 选项
rpc-secret=
```

> `dir`为下载目录。在这里，我设置为 **/data/wwwroot/www.aria2down.tk/Download**<br />
`rpc-secret`为访问秘钥，是 Aria2Ng 连接 Aria2 唯一的验证。输入自己记得的密码即可。


输入完后按`esc`，输入`:wq`保存文件。

`autoupload.sh`是下载文件后自动将下载文件移动到挂载目录下的脚本。我们暂时不进行配置，等到 rclone 时再进行配置。我们先配置`trackers-list-aria2.sh`脚本。

`trackers-list-aria2.sh`脚本是自动更新 bt 下载的 **trackers** 服务器的脚本。我们并不需要修改脚本内容，只需要把它添加进入计划任务。输入`crontab -e`进入任务计划管理。

> 第一次使用一般会遇到这种情况，我一般使用 **vim.basic**<br />
![](https://www.94ish.me/usr/uploads/2018/02/1346494786.png#alt=)


编辑方法与 vim 相同，将以下两句添加进去。

```
0 3 */7 * * /root/.aria2/trackers-list-aria2.sh
*/5 * * * * /usr/sbin/service aria2 start
```

保存后 **aria2** 即配置完毕，我们输入`bash /etc/init.d/aria2 start`启动 aria2。<br />
启动成功时我们即可看到以下配置信息:<br />
![](https://www.94ish.me/usr/uploads/2018/02/3954307145.png#alt=)<br />
我们可以利用 **Aria2Ng** 进行连接测试。<br />
![](https://www.94ish.me/usr/uploads/2018/02/2552700963.png#alt=)

<a name="7a9c0a98"></a>
## 挂载谷歌云盘

一般 VPS 的硬盘是不够我们用过瘾的。所以我们可以挂载无限空间的谷歌云盘来爽一把。

```
cd /root
wget https://raw.githubusercontent.com/chiakge/Aria2-Rclone-DirectoryLister-Aria2Ng/master/sh/rclone_debian.sh  && bash rclone_debian.sh
rm -f rclone_debian.sh
rclone config
```

**交互信息**

```
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
name> aria2down  #随便填，后面要用到
Type of storage to configure.
Choose a number from below, or type in your own value
 1 / Alias for a existing remote
   \ "alias"
 2 / Amazon Drive
   \ "amazon cloud drive"
 3 / Amazon S3 (also Dreamhost, Ceph, Minio, IBM COS)
   \ "s3"
 4 / Backblaze B2
   \ "b2"
 5 / Box
   \ "box"
 6 / Cache a remote
   \ "cache"
 7 / Dropbox
   \ "dropbox"
 8 / Encrypt/Decrypt a remote
   \ "crypt"
 9 / FTP Connection
   \ "ftp"
10 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
11 / Google Drive
   \ "drive"
12 / Hubic
   \ "hubic"
13 / Local Disk
   \ "local"
14 / Microsoft Azure Blob Storage
   \ "azureblob"
15 / Microsoft OneDrive
   \ "onedrive"
16 / Openstack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
17 / Pcloud
   \ "pcloud"
18 / QingCloud Object Storage
   \ "qingstor"
19 / SSH/SFTP Connection
   \ "sftp"
20 / Webdav
   \ "webdav"
21 / Yandex Disk
   \ "yandex"
22 / http Connection
   \ "http"
Storage> 11 #选择11
Google Application Client Id - leave blank normally.
client_id> #留空
Google Application Client Secret - leave blank normally.
client_secret> #留空
Scope that rclone should use when requesting access from drive.
Choose a number from below, or type in your own value
 1 / Full access all files, excluding Application Data Folder.
   \ "drive"
 2 / Read-only access to file metadata and file contents.
   \ "drive.readonly"
   / Access to files created by rclone only.
 3 | These are visible in the drive website.
   | File authorization is revoked when the user deauthorizes the app.
   \ "drive.file"
   / Allows read and write access to the Application Data folder.
 4 | This is not visible in the drive website.
   \ "drive.appfolder"
   / Allows read-only access to file metadata but
 5 | does not allow any access to read or download file content.
   \ "drive.metadata.readonly"
scope> 1 #选择1
ID of the root folder - leave blank normally.  Fill in to access "Computers" folders. (see docs).
root_folder_id> #留空
Service Account Credentials JSON file path  - leave blank normally.
Needed only if you want use SA instead of interactive login.
service_account_file> #留空
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine or Y didn't work
y) Yes
n) No
y/n> n  #选择n
If your browser doesn't open automatically go to the following link: https://accounts.google.com/o/oauth2/auth....  #复制到浏览器打开，获取验证码
Log in and authorize rclone for access
Enter verification code>  #填入上面获取到的验证码
Configure this as a team drive?
y) Yes
n) No
y/n> y  #选择y
Fetching team drive list...
No team drives found in your account--------------------
[Rats]
client_id = 
client_secret = 
service_account_file = 
token = {"access_token":"ya29.GltFBd7UJN2qrxdG8FnG_rMuB18ogb8QlujdL7glvXtfV"}
team_drive = 
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote
y/e/d> y  #选择y
Current remotes:

Name                 Type
====                 ====
aria2down            drive

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q  #选择q退出
```

退出后我们来设置下`rclone`的**自启动**脚本。

```
wget https://raw.githubusercontent.com/chiakge/Aria2-Rclone-DirectoryLister-Aria2Ng/master/sh/rcloned
vim rcloned
```

我们要修改以下内容

```
 #rclone name名
REMOTE='' #远程文件夹
LOCAL='' #挂载地址
```

> NAME 即是刚刚 rclone 配置是输入的`name`。这里我填写的是`aria2down`。<br />
REMOTE 填写的是**谷歌云盘中文件夹**的名称，我在云盘里新建了一个 _chikage_ 的文件夹，所以这里填写的就是`chikage`。<br />
LOCAL 填写**挂载在 vps 中位置**的地址，在这里我填写的是`/data/wwwroot/www.aria2down.tk/Cloud`<br />
输入完成后保存。我们将其设置自启, 并尝试启动。


```
mv rcloned /etc/init.d/rcloned
chmod +x /etc/init.d/rcloned
update-rc.d -f rcloned defaults
bash /etc/init.d/rcloned start
```

检测信息显示 **rclone** 启动成功即可。<br />
![](https://www.94ish.me/usr/uploads/2018/02/1016847848.png#alt=)<br />
最后我们配置下`autoupload.sh`，输入`vim /root/.aria2/autoupload.sh`

```
downloadpath='' #下载目录
rclone=''   #rclone挂载的目录
```

> downloadpath 为之前在 **aria2.conf** `dir`的值。<br />
rclone 为刚刚填入 **rcloned** `LOCAL`的值


保存后即完成了全套离线下载方案的部署。

<a name="388a5fee"></a>
## 结束语

这个教程编写大概耗了我一天的时间，其中的绝大多数脚本由自己编写并调试，在这过程中也遇到许多问题。希望我这篇教程能给你带来帮助。如果可以的话，你可以打赏一下我，不在乎金钱的大小，你的支持是我前进的动力。
