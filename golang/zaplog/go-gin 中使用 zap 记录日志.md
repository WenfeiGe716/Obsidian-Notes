## 第一步 创建zap工具类

```go
import (  
   "fmt"  
   "github.com/gin-gonic/gin"   rotatelogs "github.com/lestrrat/go-file-rotatelogs"  
   "go.uber.org/zap"   "go.uber.org/zap/zapcore"   "io"   "net"   "net/http"   "net/http/httputil"   "os"   "runtime/debug"   "strings"   "time")  
  
var (  
   Logger *zap.SugaredLogger  
)  
  
// SetupLogger 创建logger
func SetupLogger() error {  
   var err error  
   filepath := fmt.Sprintf("%s", Configs.Log["LogFilePath"])  
   infofilename := fmt.Sprintf("%s", Configs.Log["LogInfoFileName"])  
   warnfilename := fmt.Sprintf("%s", Configs.Log["LogWarnFileName"])  
   debugfilename := fmt.Sprintf("%s", Configs.Log["LogDebugFileName"])  
   fileext := fmt.Sprintf("%s", Configs.Log["LogFileExt"])  
   Logger, err = GetInitLogger(filepath, infofilename, warnfilename, debugfilename, fileext)  
   if err != nil {  
      return err  
   }  
   defer Logger.Sync()  
   return nil  
}  
  
// GetInitLogger get logger
func GetInitLogger(filepath, infofilename, warnfilename, debugfilename, fileext string) (*zap.SugaredLogger, error) {  
   encoder := getEncoder()  
   //两个判断日志等级的interface  
   //warnlevel以下属于info   
   infoLevel := zap.LevelEnablerFunc(func(lvl zapcore.Level) bool {  
      return lvl < zapcore.WarnLevel  
   })  
   //warnlevel及以上属于warn  
   warnLevel := zap.LevelEnablerFunc(func(lvl zapcore.Level) bool {  
      return lvl >= zapcore.WarnLevel  
   })  
  
   infoWriter, err := getLogWriter(filepath+"/"+infofilename, fileext)  
   if err != nil {  
      return nil, err  
   }  
   warnWriter, err2 := getLogWriter(filepath+"/"+warnfilename, fileext)  
   if err2 != nil {  
      return nil, err2  
   }  
   debugWriter, err3 := getLogWriter(filepath+"/"+debugfilename, fileext)  
   if err3 != nil {  
      return nil, err3  
   }  
   //创建具体的Logger  
   core := zapcore.NewTee(  
      zapcore.NewCore(encoder, infoWriter, infoLevel),  
      zapcore.NewCore(encoder, warnWriter, warnLevel),  
      zapcore.NewCore(encoder, debugWriter, zap.DebugLevel),  
      zapcore.NewCore(encoder, zapcore.AddSync(os.Stdout), zap.DebugLevel),  
   )  
   loggerres := zap.New(core, zap.AddCaller())  
   return loggerres.Sugar(), nil  
}  
  
// Encoder  
func getEncoder() zapcore.Encoder {  
   //encoderConfig := zap.NewProductionEncoderConfig()  
   encoderConfig := zap.NewDevelopmentEncoderConfig()  
   encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder  
   encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder  
   encoderConfig.EncodeLevel = zapcore.CapitalColorLevelEncoder  
   encoderConfig.EncodeDuration = zapcore.NanosDurationEncoder  
   encoderConfig.StacktraceKey = "stacktrace"  
   encoderConfig.FunctionKey = zapcore.OmitKey  
   return zapcore.NewConsoleEncoder(encoderConfig)  
}  
  
// LogWriter  
func getLogWriter(filePath, fileext string) (zapcore.WriteSyncer, error) {  
   warnIoWriter, err := getWriter(filePath, fileext)  
   if err != nil {  
      return nil, err  
   }  
   return zapcore.AddSync(warnIoWriter), nil  
}  
  
// 日志文件切割，按天  
func getWriter(filename, fileext string) (io.Writer, error) {  
   // 保存30天内的日志，每24小时(整点)分割一次日志  
   hook, err := rotatelogs.New(  
      filename+"-%Y%m%d%H%M."+fileext,  
      rotatelogs.WithLinkName(filename),  
      rotatelogs.WithMaxAge(time.Hour*24*30),  
      rotatelogs.WithRotationTime(time.Hour*24),  
   )  
   //供测试用，保存30天内的日志，每1分钟(整点)分割一次日志  
   //hook, err := rotatelogs.New(   
   // filename+"_%Y%m%d%H%M.log",   
   // rotatelogs.WithLinkName(filename),   
   // rotatelogs.WithMaxAge(time.Hour*24*30),   
   // rotatelogs.WithRotationTime(time.Minute*1),   
   //)   
   if err != nil {  
      //panic(err)  
      return nil, err  
   }  
   return hook, nil  
}  
  
// GinLogger 接收gin框架默认的日志  
//func GinLogger() gin.HandlerFunc {  
// return func(c *gin.Context) {  
//    start := time.Now()  
//    path := c.Request.URL.Path  
//    query := c.Request.URL.RawQuery  
//    c.Next()  
//  
//    cost := time.Since(start)  
//    Logger.Info(path,  
//       zap.Int("status", c.Writer.Status()),  
//       zap.String("method", c.Request.Method),  
//       zap.String("path", path),  
//       zap.String("query", query),  
//       zap.String("ip", c.ClientIP()),  
//       zap.String("user-agent", c.Request.UserAgent()),  
//       zap.String("errors", c.Errors.ByType(gin.ErrorTypePrivate).String()),  
//       zap.Duration("cost", cost),  
//    )  
// }  
//}  
  
// GinRecovery recover掉项目可能出现的panic，并使用zap记录相关日志
func GinRecovery(stack bool) gin.HandlerFunc {  
   return func(c *gin.Context) {  
      defer func() {  
         if err := recover(); err != nil {  
            // Check for a broken connection, as it is not really a  
            // condition that warrants a panic stack trace.            var brokenPipe bool  
            if ne, ok := err.(*net.OpError); ok {  
               if se, ok := ne.Err.(*os.SyscallError); ok {  
                  if strings.Contains(strings.ToLower(se.Error()), "broken pipe") || strings.Contains(strings.ToLower(se.Error()), "connection reset by peer") {  
                     brokenPipe = true  
                  }  
               }  
            }  
  
            httpRequest, _ := httputil.DumpRequest(c.Request, false)  
            if brokenPipe {  
               Logger.Error(c.Request.URL.Path,  
                  zap.Any("error", err),  
                  zap.String("request", string(httpRequest)),  
               )  
               // If the connection is dead, we can't write a status to it.  
               c.Error(err.(error)) // nolint: errcheck  
               c.Abort()  
               return  
            }  
  
            if stack {  
               Logger.Error("[Recovery from panic]",  
                  zap.Any("error", err),  
                  zap.String("request", string(httpRequest)),  
                  zap.String("stack", string(debug.Stack())),  
               )  
            } else {  
               Logger.Error("[Recovery from panic]",  
                  zap.Any("error", err),  
                  zap.String("request", string(httpRequest)),  
               )  
            }  
            c.AbortWithStatus(http.StatusInternalServerError)  
         }  
      }()  
      c.Next()  
   }  
}
```

## 第二步 main 方法中注册zap相关中间件

```go
// 注册zap相关中间件  
// 将 gin 日志也记录
//router.Use(util.GinLogger(), util.GinRecovery(true))  
router.Use(util.GinRecovery(true))
```