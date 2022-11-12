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

Метаданные конфигурации традиционно имеют простой и интуитивный XML формат, использующийся в этой главе, чтобы передать основные идеи и особенности Spring IoC контейнера.

> XML - не единственное допустимое представление метаданных конфигурации. Spring IoC контейнер не зависит от формата, в котором они представлены. В настоящее время многие разработчики используют [конфигурацию на основе Java](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java) в своих Spring приложениях.

Для получения информации об использовании других форм метаданных с контейнером Spring смотрите:
* [Конфигурация на основе аннотаций](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-annotation-config): Spring 2.5 предоставил поддержку метаданных конфигурации на основе аннотаций.
* [Конфигурация на основе Java](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java): Начиная с Spring 3.0, многие функции, предоставляемые проектом Spring JavaConfig, стали частью ядра Spring Framework.

Таким образом, вы можете определить bean-компоненты, внешние к классам вашего приложения, используя Java вместо файлов XML. Чтобы использовать эти новые функции, ознакомьтесь с [`@Configuration`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Configuration.html), [`@Bean`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Bean.html), [`@Import`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/Import.html), и [`@DependsOn`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/DependsOn.html) аннотациями.

Конфигурация Spring состоит как минимум из одного и, как правило, более одного определения bean-компонента, которым должен управлять контейнер. XML-метаданные этого компонента представляют из себя дочерние `<bean/>` элементы внутри родительского `<beans/>` элемента. Конфигурация Java обычно использует методы с аннотациями `@Bean` в классе `@Configuration`.

Эти определения bean-компонентов соответствуют реальным объектам, из которых состоит ваше приложение. Как правило, вы определяете объекты сервисного уровня (service objects), объекты доступа к данным (data access objects, DAOs), объекты представления, такие как экземпляры Struts `Action`, объекты инфраструктуры вроде Hibernate `SessionFactories` , JMS `Queues`, и т.д. Обычно, 

Как правило, fine-grained объекты домена в контейнере не настраиваются, поскольку за их создание и загрузку обычно отвечают DAO и бизнес-логика. Однако вы можете использовать интеграцию Spring с AspectJ для настройки объектов, которые были созданы вне контроля контейнера IoC. [См. Использование AspectJ для внедрения зависимостей в объекты домена с помощью Spring](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#aop-atconfigurable).

В следующем примере показана базовая структура метаданных конфигурации на основе XML:

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

1) Атрибут `id`  - строка-идентификатор, сопоставляющая определение с конкретным bean-компонентом.
2) Атрибут `class` определяет тип bean-компонента и содержит полное имя класса.

Значение `id` атрибута относится к взаимодействующим объектам. XML, не показанным в данном примере. См. [Dependencies](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-dependencies) для получения дополнительных сведений.