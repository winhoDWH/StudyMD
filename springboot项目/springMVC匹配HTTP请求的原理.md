# SpringMVC接收请求
## 一、 HTTP基础知识
1. 决定返回体Response的类型`Context-type`的是请求中Header的`accept`属性，该属性的值又称为`MediaType`。服务端会根据这个属性决定返回体的类型，如果服务端不支持请求的`accept`属性，将会返回**406 Not Acceptable**
2. 如果`accept`指定了多个`MediaType`,服务端也支持多个`MediaType`，那`Accpet`应同时指定每个类型的权重`QualityValue(q值)`来决定优先级；
   * 默认情况下权重：text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8；如果**不指定权重默认q=1**
3. `Content-Type`必须是具体确定的类型，不能包含*。
4. `SpringMVC`通过自动设置一系列默认的`HttpMessageConverter`，对http请求进行序列化和反序列化的操作（即读写请求体和返回体）；

## 二、 写操作，将返回值写入到返回流中
1. `org.springframework.web.servlet.mvc.method.annotation.AbstractMessageConverterMethodProcessor`类中的``writeWithMessageConverters()``方法去处理这部分；
2. 逻辑：先从Accept中获取所有mediaType类型（MIME），然后根据请求找到配置的Converter（处理请求参数转化的实现类）中适合该mediaType类型的所有mediaType类型（即框架能处理Accept的所有类型）；然后根据**q值和具体程度**来排序， 选择最后输出的mediaType类型,然后选择Converter；
```java
    /**
    *  @param value:controller方法返回的对象；
    *  @param returnType:对方法参数类型的描述，springboot封装的一个类
    *  @param inputMessage:请求输入流
    *  @param outputMessage:响应输出流
    */
    protected <T> void writeWithMessageConverters(@Nullable T value, MethodParameter returnType,
			ServletServerHttpRequest inputMessage, ServletServerHttpResponse outputMessage)
			throws IOException, HttpMediaTypeNotAcceptableException, HttpMessageNotWritableException {
    //将返回类型分成：字符类型、对象类型，分别获取body（controller返回对象）、valueType（返回对象的class）、targetType（返回对象的type）；
    //对于资源类型返回，做特别处理
    if (isResourceType(value, returnType)) {
		.......
    }
    HttpServletRequest request = inputMessage.getServletRequest();
    /**
    * getAcceptableMediaTypes方法说明
    * 从请求的accept中获取mediaType;
    */
	List<MediaType> acceptableTypes = getAcceptableMediaTypes(request);
    /**
    * getProducibleMediaTypes方法说明
    * 根据请求和配置上的Converter查看支持转换的mediaType类型，一个个遍历检查Converter；
    */
	List<MediaType> producibleTypes = getProducibleMediaTypes(request, valueType, targetType);
    if (body != null && producibleTypes.isEmpty()) {
        throw new HttpMessageNotWritableException(
                "No converter found for return value of type: " + valueType);
    }
    /**
    * acceptableTypes：请求中允许的返回类型；producibleTypes：程序支持的返回类型
    * requestedType.isCompatibleWith(producibleType)：判断支持的类型中有哪些是允许的类型；
    * getMostSpecificMediaType方法：找出两个之间"更具体"的类型，更具体的意思可以理解成是：不带星号*，或者找子类；
    */
    List<MediaType> mediaTypesToUse = new ArrayList<>();
    for (MediaType requestedType : acceptableTypes) {
        for (MediaType producibleType : producibleTypes) {
            if (requestedType.isCompatibleWith(producibleType)) {
                mediaTypesToUse.add(getMostSpecificMediaType(requestedType, producibleType));
            }
        }
    }
    //根据Q值和具体程度对类型进行排序，默认所有类型的Q值为1
    MediaType.sortBySpecificityAndQuality(mediaTypesToUse);
    //选择具体默认的返回类型
    for (MediaType mediaType : mediaTypesToUse) {
        if (mediaType.isConcrete()) {
            selectedMediaType = mediaType;
            break;
        }
        else if (mediaType.isPresentIn(ALL_APPLICATION_MEDIA_TYPES)) {
            selectedMediaType = MediaType.APPLICATION_OCTET_STREAM;
            break;
        }
    }
    //根据converter的添加顺序（优先级），找到一个能处理确定返回类型selectedMediaType的converter，并对结果进行处理；
    if (selectedMediaType != null) {
        selectedMediaType = selectedMediaType.removeQualityValue();
        for (HttpMessageConverter<?> converter : this.messageConverters) {
            GenericHttpMessageConverter genericConverter = (converter instanceof GenericHttpMessageConverter ?
                    (GenericHttpMessageConverter<?>) converter : null);
            if (genericConverter != null ?
                    ((GenericHttpMessageConverter) converter).canWrite(targetType, valueType, selectedMediaType) :
                    converter.canWrite(valueType, selectedMediaType)) {
                body = getAdvice().beforeBodyWrite(body, returnType, selectedMediaType,
                        (Class<? extends HttpMessageConverter<?>>) converter.getClass(),
                        inputMessage, outputMessage);
                if (body != null) {
                    Object theBody = body;
                    LogFormatUtils.traceDebug(logger, traceOn ->
                            "Writing [" + LogFormatUtils.formatValue(theBody, !traceOn) + "]");
                    addContentDispositionHeader(inputMessage, outputMessage);
                    if (genericConverter != null) {
                        genericConverter.write(body, targetType, selectedMediaType, outputMessage);
                    }
                    else {
                        ((HttpMessageConverter) converter).write(body, selectedMediaType, outputMessage);
                    }
                }
                else {
                    if (logger.isDebugEnabled()) {
                        logger.debug("Nothing to write: null body");
                    }
                }
                return;
            }
        }
	}
```
3. 原生的fastJsonConverter因为其mediaType是*/*，所以所有类型的反序列化和序列化在框架中都可以调用该Converter；**在写操作中，如果将fastJsonConverter的优先级调到最高，则无论返回是不是application/json，甚至禁用返回类型为application/json，都会调用fastJsonConverter进行处理，返回的都是json格式数据，但content-Type可能是别的类型！**；

## 三、读取操作-请求参数加上@RequestBody的情况
1. 调用了AbstractMessageConverterMethodArgumentResolver类（写操作的父类）的readWithMessageConverters方法
```java
    /**
    * @param inputMessage：请求输入流
    * @param parameter:请求对应controller的方法参数的描述
    * @param targetType：请求对应controller方法参数的类型
    */
    protected <T> Object readWithMessageConverters(HttpInputMessage inputMessage, MethodParameter parameter,Type targetType) throws IOException, HttpMediaTypeNotSupportedException, HttpMessageNotReadableException {
        /**
        * 获取变量contentType：请求的Content-Type，从inputMessage中获取；
        * 如果请求没有Content-Type，则去默认值application/octet-stream，对应：MediaType.APPLICATION_OCTET_STREAM
        */
        /**
        * 下面这段源代码是使用ResolvableType对被请求的controller的方法参数进行提取，然后封装成targetClass
        */
        if (targetClass == null) {
			ResolvableType resolvableType = ResolvableType.forMethodParameter(parameter);
			targetClass = (Class<T>) resolvableType.resolve();
		}
        EmptyBodyCheckingHttpInputMessage message;
		try {
			message = new EmptyBodyCheckingHttpInputMessage(inputMessage);
            //循环访问所有的转换器
			for (HttpMessageConverter<?> converter : this.messageConverters) {
				Class<HttpMessageConverter<?>> converterType = (Class<HttpMessageConverter<?>>) converter.getClass();
				//挑出转换器中实现了GenericHttpMessageConverter该类的转换器
                GenericHttpMessageConverter<?> genericConverter =
						(converter instanceof GenericHttpMessageConverter ? (GenericHttpMessageConverter<?>) converter : null);
                /*
                * 这个if是先对转换器做类型判断，然后再判断该转换器能否读取请求体中的参数；
                * 这里转换器分为两种，一种是实现了GenericHttpMessageConverter类的，用genericConverter变量代表，另一种是只实现了HttpMessageConverter类的转换器；
                * 首先判断是不是实现了GenericHttpMessageConverter类的转换器，是的话，调用其canRead方法；
                * 其次是看被请求的controller方法是否有参数（targetClass != null），有的话，这时候的转换器也是只实现了HttpMessageConverter类的转换器，便调用其canRead方法；
                */
				if (genericConverter != null ? genericConverter.canRead(targetType, contextClass, contentType) :
						(targetClass != null && converter.canRead(targetClass, contentType))) {
					if (message.hasBody()) {
                        //下面都是使用转换器的read（），对请求体进行转换
                        //getAdvice().beforeBodyRead()是在转换前，对请求体进行处理
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
		}
		catch (IOException ex) {
			throw new HttpMessageNotReadableException("I/O error while reading input message", ex, inputMessage);
		}
    }
```
1. 主要处理过程：先获取请求中的`Context-type`属性，得到请求体类型说明`media-Type`；如果没有该属性，则默认为：`application/octet-stream`类型；然后根据`media-Type`选取`HttpMessageConverter`，然后处理请求体；
1. 这里对请求参数的转换器分为两种，一种是实现了`GenericHttpMessageConverter`类的，另一种是只实现了`HttpMessageConverter`类的转换器。

### GenericHttpMessageConverter类

```java
boolean canRead(Type type, Class<?> contextClass, MediaType mediaType);
```

### HttpMessageConverter类
```java
boolean canRead(Class<?> clazz, MediaType mediaType);
```

## 四、 读取操作-请求参数加上@RequestParam
1. 当Controller类的方法参数不带注解时，实际上和加上注解@RequestParam是一样的处理方法，只是@RequestParam可以设置一些约束；
2. 当使用@RequestParam注解和@RequestPart注解时，会调用InvocableHandlerMethod类的getMethodArgumentValues方法，该方法通过for循环遍历每一个controller方法参数，为每个参数寻找一个处理器，然后根据请求进行赋值
   ```java
    /**
     * MethodParameter[] parameters是请求对应的controller方法的参数列表
     * 
    **/
    MethodParameter[] parameters = getMethodParameters();
    Object[] args = new Object[parameters.length];
    for (int i = 0; i < parameters.length; i++) {
        MethodParameter parameter = parameters[i];
        //supportsParameter方法是判断现在已有的参数处理器是否支持处理该类型参数
        if (!this.resolvers.supportsParameter(parameter)) {
            throw new IllegalStateException(formatArgumentError(parameter, "No suitable resolver"));
        }
        try {
            //有合适的处理器，则将参数进行处理
            args[i] = this.resolvers.resolveArgument(parameter, mavContainer, request, this.dataBinderFactory);
        }
        catch (Exception ex) {
            // Leave stack trace for later, exception may actually be resolved and handled...
            if (logger.isDebugEnabled()) {
                String exMsg = ex.getMessage();
                if (exMsg != null && !exMsg.contains(parameter.getExecutable().toGenericString())) {
                    logger.debug(formatArgumentError(parameter, exMsg));
                }
            }
            throw ex;
        }
    }
   ```
3. 进一步查看`this.resolvers.resolveArgument(...)`调用源码
    ```java
    public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
			NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {
        //该方法作用是：从该类的缓存中获取一个能处理当前参数的处理器
        //HandlerMethodArgumentResolver类是所有处理器的基础类
		HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
		if (resolver == null) {
			throw new IllegalArgumentException("Unsupported parameter type [" +
					parameter.getParameterType().getName() + "]. supportsParameter should be called first.");
		}
		return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
	}
    ```

### HandlerMethodArgumentResolver类-请求参数处理器
1. 上面处理器都是继承该类的，该类有两个方法；
2. `boolean supportsParameter(MethodParameter parameter);`用来判断是否适合处理当前方法参数；
3. `Object resolveArgument(....)`:根据请求将方法参数赋值；

### 实体类解释器
ModelAttributeMethodProcessor

### 普通参数解释器
RequestParamMethodArgumentResolver，实际上用的是AbstractNamedValueMethodArgumentResolver

### 处理上传文件
1. 使用的是：`RequestPartMethodArgumentResolver`类，继承了HandlerMethodArgumentResolver类；
2. 当使用from-data上传文件时，请求request会被转换成`StandardMultipartHttpServletRequest`；（DispatcherServlet类调用doDispatch方法时，对请求进行判断，如果请求是带有文件上传性质的就会**调用解析器**转化为`StandardMultipartHttpServletRequest`）
3. 由于已经被转化为`StandardMultipartHttpServletRequest`类型了（转化的时候已经将上传文件封装成`MultipartFile`对象了）；所以只需要将对应名称的`MultipartFile`对象取出赋值到方法参数即可；
4. 所以spring在有上传文件的`controller`时，一般都是使用`MultipartFile`类型作为文件的接收参数；
   ```
   @RequestMapping(value = "testFromData", method = RequestMethod.POST)
    public BaseResponse testFormData(@RequestPart("file")MultipartFile file){
        return new BaseResponse("0", "success");
    }
   ```
   * 上面@RequestPart注解标注文件上传的key名为file；
   * 

## 参考文献
https://segmentfault.com/u/shierye/articles?sort=vote 


https://blog.csdn.net/u012881904/article/details/80813294 理解type等类

https://www.cnblogs.com/FraserYu/p/11343803.html 解析实现过程的

https://blog.csdn.net/as513385/article/details/93512699 加与不加@RequestParam标签

https://www.cnblogs.com/throwable/p/11980555.html SpringMVC请求参数接收总结(一)