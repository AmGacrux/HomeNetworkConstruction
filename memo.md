## 作ろうとしてるもの

 ![ネットワーク構成図](./abstruct.jpg)

## 環境

* ONU
  * プロバイダ：Softbank光
* ホームゲートウェイ(Wi-fiルーター機能付き)
  * Router #1
* ルーター
  * Router #2

## サーバーマシン(物理)

* RaspberryPi Model A+

Raspbian Stretch Lite

    ersion:November 2018
    Release date:2018-11-13
    Kernel version:4.14
    Release notes:Link

https://www.raspberrypi.org/downloads/raspbian/

## インストール直後の下準備

Wolfram、mathematica、Scratch、LibreOfficeの削除

```shell
$ sudo apt-get purge wolfram-engine
$ sudo apt-get purge sonic-pi
$ sudo apt-get purge scratch
$ sudo apt-get remove --purge libreoffice*
$ sudo apt-get clean
$ sudo apt-get autoremove
```

Raspberry Pi 不要パッケージの削除
https://qiita.com/NaotakaSaito/items/f6e1ae206963b971028e


* RaspberryPi3 Model B

CentOS 7.6 armhfp(Arm32) Minimal image for RaspberryPi 2/3

http://isoredirect.centos.org/altarch/7/isos/armhfp/CentOS-Userland-7-armv7hl-RaspberryPI-Minimal-1810-sda.raw.xz

ARM版CentOSのインストールは↓から。

https://wiki.centos.org/SpecialInterestGroup/AltArch/armhfp

