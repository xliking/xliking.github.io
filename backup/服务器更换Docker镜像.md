**打开这个目录，没有就创建一个**
```
/etc/docker/daemon.json
```
打开 https://registry.docker-cn.com 会获取一个加速链接
编写一个配置文件，这里我就用我的
```
{
 
  "registry-mirrors": ["https://qg6oyurf.mirror.aliyuncs.com"]
 
}
```
**保存关闭，运行一下命令**
```
sudo systemctl daemon-reload
sudo systemctl restart docker
```