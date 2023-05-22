# context
并发安全的
go1.7引入的标准库
作用：在上下文中传递除了业务参数之外的额外信息
例如：在微服务业务中，需要整成业务链条整体的超时时间信息

## 什么时候可以用到contex
- 上下文信息传递
  在http服务器中rebase把当前的commit放到公共分支的最后面，merge把当前的commit和公共分支合并在一起；2、用merge命令解决完冲突后会产生一个commit，而用rebase命令解决完冲突后不会产生额外的comm
- 控制子协程的运行
- 超时控制的方法调用
- 可以取消的方法调用

## context三种类型
- cancel可取消的context
- timerCtx 增加了超时时间
- ValueCtx 存储goroutine共享变量 如cookie、session
![img.png](img_5.png)