Dockerfile 文件

``` shell
 FROM ubuntu:18.04  
 RUN apt-get update  
 RUN apt-get install -y openssh-server  
 RUN mkdir /var/run/sshd  
 RUN echo 'root:root' |chpasswd  
 RUN sed -ri 's/^#?PermitRootLogin\s+.*/PermitRootLogin yes/' /etc/ssh/sshd_config  
 RUN sed -ri 's/UsePAM yes/#UsePAM yes/g' /etc/ssh/sshd_config  
 RUN mkdir /root/.ssh  
 # 配置 golang  
 RUN apt-get install -y wget git gcc  
 RUN wget -P /tmp https://dl.google.com/go/go1.19.10.linux-arm64.tar.gz  
 RUN tar -C /usr/local -xzf /tmp/go1.19.10.linux-arm64.tar.gz  
 RUN rm -rf /tmp/go1.19.10.linux-arm64.tar.gz  
 ENV GOPATH /go  
 ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH  
 RUN mkdir -p $GOPATH/src $GOPATH/bin && chmod -R 777 $GOPATH  
 RUN apt-get clean && \  
     rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*  
 EXPOSE 22  
 CMD ["/usr/sbin/sshd", "-D"]
```