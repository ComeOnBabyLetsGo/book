# Docker部署springboot

### 1、制作项目启动脚本

创建.sh文件，文件内容如下：

```sh
java -server -Xms512m -Xmx512m -jar /data/DataVerificationServer.jar
```
`/data`目录为容器内部文件目录, 不用改变
`DataVerificationServer.jar`是项目jar包名称  

**如果要指定外部配置文件**

```sh
java -server -Xms512m -Xmx512m -jar /data/DataVerificationServer.jar --spring.config.additional-location=/data/config/application.yml
```

>补充知识点   
>
>1.   –spring.config.location=“D:/xxx/system.properties”   会使默认配置文件失效
>2.   –spring.config.additional-location   用于追加配置文件。原有的application.properties或application.yml文件均有效

### 2、添加sh文件执行权限

```powershell
chmod +x startServer.sh
```

### 3、制作Dockerfile，内容如下：

```dockerfile
FROM openjdk:8u201-jdk-alpine

# 中文环境
ENV LANG zh_CN.UTF-8
ENV LC_ALL zh_CN.utf8

#时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo '$TZ' > /etc/timezone

# 字体文件 可以不要
RUN apk add --update ttf-dejavu fontconfig && rm -rf /var/cache/apk/*


CMD ["sh","-c","/data/startServer.sh"]
```

注意 CMD  执行的脚本路径  `/data/startServer.sh`

### 4、创建镜像，命令如下：

```
docker build -t jdk:8u201 .
```

### 5、创建并启动容器

注意 -v 挂载的文件夹路径  把物理机  项目jar与启动脚本 所在文件夹挂载到容器内`/data`目录
所以上边的启动脚本  Dockerfile目录都是用的`/data`目录

测试服务器

```shell
docker run  -dit --network projectBridge -v /kingdom/gw_server:/data --name gw_server  -p 9999:9999  jdk:8u201

# /kingdom/report/aj-report-2.0.2
docker run  -dit --network projectBridge -v /kingdom/report:/data --name aj_report  -p 9095:9095  jdk:8u201

docker run  -dit --network projectBridge -v /kingdom/zx_feed:/data --name zx_feed_server3  -p 8403:8403  jdk:8u201

```

生产服务器

```shell
# 生产  网关管理
docker run -dit -v /iotserver/gw_server:/data  --name gw_server -p 9001:9001 alpinejdk
```

