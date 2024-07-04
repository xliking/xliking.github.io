**登录** 
```
docker login --username=aliyun6760578873 registry.cn-hangzhou.aliyuncs.com
 ```
**从Registry中拉取镜像**
```
docker pull registry.cn-hangzhou.aliyuncs.com/xlike/midj:[镜像版本号]
```
**将镜像推送到Registry**
```
$ docker login --username=aliyun6760578873 registry.cn-hangzhou.aliyuncs.com
$ docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/xlike/midj:[镜像版本号]
$ docker push registry.cn-hangzhou.aliyuncs.com/xlike/midj:[镜像版本号]
```