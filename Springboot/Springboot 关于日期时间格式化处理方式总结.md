---
typora-copy-images-to: ipic
---

# Springboot 关于日期时间格式化处理方式总结

> 项目中使用LocalDateTime系列作为DTO中时间的数据类型，但是SpringMVC收到参数后总报错，为了配置全局时间类型转换，尝试了如下处理方式。
>
> **注：本文基于Springboot2.x测试，如果无法生效可能是spring版本较低导致的。PS：如果你的Controller中的LocalDate类型的参数啥注解（RequestParam、PathVariable等）都没加，也是会出错的，因为默认情况下，解析这种参数是使用`ModelAttributeMethodProcessor`进行处理，而这个处理器要通过反射实例化一个对象出来，然后再对对象中的各个参数进行convert，但是LocalDate类没有构造函数，无法反射实例化因此会报错！！！**

## 完成目标

- 请求入参为 String（指定格式）转 Date，支持get、post（content-type=application/json）
- 返回数据为Date类型转为指定的日期时间格式字符创
- 支持Java8 日期 API，如：`LocalTime`、`localDate` 和 `LocalDateTime`

## **GET请求及POST表单日期时间字符串格式转换**

> 这种情况要和时间作为Json字符串时区别对待，因为前端json转后端pojo底层使用的是Json序列化Jackson工具（`HttpMessgeConverter`）；而时间字符串作为普通请求参数传入时，转换用的是`Converter`，两者在处理方式上是有区别。

### 使用自定义参数转换器（Converter）

实现 org.springframework.core.convert.converter.Converter，自定义参数转换器，如下：

```java
@Configuration
public class DateConverterConfig {
    @Bean
    public Converter<String, LocalDate> localDateConverter() {
      	return new Converter<String, LocalDate>() {
            @Override
            public LocalDate convert(String source) {
                return LocalDate.parse(source, DateTimeFormatter.ofPattern("yyyy-MM-dd"));
            }
        };
    }

    @Bean
    public Converter<String, LocalDateTime> localDateTimeConverter() {
        return new Converter<String, LocalDateTime>() {
            @Override
            public LocalDateTime convert(String source) {
                return LocalDateTime.parse(source, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
            }
        };
    }
}
```

点评：以上两个bean会注入到spring mvc的参数解析器（好像叫做`ParameterConversionService`），当传入的字符串要转为LocalDateTime类时，spring会调用该Converter对这个入参进行转换。

**注意：**`关于自定义的参数转换器 Converter，这里我遇到了一个坑，我再这里详细记录下`，本来我的想法是为了代码精简，将上面匿名内部类的写法精简成lambda表达式的方式：

```java
		@Bean
    @ConditionalOnBean(name = "requestMappingHandlerAdapter")
    public Converter<String, LocalDate> localDateConverter() {
        return source -> LocalDate.parse(source, DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT));
    }
```

当我再次启动项目时却出现了异常：

```
Caused by: java.lang.IllegalArgumentException: Unable to determine source type <S> and target type <T> for your Converter [com.example.demo126.config.MappingConverterAdapter$$Lambda$522/817994751]; does the class parameterize those types?
```

![](https://blog-figure-bed.oss-cn-shanghai.aliyuncs.com/2020/2020-03-06-152938.jpg)

百思不得其解，在查阅了资料才得知一二：

web项目启动注册`requestMappingHandlerAdapter`的时候会初始化`WebBindingInitializer`

```java
adapter.setWebBindingInitializer(getConfigurableWebBindingInitializer());
```

而`ConfigurableWebBindingInitializer`需要`FormattingConversionService`, 而`FormattingConversionService`会将所有的`Converter`添加进来，添加的时候需要获取泛型信息：

```java
@Override
public void addFormatters(FormatterRegistry registry) {
    for (Converter<?, ?> converter : getBeansOfType(Converter.class)) {
      	registry.addConverter(converter);
    }
    for (GenericConverter converter : getBeansOfType(GenericConverter.class)) {
      	registry.addConverter(converter);
    }
    for (Formatter<?> formatter : getBeansOfType(Formatter.class)) {
      	registry.addFormatter(formatter);
    }
}
```

添加Converter.class 一般是通过接口获取两个泛型的具体类型

```java
public ResolvableType as(Class<?> type) {
    if (this == NONE) {
      return NONE;
    }
    Class<?> resolved = resolve();
    if (resolved == null || resolved == type) {
      return this;
    }
    for (ResolvableType interfaceType : getInterfaces()) {
      ResolvableType interfaceAsType = interfaceType.as(type);
      if (interfaceAsType != NONE) {
        return interfaceAsType;
      }
    }
    return getSuperType().as(type);
}
```

Lambda表达式的接口是`Converter`，并不能得到具体的类型，在窥探了SpringMVC源码后才得知原来如此，既然指导了原因，那解决办法：

- 最简单的方法就是不适用Lambda表达式，还是老老实实的使用匿名内部类，这样就不会存在上述问题

- 或者就是等`requestMappingHandlerAdapter`bean注册完成之后再添加自己的`converter`就不会注册到`FormattingConversionService`中

  ```java
  @Bean
  @ConditionalOnBean(name = "requestMappingHandlerAdapter")
  public Converter<String, LocalDateTime> localDateTimeConverter() {
    return source -> LocalDateTime.parse(source, DateTimeUtils.DEFAULT_FORMATTER);
  }
  ```

还可以对前端传递的string进行正则匹配，如yyyy-MM-dd HH:mm:ss、yyyy-MM-dd、 HH:mm:ss等，进行匹配。以适应多种场景。

```java
@Component
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String value) {
        /**
         * 可对value进行正则匹配，支持日期、时间等多种类型转换
         * 这里我偷个懒，在匹配Date日期格式时直接使用了 hutool 为我们已经写好的解析工具类，这里就不重复造轮子了
         * cn.hutool.core.date.DateUtil
         * @param value
         * @return
         */
        return DateUtil.parse(value.trim());
    }
}
```

**注：**这里我偷个懒，在匹配Date日期格式时直接使用了 hutool 为我们已经写好的解析工具类，这里就不重复造轮子了，下面的方法同样使用了该工具类，想要在自己的项目中使用该工具类也很简单，在项目pom文件中引入hutool的依赖就可以了，如下：

```xml
<!--hu tool 工具类-->
<dependency>
  <groupId>cn.hutool</groupId>
  <artifactId>hutool-all</artifactId>
  <version>5.1.3</version>
</dependency>
```

### 使用Spring注解

使用spring自带注解@DateTimeFormat(pattern = "yyyy-MM-dd")，如下：

```java
@DateTimeFormat(pattern = "yyyy-MM-dd")
private Date startDate;
```

如果使用了自定义参数转化器，Spring会优先使用该方式进行处理，即Spring注解不生效。

### 使用ControllerAdvice配合initBinder

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.registerCustomEditor(LocalDate.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) throws IllegalArgumentException {
                setValue(LocalDate.parse(text, DateTimeFormatter.ofPattern("yyyy-MM-dd")));
            }
        });
        binder.registerCustomEditor(LocalDateTime.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) throws IllegalArgumentException {
                setValue(LocalDateTime.parse(text, DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
            }
        });
        binder.registerCustomEditor(LocalTime.class, new PropertyEditorSupport() {
            @Override
            public void setAsText(String text) throws IllegalArgumentException {
                setValue(LocalTime.parse(text, DateTimeFormatter.ofPattern("HH:mm:ss")));
            }
        });
    }
}
```

从名字就可以看出来，这是在controller做环切（这里面还可以全局异常捕获），在参数进入handler之前进行转换；转换为我们相应的对象。

## JSON入参及返回值全局处理

请求类型为:post,content-type=application/json， 后台用`@RequestBody`接收，默认接收及返回值格式为: `yyyy-MM-dd HH:mm:ss`

### 修改 application.yml 文件

在application.propertities文件中增加如下内容：

```yaml
spring:
	jackson:
		date-format: yyyy-MM-dd HH:mm:ss
		time-zone: GMT+8
```

- 支持（content-type=application/json）请求中格式为 `yyyy-MM-dd HH:mm:ss`的字符串，后台用`@RequestBody`接收，及返回值date转为`yyyy-MM-dd HH:mm:ss`格式string；
- 不支持（content-type=application/json）请求中yyyy-MM-dd等类型的字符串转为date；
- 不支持java8日期api;

### 利用Jackson的JSON序列化和反序列化

```java
@Configuration
public class JacksonConfig {

    /** 默认日期时间格式 */
    public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    /** 默认日期格式 */
    public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
    /** 默认时间格式 */
    public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";

    @Bean
    public MappingJackson2HttpMessageConverter mappingJackson2HttpMessageConverter() {
        MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter();
        ObjectMapper objectMapper = new ObjectMapper();

        // 忽略json字符串中不识别的属性
        objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        // 忽略无法转换的对象 
        objectMapper.configure(SerializationFeature.FAIL_ON_EMPTY_BEANS, false);
        // PrettyPrinter 格式化输出
        objectMapper.configure(SerializationFeature.INDENT_OUTPUT, true);
        // NULL不参与序列化
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

        // 指定时区
        objectMapper.setTimeZone(TimeZone.getTimeZone("GMT+8:00"));
        // 日期类型字符串处理
        objectMapper.setDateFormat(new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT));

        // java8日期日期处理
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        javaTimeModule.addSerializer(LocalDateTime.class, new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        javaTimeModule.addSerializer(LocalDate.class, new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        javaTimeModule.addSerializer(LocalTime.class, new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDateTime.class, new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDate.class, new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        javaTimeModule.addDeserializer(LocalTime.class, new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
        objectMapper.registerModule(javaTimeModule);

        converter.setObjectMapper(objectMapper);
        return converter;
    }
}
```

总结：

- 支持（content-type=application/json）请求中格式为`yyyy-MM-dd HH:mm:ss`的字符串，后台用`@RequestBody`接收，及返回值Date转为`yyyy-MM-dd HH:mm:ss`格式String；
- 支持java8日期api；
- 不支持（content-type=application/json）请求中`yyyy-MM-dd`等类型的字符串转为Date；

以上两种方式为JSON入参的全局化处理，推荐使用方式二，尤其适合大型项目在基础包中全局设置。

## JSON入参及返回值局部差异化处理

场景： 假如全局日期时间处理格式为：`yyyy-MM-dd HH:mm:ss`，但是某个字段要求接收或返回日期`yyyy-MM-dd`。

**方式一**

使用springboot自带的注解`@JsonFormat(pattern = "yyyy-MM-dd")`，如下所示：

```java
@JsonFormat(pattern = "yyyy-MM-dd", timezone="GMT+8")
private Date releaseDate;
```

**点评：** springboot默认提供，功能强大，满足常见场景使用，并可指定时区。

**方式二**

自定义日期序列化与反序列化，如下所示：

```java
/**
 * 日期序列化
 */
public class DateJsonSerializer extends JsonSerializer<Date> {
    @Override
    public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        jsonGenerator.writeString(dateFormat.format(date));
    }
}

/**
 * 日期反序列化
 */
public class DateJsonDeserializer extends JsonDeserializer<Date> {
    @Override
    public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException {
        try {
            SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
            return dateFormat.parse(jsonParser.getText());
        } catch (ParseException e) {
            throw new RuntimeException(e);
        }
    }
}

/**
 * 使用方式
 */
@JsonSerialize(using = DateJsonSerializer.class)
@JsonDeserialize(using = DateJsonDeserializer.class)
private Date releaseDate;
```

## 日期时间格式化处理方式完整配置

```java
@Configuration
public class DateHandlerConfig {

    /** 默认日期时间格式 */
    public static final String DEFAULT_DATE_TIME_FORMAT = "yyyy-MM-dd HH:mm:ss";
    /** 默认日期格式 */
    public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
    /** 默认时间格式 */
    public static final String DEFAULT_TIME_FORMAT = "HH:mm:ss";

    /**
     * LocalDate转换器，用于转换RequestParam和PathVariable参数
     * `@ConditionalOnBean(name = "requestMappingHandlerAdapter")`: 等requestMappingHandlerAdapter bean注册完成之后
     * 再添加自己的`converter`就不会注册到`FormattingConversionService`中
     */
    @Bean
    @ConditionalOnBean(name = "requestMappingHandlerAdapter")
    public Converter<String, LocalDate> localDateConverter() {
        return source -> LocalDate.parse(source, DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT));
    }

    /**
     * LocalDateTime转换器，用于转换RequestParam和PathVariable参数
     */
    @Bean
    @ConditionalOnBean(name = "requestMappingHandlerAdapter")
    public Converter<String, LocalDateTime> localDateTimeConverter() {
        return source -> LocalDateTime.parse(source, DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT));
    }

    /**
     * LocalTime转换器，用于转换RequestParam和PathVariable参数
     */
    @Bean
    @ConditionalOnBean(name = "requestMappingHandlerAdapter")
    public Converter<String, LocalTime> localTimeConverter() {
        return source -> LocalTime.parse(source, DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT));
    }

    /**
     * Date转换器，用于转换RequestParam和PathVariable参数
     * 这里关于解析各种格式的日期格式采用了 hutool 的日期解析工具类
     */
    @Bean
    public Converter<String, Date> dateConverter() {
        return new Converter<String, Date>() {
            @Override
            public Date convert(String source) {
                return DateUtil.parse(source.trim());
            }
        };
    }

    /**
     * Json序列化和反序列化转换器，用于转换Post请求体中的json以及将我们的对象序列化为返回响应的json
     */
    @Bean
    public ObjectMapper objectMapper(){
        ObjectMapper objectMapper = new ObjectMapper();
        objectMapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        objectMapper.disable(DeserializationFeature.ADJUST_DATES_TO_CONTEXT_TIME_ZONE);

        //LocalDateTime系列序列化和反序列化模块，继承自jsr310，我们在这里修改了日期格式
        JavaTimeModule javaTimeModule = new JavaTimeModule();
        javaTimeModule.addSerializer(LocalDateTime.class,new LocalDateTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        javaTimeModule.addSerializer(LocalDate.class,new LocalDateSerializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        javaTimeModule.addSerializer(LocalTime.class,new LocalTimeSerializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDateTime.class,new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_TIME_FORMAT)));
        javaTimeModule.addDeserializer(LocalDate.class,new LocalDateDeserializer(DateTimeFormatter.ofPattern(DEFAULT_DATE_FORMAT)));
        javaTimeModule.addDeserializer(LocalTime.class,new LocalTimeDeserializer(DateTimeFormatter.ofPattern(DEFAULT_TIME_FORMAT)));


        //Date序列化和反序列化
        javaTimeModule.addSerializer(Date.class, new JsonSerializer<>() {
            @Override
            public void serialize(Date date, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
                SimpleDateFormat formatter = new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT);
                String formattedDate = formatter.format(date);
                jsonGenerator.writeString(formattedDate);
            }
        });
        javaTimeModule.addDeserializer(Date.class, new JsonDeserializer<>() {
            @Override
            public Date deserialize(JsonParser jsonParser, DeserializationContext deserializationContext) throws IOException, JsonProcessingException {
                SimpleDateFormat format = new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT);
                String date = jsonParser.getText();
                try {
                    return format.parse(date);
                } catch (ParseException e) {
                    throw new RuntimeException(e);
                }
            }
        });

        objectMapper.registerModule(javaTimeModule);
        return objectMapper;
    }
}
```

## 扩充源码：深入研究SpringMVC数据绑定过程

> 接下来进入debug模式，看看mvc是如何将我们request中的参数绑定到我们controller层方法入参的；

写一个简单controller，打个断点看看方法调用栈：

```java
    @GetMapping("/getDate")
    public LocalDateTime getDate(@RequestParam LocalDate date,
                                 @RequestParam LocalDateTime dateTime,
                                 @RequestParam Date originalDate) {
        System.out.println(date);
        System.out.println(dateTime);
        System.out.println(originalDate);
        return LocalDateTime.now();
    }
```

调用接口以后，我们看下方法调用栈中一些关键方法：

```cpp
//进入DispatcherServlet
doService:942, DispatcherServlet
//处理请求
doDispatch:1038, DispatcherServlet
//生成调用链（前处理、实际调用方法、后处理）
handle:87, AbstractHandlerMethodAdapter
//反射获取到实际调用方法，准备开始调用
invokeHandlerMethod:895, RequestMappingHandlerAdapter
invokeAndHandle:102, ServletInvocableHandlerMethod
//这里是关键，参数从这里开始获取到
invokeForRequest:142, InvocableHandlerMethod
doInvoke:215, InvocableHandlerMethod
//这个是Java reflect调用，因此一定是在这之前获取到的参数
invoke:566, Method
```

根据上述分析，发现`invokeForRequest:142, InvocableHandlerMethod`这里的代码是用来拿到实际参数的：

```java
    @Nullable
    public Object invokeForRequest(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer,
            Object... providedArgs) throws Exception {
        //这个方法是获取参数的，在这里下个断
        Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
        if (logger.isTraceEnabled()) {
            logger.trace("Arguments: " + Arrays.toString(args));
        }
        //这里开始调用方法
        return doInvoke(args);
    }
```

进入这个方法看看是什么操作：

```java
protected Object[] getMethodArgumentValues(NativeWebRequest request, @Nullable ModelAndViewContainer mavContainer, Object... providedArgs) throws Exception {
    //获取方法参数数组，包含了入参信息，比如类型、泛型等等
    MethodParameter[] parameters = getMethodParameters();
    //这个用来存放一会从request parameter转换的参数
    Object[] args = new Object[parameters.length];
    for (int i = 0; i < parameters.length; i++) {
      MethodParameter parameter = parameters[i];
      parameter.initParameterNameDiscovery(this.parameterNameDiscoverer);
      //这里看起来没啥卵用（providedArgs为空）
      args[i] = resolveProvidedArgument(parameter, providedArgs);
      //这里开始获取到方法实际调用的参数，步进
      if (this.argumentResolvers.supportsParameter(parameter)) {
        //从名字就看出来：参数解析器解析参数
        args[i] = this.argumentResolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
        continue;
      }
    }
    return args;
}
```

进入resolveArgument看看：

```java
public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                              NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {
  //根据方法入参，获取对应的解析器
  HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
  //开始解析参数（把请求中的parameter转为方法的入参）
  return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
}
```

这里根据参数获取相应的参数解析器，看看内部如何获取的：

```java
//遍历，调用supportParameter方法，跟进看看
for (HandlerMethodArgumentResolver methodArgumentResolver : this.argumentResolvers) {
  if (methodArgumentResolver.supportsParameter(parameter)) {
    result = methodArgumentResolver;
    this.argumentResolverCache.put(parameter, result);
    break;
  }
}
```

**这里，遍历参数解析器，查找有没有适合的解析器！那么，有哪些参数解析器呢(我测试的时候有26个)？？？我列出几个重要的看看，是不是很眼熟！！！**

```java
{RequestParamMethodArgumentResolver@7686} 
{PathVariableMethodArgumentResolver@8359} 
{RequestResponseBodyMethodProcessor@8366} 
{RequestPartMethodArgumentResolver@8367} 
```

我们进入最常用的一个解析器看看他的supportsParameter方法，发现就是通过参数注解来获取相应的解析器的。

```java
    public boolean supportsParameter(MethodParameter parameter) {
        //如果参数拥有注解@RequestParam，则走这个分支（知道为什么上文要对RequestParam和Json两种数据区别对待了把）
        if (parameter.hasParameterAnnotation(RequestParam.class)) {
            //这个似乎是对Optional类型的参数进行处理的
            if (Map.class.isAssignableFrom(parameter.nestedIfOptional().getNestedParameterType())) {
                RequestParam requestParam = parameter.getParameterAnnotation(RequestParam.class);
                return (requestParam != null && StringUtils.hasText(requestParam.name()));
            }
            else {
                return true;
            }
        }
        //......
    }
```

**也就是说，对于`@RequestParam`和`@RequestBody`以及`@PathVariable`注解的参数，SpringMVC会使用不同的参数解析器进行数据绑定！**
 **那么，这三种解析器分别使用什么Converter解析参数呢？我们分别进入三种解析器看一看：**
 **首先看下`RequestParamMethodArgumentResolver`发现内部使用WebDataBinder进行数据绑定，底层使用的是ConversionService （也就是我们的Converter注入的地方）**

```java
WebDataBinder binder = binderFactory.createBinder(webRequest, null, namedValueInfo.name);
//通过DataBinder进行数据绑定的
arg = binder.convertIfNecessary(arg, parameter.getParameterType(), parameter);
```

```java
//跟进convertIfNecessary()
public <T> T convertIfNecessary(@Nullable Object value, @Nullable Class<T> requiredType,
                                @Nullable MethodParameter methodParam) throws TypeMismatchException {

  return getTypeConverter().convertIfNecessary(value, requiredType, methodParam);
}
```

```java
//继续跟进，看到了把
ConversionService conversionService = this.propertyEditorRegistry.getConversionService();
if (editor == null && conversionService != null && newValue != null && typeDescriptor != null) {
  TypeDescriptor sourceTypeDesc = TypeDescriptor.forObject(newValue);
  if (conversionService.canConvert(sourceTypeDesc, typeDescriptor)) {
    try {
      return (T) conversionService.convert(newValue, sourceTypeDesc, typeDescriptor);
    }
    catch (ConversionFailedException ex) {
      // fallback to default conversion logic below
      conversionAttemptEx = ex;
    }
  }
}
```

**然后看下`RequestResponseBodyMethodProcessor`发现使用的转换器是HttpMessageConverter类型的：**

```java
//resolveArgument方法内部调用下面进行参数解析
Object arg = readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType());

//step into readWithMessageConverters()，我们看到这里的Converter是HttpMessageConverter
for (HttpMessageConverter<?> converter : this.messageConverters) {
  Class<HttpMessageConverter<?>> converterType = (Class<HttpMessageConverter<?>>) converter.getClass();
  GenericHttpMessageConverter<?> genericConverter =
    (converter instanceof GenericHttpMessageConverter ? (GenericHttpMessageConverter<?>) converter : null);
  if (genericConverter != null ? genericConverter.canRead(targetType, contextClass, contentType) :
      (targetClass != null && converter.canRead(targetClass, contentType))) {
    if (message.hasBody()) {
      HttpInputMessage msgToUse =
        getAdvice().beforeBodyRead(message, parameter, targetType, converterType);
      body = (genericConverter != null ? genericConverter.read(targetType, contextClass, msgToUse) :
              ((HttpMessageConverter<T>) converter).read(targetClass, msgToUse));
      body = getAdvice().afterBodyRead(body, msgToUse, parameter, targetType, converterType);
    }
    else {
      body = getAdvice().handleEmptyBody(null, message, parameter, targetType, converterType);
    }
    break;
  }
}
```

**最后看下`PathVariableMethodArgumentResolver`发现 和RequestParam走的执行路径一致（二者都是继承自AbstractNamedValueMethodArgumentResolver解析器），因此代码就不贴了。**

## 总结

如果要转换request传来的参数到我们指定的类型，根据入参注解要进行区分：

- 如果是RequestBody，那么通过配置ObjectMapper（这个玩意儿会注入到Jackson的HttpMessagConverter里面，即`MappingJackson2HttpMessageConverter`中）来实现Json格式数据的序列化和反序列化；
- 如果是RequestParam或者PathVariable类型的参数，通过配置Converter实现参数转换（这些Converter会注入到ConversionService中）。