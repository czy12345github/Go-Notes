## 1.安装 Protocol Compiler
https://github.com/protocolbuffers/protobuf/releases

设置安装路径 $PROTOBUFPATH

## 2.安装相关 protoc 插件
```
cd $GOPATH/src/github.com/gogo
git clone https://github.com/gogo/protobuf.git
go install github.com/gogo/protobuf/protoc-gen-gogoslick
```

## 3.安装 protoactor-go
https://github.com/AsynkronIT/protoactor-go

编译 *.proto 时，-I 选项指示 protoc 寻找待编译及 import 的 .proto 文件的路径，运行：
```
protoc -I=. -I=$GOPATH/src -I=$PROTOBUFPATH/include --gogoslick_out=plugins=grpc:. *.proto
```
