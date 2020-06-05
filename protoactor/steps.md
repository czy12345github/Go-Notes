## 定义 Actor
```go
type MyActor struct{
  ...
  state State
}

func (a *MyActor) Receive(context actor.Context){
  switch msg := context.Message().(type){
  case *actor.Started:
  case *actor.Stopping:
  case *actor.Stopped:
  case *actor.Restarting:
  case *MyMessageType:
  ...
  }
}
```

## 启动 actor
```go
func main(){
  // 监听端口
  remote.Start("<ip>:<port>")
  
  context = actor.EmptyRootContext
  
  props = actor.PropsFromProducer(func() actor.Actor{ return &MyActor{} })
  
  // <name> 可以区分同一地址内不同的actor
  context.SpawnNamed(props, "<name>")
}
```

## 发送消息
```go
// 属于接口 sendPart
// 接收端 context.Sender() 将是 nil
func (ctx *actorContext) Send(pid *PID, message interface{}) {
	ctx.sendUserMessage(pid, message)
}

// 属于接口 sendPart
func (ctx *actorContext) Request(pid *PID, message interface{}) {
	env := &MessageEnvelope{
		Header:  nil,
		Message: message,
		Sender:  ctx.Self(),
	}

	ctx.sendUserMessage(pid, env)
}

// 属于接口 sendPart
// 可用于获取返回消息
RequestFuture(pid *PID, message interface{}, timeout time.Duration) *Future

// 属于接口 basePart
// 配合发送端 Request 使用
func (ctx *actorContext) Respond(response interface{}) {
	// If the message is addressed to nil forward it to the dead letter channel
	if ctx.Sender() == nil {
		deadLetter.SendUserMessage(nil, response)
		return
	}

	ctx.Send(ctx.Sender(), response)
}
```

## 设置 SenderMiddleware
在发送消息之前或之后执行额外操作，参考示例 remoteheader。
```go
type SenderMiddleware func(next SenderFunc) SenderFunc

func (props *Props) WithSenderMiddleware(middleware ...SenderMiddleware) *Props

// eg
func MySenderMiddleware(next SenderFunc) SenderFunc{
  return func(ctx actor.SenderContext, target *actor.PID, envelope *actor.MessageEnvelope){
    ... // 发送消息前执行的操作
    next(ctx, target, envelope)
    ... // 发送消息后执行的操作
  }
}
```
