---
share: true
alias: ESB
---


Архитектурный паттерн построения интеграции между приложениями через отдельный центральный компонент. Который производит трансформацию данных, роутинг сообщений, композицию сообщений и реализует различные коммуникационные протоколы.

В настоящий момент считается антипаттерном.

1. На ESB часто просачивается бизнес логика из систем, которую не успевают сделать команды самих систем.
2. Команда, обслуживающая интеграцию, становится узким местом. Обладая своим бэклогом и релизным циклом остальным системам приходится под них подстраиваться.
3. Интеграционная команда не обладает знаниями домена систем, которые интегрирует, поэтому не могут качественно спроектировать интеграцию.
4. Непрозрачное бюджетирование. Бизнес-линии сливаются в «общий котёл» интеграции без видимых результатов для «своего» бизнеса.

Если попытаться отказаться от ESB просто переписав всю интеграцию на микросервисы, то это не решит ни одну из трех проблем. Кажется более перспективным доверить интеграцию командам самих систем. Чтобы упорядочить и упростить интеграцию между системами, можно придумать стандарты или общие сервисы/платформу интеграции.

## Ссылки
* [Microservices - Smart endpoints and dumb pipes](https://martinfowler.com/articles/microservices.html#SmartEndpointsAndDumbPipes)
- [ArchDays 2019 • Эволюция корпоративной интеграции • Андрей Шиллинг - YouTube](https://www.youtube.com/watch?v=uG08poIN-wU&ab_channel=%D0%9A%D0%BE%D0%BD%D1%84%D0%B5%D1%80%D0%B5%D0%BD%D1%86%D0%B8%D1%8FArchDays)
- [Доменно-ориентированный подход к интеграции](https://systems.education/integration-with-ddd)
