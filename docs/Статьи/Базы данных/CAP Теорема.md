---
share: true
---

В распределенных системах есть три компромиссных по отношению друг к другу свойства: согласованность, доступность и терпимость к разделению. При отказе удается сохранить только два из них.
Согласованность является характеристикой системы, означающей, что при обращении к нескольким узлам будет получен один и тот же ответ. Доступность означает, что на каждый запрос будет получен ответ. Терпимость к разделению является способностью системы справляться с тем фактом, что установить связь между ее частями порой становится невозможно.

## Ссылки
- [CAP-теорема простым, доступным языком / Хабр](https://habr.com/ru/post/130577/)
- [Eventually Consistent - Revisited](https://www.allthingsdistributed.com/2008/12/eventually_consistent.html)
- [Consistency Models](https://jepsen.io/consistency)