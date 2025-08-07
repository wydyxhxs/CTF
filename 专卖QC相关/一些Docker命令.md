#docker [[docker]]
```bash
chmod +x build-docker.sh    #
./build-docker.sh     #运行项目目录下的./build-docker.sh脚本，构建镜像
docker save -o tobacco-assistant.tar tobacco-assistant   #将项目打包为docker镜像`tobacco-assistant.tar` 
```

```bash
查看日志: docker logs -f tobacco-assistant
停止服务: docker stop tobacco-assistant
```
在客户机上：
```bash
docker load -i tobacco-assistant.tar    #载入容器
docker run -d -p 9380:9380 tobacco-assistant    #启动容器
```
查看容器列表：
```bash
docker ps -a    #查看容器列表
docker stop <容器ID或容器名>     #停止容器
docker rm tobacco-assistant     #删除容器
docker rmi tobacco-assistant     #删除镜像
```
### 删除容器和删除镜像的区别
>- **容器是运行的结果**，删除容器只是把这个运行实例清掉，但 **镜像还在**，可以再次 `docker run`。
- **镜像是模板**，删除镜像后就没有“模板”，只能重新导入或重新 build。
