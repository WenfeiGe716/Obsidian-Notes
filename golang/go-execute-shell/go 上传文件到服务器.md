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

