# Spring
Данная часть справочной документации покрывает все технологии, являющиеся неотъемлемой частью Spring Framework.
В первую очередь, это контейнер Инверсии Управления ([IoC](https://en.wikipedia.org/wiki/Inversion_of_control)) Spring Framework. За его тщательным рассмотрением следует подробное описание технологий аспектно-ориентированного программирования ([AOP](https://en.wikipedia.org/wiki/Aspect-oriented_programming)) Spring. Spring Framework имеет собственную структуру AOP, концептуально легкую в понимании и удовлетворяющую 80% AOP потребностей в корпоративной Java-разработке. Также предоставляется информация о внедрении Spring с AspectJ (в настоящее время самая богатая - "с точки зрения возможностей" - и, безусловно, самая зрелая реализация AOP в корпоративной среде Java).

### 1. The IoC Container
Эта глава посвящена контейнеру Инверсии Управления (IoC).

### 1.1 Введение в Spring IoC контейнер и Beans
В этой главе рассматривается реализация Spring Framework принципа инверсии управления (IoC). IoC также называют внедрением зависимостей (DI). Это процесс, в котором объекты определяют свои зависимости (то есть другие объекты, с которыми они работают) только через аргументы конструктора, аргументы фабричного метода или свойства, установленные для экземпляра объекта после его создания в конструкторе или в фабричном методе. Затем контейнер внедряет эти зависимости при создании bean-компонента. Такой процесс, по сути, является обратным (отсюда и название "Инверсия Управления") самому bean-компоненту, контролирующему создание экземпляров или расположение его зависимостей c помощью прямого построения классов или механизма вроде шаблона Service Locator.

Пакеты [`org.springframework.beans`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/package-summary.html) и [`org.springframework.context`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/package-summary.html) являются основой IoC контейнера Spring Framework. Интерфейс [`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.3.23/javadoc-api/org/springframework/beans/factory/BeanFactory.html) предоставляет расширенный механизм конфигурации, способный управлять любым типом объектов.  [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.23/javadoc-api/org/springframework/context/ApplicationContext.html) наследуется от `BeanFactory` и добавляет:
* Упрощенную интеграцию функций Spring AOP
* Обработчик ресурсов сообщений (для интернационализации) 
* Публикации событий
* Специальные контексты прикладного уровня, такие как `WebApplicationContext` для web-приложений

Короче говоря, `BeanFactory` содержит инструменты для конфигурации и базовые функции, а `ApplicationContext` добавляет больше функций, специфичных для задачи. `ApplicationContext` является надмножеством `BeanFactory`, и в этой главе при описании контейнера IoC Spring будет использоваться именно он. Для получения больших сведений об использовании `BeanFactory` вместо `ApplicationContext` обратитесь к главе, описывающей [`BeanFactory API`](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanfactory).

В Spring объекты, формирующие основу вашего приложения и управляемые контейнером Spring IoC, называются bean-компонентами. Компонент — это объект, который создается, собирается и управляется контейнером Spring IoC. В противном случае bean-компонент — это просто один из многих объектов вашего приложения. Bean-компоненты и зависимости между ними отражаются в метаданных конфигурации, используемых контейнером.

### 1.2. Обзор контейнера
Интерфейс `org.springframework.context.ApplicationContext` представляет контейнер Spring IoC и отвечает за создание, настройку и сборку компонентов. Cчитывая метаданные конфигурации, контейнер получает инструкции о том, какие объекты создавать, настраивать и собирать. Метаданные представлены в формате XML, аннотациях или коде Java. Этот механизм позволяет выражать объекты, составляющие приложение, и все взаимозависимости между ними.

Несколько реализаций интерфейса `ApplicationContext` поставляются с Spring. В автономных приложениях обычно создается экземпляр [`ClassPathXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.23/javadoc-api/org/springframework/context/support/ClassPathXmlApplicationContext.html) или [`FileSystemXmlApplicationContext`](https://docs.spring.io/spring-framework/docs/5.3.23/javadoc-api/org/springframework/context/support/FileSystemXmlApplicationContext.html). Хотя XML был традиционным форматом для определения метаданных конфигурации, вы можете указать контейнеру использовать аннотации или код Java в качестве формата метаданных, предоставив небольшой объем конфигурации XML, чтобы декларативно включить поддержку этих дополнительных форматов метаданных.

В большинстве сценариев приложений явный пользовательский код не требуется для создания одного или нескольких экземпляров контейнера Spring IoC. Например, в сценарии веб-приложения обычно достаточно восьми (или около того) строк стандартного XML-кода веб-дескриптора в файле web.xml приложения (см. [Удобное создание экземпляров `ApplicationContext` для веб-приложений](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#context-create)). Если вы используете [Spring Tools для Eclipse](https://spring.io/tools) (среда разработки на основе Eclipse), вы можете легко создать эту шаблонную конфигурацию несколькими щелчками мыши или нажатиями клавиш.

На следующей диаграмме показано общее представление о том, как работает Spring. Классы вашего приложения объединяются с метаданными конфигурации, так что после создания и инициализации `ApplicationContext` у вас есть полностью настроенная и исполняемая система или приложение.
![[Pasted image 20221112021714.png]]
**Рисунок 1. Контейнер Spring IoC**

### 1.2.1. Метаданные конфигурации
Как показано на предыдущей диаграмме, контейнер Spring IoC использует форму метаданных конфигурации. Метаданные сообщают контейнеру Spring о том, как создавать, настраивать и собирать объекты в вашем приложении.

Метаданные конфигурации традиционно имеют простой и интуитивный XML формат, использование которого преобладает в этой главе
Configuration metadata is traditionally supplied in a simple and intuitive XML format, which is what most of this chapter uses to convey key concepts and features of the Spring IoC container.

XML-based metadata is not the only allowed form of configuration metadata. The Spring IoC container itself is totally decoupled from the format in which this configuration metadata is actually written. These days, many developers choose [Java-based configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java) for their Spring applications.

For information about using other forms of metadata with the Spring container, see:

-   [Annotation-based configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config): Spring 2.5 introduced support for annotation-based configuration metadata.
-   [Java-based configuration](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java): Starting with Spring 3.0, many features provided by the Spring JavaConfig project became part of the core Spring Framework. Thus, you can define beans external to your application classes by using Java rather than XML files. To use these new features, see the [`@Configuration`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html), [`@Bean`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html), [`@Import`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html), and [`@DependsOn`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html) annotations.

Spring configuration consists of at least one and typically more than one bean definition that the container must manage. XML-based configuration metadata configures these beans as `<bean/>` elements inside a top-level `<beans/>` element. Java configuration typically uses `@Bean`-annotated methods within a `@Configuration` class.

These bean definitions correspond to the actual objects that make up your application. Typically, you define service layer objects, data access objects (DAOs), presentation objects such as Struts `Action` instances, infrastructure objects such as Hibernate `SessionFactories`, JMS `Queues`, and so forth. Typically, one does not configure fine-grained domain objects in the container, because it is usually the responsibility of DAOs and business logic to create and load domain objects. However, you can use Spring’s integration with AspectJ to configure objects that have been created outside the control of an IoC container. See [Using AspectJ to dependency-inject domain objects with Spring](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-atconfigurable).

The following example shows the basic structure of XML-based configuration metadata:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="..." class="...">  
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <bean id="..." class="...">
        <!-- collaborators and configuration for this bean go here -->
    </bean>

    <!-- more bean definitions go here -->

</beans>
```

The `id` attribute is a string that identifies the individual bean definition.

The `class` attribute defines the type of the bean and uses the fully qualified classname.

The value of the `id` attribute refers to collaborating objects. The XML for referring to collaborating objects is not shown in this example. See [Dependencies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-dependencies) for more information.