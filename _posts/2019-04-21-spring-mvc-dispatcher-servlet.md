---
layout: post
title: "[Spring MVC] DispatcherServlet에 대해서 알아보자"
date: 2019-04-21 12:30:30
category: 
 
- Spring
tags: 
- Spring
redirect_from: 
- 2019/04/21/spring-mvc-dispather-servlet/
---


> DispatcherSevlet의 기본전략과 여러가지 유형에 대해서 알아보자.

### DispatcherServlet 기본전략 

```java
protected void initStrategies(ApplicationContext context) {
		initMultipartResolver(context);
		initLocaleResolver(context);
		initThemeResolver(context);
		initHandlerMappings(context);
		initHandlerAdapters(context);
		initHandlerExceptionResolvers(context);
		initRequestToViewNameTranslator(context);
		initViewResolvers(context);
		initFlashMapManager(context);
	}
```

DispatcherServlet 의 초기화 전략은 다음과 같다. 초기화 구조는 거의다 비슷하기 때문에 여기에서는 `initViewResolvers(context)` 를 살펴보자. 

```java
	/**
	 * Initialize the ViewResolvers used by this class.
	 * <p>If no ViewResolver beans are defined in the BeanFactory for this
	 * namespace, we default to InternalResourceViewResolver.
	 */
	private void initViewResolvers(ApplicationContext context) {
		this.viewResolvers = null;

		if (this.detectAllViewResolvers) {
			// Find all ViewResolvers in the ApplicationContext, including ancestor contexts.
			Map<String, ViewResolver> matchingBeans =
					BeanFactoryUtils.beansOfTypeIncludingAncestors(context, ViewResolver.class, true, false);
			if (!matchingBeans.isEmpty()) {
				this.viewResolvers = new ArrayList<>(matchingBeans.values());
				// We keep ViewResolvers in sorted order.
				AnnotationAwareOrderComparator.sort(this.viewResolvers);
			}
		}
		else {
			try {
				ViewResolver vr = context.getBean(VIEW_RESOLVER_BEAN_NAME, ViewResolver.class);
				this.viewResolvers = Collections.singletonList(vr);
			}
			catch (NoSuchBeanDefinitionException ex) {
				// Ignore, we'll add a default ViewResolver later.
			}
		}

		// Ensure we have at least one ViewResolver, by registering
		// a default ViewResolver if no other resolvers are found.
		if (this.viewResolvers == null) {
			this.viewResolvers = getDefaultStrategies(context, ViewResolver.class);
			if (logger.isTraceEnabled()) {
				logger.trace("No ViewResolvers declared for servlet '" + getServletName() +
						"': using default strategies from DispatcherServlet.properties");
			}
		}
	}
```

크게 `this.detectAllViewResolvers` 의 boolean 값(default = true) 에 의해서 viewReoslvers 들을 찾는다. 그리고 만약에 `this.viewResolvers` 가 null 이면 `getDefaultStrageies()` default 전략으로 viewResolver를 찾는다. 



아래는 `DispacherServlet.properties` 에 정의되어 있는 기본전략이다.

```properties
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter

org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

여기에서 `ViewSolver` 의 기본 전략은 `InternalResourceViewResovler` 이다





### 다양한 DispatcherServlet 전략들



#### MultipartResolver

- 파일 업로드 요청 처리에 필요한 인터페이스
- HttpServletRequest를 MultipartHttpServletRequest로 변환해주어 요청이 담고 있는 File을 꺼낼 수 있는 API 제공.



#### LocaleResolver

- 클라이언트의 위치(Locale) 정보를 파악하는 인터페이스
- 기본 전략은 요청의 accept-language를 보고 판단.



#### ThemeResolver

- 애플리케이션에 설정된 테마를 파악하고  변경할 수 있는 인터페이스
- 참고: <https://memorynotfound.com/spring-mvc-theme-switcher-example/>



#### HandlerMapping

- 요청을 처리할 핸들러를 찾는 인터페이스



#### HandlerAdapter

- HandlerMapping이 찾아낸 “핸들러”를 처리하는 인터페이스



#### HandlerExceptionResolver

- 요청 처리 중에 발생한 에러 처리하는 인터페이스



#### RequestToViewNameTranslator

- 핸들러에서 뷰 이름을 명시적으로 리턴하지 않은 경우, 요청을 기반으로 뷰 이름을 판단하는 인터페이스

```java
		@GetMapping("/sample")
    public String sample() {
        return "sample";
    }
		
		//아무것도 리턴을 void로 해도, /sample 요청에 의해서 viewName으로 바꿔준다.
		@GetMapping("/sample")
    public void sample() {}

```





#### ViewResolver

- 뷰 이름(string)에 해당하는 뷰를 찾아내는 인터페이스



#### FlashMapManager

- 폼정보를 submit하고 POST 요청을 했을 때 -> GET 요청으로 화면을 리다이렉트 시키는 목적
  여기에서 브라우저 화면을 refresh 했을때, 다시 form 정보로 보여지지 않게 하기 위함 !!

