# Swagger2 安装与使用



### 安装配置步骤

1. Maven依赖安装
```
<!-- swagger2 -->
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```

2. Swagger的Java配置文件
```
package com.arto;/**
 * Created by lee on 20-3-6.
 */

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.bind.annotation.RestController;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

/**
 * @ClassName Swagger2Config
 * @Description TODO
 * @Author lee
 * @Date 20-3-6 上午9:06
 * @Verion 1.0
 */

@Configuration
@EnableSwagger2
public class Swagger2Config {
    @Bean
    public Docket createRestApi(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(new ApiInfoBuilder()
                        .title("KHYX Restful API")
                        .description("KHYX Restful API")
                        .termsOfServiceUrl("http://www.bejson.com/")
                        .version("1.0")
                        .build())
                .select()
                .apis(RequestHandlerSelectors.withClassAnnotation(RestController.class))
                .paths(PathSelectors.any())
                .build();
    }
}

```

3. Controller中API文档说明注解添加
```
package com.arto.common.shiro.web;
/** Created by lee on 19-6-28. */
import com.arto.common.config.AppType;
import com.arto.common.config.HttpCode;
import com.arto.common.config.PathConfig;
import com.arto.common.entity.ApiResult;
import com.arto.common.entity.MyToken;
import com.arto.common.entity.ShiroUserInfo;
import com.arto.common.util.UserUtils;
import io.swagger.annotations.ApiImplicitParams;
import io.swagger.annotations.ApiOperation;
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.subject.Subject;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

/** @ClassName ShiroUserControll @Description TODO @Author lee @Date 19-6-28 上午11:28 @Verion 1.0 */
@Controller
@RequestMapping(PathConfig.Common.PATH_SHIRO)
public class ShiroUserController {

  @Value("${arto.freemarker.html-location}")
  private String artoFreemarkerHtmlLocation;

  Logger logger = LoggerFactory.getLogger(this.getClass());
  public static final int INITIAL_CAPACITY = 16;

  /**
   * PC登录页面
   *
   * @return
   */
  @ApiOperation(value = "登录页面跳转", httpMethod = "GET/POST")
  @ApiImplicitParams({})
  @RequestMapping(
    value = "/login",
    method = {RequestMethod.GET, RequestMethod.POST}
  )
  public String login() {
    return "sys/login";
  }

  @ApiOperation(value = "测试用户接口", httpMethod = "GET/POST")
  @ApiImplicitParams({})
  @RequestMapping(
    value = "/getusr",
    method = {RequestMethod.GET, RequestMethod.POST}
  )
  @ResponseBody
  public ShiroUserInfo getUser() {
    ShiroUserInfo su = new ShiroUserInfo();
    su.setId("test");
    su.setUserName("test");
    return su;
  }

  @ApiOperation(value = "登录请求验证", httpMethod = "GET/POST")
  @ApiImplicitParams({})
  @RequestMapping(
    value = "/dologin",
    method = {RequestMethod.GET, RequestMethod.POST}
  )
  public String dologin(String username, String password, AppType apptype) {
    Subject subject = SecurityUtils.getSubject();
    ShiroUserInfo principal = UserUtils.getUser();
    if (principal == null) {
      try {
        SecurityUtils.getSubject().login(new MyToken(username, password, apptype));
        principal = (ShiroUserInfo) subject.getPrincipal();
      } catch (Exception e) {
        logger.error(e.getMessage(), e);
        ApiResult<ShiroUserInfo> ar = new ApiResult<ShiroUserInfo>();
        ar.setCode(HttpCode.CODE_401.getCode());
        ar.setMsg("用户名或密码不对！");
        return "redirect:" + PathConfig.Common.PATH_SHIRO + "/login";
      }
    }
    ApiResult<ShiroUserInfo> ar = new ApiResult<ShiroUserInfo>();
    ar.setData(principal);
    return "redirect:/home";
  }

  @ApiOperation(value = "登出", httpMethod = "GET/POST")
  @ApiImplicitParams({})
  @RequestMapping(
    value = "/logout",
    method = {RequestMethod.GET, RequestMethod.POST}
  )
  public String logout() {
    UserUtils.logout();
    return "redirect:/home";
    //    return  PathConfig.Common.PATH_SHIRO + "/login";
  }
}

```

4. 现在访问 http://localhost:8080/khyx/swagger-ui.html 应该能看到效果了, 但如果出现404

- 处理方法：在 GitHub 上[下载 SwaggerUI 项目](https://github.com/swagger-api/swagger-ui
)，建议[下载2.0分支版本](https://github.com/swagger-api/swagger-ui/tree/2.x)，界面清爽点，将dist下所有内容拷贝到本地项目resource/static/swagger下面, 并修改 index.html
```
//url = "http://petstore.swagger.io/v2/swagger.json";        
url = "http://localhost:8080/v2/api-docs";
```
然后访问 http://localhost:8080/static/swagger/index.html



### 注解使用方法

1. @Api：用在请求的类上，说明该类的作用

```
@Api：用在请求的类上，说明该类的作用
    tags="说明该类的作用"
    value="该参数没什么意义，所以不需要配置"
    
示例
@Api(tags="用户注册Controller")

```



2. @ApiOperation：用在请求的方法上，说明方法的作用

```
@ApiOperation："用在请求的方法上，说明方法的作用"
    value="说明方法的作用"
    notes="方法的备注说明"
    
示例
@ApiOperation(value="用户注册",notes="手机号、密码都是必输项，年龄随边填，但必须是数字")

```



3. @ApiImplicitParams：用在请求的方法上，包含一组参数说明

```
@ApiImplicitParams：用在请求的方法上，包含一组参数说明
    @ApiImplicitParam：用在 @ApiImplicitParams 注解中，指定一个请求参数的配置信息       
        name：参数名
        value：参数的汉字说明、解释
        required：参数是否必须传
        paramType：参数放在哪个地方
            · header --> 请求参数的获取：@RequestHeader
            · query --> 请求参数的获取：@RequestParam
            · path（用于restful接口）--> 请求参数的获取：@PathVariable
            · body（不常用）
            · form（不常用）    
        dataType：参数类型，默认String，其它值dataType="Integer"       
        defaultValue：参数的默认值
    
示例
@ApiImplicitParams({
    @ApiImplicitParam(name="mobile",value="手机号",required=true,paramType="form"),
    @ApiImplicitParam(name="password",value="密码",required=true,paramType="form"),
    @ApiImplicitParam(name="age",value="年龄",required=true,paramType="form",dataType="Integer")
})

```



4. @ApiResponses：用于请求的方法上，表示一组响应

```
@ApiResponses：用于请求的方法上，表示一组响应
    @ApiResponse：用在@ApiResponses中，一般用于表达一个错误的响应信息
        code：数字，例如400
        message：信息，例如"请求参数没填好"
        response：抛出异常的类
    
示例
@ApiOperation(value = "select1请求",notes = "多个参数，多种的查询参数类型")
@ApiResponses({
    @ApiResponse(code=400,message="请求参数没填好"),
    @ApiResponse(code=404,message="请求路径没有或页面跳转路径不对")
})

```



5. @ApiModel：用于响应类上，表示一个返回响应数据的信息

```
@ApiModel：用于响应类上，表示一个返回响应数据的信息
            （这种一般用在post创建的时候，使用@RequestBody这样的场景，
            请求参数无法使用@ApiImplicitParam注解进行描述的时候）
    @ApiModelProperty：用在属性上，描述响应类的属性
    
示例
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;

import java.io.Serializable;

@ApiModel(description= "返回响应数据")
public class RestMessage implements Serializable{

    @ApiModelProperty(value = "是否成功")
    private boolean success=true;
    @ApiModelProperty(value = "返回对象")
    private Object data;
    @ApiModelProperty(value = "错误编号")
    private Integer errCode;
    @ApiModelProperty(value = "错误信息")
    private String message;

    /* getter/setter */
}

```

