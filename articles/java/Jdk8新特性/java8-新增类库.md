<h1> 新增类库 </h1>

## Base64				

java.util.Base64
注意,这个是JDK8的新特性,在JDK8中,Base64已经是 JDK的标准类库

加密:传入的是字节数组,返回的是String

解密:传入的是String,返回的是字节数组
```java
String text = "Base64";

// 加密
String encoded = Base64.getEncoder().encodeToString(text.getBytes(StandardCharsets.UTF_8));		

System.out.println(encoded);	

//解密
String decoded = new String(Base64.getDecoder().decode(encoded),StandardCharsets.UTF_8); 
System.out.println(decoded);
```
