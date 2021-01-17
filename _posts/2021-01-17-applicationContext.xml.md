---
layout: post
title: "applicationContext.xml/dispatcher-servlet.xml"
subtitle: "applicationContext.xml, dispatcher-servlet.xml에 대해 알아보자."
date: 2021-01-17 18:00:00 +0900
background: '/img/posts/01.jpg'
categories: ['BackEnd','Spring']
---
# applicationContext 이란?
- Root Context이자 스프링에 의해 생성되는 Bean에 대한 Spring IoC Container입니다.
- BeanFactory를 상속받는 Context
- 여러 Servlet에서 공통으로 사용할 Bean을 등록하는 Context입니다.
- @Transactional 으로 트랜잭션을 이용해야할 때 ApplicationContext에 있는 Service에서만 트랜잭션이 정상 작동합니다.
- 보통 데이터 베이스와 관련한 설정이 들어가며, 모든 서블릿과 필터에 우선적으로 필요한 내용을 포함한다

~~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="userBean" class="com.example.diexam01.UserBean"></bean>
	<bean id="e" class="com.example.diexam01.Engine"></bean>
	<bean id="c" class="com.example.diexam01.Car">
		<property name="engine" ref="e"></property>
	</bean>
</beans>
~~~~

# DispatcherServlet 이란?
DispatcherServlet은 서버로 들어오는 모든 요청을 받아서 처리하고 공통처리 작업을 처리한 후, 이외의 작업은 적절한 세부 Controller에게, 예외 발생 시 일괄적인 방식으로 에러를 처리해주는 프론트 컨트롤러이다.

~~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/mvc
       http://www.springframework.org/schema/mvc/spring-mvc.xsd
       http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context.xsd" >

    <mvc:annotation-driven></mvc:annotation-driven>
    <context:component-scan base-package="controller"></context:component-scan>

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>

</beans>
~~~~

- `<beans xmlsns=>`관련 : 아래의 bean에서 사용하기 위한 클래스를 사용하기 위해 명시해주는것.
- `<mvc:annotation-driven>` : Annotation을 활용할 때 기본적인 Default 방식을 설정해준다
- `<context:component-scan>` : 의존성 주입을 위해 해당 클래스 모두를 bean으로 등록해야 하는데, `<context:component-scan>`을 통해 자동으로 처리하게 해 주는 것
    - `base-package="controller"` : controller으로 시작하는 패키지 모두를 등록하겠다는 것
- `View Resolver` : 컨트롤러는 최종적으로 결과를 출력할 뷰와 뷰에 전달한 객체를 담고 있는 ModelAndView 객체를 리턴한다. DispatcherServlet은 ViewResolver를 통해 결과를 출력할 View 객체를 구하고, 해당 객체를 이용하여 내용을 생성한다.
![image](https://user-images.githubusercontent.com/46861704/104835273-e2766900-58e8-11eb-936d-e0cf166d6c26.png)


# Reference
> - [기본기를 쌓는 정아마추어 코딩블로그](https://jeong-pro.tistory.com/222)
> - [Spring Bean 생성 및 사용](https://lazymankook.tistory.com/67)
> - [spring > dispatcher-servlet](http://www.incodom.kr/spring/dispatcher-servlet)
> - [[Spring] Dispatcher-Servlet 설정](https://mangkyu.tistory.com/13)
> - [[Spring] 스프링 기초 - ViewResolver 설정](https://themach.tistory.com/107)