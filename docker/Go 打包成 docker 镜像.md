
前置环境准备：docker、docker-compose
## 第一步 ： 获取基础镜像
1、拉取镜像
```shell
 docker pull golang:1.15.7-alpine3.13
```
2、验证版本
```shell
 docker run -it --rm golang:1.15.7-alpine3.13 go version
```
3、运行
```shell
 docker run -it --rm golang:1.15.7-alpine3.13 sh
```
## 第二步 ：构建镜像
### 方法 一 单纯构建（大）
1、编写 Dockerfile
```dockerfile
 FROM golang:1.15.7-alpine3.13 # 基础镜像  
 WORKDIR /go/src # 设置工作目录   
 ADD . /go/src # 向镜像中新增文件  
 RUN cd /go/src  &&  go build -o main # 执行命令  
 CMD ["/go/src/main"] # 启动后执行的命令
```
2、构建镜像
```shell
 docker image build -t build-go-image:v1.0 .
```
-t 构建的镜像的标签名
注意：不要漏掉 .
3、查看存在的镜像
```shell
 docker image ls
```
4、执行镜像
```shell
 docker run -it --rm build-go-image:v1.0
```
### 方法二 分阶段构建（小）
1、编写 Dockerfile
```
 # 编译阶段  
 FROM golang:1.15.7-alpine3.13 AS buildStage  
 WORKDIR /go/src # 设置工作目录   
 ADD . /go/src # 向镜像中新增文件  
 RUN cd /go/src  &&  go build -o main # 执行命令  
 # 打包阶段  
 FROM alpine:latest  
 WORKDIR /app  
 COPY --from=buildStage /go/src/main /app/  
 ENTRYPOINT ./main  
```
2、构建镜像
docker image build -t build-go-image:v2.0 .
-t 构建的镜像的标签名
注意：不要漏掉 .
3、查看存在的镜像
docker image ls
4、执行镜像
docker run -it --rm build-go-image:v2.0