----------------------------
Spring-boot websocket		|
----------------------------
	# 关于WebSocket的自动配置包
		org.springframework.boot.autoconfigure.websocket
	
	# 如果是war形式运行的websocket,那么 endpoint 就应该交给容器管理,而不是 ServerEndpointExporter

----------------------------
Spring-boot websocket整1	|
----------------------------
	# 导入依赖
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
	
	# 注册Bean
		* ServerEndpointExporter
		@Configuration  
		public class WebSocketConfiguration {  
			@Bean  
			public ServerEndpointExporter serverEndpointExporter (){  
				return new ServerEndpointExporter();  
			}  
		}  
	
	# 使用注解开发WebSocket端点
		 
		@Component 
		@ServerEndpoint("/websocket")		//必须以斜线开头,不然启动报错
		public WebSocketEndpoint{
			@OnMessage
			@OnOpen
			@OnClose
			@OnError
		}

	# 注意
		* @OnError 要添加参数:Throwable ,不然启动异常