
## 第一步：安装 swag 命令

```shell
 go get -u github.com/swaggo/swag/cmd/swag
```

> 注意：
> 如果执行之后，swag -v 提示命令不存在，进行下面的操作：
> 执行 go env 查看 gopath 对应的目录
> 进入 $GOPATH/pkg/mod/github.com/swaggo/swag@v1.16.2/cmd/swag
> 执行 go build -o swag
> 将编译好的文件 移动到 /usr/local/go/bin 下

第二步：在项目包含main.go文件的目录下执行:生成docs文件
```shell
 swag init
```
第三步：安装 gin-swagger
```sh
 go get -u github.com/swaggo/gin-swagger
```
第四步：安装 swaggo/files
```sh
 go get -u github.com/swaggo/files
```
## go-swapper注解规范说明

``` go
 // @Summary 摘要  
 // @Description 描述  
 // @Description 接口的详细描述  
 // @Id 全局标识符  
 // @Version 接口版本号  
 // @Tags 接口分组，相当于归类  
 // @Accept json 浏览器可处理数据类型  
 // @Produce json 设置返回数据的类型和编码  
 // @Param 参数格式 从左到右：参数名、入参类型、数据类型、是否必填和注释  例：id query int true "ID"  
 // @Success 响应成功 从左到右：状态码、参数类型、数据类型和注释  例：200 {string} string "success"  
 // @Failure 响应失败 从左到右：状态码、参数类型、数据类型和注释  例：400 {object}  string "缺少参数 ID"  
 // @Router 路由： 地址和http方法  例：/api/user/{id} [get]  
 // @contact.name 接口联系人  
 // @contact.url 联系人网址  
 // @contact.email 联系人邮箱  
   ### 增加token验证方法  
 // @securityDefinitions.apikey ApiKeyAuth  安全方式  
 // @in header  token携带的位置，这里是在header中  
 // @name Authorization  heaer中的名称  
```
### 参数param的几种类型
```
 query 形如 /user?userId=1016  
 body 需要将数据放到 body 中进行请求  
 formData multipart/form-data* 请求  
 path 形如 /user/1016  
 header header头信息
```
### main函数添加注解
```go
 // @title 接囗文档  
 // @version 1.0  
 // @description 上链项目  
 // @termsofservice https://github.com/xxxx  
 // @contact.name Tracy  
 // @contact.email xxxx@126.com  
 // @host 127.0.0.1:9090
```
> 注：main函数中的@host，其他所有路由接囗都默认成此地址，如有特殊可以在路由接囗中添加@host，覆盖main的@host , 不加@host 可以设置成动态的，这样方便部署任何平台，不依赖HOST， 如下所示：
> 
>  // @title 接囗文档  
>  // @version 1.0  
>  // @description 上链项目  
>  // @termsofservice https://github.com/xxxx  
>  // @contact.name Tracy  
>  // @contact.email xxxx@126.com
### 路由函数添加注解
```go
// @Summary 个人注册  
// @title Swagger API  
// @version 1.0  
// @Tags 劳动者管理  
// @description  个人注册接囗  
// @BasePath /labourer/register  
// @Produce  json  
// @Param phone body LabourerInfo true "劳动者信息"  
// @Success 200 {object} RespData "{"code":200,"data":{},"msg":"ok"}"  
// @Router /labourer/register [post]
```
### 添加访问路由
```go
// 添加[swagger](https://so.csdn.net/so/search?q=swagger&spm=1001.2101.3001.7020)访问路由  
 r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
```
### 格式化swag注解
```sh
swag fmt
```
### 生成文档
```sh
swag init
```
> 注：每次添加或修改注解后，都需要使用swag init命令重新生成文档，使其生效 swag init 默认通过项目根目录中的main.go文件生成，如果main.go不在根目录中，如在cmd/main.go中，则要使用参数-g, 命令：
> 
> swag init -g ./cmd/main.go
