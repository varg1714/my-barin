---
source: https://mp.weixin.qq.com/s/r4Qs2mSIhDOvOMDzmoUKtQ
create: 2025-04-17 21:39
read: true
tags:
  - Spring
  - Java
  - 框架使用
knowledge: true
knowledge-date: 2025-10-20
summary: "[[Spring 的参数校验]]"
---
![](https://mmbiz.qpic.cn/mmbiz_jpg/Z6bicxIx5naLFZ2mB9wuDGL8DRZg7Rf2qPuzNhvG7snQ8v1LWAPPOCOjZhRrrcRpBmjibCv0Xm92SlMv1MbciaZ7Q/640?wx_fmt=jpeg&from=appmsg)

阿里妹导读

本文介绍了在 Java Spring Boot 开发中如何使用 JSR 303 和 Hibernate Validator 进行参数校验，以避免冗余的 if-else 判断。文章涵盖了基本注解的使用、全局异常处理、分组校验、嵌套对象校验、快速失败配置以及自定义校验规则等实用技巧。

一、参数校验简介

在实际工作中，得到数据得到的第一步就是校验数据的正确性，如果存在录入上的问题，一般会通过注解校验，发现错误后返回给用户，但是对于一些逻辑上的错误，比如购买金额 = 购买数量 * 单价，这样的规则就很难使用注解方式验证了，此时可以使用 Spring 提供的验证器 (Validator) 规则去验证。

由于 Validator 框架通过硬编码完成数据校验，在实际开发中会显得比较麻烦，因此现在开发更加推荐使用 JSR 303 完成服务端的数据校验。

Spring3 开始支持 JSR 303 验证框架，JSR 303 是 Java 为 Bean 数据合法性校验所提供的标准框架。JSR 303 支持 XML 风格的和注解风格的验证，通过在 Bean 属性上标注类似于 @NotNull、@Max 等的标准注解指定校验规则，并通过标准的验证接口对 Bean 进行验证。

可以查看详细内容并下载 JSR 303 Bean Validation**①**。JSR 303 不需要编写验证器，它定义了一套可标注在成员变量、属性方法上的校验注解，如下表所示：

<table><tbody><tr><td colspan="1" rowspan="1"><section><span><strong>约束</strong></span></section></td><td colspan="1" rowspan="1"><section><span><strong>说明</strong></span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Null</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须为 Null</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@NotNull</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须不为 Null</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@AssertTrue</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须为 true</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@AssertFalse</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须为 false</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Min(value)</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须是一个数字，其值必须大于等于最小值</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Max(value)</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须是一个数字，其值必须小于等于最大值</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@DecimalMin(value)</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须是一个数字，其值必须大于等于最小值</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@DecimalMax(value)</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须是一个数字，其值必须小于等于最大值</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Size(max,min)</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素的大小必须在指定的范围内</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Digits(integer,fraction)</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须是一个数字，其值必须在可接受范围内</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Past</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须是一个过去的日期</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Future</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须是一个将来的日期</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Pattern(value)</span></section></td><td colspan="1" rowspan="1"><section><span>被直接的元素必须符合指定的正则表达式</span></section></td></tr></tbody></table>

Hibernate Validator 是 JSR 303 的一个参考实现, 除了支持所有标准的校验注解之外，还扩展了如下表所示的注解。

<table><tbody><tr><td colspan="1" rowspan="1"><section><span><strong>约束</strong></span></section></td><td colspan="1" rowspan="1"><section><span><strong>说明</strong></span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@NotBlank</span></section></td><td colspan="1" rowspan="1"><section><span>检查被注解的元素是不是 Null，以及被去掉前后空格的长度是否大于 0</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Email</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须是电子邮件格式</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@URL</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须是合法的 URL 地址</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Length</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的字符串的字符串的大小必须在指定的范围内</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@NotEmpty</span></section></td><td colspan="1" rowspan="1"><section><span>校验集合元素或数组元素或者字符串是否非空。通常作用于集合字段或数组字段，此时需要集合或者数字的元素个数大于 0。也可以作用于字符串，此时校验字符串不能为 null 或空串（可以是一个空格）。</span></section></td></tr><tr><td colspan="1" rowspan="1"><section><span>@Range</span></section></td><td colspan="1" rowspan="1"><section><span>被注解的元素必须在合适的范围内</span></section></td></tr></tbody></table>

① http://jcp.org/en/jsr/detail?id=303

二、实战 

spring-boot-starter-validation 是 Spring Boot 框架中的一个模块，它构建在 Hibernate Validator 的基础之上，并提供了对 Bean Validation 的集成支持。它可以通过 SpringBoot 的自动配置机制轻松集成到 Spring 应用程序中。使用 spring-boot-starter-validation，你可以在 SpringBoot 应用程序中使用 Hibernate Validator 的功能，而不需要显式地配置 Hibernate Validator。

因此，Hibernate Validator 是一个独立的验证框架，而 spring-boot-starter-validation 则是一个为 Spring Boot 应用程序提供集成 Hibernate Validator 的模块。

在这里搭建一个简单的 SpringBoot 项目，并引入如下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**2.1 全局异常处理**

```java
@Slf4j
@ControllerAdvice
public class GlobalExceptionHandler {
    //参数校验异常
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    public Result error(MethodArgumentNotValidException e){
        log.warn(e.getMessage());
        return Result.fail()
                .code(ResultCodeEnum.ARGUMENT_VALID_ERROR.getCode())
                .message(e.getBindingResult().getFieldError().getDefaultMessage());
    }
    //参数校验异常
    @ExceptionHandler(BindException.class)
    @ResponseBody
    public Result error(BindException e){
        log.warn(e.getMessage());
        StringBuilder sb = new StringBuilder();
        for (ObjectError error : e.getBindingResult().getAllErrors()) {
            sb.append(error.getDefaultMessage());
        }
        return Result.fail()
                .code(ResultCodeEnum.ARGUMENT_VALID_ERROR.getCode())
                .message(sb.toString());
    }
    //参数校验异常
    @ExceptionHandler(ConstraintViolationException.class)
    @ResponseBody
    public Result error(ConstraintViolationException e){
        log.warn(e.getMessage());
        StringBuilder sb = new StringBuilder();
        for (ConstraintViolation<?> violation : e.getConstraintViolations()) {
            sb.append(violation.getMessage());
        }
        return Result.fail()
                .code(ResultCodeEnum.ARGUMENT_VALID_ERROR.getCode())
                .message(sb.toString());
    }
    //全局异常
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public Result error(Exception e){
        log.warn(e.getMessage());
        return Result.fail().message("执行了全局异常处理");
    }
}
```

**2.2 快速开始**

2.2.1 实体类

```java
@Data
public class UserCreateVO {
    @NotBlank(message = "用户名不能为空")
    private String userName;

    @NotBlank(message = "姓名不能为空")
    private String name;

    @Size(min=11,max=11,message = "手机号长度不符合要求")
    private String phone;

    @NotNull(message = "性别不能为空")
    private Integer sex;
}
```

```java
@Data
public class UserUpdateVO {

    @NotBlank(message = "id不能为空")
    private String id;

    @NotBlank(message = "用户名不能为空")
    private String userName;

    @NotBlank(message = "姓名不能为空")
    private String name;

    @Size(min=11,max=11,message = "手机号长度不符合要求")
    private String phone;

    @NotNull(message = "性别不能为空")
    private Integer sex;
}
```

2.2.1 定义请求接口

```java
@Validated
@RestController
@RequestMapping("/user")
public class UserController {

    @PostMapping("create")
    public Result createUser(@Validated @RequestBody UserCreateVO userCreateVo){
        return Result.ok("参数校验成功");
    }

    @PostMapping("update")
    public Result updateUser(@Validated @RequestBody UserUpdateVO userUpdateVo){
        return Result.ok("参数校验成功");
    }
}
```

2.2.2 测试

新增接口测试：

这里故意不写用户名

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhicOPmE0LkYVGQpGoWG6W6Jayf9icoL50W8lyrkGA8X9CcmI8R1v0Rfhdw/640?wx_fmt=png)

更新接口测试：

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhicp8Hqh6rLMFgJoFzf9FZT1ElQic0dQwwZbpd9MMRoicR0714HErv7arBw/640?wx_fmt=png)

**2.3 路径传参校验**

有时候我们是通过 @Pathvariable 注解实现参数传递的，这个时候校验如下：

```java
@GetMapping("getUserById/{id}")
public Result getUserById(@PathVariable @Size(min=2,max=5,message = "id长度不符合要求") String id){
    return Result.ok("参数校验成功");
}
```

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhicFnksvd4zF1bDULyW4FmwgDhdhYtWxYArjgBqK5mNEAKMUrRsmcbM7Q/640?wx_fmt=png)

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhicHDziaFhcCn2s0vVS9j7vSLahYdlnbXd5yyqa8KCqQk8eZ8yIMNkGq8w/640?wx_fmt=png)

可以看到，校验规则是生效的。

**2.4 分组校验**

我们前边写了两个 VO 类：UserCreateVO 和 UserUpdateVO。但是如果每个请求都要定制 VO 类的话，那我们还不如直接用 if else 梭哈呢。

比如我们新增的时候一般不需要 id，但是修改的时候需要传入 id。

2.4.1 定义分组校验接口

```java
//分组校验
public class UserGroup {
    public interface CreateGroup extends Default {
    }
    public interface UpdateGroup extends Default {
    }
}
```

2.4.2 统一 VO 校验类

```java
@Data
public class UserVo {

    @NotBlank(message = "id不能为空",groups = UserGroup.UpdateGroup.class)
    private String id;

    @NotBlank(message = "用户名不能为空",groups = 
            {UserGroup.CreateGroup.class,UserGroup.UpdateGroup.class})
    private String userName;

    @NotBlank(message = "姓名不能为空",groups = 
            {UserGroup.CreateGroup.class,UserGroup.UpdateGroup.class})
    private String name;

    @Size(min=11,max=11,message = "手机号长度不符合要求",groups = 
            {UserGroup.CreateGroup.class,UserGroup.UpdateGroup.class})
    private String phone;

    @NotNull(message = "性别不能为空",groups = 
            {UserGroup.CreateGroup.class,UserGroup.UpdateGroup.class})
    private Integer sex;
}
```

2.4.3 修改接口

参数上边添加 @Validated 接口，并指定分组名称, 并统一使用 UserVO 类接收参数。

```java
@Validated
@RestController
@RequestMapping("/user")
public class UserController {

    @PostMapping("create")
    public Result createUser(@Validated(UserGroup.CreateGroup.class) @RequestBody UserVO userVO) {
        return Result.ok("参数校验成功");
    }

    @PutMapping("update")
    public Result updateUser(@Validated(UserGroup.UpdateGroup.class) @RequestBody UserVO userVO) {
        return Result.ok("参数校验成功");
    }

    @GetMapping("getUserById/{id}")
    public Result getUserById(@PathVariable @Size(min = 2, max = 5, message = "id长度不符合要求") String id) {
        return Result.ok("参数校验成功");
    }
}
```

2.4.4 测试

新增接口：

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhic9MZGniaPrKLTmh3DMZVcSaicwKeDCMiatZBo25UwLbvlibBsPJxM2VsmDw/640?wx_fmt=png)

修改接口：

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhicKiaCjbCs60Ss8yHQm3Ftv6rtwZBc6icTvHejmgAW6jDksNDsLYAoLUicA/640?wx_fmt=png)

**2.5 嵌套对象校验**

存在嵌套对象校验的时候，使用 @Valid 注解解决。

2.5.1 添加一个角色实体

```java
@Data
public class Role {
    @NotBlank(message = "角色名称不能为空")
    private String roleName;
}
```

2.5.2 修改 VO 类

在 User 类中引入角色实体，并加入 @Valid 注解。

```java
@Data
public class UserVo {

    @NotBlank(message = "id不能为空",groups = UserGroup.UpdateGroup.class)
    private String id;

    @NotBlank(message = "用户名不能为空",groups =
            {UserGroup.CreateGroup.class,UserGroup.UpdateGroup.class})
    private String userName;

    @NotBlank(message = "姓名不能为空",groups =
            {UserGroup.CreateGroup.class,UserGroup.UpdateGroup.class})
    private String name;

    @Size(min=11,max=11,message = "手机号长度不符合要求",groups =
            {UserGroup.CreateGroup.class,UserGroup.UpdateGroup.class})
    private String phone;

    @NotNull(message = "性别不能为空",groups =
            {UserGroup.CreateGroup.class,UserGroup.UpdateGroup.class})
    private Integer sex;

    @Valid
    @NotNull(message = "角色信息不能为空")
    private Role role;
}
```

2.5.3 测试

测试新增接口，故意不传角色信息：

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhic0rSTibfTyibG9icbrqNffGQ0NNa0KG7fehibEJgeiaicqVw8pAWqsGTaOfVQ/640?wx_fmt=png)

测试修改接口，故意不传角色信息：

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhicEfAoW0cpJIpd6iaFgibt0XrAtxKOcAhc9bib9WZzQWlsPyKrBIhp8xWYQ/640?wx_fmt=png)

**2.6 快速失败配置**

默认 validation 的校验规则默认会检查所有属性的校验规则，在每一条规则都校验完成后才抛出异常，这样子难免效率会很低，我们希望在发现错误后就不再向后检查，可以通过 “快速失败” 配置解决，参考代码如下：

```java
/**
 * 参数校验相关配置
 */
@Configuration
public class ValidConfig {
    /**
     * 快速返回校验器
     * @return
     */
    @Bean
    public Validator validator(){
        ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
                .configure()
                //快速失败模式
                .failFast(true)
                .buildValidatorFactory();
        return validatorFactory.getValidator();
    }

    /**
     * 设置快速校验，返回方法校验处理器
     * 使用MethodValidationPostProcessor注入后，会启动自定义校验器
     * @return
     */
    @Bean
    public MethodValidationPostProcessor methodValidationPostProcessor(){
        MethodValidationPostProcessor methodValidationPostProcessor = new MethodValidationPostProcessor();
        methodValidationPostProcessor.setValidator(validator());
        return methodValidationPostProcessor;
    }
}
```

**2.7 自定义校验规则**

有时候框架自带的校验无法满足我们业务的需求，这个时候可以根据自己的需求定制校验规则。

这里我们自定义一个手机号校验的注解，**仅作示例使用，因为官方已经提供了手机号的校验注解**。

2.7.1 定义注解

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD, ElementType.FIELD, ElementType.ANNOTATION_TYPE, ElementType.CONSTRUCTOR, ElementType.PARAMETER, ElementType.TYPE_USE})
@Constraint(validatedBy = PhoneValidator.class)
public @interface PhoneValid {
    String message() default "请填写正确的手机号";

    Class<?>[ ] groups() default {};


    Class<? extends Payload>[ ] payload() default {};

}
```

@Constraint 注解是 Hibernate Validator 的注解，用于实现自定义校验规则，通过 validatedBy 参数指定进行参数校验的实现类。

2.7.2 校验实现类

实现 ConstraintValidator 接口。

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhicEpmVib6fj4b0OSZREEZMqlnWf4DLneOEkVicFL3EbicYfD1Pj5UJib57LQ/640?wx_fmt=png)

在 ConstraintValidator 中，第一个参数为注解，即 Annotation，第二个参数是泛型。

```java
public class PhoneValidator implements ConstraintValidator<PhoneValid,Object> {
    /**
     * 11位手机号的正则表达式,以13、14、15、17、18头
     * ^：匹配字符串的开头
     * 13\d：匹配以13开头的手机号码
     * 14[579]:匹配以145、147、149开头的手机号
     * 15[^4\D]：匹配以15开头且第3位数字不为4的手机号码
     * 17[^49\D]:匹配以17开头且第3位数字部位4或9的手机号码
     * 18\d :匹配以18开头的手机号码
     * \d{8}：匹配手机号码的后8位，即剩余的8个数字
     * $：匹配字符串的结尾
     */
    public static final String REGEX_PHONE="^(13\\d|14[579]|15[^4\\D]|17[^49\\D]|18\\d)\\d{8}$";


    //初始化注解
    @Override
    public void initialize(PhoneValid constraintAnnotation) {
        ConstraintValidator.super.initialize(constraintAnnotation);
    }

    //校验参数，true表示校验通过，false表示校验失败
    @Override
    public boolean isValid(Object o, ConstraintValidatorContext constraintValidatorContext) {
        String phone = String.valueOf(o);
        if(phone.length()!=11){
            return false;
        }
        //正则校验
        return phone.matches(REGEX_PHONE);
    }
}
```

2.7.3 在实体中添加自定义注解

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhic4eO0EUHhcNrs4TXDII5ibVibRvLzl91Ek6ZzfH2AOelo9VmWAqSM4bkg/640?wx_fmt=png)

2.7.4 测试自定义校验规则

先使用一个正确的手机号测试，校验成功。

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhicd9Xf44hBLO98QSGGoA4dRSpRiaxbicRCAr76fia32QhAF616Z2k3gCQMg/640?wx_fmt=png)

再用一个错误的手机号测试，校验失败。

![](https://mmecoa.qpic.cn/sz_mmecoa_png/7TJbnpickD6MHGQ6cfbScRibxgBckPkEhicibs641HHtoLsYU5fs1YLy5EkIx2ot6mKH03S3apkrgLXmicW7ianmeU9A/640?wx_fmt=png)

就介绍到这里，其实参数校验的用法很多，这里仅列举常用功能。

**参考文献**

[01]. Hibernate Validator

_https://hibernate.org/validator/documentation/_

[02]. SpringBoot + validator 优雅参数校验，消除 if-else

_https://juejin.cn/post/7246800194463957053_

[03]. 新来的一个同事，把 SpringBoot 参数校验玩的那叫一个优雅

_https://juejin.cn/post/7322275119592996927_

[04]. 快速失败 (Fail Fast) Spring Validation

_https://blog.csdn.net/liuqiang211/article/details/107979269_