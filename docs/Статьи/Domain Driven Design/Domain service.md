---
share: true
alias: доменный сервис
---


Одна из концепций [[Статьи/Domain Driven Design/Domain Driven Design|DDD]]. Класс с доменной логикой, который при этом не содержит состояния. Используется для изоляции доменной логики там, где ее нельзя поместить внутрь какого-либо доменного объекта. Повышает удобство тестирования этой логики, по этой причине не должен содержать вызовов внешних систем или обращение к IO.