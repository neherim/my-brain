---
share: true
alias: связанный контекст
---


Связанный контекст - это граница внутри [[Статьи/Domain Driven Design/Бизнес домен|домена]] в котором поддерживается [[Статьи/Domain Driven Design/Ubiquitous language|единый язык]] и применяется конкретная [[Статьи/Domain Driven Design/Доменная модель|доменная модель]].

## Отличие от поддомена
Часто границы поддоменов совпадают с В отличие от [[Статьи/Domain Driven Design/Subdomain|поддомена]] границы связанного контекста формируются разработчиками. Связанный контекст может содержать в себе несколько поддоменов, поддомен может содержать несколько связанных контекстов.

## Пример
Например, объект физического мира *книга* означает в разных связанных контекстах разные вещи:
 * *Catalog:* обложка, название, автор, объем, формат
 * *Shipping:* вес, размеры, запрет на доставку в страны
 * *Prices:* цена, скидки
 * *Recommendation:* книги, которые часто покупают вместе с этой
 * *Reviews:* оценки и отзывы
 В разных контекстах нам важны разные свойства *книги*. Для работника склада абсолютно все равно как называется книга или кто ее автор, ему важен вес и размеры.

## Зачем нужно выделять связанные контексты?
Позволяют моделировать различные аспекты проблемы не контактируя с другими частями бизнеса. В примере выше, выделив модуль *Shipping* в отдельный связанный контекст мы можем моделировать в нем объект *книга* игнорируя все ее свойства, кроме тех, что интересны в рамках нашего контекста. Если мы этого не сделаем, то можем прийти к антипаттерну God Object, при котором объект *книга* превратится в свалку сотен никак не связанных друг с другом свойств.

## Способы определения
 - Следить за словами и выражениями, которые используют доменные эксперты. Разные контексты имеют разные словари. В разных связанных контекстах часто одно и то же слово означает разные вещи или одна вещь имеет два разных названия.
   Например, клиента депозитария аналитик, отвечающий за онбординг клиентов, будет называть "физическим лицом". А аналитик депозитарной системы будет называть его же словом "депонент".
 - Если одни и те же данные нужны в разных контекстах - это тревожный звонок, что границы были выбраны неправильно.
 - Разные доменные эксперты. В сложных областях есть несколько экспертов, которые знают каждый про свою область. Эти "области" могут и оказаться связанными контекстами.
 - Иногда каждый большой шаг бизнес процесса и есть связанный контекст.
