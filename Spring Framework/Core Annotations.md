## Обзор

При помощи аннотаций мы можем использовать возможности Spring DI.
Данные аннотации называются Spring core аннотациями и расположены они в следующих пакетах:
org.springframework.beans.factory.annotation и org.springframework.context.annotation packages.

## Аннотации относящиеся к DI механизму:

### @Autowired
Мы можем использовать аннотацию @Autowired,
чтобы пометить зависимость, которую Spring должен внедрить.
Мы можем использовать данную аннотацию на **конструкторах**, **сеттерах** и **полях**.

Внедрение зависимостей на **конструкторе**:
```java
class Car {
    Engine engine;

    @Autowired
    Car(Engine engine) {
        this.engine = engine;
    }
}
```
Внедрение зависимостей на **сеттерах**:
```java
class Car {
Engine engine;

    @Autowired
    void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```
Внедрение зависимостей **на полях**:
```java
class Car {
    @Autowired
    Engine engine;
}
```

@Autowired имеет аргумент _required_, который имеет стандартное значение _true_.
Данный аргумент определяет поведение Spring в случае, если не был найден подходящий _bean_ для внедрения.
В данном случае при значении _true_ будет выброшено исключение, иначе ничего не внедрится.

Подробнее про аннотацию @Autowired


### @Bean
Аннотация @Bean помечает фабричный метод, который создает экземпляр Spring Bean.
```java
@Bean
Engine engine() {
    return new Engine();
}
```
Spring вызывает данный метод, когда ему необходимо получить экземпляр,
тип которого совпадает с возвращаемым значением фабричного метода.

Созданный Bean будет иметь такое же имя, как и название фабричного метода. 
Если нам необходимо, чтобы имя Bean'a было отличным,
мы можем использовать аргументы name или value, чтобы задать собственное название для Bean'a.
На самом деле агрумент value это псевдоним агрумента name и функциональных различий в их использовании нет.
```java
@Bean("engine")
Engine getEngine() {
    return new Engine();
}
```
**Все фабричные Bean методы должны быть расположены в классах, отмеченных аннотацией @Configuration**


### @Qualifier
Мы можем использовать аннотацию @Qualifier совместно с аннотацией @Autowired
для того, чтобы уточнить идентификатор или имя нужного при внедрении Bean'a, когда это может быть необходимо.
Например, данная необходимость может возникнуть в случае неопределенности,
когда Spring не может понять какой конкретно Bean ему стоит внедрить, так как могут подойти два Bean'a

Например, следующие два Bean'a реализуют один и тот же интерфейс:
```java
class Bike implements Vehicle {}

class Car implements Vehicle {}
```
В данном случае, если будет необходимо внедрить bean _Vehicle_, это приведет к неопределённости.
Для разрешения данной проблемы используется аннотаций @Qualifier.

При помощи внедрения зависимостей с помощью **конструктора**:
```java
@Autowired
Biker(@Qualifier("bike") Vehicle vehicle) {
    this.vehicle = vehicle;
}
```
При помощи внедрения зависимостей с помощью **сеттера**:
```java
@Autowired
void setVehicle(@Qualifier("bike") Vehicle vehicle) {
    this.vehicle = vehicle;
}
```
или
```java
@Autowired
@Qualifier("bike")
void setVehicle(Vehicle vehicle) {
    this.vehicle = vehicle;
}
```
При помощи внедрения зависимостей с помощью **поля**:
```java
@Autowired
@Qualifier("bike")
Vehicle vehicle;
```
Подробнее здесь


### @Required 
Аннотация @Required используется на сеттерах, чтобы указать,
что bean свойство должно быть определено в файле XML конфигурации.
Если это не так, будет выброшено исключение - _BeanInitializationException_
```java
@Required
void setColor(String color) {
    this.color = color;
}
```
```xml
<bean class="com.baeldung.annotations.Bike">
    <property name="color" value="green" />
</bean>
```


### @Value
Мы можем внедрять значения в наши Bean'ы используя аннотацию @Value. Это совместимо с конструкторами, сеттарами, полями.

Внедрение при помощи **конструктора**:
```java
Engine(@Value("8") int cylinderCount) {
    this.cylinderCount = cylinderCount;
}
```
Внедрение при помощи **сеттера**:
```java
@Autowired
void setCylinderCount(@Value("8") int cylinderCount) {
    this.cylinderCount = cylinderCount;
}
```
или
```java
@Value("8")
void setCylinderCount(int cylinderCount) {
    this.cylinderCount = cylinderCount;
}
```
Внедрение при помощи **поля**:
```java
@Value("8")
int cylinderCount;
```
Конечно, прямое внедрение статических значений является не очень полезным.
Но мы можем подтянуть значения, которые определены в наших .properties файлах или .yaml файлах.

Предположим, что у нас есть следующий .properties файл:
```properties
engine.fuelType=petrol
```
Тогда мы можем внедрить значение "petrol", которое прописано в engine.fuelType при помощи:
```java
@Value("${engine.fuelType}")
String fuelType;
```
Мы можем использовать @Value также используя SpEL(Spring Expression Language). Об этом подробнее здесь


### @DependsOn
Мы можем использовать данную аннотацию, чтобы указать Spring, чтобы перед подгрузкой Bean'a,
сначала подгрузились Bean'ы, указанные в аннотации, обычно это компоненты, от которых зависит текущий подгружаемый Bean.
Описанное поведение является автоматическим для Spring, но данная аннотация может быть полезна для подгрузки неявных компонентов.
```java
@DependsOn("engine")
class Car implements Vehicle {}
```
```java
@Bean
@DependsOn("fuel")
Engine engine() {
    return new Engine();
}
```


### @Lazy
Мы используем аннотацию @Lazy, когда нам необходима ленивая подгрузка (lazy loading) нашего Bean'a.
По умолчанию Spring создает все одиночные (singleton) Bean'ы в режиме стремительной загрузки (eager loading).
Это означает, что Bean'ы создаются при запуске приложения.
Иногда нам может потребоваться иное поведение, которое может обеспечить аннотация @Lazy.

Аннотация может работать по-разному, в зависимости от того, где мы её разместим:
* Фабричные метода Bean'ов, отмеченные аннотацией @Bean для задержания вызова метода, соответственно и задержки создания Bean'а.
* На уровне класса, помеченного аннотацией @Configuration.
В данном случае будут затронуты все методы класса, которые помечены аннотацией @Bean.
* На уровне класса, помеченного аннотацией @Component, который не является классом конфигурацией. Обеспечивает lazy подгрузку component bean'а.
* На уровне конструктора, сеттера, поля помеченного аннотацией @Autowired. В данном случае внедрение зависимости будет происходить лениво.

Более подробно здесь.


### @Lookup
Метод помеченный аннотацией @Lookup говорит Spring о необходимости возврата экземпляра возвращаемого типа метода при его вызове.

Подробнее здесь.

### @Primary
Данная аннотация необходима, когда нам нужно объявить несколько Bean'ов одного типа и указать явно какой из них должен быть приоритетнее.
Это позволяет избежать путаницы для Spring, так как он на самом деле понятия не имеет какой именно Bean нужно внедрить,
когда у нас есть несколько подходящих кандидатов. Конечно, можно использовать аннотацию @Qualifier,
но это может быть неудобно, когда мы очень часто внедряем один Bean.
Таким образом мы можем пометить аннотацией @Primary самый используемый Bean.

```java
@Component
@Primary
class Car implements Vehicle {}

@Component
class Bike implements Vehicle {}

@Component
class Driver {
    @Autowired
    Vehicle vehicle;
}

@Component
class Biker {
    @Autowired
    @Qualifier("bike")
    Vehicle vehicle;
}
```

В этом примере:
* Car это первичный(primary) вид транспорта.
* В классе Driver будет внедрен объект Car, так как мы явно указали это при помощи аннотации @Primary.
* В классе Biker будет внедрен объект Bike, так как в этом случае мы явно укали внедряемый объект при помощи аннотации @Qualifier.


### @Scope
Мы используем аннотацию @Scope, чтобы указать область скоуп Bean'a.
Область действия Bean-компонента определяет жизненный цикл и видимость этого Bean-компонента в контекстах, в которых мы его используем.

Существуют разные виды скоупов: singleton, prototype, request, session, globalSession и другие.

Пример использования аннотации @Scope:
```java
@Component
@Scope("prototype")
class Engine {}
```

Подробнее про скоупы здесь


## Аннотации конфигурации

### @Profile
Если мы хотим использовать определенный метод отмеченный аннотацией @Component или @Bean
только во время активности какого-либо профиля, мы можем пометить метод аннотацией @Profile, указав его имя.

```java
@Component
@Profile("sportDay")
class Bike implements Vehicle {}
```

Подробнее про профили здесь


### @Import
Мы можем использовать класс помеченный аннотацией @Configuration без использования сканирования компонентов.
Для этого нам нужно указать данный класс в качестве аргумента аннотации @Import.

```java
@Import(VehiclePartSupplier.class)
class VehicleFactoryConfig {}
```

### @ImportResource
Мы можем импортировать XML-конфигурации с помощью этой аннотации.
Мы можем указать расположения XML-файлов с помощью аргумента locations или с помощью его псевдонима, аргумента value:

```java
@Configuration
@ImportResource("classpath:/annotations.xml")
class VehicleFactoryConfig {}
```


### @PropertySource
При помощи данной аннотации мы можем определить файл свойств для настройки нашего приложения.
```java
@Configuration
@PropertySource("classpath:/annotations.properties")
class VehicleFactoryConfig {}
```
С Java 8 внутри аннотации @PropertySource реализуется функционал повторяющихся аннотаций.
Это значит, что мы можем использовать данную аннотацию, чтобы указать несколько файлов свойств.
```java
@Configuration
@PropertySource("classpath:/annotations.properties")
@PropertySource("classpath:/vehicle-factory.properties")
class VehicleFactoryConfig {}
```


### @PropertySources
Мы можем использовать данную аннотацию для указания нескольких файлов свойств @PropertySource.
```java
@Configuration
@PropertySources({ 
    @PropertySource("classpath:/annotations.properties"),
    @PropertySource("classpath:/vehicle-factory.properties")
})
class VehicleFactoryConfig {}
```