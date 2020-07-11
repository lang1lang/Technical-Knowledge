# Golang环境配置

1. Golang下载地址：https://golang.org/dl/go1.13.12.linux-amd64.tar.gz
2. 解压go1.13.12.linux-amd64.tar.gz至/usr/local/go
3. 将/usr/local/go/bin/go和/usr/local/go/bin/gofmt拷贝至/usr/bin/
4. 创建go开发路径$GOPATH：/data/go-env-nice
5. 配置/etc/profile

```
# Enable the go modules feature
export GO111MODULE=on
# Set the GOPROXY environment variable
export GOPROXY=https://goproxy.io  #https://gocenter.io
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
export GOPATH=/data/go-env-nice
export PATH=$PATH:$GOPATH/bin
```

6. 查看go版本

```
go version
go version go1.13.12 linux/amd64
```

# vim环境配置

