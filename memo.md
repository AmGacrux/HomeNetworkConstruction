# 作ろうとしてるもの

 ![ネットワーク構成図](./abstruct.jpg)

# 環境

* ONU
  * Softbank光
* ホームゲートウェイ
  * Router #1
    * Foxconn WMTA-2.3
      * Wi-fiルーター機能付き
* ルーター
  * Router #2
    * Cisco C841M
* サーバーマシン
  * Server #1
    * RaspberryPi Model A+
      * Zabbixで下の#2,3を監視する専用のマシン
        * スペック足りてる？
  * Server #2
    * RaspberryPi3 Model B+
      * 自分用の外から見れるツール用
        * 一度立てたらソフトの構成をそうそう変えないだろうからこれで
          * スペック足りてる？？
  * Server #3
    * Dell Vostro 270s
      * 外部向けの公開用Webサーバー
        * 用途が決まっちゃいないのにこの中では一番スペックが高い
* 外付けHDD
  * 2台でRAID1組んでServer#1,2,3のバックアップ用に

## 物理マシンの設定

### RaspberryPi Model A+

* OS: Raspbian Stretch Lite  

    Version:November 2018  
    Release date:2018-11-13  
    Kernel version:4.14  
    Release notes:Link  

* ストレージ: microSD 16GB (class10)

https://www.raspberrypi.org/downloads/raspbian/

インストールはWindows機にアダプタで繋いだmicroSDに、  
SDフォーマッターしてからWin32DiskImagerで書き込むいつもの流れ。

#### インストール直後の下準備

SSHを有効化するまでは適当なディスプレイと  
USBハブにLANとマウス、キーボードをつないで操作する。  

IPアドレスも現段階では特に設定せずに  
Router #1のLANポートに繋いで割り振られたIPを使う。  
DHCPクライアントを表示する機能を使うと便利。  

![DHCPの一覧](rt01_sample01_dhcp.png)

###### ファームウェアの更新

出力は省略。  
適宜Y(Yes)を入力すること。

結構時間かかる。

```shell
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade

$ uname -a
Linux raspberrypi 4.14.50+ #1122 Tue Jun 19 12:21:21 BST 2018 armv6l GNU/Linux

$ vcgencmd version
Jun  7 2018 15:31:38
Copyright (c) 2012 Broadcom
version 4800f08a139d6ca1c5ecbee345ea6682e2160881 (clean) (release)

$ sudo rpi-update
$ sudo reboot

$ uname -a
Linux raspberrypi 4.14.97+ #1197 Mon Feb 4 21:00:28 GMT 2019 armv6l GNU/Linux
$ vcgencmd version
Jan 22 2019 16:50:58
Copyright (c) 2012 Broadcom
version 7bfabcecab2918f85a2a217b389e256eac696962 (clean) (release) (start)
```

##### SSHの有効化

```shell
$ sudo raspi-config
```

![raspi-config](./srv01_config01_ssh.png)

ここの *"5 Interfacing Options"* から、  
*"P2 SSH"* を選び続く確認画面で "Yes" しておく。

![raspi-config](./srv01_config02_ssh.png)

これでSSH接続ができるようになる。
ログインユーザ名とパスワードは、  
``` pi ``` と ``` raspberry ``` 。

###### 不要なソフトと設定ファイルの削除

Minibianを直接入れられたらよかったんだけど、  
更新が2016年で止まってしまってるのが気になるので  
最新のRaspianから手作業で掃除することにした。  
(多いので節を分割)

* Wolfram、mathematica、Scratch、LibreOfficeの削除

```shell
$ sudo apt-get purge wolfram-engine
$ sudo apt-get purge sonic-pi
$ sudo apt-get purge scratch
$ sudo apt-get remove --purge libreoffice*
$ sudo apt-get clean
$ sudo apt-get autoremove
```

##### 参考資料

Raspberry Pi 不要パッケージの削除
https://qiita.com/NaotakaSaito/items/f6e1ae206963b971028e


#### RaspberryPi3 Model B

* OS:CentOS 7.6 armhfp(Arm32) Minimal image for RaspberryPi 2/3
* ストレージ: microSD 32GB (class10)

http://isoredirect.centos.org/altarch/7/isos/armhfp/CentOS-Userland-7-armv7hl-RaspberryPI-Minimal-1810-sda.raw.xz

https://wiki.centos.org/SpecialInterestGroup/AltArch/armhfp

こちらも落としてきた.raw.xzファイルを  
WSLで ``` $ xz -d ファイル名 ``` して解凍された.rawファイルをフォーマットしたmicroSDにWin32DiskImagerで書き込めばOK。 

ログインユーザ名とパスワードは、  
``` root ``` と ``` centos ``` 。

SDカードの容量に対してパーティションサイズが少なく認識されているのを直す。  

```shell
# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       1.4G  896M  451M  67% /
devtmpfs        460M     0  460M   0% /dev
tmpfs           464M     0  464M   0% /dev/shm
tmpfs           464M   12M  452M   3% /run
tmpfs           464M     0  464M   0% /sys/fs/cgroup
/dev/mmcblk0p1  667M   38M  629M   6% /boot
tmpfs            93M     0   93M   0% /run/user/0
```
```shell
# cat /root/README
== CentOS 7 userland ==

If you want to automatically resize your / partition, just type the following (as root user):
rootfs-expand
```

```shell
# rootfs-expand
# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        28G  899M   28G   4% /
devtmpfs        460M     0  460M   0% /dev
tmpfs           464M     0  464M   0% /dev/shm
tmpfs           464M   13M  452M   3% /run
tmpfs           464M     0  464M   0% /sys/fs/cgroup
/dev/mmcblk0p1  667M   38M  629M   6% /boot
tmpfs            93M     0   93M   0% /run/user/0
```

```shel
# uname -a
Linux localhost 4.14.82-v7.1.el7 #1 SMP Sun Nov 25 22:42:27 UTC 2018 armv7l armv7l armv7l GNU/Linux
# vcgencmd version
Nov  4 2018 16:31:07
Copyright (c) 2012 Broadcom
version ed5baf9520a3c4ca82ba38594b898f0c0446da66 (clean) (release)

# yum update

# uname -a
Linux localhost 4.14.91-v7.1.el7 #1 SMP Sun Jan 6 12:24:04 UTC 2019 armv7l armv7l armv7l GNU/Linux
# vcgencmd version
Dec 17 2018 23:56:39
Copyright (c) 2012 Broadcom
version da468960fe03ecbaa8e3f1ee01c7217c3bd01fa8 (clean) (release)
```

##### 参考資料

