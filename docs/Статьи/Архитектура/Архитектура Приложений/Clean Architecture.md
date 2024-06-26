---
share: true
aliases:
  - Onion Architecture
  - Hexogonal Architecture
  - Ports And Adapters
---


Метод построения [[Статьи/Архитектура/Архитектура|архитектуры]] приложений, в котором мы пытаемся максимально абстрагировать бизнес логику от деталей реализации вроде БД, UI, внешних сервисов, фреймворков. Также известна как Onion Architecture или Hexogonal Architecture, Ports And Adapters.

![500](attachments/clean_arch.png)

## Слои
В чистой архитектуре выделяют следующие слои:
1. Presentation Layer (User Interface Layer)
   Слой представления. Отвечает за интерфейс взаимодействия с пользователем.   
2. Application Layer (Service Layer)
   Слой сервисов. Этот слой должен оставаться тонким, он не содержит бизнес правил а только координирует работу других слоев для выполнения бизнес функции. Он не имеет состояния, отражающего бизнес-ситуацию, но может иметь состояние, отражающее ход выполнения задачи для пользователя или программы.
3. Domain Layer (Model Layer/Business Logic Layer)
   Доменный слой. Отвечает за описание бизнес правил и состояния. На этом слое находится самый важный код приложения - бизнес логика.
4. Infrastructure Layer (Data access layer)
   Слой инфраструктуры. HTTP эндпоинты, внешний API, интеграция с очередями, взаимодействие с базой данных.
   
## Отличия от других типов архитектуры
Основные отличия от [[Статьи/Архитектура/Архитектура Приложений/Layered Architecture|слоистой архитектуры]]:
1. В слоистой архитектуре доступ к БД происходит из бизнес логики. В чистой архитектуре слой с бизнес логикой находится в центре приложения и не имеет прямого доступа к слою доступа к данным.
2. Используется Dependency inversion principle для доступа из внутренних слоев к внешним.

## Application Service
Это классы внутри Application layer. После того как приняли запрос от внешней системы в слой *Infrastructure* (Например, Spring Controller или Kafka Listener), вызываем метод бизнес логики из *Applicatio Service* в слое *Application*. Сервис чаще всего стартует транзакцию в БД, проводит бизнес валидацию запроса, загружает [[Статьи/Domain Driven Design/Агрегат|агрегат]] из БД через репозиторий. Далее вызывает необходимый метод загруженного агрегата, который принадлежит *Domain Layer*. Методы агрегата меняют его внутреннее состояние. Далее сервис сохраняет агрегат в БД и производит дополнительные действия, например публикация событий в очереди, если необходимо. 

Пример:
```java
@Service
@RequiredArgsConstructor
public class MoneyTransferService {
    private final AccountRepository accountRepository;

    @Transactional
    public void reserveMoney(Long operationId, Long accountId, Integer amount) {
        var account = accountRepository.getByIdWithLock(accountId);
        account.reserveMoney(operationId, amount);
    }

```

## Domain Service
Помимо *Application Service*, иногда стоит использовать *Domain Service*. И Domain и Application сервисы это классы без состояния, которые находятся над агрегатами, но при этом *Domain Service* содержат бизнес логику, и в отличии от *Application Service*, и не должны содержать IO зависимостей.
Domain Service содержит бизнес логику, которая либо не принадлежит конкретному агрегату, либо объединяет несколько агрегатов. Его имеет смысл отделять от *Application Service* если мы хотим легко протестировать эту логику.

## Направление зависимостей
Главный принцип - строгое направление зависимостей от внешнего слоя к внутреннему. Доменный слой не должен содержать никаких прямых вызовов внешних сервисов или БД. Слой инфраструктуры не может обойти слой сервисов и вызывать напрямую методы агрегата.
Если, например, слой application хочет сделать вызов внешнего API, то он должен использовать dependency inversion principle. То есть в пакете application создаем интерфейс, представляющий внешний API. А уже имплементация этого интерфейса будет в пакете infrastructure.
Нормально работают схемы, при которых соединяют слои Infrastructure и Application.

## Разделение модели на запись и чтение
[[Статьи/Domain Driven Design/Агрегат|Агрегаты]] используются только для модификации данных, для чтений стоит использовать отдельные DTO классы, в которые загружать только необходимые данные, используя паттерн [[Статьи/Архитектура/Архитектура Систем/CQRS|CQRS]]. Это позволит сделать код более оптимальным, избежать лишних блокировок и не загрязнять агрегат лишними данными, не нужными для бизнес логики.

Для повышения качества и надежности доменной модели необходимо всеми силами добиваться [[Статьи/Архитектура/Архитектура Приложений/Domain model purity|Domain model purity]]. Это самая важная цель всего подхода. Не важно в каком стиле написана программа ФП или ООП, необходимо стремиться к тому, чтобы бизнес логика была отделена от IO. Это позволит ее качественно протестировать большим количеством быстрых Unit test'ов. И значительно повысит читабельность кода, а значит и уменьшит стоимость поддержи такого кода.

## Тестирование
Основная часть бизнес логики, которую мы хотим покрыть тестами, находится в чистом доменном слое, на нее пишем юнит тесты. Слой *Application* содержит чаще всего только интеграционный "клей" и нуждается скорее в интеграционных или end-to-end тестах.

## Когда использовать?
Как и [[Статьи/Domain Driven Design/Domain Driven Design|DDD]] чистую архитектуру имеет смысл использовать только если приложение содержит сложную доменную логику. Иначе принесет больше вреда чем пользы.

## Ссылки
- [Vertical Slice Architecture, not Layers! - YouTube](https://www.youtube.com/watch?v=L2Wnq0ChAIA&ab_channel=CodeOpinion)
- [DevoxxUA 2021. Victor Rentea. A Clean, Pragmatic Architecture - YouTube](https://www.youtube.com/watch?v=fnl-w1Jwgys&ab_channel=DevoxxUkraine)
- [Clean Coder Blog](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- [Clean Architecture: A Craftsman's Guide to Software Structure and Design: Books](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)
- [Domain services vs Application services · Enterprise Craftsmanship](https://enterprisecraftsmanship.com/posts/domain-vs-application-services/)
- [Проектирование Сервисного Слоя и Логики Приложения — @emacsway's blog](https://emacsway.github.io/ru/service-layer/)
