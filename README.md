

本项目是个人学习项目，是对
[restful-api-demo](https://github.com/wendell-dev/restful-api-demo)
的注释学习。

有能力的时候会贡献原项目。

---
# 基于springBoot编写的RESTFul API
本项目可用于快速搭建基于springBoot的RESTFul API服务，同时集成了swagger作为接口的在线文档与调试工具，数据交互格式建议是JSON格式。

## 增强理解

[Spring Boot集成swagger2生成接口文档](https://www.jianshu.com/p/a115c9367a59)

[自定义RESTful API服务规范](https://www.jianshu.com/p/bdea0385a77e)


## RESTFul API
首先本项目是一个RESTFul API服务的demo，与此同时再集成了一些做API常用的工具。 对于RESTFul API服务各有各的见解，网上大多是自己封装了controller层统一格式返回，通常情况下，不管你怎么请求，它总是响应你的http状态码为200。 而本项目中充分结合了HTTP状态码规范，使用ResponseEntity + HttpStatus的方式完成我们的API。当然，你想做一个完全具有RESTFul风格的API，你需要具有良好的RESTFul风格的资源设计能力。

## 全局异常处理
采用@RestControllerAdvice + @ExceptionHandler的方式对全局异常进行处理，同时加入了常见的一些自定义异常类。

## 参数验证器
采用spring提供的@Validated注解结合hibernate的validator进行验证，你只需要在你的验证实体对象中使用验证注解，如@NotNull、@NotBlank等，同时在你的controller方法中加入@Validated注解即可，验证结果信息已经由全局异常处理器帮你做好了。

## TOKEN验证
当我们的API需要登录后才能访问时，简单做法是登录验证成功后给客户端生成一个token，客户端后续的请求都需要带上这个token参数，服务端对这个token进行验证，验证通过即可访问API。本项目中也集成了token的生成，同时通过拦截器统一验证了token的有效性，这依赖于redis来存储token，但这也是比较流行的做法。
你只需要在controller中需要的地方加入@AccessToken注解即可，同时如果你需要当前登录的用户信息，只需要在方法参数中加入@UserPrincipal注解修饰参数UserPrincipalVO即可。
代码示例：
```
// 在登录业务类中注入用户TOKEN组件
@Autowired
private UserTokenComponent userTokenComponent;

// 登录
public UserPrincipalVO login(String account, String pwd) {
  // 登录逻辑验证 ~~~~~
  // 验证成功后,可得到用户信息
  // 根据用户信息创建token, 可以把用户其它信息填充进UserPrincipalVO中,提供了全参的构造方法
  UserPrincipalVO userPrincipalVO = new UserPrincipalVO(account);
  return userTokenComponent.createToken(userPrincipalVO);
}
```
```
@ApiOperation(value = "需要登录后才能访问的API")
@GetMapping("/token")
@AccessToken
public ResponseEntity<UserPrincipalVO> testToken(@ApiIgnore @UserPrincipal UserPrincipalVO user) {
	return ResponseEntity.ok(user);
}
```

## 参数签名验证
当我们的API需要作为开放接口时，一般会为接入方分配对应的accessKey和secret，接入方每次请求我们的API时，需要把accessKey和secret与其他参数进行统一的方式签名得到签名串sign，同时把sign作为参数传送给服务端（通常在header中），服务端对请求进行处理后验证sign是否正确，验证通过即可访问API。本项目中同样集成了请求签名，通过拦截器统一进行签名验证，同时也支持使用@RequestBody注解的请求参数。你只需要在controller中需要的地方加入@Sign注解即可，非常简单。
代码示例：
```
@GetMapping("/sign")
@Sign
public ResponseEntity<String> signTest(@RequestParam Long id, @RequestParam String name) {
	// 返回结果为JSON格式
	return ResponseEntity.ok(MsgEnum.SUCCESS.msg("签名验证成功"));
}
```

## 运行项目 - 本地maven构建
```
## 打包
mvn clean package
## 运行项目
java -jar target/restful-api-demo-0.0.1-SNAPSHOT.jar
## 测试访问地址
http://localhost:8081/swagger-ui.html
```

## 运行项目 - 本地docker环境中构建
```
## dockerfile-maven-plugin打包并构建镜像
mvn clean package dockerfile:build
## 或者打包后使用docker命令构建镜像
mvn clean package
docker build -t wendell/restful-api-demo .
## 启动容器
docker run-p 8081:8081 -d --name restful-api-demo wendell/restful-api-demo
## 测试访问地址
http://localhost:8081/swagger-ui.html
```




