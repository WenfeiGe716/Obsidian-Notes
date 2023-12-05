## 第一步创建 SSH 客户端

```go
// NewSSHClient connects to an SSH host and via username:password@host:port
func NewSSHClient(username, password, host string, port int) (*ssh.Client, error) {  
   clientConfig := &ssh.ClientConfig{  
      User:    username,  
      Auth:    []ssh.AuthMethod{ssh.Password(password)},  
      Timeout: 30 * time.Second,  
      HostKeyCallback: func(hostname string, remote net.Addr, key ssh.PublicKey) error {  
         return nil  
      },  
   }  
  
   addr := fmt.Sprintf("%s:%d", host, port)  
   return ssh.Dial("tcp", addr, clientConfig)  
}
```

## 第二步建立SFTP连接

```go
func newSFTPSession(username, password, host string, port int) (*sftp.Client, error) {  
   client, err := NewSSHClient(username, password, host, port)  
   if err != nil {  
      return nil, err  
   }  
  
   return sftp.NewClient(client)  
}
```



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

```go
func ExecuteShellRealOutputWithUser(userName, password, host, command string) error {  
   session, err := newSSHSession(userName, password, host, 22)  
   if err != nil {  
      Logger.Errorf("远程SSH连接失败: %s\n", err)  
      return fmt.Errorf("远程SSH连接失败: %s", err)  
   }  
   defer session.Close()  
   // Create pipes for capturing command output and error  
   stdout, stdoutPipe := io.Pipe()  
   stderr, stderrPipe := io.Pipe()  
   // Combine output and error into a single channel  
   var wg sync.WaitGroup  
   wg.Add(2)  
   Logger.Debugln(command)  
   go func() {  
      defer wg.Done()  
      // Log standard output in real-time  
      logReader(stdout)  
   }()  
   go func() {  
      defer wg.Done()  
      // Log standard error in real-time  
      logReader(stderr)  
   }()  
   all := fmt.Sprintf("%s", command)  
   // Set command output and error to the pipes  
   session.Stdout = stdoutPipe  
   session.Stderr = stderrPipe  
   // Start the command  
   err = session.Run(all)  
   // Close pipes to signal the log readers to finish  
   stdoutPipe.Close()  
   stderrPipe.Close()  
  
   // Wait for the log readers to finish  
   wg.Wait()  
  
   if err != nil {  
      Logger.Errorf("远程SSH执行失败: %s\n", err)  
   }  
   return err  
}  
  
// logReader reads from the reader and logs the output in real-timefunc logReader(reader io.Reader) {  
   scanner := bufio.NewScanner(reader)  
   for scanner.Scan() {  
      Logger.Debugln(scanner.Text())  
   }  
}
```