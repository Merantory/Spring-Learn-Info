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