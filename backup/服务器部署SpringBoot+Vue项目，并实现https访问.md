**首先我们需要一台服务器，自己买一个就行，但必须要备案，一个域名**
所以推荐一个 [国外免备案，可以支付宝支付](https://my.racknerd.com/)
买完之后，安装一个[宝塔面板](https://www.bt.cn/)，官网支持远程安装
Centos安装脚本
```
yum install -y wget && wget -O install.sh https://download.bt.cn/install/install_6.0.sh && sh install.sh ed8484bec
```
Ubuntu/Deepin安装脚本
```
wget -O install.sh https://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh ed8484bec
```
**第一步，我们在宝塔面板安装Nginx**






