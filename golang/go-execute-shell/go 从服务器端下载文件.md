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

第三步下载文件

```go
 func FileDownload(username, password, host string, port int, remoteRelativePath, localPath string) error {  
   sftpClient, err := newSFTPSession(username, password, host, port)  
   if err != nil {  
      return err  
   }  
   defer sftpClient.Close()  
  
   // filePath := path.Join(node.TestPath, fmt.Sprintf("node%d", nodeIndex), remoteRelativePath)  
   remoteFile, err := sftpClient.Open(remoteRelativePath)  
   if err != nil {  
      return err  
   }  
  
   _ = os.Mkdir(path.Dir(localPath), 0777)  
   localFile, err := os.Create(localPath)  
   if err != nil {  
      return err  
   }  
   _, err = io.Copy(localFile, remoteFile)  
   return err  
}
```
