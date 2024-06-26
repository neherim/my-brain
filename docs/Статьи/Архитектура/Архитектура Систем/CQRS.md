---
share: true
---


Command and Query Responsibility Segregation (CQRS) - принцип CQRS разделяет назначение методов на запросы на чтение данных и команд на обработку данных, а так же разделение моделей на чтение и на запись.
В отличае от CQRS принцип CQR не настаивает на разделении моделей на чтение и запись.

CQRS говорит, что необходимо разделять модели на запись и на чтение, так как по сути они решают разные задачи и могут развиваться независимо друг от друга.

Модель на запись используется для бизнес логики, модель для чтения используется, чтобы отобразить в UI или вернуть через API.

Гибкость, создаваемая использованием CQRS, позволяет системе лучше развиваться с течением времени и не позволяет командам обновления вызывать конфликты слияния на уровне домена. Тем самым уменьшая [[Статьи/Архитектура/Архитектура Приложений/Coupling|связанность]].

## Способ реализации
Технически CQRS может быть реализован несколькими способами, например:
1. Модель на чтение и на запись смотрят на одну таблицу в БД.
2. Модель на запись при сохранении в БД может обновлять отдельную таблицу - источник для модели на чтения. Сама таблица может быть спроектирована специально для быстрого поиска записей.
3. Модель на чтение хранится в другой БД, оптимизированной для быстрого чтения и поиска. При обновлении модели на запись выпускается событие, которое асинхронно обновляет модель на чтение в отдельной БД. Поскольку при этом обновление происходит не мгновенно нам придется иметь дело с [[Статьи/Архитектура/Архитектура Систем/Eventual consistency|Eventual consistency]].

## Когда использовать?

### Внутри одного приложения
В рамках одного приложения обязательно стоит использовать, если ваша доменная модель достаточно сложна. Самый простой способ это сделать отдельно класс для записи и класс для чтения модели. Основную ценность для нас представляют классы на запись, их мы используем в бизнес логике, они отвечают за целостность данных в нашем приложении. Класс на запись мы будем использовать для отображения на UI или в API.

Например, класс на запись:
```java
public class Account {
  private Integer balance;
  private Set<MoneyReservation> moneyReservations;

  public Integer withdrawReserved(Long operationId) {...}
  public void deposit(Integer amount) {...}
  public void reserveMoney(Long operationId, Integer amount) {...}
  public Optional<MoneyReservation> removeReservation(Long operationId) {...}
}
```

класс на чтение:
```java
public class AccountView {
  private Integer totalBalance;
  private Integer availableBalance;
  private String ownerName;

  public Integer getAvailableBalance() {...}
  public Integer getTotalBalance() {...}
  public String getOwnerName() {...}
}
```
Оба класса смотрят в одни и те же таблицы, но класс `Account` содержит только поля необходимые для выполнения своих функций по переводу денег и закрытию счета. Эти операции не зависят от ФИО владельца счета, поэтому его мы оставляем только в классе на чтение `AccountView`. Оба класса могут развиваться независимо. Так же нет проблем с ленивой загрузкой связей в ORM, так как на UI мы отдаем полностью загруженный из БД объект.

### На уровне системы
Если нужна высокая скорость на чтение и есть возможность использовать для этого более подходящее хранилище данных. 