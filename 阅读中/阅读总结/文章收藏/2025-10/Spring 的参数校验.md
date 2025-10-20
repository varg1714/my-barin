---
source:
  - "[[优雅的参数校验，告别冗余 if-else]]"
create: 2025-10-20
---

## 1. 核心要点

1. **基础概念**
    * **JSR 303 (Bean Validation):** 是 Java 为 Bean 数据合法性校验提供的标准框架，通过注解（如 `@NotNull`, `@Size`）定义校验规则。
    * **Hibernate Validator:** 是 JSR 303 的一个参考实现，并提供了一些扩展注解（如 `@NotBlank`, `@Email`）。
    * **依赖引入:** 在 Spring Boot 项目中，只需添加 `spring-boot-starter-validation` 依赖即可。

2. **全局异常处理**
    * 通过创建一个带有 `@ControllerAdvice` 注解的类，可以全局捕获参数校验异常。
    * 使用 `@ExceptionHandler` 来处理特定异常，如 `MethodArgumentNotValidException` (用于 `@RequestBody` 校验失败) 和 `ConstraintViolationException` (用于 `@PathVariable` 或 `@RequestParam` 校验失败)，并返回统一的错误响应。

3. **核心校验技巧**
    * **基本校验:** 在 Controller 方法的参数前使用 `@Validated` 注解，并在 DTO/VO 对象的字段上添加相应的校验注解。
    * **路径传参校验:** 直接在 Controller 方法中使用 `@PathVariable` 注解的参数上添加校验注解，如 `@PathVariable @Size(...) String id`。
    * **分组校验 (Group Validation):**
        * **场景:** 当多个操作（如“新增”和“更新”）共用一个 VO，但校验规则不同时（例如更新时需要校验 `id`，新增时不需要）。
        * **实现:**
            1. 定义分组接口（如 `CreateGroup.class`, `UpdateGroup.class`）。
            2. 在 VO 的字段注解上通过 `groups` 属性指定该校验规则属于哪个分组。
            3. 在 Controller 方法上使用 `@Validated(UserGroup.CreateGroup.class)` 来指定本次校验使用哪个分组的规则。
    * **嵌套对象校验:**
        * **场景:** 当一个 VO 对象内部包含了另一个需要校验的对象时。
        * **实现:** 在嵌套对象的字段上添加 `@Valid` 注解，即可触发对该嵌套对象内部字段的校验。

4. **高级功能**
    * **快速失败 (Fail-Fast):**
        * **作用:** 默认情况下，校验器会检查所有字段的规则再统一返回错误。配置快速失败后，一旦遇到第一个校验不通过的字段，就会立即中断并返回错误，提高效率。
        * **实现:** 通过 `@Configuration` 创建一个 `Validator` Bean，并设置 `.failFast(true)`。
    * **自定义校验规则:**
        * **作用:** 当内置注解无法满足复杂的业务需求时，可以自定义校验逻辑。
        * **实现步骤:**
            1. **定义注解:** 创建一个新的注解接口（如 `@PhoneValid`），并使用 `@Constraint(validatedBy = PhoneValidator.class)` 指定其校验器。
            2. **编写校验器:** 创建一个实现 `ConstraintValidator<YourAnnotation, FieldType>` 接口的类，在 `isValid()` 方法中编写具体的校验逻辑。
            3. **使用注解:** 在 VO 的字段上应用你自定义的注解。