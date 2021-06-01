----------------------------
Spring boot-异常处理1		|
----------------------------
	# 使用 Spring Boot 提供的处理器
	# 可以处理 404,500等异常
	1,自动异常处理 Controller ,实现接口: org.springframework.boot.autoconfigure.web.ErrorController.ErrorController
	2,覆写方法
		
		@Controller
		public class ErrorController implements org.springframework.boot.autoconfigure.web.ErrorController {

			private static final Logger LOGGER = LoggerFactory.getLogger(ErrorController.class);

			/**
			 * 错误页面的目录/路径
			 * classpatg:/templates/error/error.html
			 */
			private static final String ERROR_PATH = "error";

			/**
			 * 异常的时候,系统会自动跳转到这个路径,并且会在 request 域中存放很多关于异常的信息
			 * 可以根据异常状态码来判断是什么异常
			 * 也可以根据请求类型(ajax)决定是返回ModelAndView还是直接响应Ajax数据
			 * @param modelAndView
			 * @param request
			 * @param response
			 * @return
			 */
			@RequestMapping(value = ERROR_PATH,method = RequestMethod.GET)
			public ModelAndView error(ModelAndView modelAndView,
									  HttpServletRequest request,
									  HttpServletResponse response){
				Class exceptionType = (Class) request.getAttribute("javax.servlet.error.exception_type");   //异常类型
				Throwable exception = (Throwable) request.getAttribute("javax.servlet.error.exception");    //异常类,只有在 500 异常的清空下,该值不为空
				Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");     //HTTP异常状态码
				String servletName = (String) request.getAttribute("javax.servlet.error.servlet_name");     //异常的Servlet
				String erroPath = (String) request.getAttribute("javax.servlet.error.request_uri");         //发生了异常的请求地址(并不是当前地址-request.getRequestURI())
				LOGGER.error("服务器异常, erroPath={}, message={}",erroPath,exception.getMessage());
				/**
				 * 防止部分浏览器、路由器（如小米）等劫持不显示自己的错误页面，强制将code设置为200
				 * 但这样ajax就无法检测错误状态
				 */
				response.setStatus(HttpServletResponse.SC_OK);
				return new ModelAndView(ERROR_PATH + "/error");
			}

			/**
			 * 异常的时候,系统会自动调用这个方法,获取到异常的路径.然后对该路径执行请求
			 * @return
			 */
			@Override
			public String getErrorPath() {
				return ERROR_PATH;
			}
		}

----------------------------
Spring boot-异常处理2		|
----------------------------
	# 使用 Spring 提供的处理器
	# 仅仅只能处理掉 500 服务器异常,404 之类的不会处理
	# 实现 HandlerExceptionResolver 注册到IOC来实现全局的异常处理

		@Component
		public class ExceptionController implements HandlerExceptionResolver {

			private static final Logger LOGGER = LoggerFactory.getLogger(ExceptionController.class);
			
			/*
			 * 可以根据请求的类型(Ajax)来确定是返回mv还是直接响应ajax数据
			 * 这种方式,只能处理掉服务器的异常,无法处理 404 之类的异常
			 */
			@Override
			@ExceptionHandler
			public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
				HandlerMethod handlerMethod = (HandlerMethod) o;		//把handler参数强转
				Method method = handlerMethod.getMethod();				//获取到请求异常的方法对象
				return new ModelAndView("error/error");
			}
		}
	

	# 或者
	# 仅仅只能处理掉 500 服务器异常,404 之类的不会处理
	# 标识 @ControllerAdvice 超级Controller注解,注册到IOC,来实现全局异常的处理
		@ControllerAdvice
		public class ExceptionController {

			private static final Logger LOGGER = LoggerFactory.getLogger(ExceptionController.class);
			
			/*
			 * 可以根据请求的类型(Ajax)来确定是返回mv还是直接响应ajax数据
			 * 这种方式,只能处理掉服务器的异常,无法处理 404 之类的异常
			 */
			@ExceptionHandler
			public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
				HandlerMethod handlerMethod = (HandlerMethod) o;		//把handler参数强转
				Method method = handlerMethod.getMethod();				//获取到请求异常的方法对象
				return new ModelAndView("error/error");
			}
		}