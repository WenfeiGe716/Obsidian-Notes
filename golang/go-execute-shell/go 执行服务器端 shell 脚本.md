##  第一种：等待 shell 脚本执行完成

```go
func ExecuteShell(username, password, host string, port int, command string) error {  
   session, err := newSSHSession(username, password, host, port)  
   if err != nil {  
      Logger.Errorf("远程SSH连接失败: %s\n", err)  
      return fmt.Errorf("远程SSH连接失败: %s", err)  
   }  
   defer session.Close()  
  
   Logger.Debugln(command)  
   all := fmt.Sprintf("export %s; export PATH=$PATH:$ThrustPath; %s", node.Environment, command)  
   output, err1 := session.CombinedOutput(all)  
   if err1 != nil {  
      Logger.Errorln(string(output))  
      return fmt.Errorf("远程SSH执行失败: %s", err1)  
   }  
   Logger.Debugln(string(output))  
  
   return nil  
}
```

## 第二种：不等待脚本执行完成并实时通过 Logger 打印

