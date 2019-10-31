# Github文档已不再维护，请移步码云https://gitee.com/xiaoyudeguang/projects 查看相关项目
# easy-netty

#### 介绍
将netty服务端编写简化到一个注解就能完成的程度，客户端编写甚至更简单。。。

## 使用说明

### Maven引用(如果版本有变化，请自行去maven中央仓库引用)
```
 <dependency>
      <groupId>io.github.xiaoyudeguang</groupId>
      <artifactId>easy-netty</artifactId>
      <version>1.2.5-RELEASE</version>
 </dependency>
```

### netty服务端

```
@Server(port = 9000)
public class NettyServer extends AbstractNettyServer{

	@Override
	public String handleMsg(String msg) {
		Console.log(this.getClass(), msg);
		return EasyBuffer.wrapper(this.hashCode(),"=>",System.currentTimeMillis());
	}
}
```
> 注意，@Server注解和继承AbstractNettyServer类是必须的。
### netty客户端

#### 1. netty短连接版阻塞式客户端(会等待结果后返回)

```
NettyUtils.getInstance("127.0.0.1", 9000).sendMsg(msg);
```
#### 2. netty长连接版非阻塞式客户端（遵循netty原生规则）

```
@Sharable
public class NettyClient extends AbstractNettyClient{

	public NettyClient(String key, String host, int port) throws Exception {
		super(key, host, port);
	}

	@Override
	public void doOnConnection(ChannelHandlerContext ctx) throws Exception {
	       Console.log(this.getClass(), "初次建立连接时做点啥");
           this.write(hex);
	       this.writeBytes(bytes);
	       this.writeMsg(msg);
	}

	@Override
	public Object doService(Channel channel, byte[] msg) throws Exception {
		Console.log(this.getClass(), "服务端响应时做点啥");
		return null;
	}
}
```
> 注意，@Sharable和继承AbstractNettyClient类是必须的。构造方法的key，可以随意设置，只是用于保存Netty连接。
> 怎么向服务端输出点啥呢？你可以用doOnConnection()方法里提到的三个write()方法来输出内容到服务端。
> 在doService()方法里，返回null将不会输出东西到服务端；同样，你也可以直接调用三个write()方法来蹂躏服务端。
#### 3. netty长连接版阻塞式客户端(会等待结果后返回)

```
public String test(String key, String host, int port, String data) throws Exception {
	BlockNettyClient nettyClient = CacheUtils.get(key);
	//如果从未与TCPServer建立过连接，就建立一个连接并缓存起来；否则，直接从缓存里取出已经建立好的连接发送消息
	if(nettyClient == null) {
		nettyClient = new BlockNettyClient(key, host, port, data);
		ThreadManager.execute(new Thread(nettyClient));
		CacheUtils.put(key, nettyClient);
	}else { 
		nettyClient.doSend(data);
	}
	return nettyClient.getResponse();
}
```
