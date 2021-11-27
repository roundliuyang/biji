## docker

```awk
# 搜索tomcat镜像
docker search tomcat

# 拉取tomcat镜像
docker pull tomcat

# 查看所有镜像
docker images

# 运行nginx容器，-d 后台运行容器并打印出容器ID，-p 将容器的8080端口映射到主机的3355端口
docker run -d -p 3355:8080 --name tomcat01 tomcat

# 进入容器
docker exec -it tomcat01 /bin/bash

# 进入容器拷贝文件
cd /usr/local/tomcat/
cp -r webapps.dist/* webapps/
exit

# 查看所有运行中的容器：
docker ps

# 通过容器名称停止容器：
docker container stop tomcat01

# 通过容器名称移除容器：
docker container rm tomcat01
```