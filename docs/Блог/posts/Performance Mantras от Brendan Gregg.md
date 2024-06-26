---
share: true
categories:
  - common
slug: perf
date: 2023-11-07
---

Представим, что мы для разработали сервис, который проверяет документы на наличие в них запрещенных слов. Но нагрузочные тесты показали, что он слишком медленно работает. Как выбрать стратегию для оптимизации производительности? С чего начать?
<!-- more -->

Сначала определим что делает наш сервис. Допустим он принимает документ из Kafka, раскладывает по таблицам в БД, затем сканирует на наличие запрещенных слов и возвращает результат клиенту. После профилирования выясняем, что больше всего на [[Статьи/Архитектура/Latency|latency]] влияет сохранение документов в БД.

Наверное на этом моменте кто-то уже побежал просить сервер помощнее или выбирать NoSQL базу на замену PostgreSQL. Но есть и другой подход, более инженерный. Например, [Performance Analysis Methodology](https://www.brendangregg.com/methodology.html) которую описал Брэндан Грегг в своем блоге.

Узкое место в производительности нашего сервиса мы уже обнаружили, теперь давайте подумаем, что можно с ним сделать. Для этого применим одну из техник Брендана под названием **Performance Mantras**. 
Итак, какие у нас есть варианты решения проблемы производительности:
1. Don't do it
2. Do it, but don't do it again
3. Do it less
4. Do it later
5. Do it when they're not looking
6. Do it concurrently
7. Do it cheaper

>[!info]
>**Brendan Gregg** человек очень известный в узких кругах. Работал перфоманс инженером в топовых компания вроде Oracle, Netflix, Intel. Изобретатель такой штуки как [flame graph](https://www.brendangregg.com/flamegraphs.html), автор книг по перфомансу и замечательного [блога](https://www.brendangregg.com/index.html)

---

Представим, как может выглядеть разговор с заказчиком в нашем примере:
\- Можем ли мы вообще не сохранять входящие документы? Это позволит сильно улучшить производительность. *(Don't do it)*
\- Не можем, пользователи хотят видеть в UI эти документы. Все входящие документы нужно сохранять.

\- Но эти документы уже сохранены в сервисе отправителе, можем ли мы использовать их API для вывода в UI? *(Do it, but don't do it again)*
\- Нет, в системе отправителе нет подходящего API для поиска и фильтрации по этим документам.

\- Нам точно нужно сохранять все входящие документы? Можем ли ограничится только документами с найденными совпадениями? Или хотя бы сохранять не весь документ целиком, а только его часть? Ведь для вывода в UI используется только часть его полей. *(Do it less)*
\- Нам точно нужны абсолютно все проверенные документы в БД со всеми полями. Да для списка документов все поля не нужны, но мы планируем добавить в UI полную карточку со всеми полями.

\- Хорошо, мы можем сделать 2 независимых слушателя топика входящих документов, один их только проверят, а второй, со своей скоростью, сохраняет в БД. Решение будет [[Статьи/Архитектура/Архитектура Систем/Eventual consistency|eventual consistency]], это допустимо? *(Do it when they're not looking)*
\- Мы не можем этого допустить. Документ должен попасть в БД строго в одной транзакции с результатом проверки.

\- Все ясно, тогда мы постараемся сделать сохранение документа в БД и его проверку в двух параллельных потоках в рамках обработки одного запроса *(Do it concurrently)*. А если не поможет, то рассмотрим вариант замены БД на более подходящую для нагрузок частой записи *(Do it cheaper)*.

---

Прелесть этих "мантр" еще и в порядке, в котором они расположены. Сначала идут самые дешевые для реализации и при этом эффективные подходы. Заметьте, что чем дальше продвигался диалог, тем наши варианты становились сложнее в реализации и менее перспективными в плане ожидаемой производительности.


Какой общий вывод, на мой взгляд, можно сделать из этих "мантр"?
- Иногда перед началом оптимизаций стоит поговорить с бизнесом. Чем он готов пожертвовать ради производительности? Это поможет сузить количество доступных вариантов оптимизаций.  

- Старайтесь сначала рассматривать самые простые и дешевые варианты оптимизации. И только если они совсем не подходят, тогда пускайтесь во все тяжкие. 
  