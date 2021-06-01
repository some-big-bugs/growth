----------------------------
applicationEvent			|
----------------------------
	# 专门监听applicationContent的各种事件 
	# 实现接口 ApplicationListener<E extends ApplicationEvent>
	# 通过该接口的泛型来决定要监听的事件(ApplicationEvent子类)
		SpringApplicationEvent
		ApplicationStartingEvent
		....
	# demo
		@Component
		public class ApplicationStartingListener implements ApplicationListener<SpringApplicationEvent> {

			private static final Logger LOGGER = LoggerFactory.getLogger(ApplicationStartingListener.class);

			@Autowired
			private ApplicationContext applicationContext;

			@Override
			public void onApplicationEvent(SpringApplicationEvent event) {
				LOGGER.debug("application 启动ok");
				//注入 root context
				SpringContext.setApplicationContext(this.applicationContext);
				//启动UDP服务
				Server.start();
			}
		}