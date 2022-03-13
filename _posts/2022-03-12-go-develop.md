这里记录一下 go 的简单安装过程

平台：
ArchLinux

1. 安装 go

````
sudo pacman -S go
````

使用 `go version` 检查安装是否正确

2. 改变环境变量，向 `.bashrc` 文件中加入下列内容

````
export GOPATH=$HOME/path/to/goproject
export GOBIN=$GOPATH/bin
````

3. 可以设置 GOPROXY

````
go env -w GOPROXY="https://goproxy.cn,direct"
````

4. 开始 go mod 

````
go env -w GO111MODULE="on"
````

之后可以写一个 `go` 的 `demo` 测试:

````go
packge main

import fmt

func main() {
    fmt.Println("HelloWorld!")
}
````
