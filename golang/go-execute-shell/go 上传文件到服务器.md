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

## 第三步上传文件

```go
func UploadFile(username, password, host string, port int, localFilePath, remotePath string) error {  
   sftpClient, err := newSFTPSession(username, password, host, port)  
   if err != nil {  
      return err  
   }  
   defer sftpClient.Close()  
  
   return uploadFile(sftpClient, localFilePath, remotePath)  
}
```

## 第四步 uploadFile 方法

```go
 func uploadFile(sftpClient *sftp.Client, localFilePath string, remotePath string) error {  
   srcFile, err := os.Open(localFilePath)  
   if err != nil {  
      Logger.Errorf("open file %s error: %s\n", localFilePath, err)  
      return fmt.Errorf("open file %s error: %s", localFilePath, err)  
   }  
   defer srcFile.Close()  
  
   remoteFileName := path.Base(localFilePath)  
   dstFile, err := sftpClient.Create(path.Join(remotePath, remoteFileName))  
   if err != nil {  
      Logger.Errorf("create SFTP client error: %s\n", path.Join(remotePath, remoteFileName))  
      return fmt.Errorf("create SFTP client error: %s", path.Join(remotePath, remoteFileName))  
   }  
   defer dstFile.Close()  
  
   _, err = io.Copy(dstFile, srcFile)  
   return err  
}
```