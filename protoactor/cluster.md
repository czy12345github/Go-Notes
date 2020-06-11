## 配置 consul

### 1.安装 https://www.consul.io/downloads.html

### 2.设置节点
简单设置可参考 https://learn.hashicorp.com/consul/getting-started/join

## protoactor 主要流程

### 1.编写 protos.proto 并生成相关代码
```
message HelloRequest {
  string name = 1;
}

message HelloResponse {
  string message = 1;
}

// 生成代码中将其定义为接口，用户自定义结构体以实现所需功能
// 一个 service 对应一个 actor
service Hello {
  rpc SayHello (HelloRequest) returns (HelloResponse) {}
}
```
代码生成指令：
```
protoc -I=. -I=$GOPATH/src --gogoslick_out=. protos.proto 
protoc -I=. -I=$GOPATH/src --gograin_out=. protos.proto 
```

### 2.自定义结构体实现所需功能
```go
type hello struct {
	cluster.Grain
}

func (*hello) Terminate()  {}

func (h *hello) SayHello(r *HelloRequest, ctx cluster.GrainContext) (*HelloResponse, error) {
	return &HelloResponse{Message: "hello " + r.Name + " from " + h.ID()}, nil
}

func init() {
	// apply DI and setup logic
	HelloFactory(func() Hello { return &hello{} })
}
```

### 3.启动 cluster 并调用集群服务
```go
func main() {
  // this node knows about Hello kind
  // 多个 consul 节点可以注册同一类型的服务
  remote.Register("Hello", actor.PropsFromProducer(func() actor.Actor {
    return &shared.HelloActor{}
  }))

  cp, err := consul.New()
  if err != nil {
    log.Fatal(err)
  }
  // 启动集群后才能调用服务，<ip> 为运行 consul 时绑定的 ip
  cluster.Start("mycluster", "<ip>:<port>", cp)
  
  // hello 为生成的类型 *shared.HelloGrain，调用服务时将发送消息到 HelloActor
  // <ID> 用于区分不同的 actor 实例，第一次调用服务时会在注册此类服务的某个节点生成 HelloActor 实例
  hello := shared.GetHelloGrain("<ID>")
  res, err := hello.SayHello(&shared.HelloRequest{Name: "Roger"})
  if err != nil {
    log.Fatal(err)
  }
  log.Printf("Message from grain: %v", res.Message)
}
```
