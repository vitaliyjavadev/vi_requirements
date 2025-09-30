# repository_layer

Репозиторий -
это [шаблон проектирования](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design#the-repository-pattern) (
не GoF), который предлагает абстракции по работе с данными доменных моделей.
Работа с данными предполагает CRUD (Create, Read, Update, Delete) операции.
Большинство проектов используют этот шаблон. 

При использовании этого шаблона нужно соблюдать следующие правила:

1. **Положение репозиториев.** Классы-репозитории должны располагаться в пакете
   `{artifact_id}.<project>.repository`
2. **Названия классов.** Префикс берется от пакета, например, `UserRepository`, а не
   `UserDbStore`;
3. **Соблюдение SOLID.** Особенно стоит учитывать принципы ниже:
    1. **SRP**. Класс репозиторий должен работать со своей доменной моделью.
       Например, `MovieRepository` не должен заниматься
       хранением `Session` модели;
    2. **OCP**. Класс репозиторий должен реализовывать интерфейс. Например,
       `JdbcUserRepository` должен реализовывать `UserRepository`;
    3. **DIP**. Класс репозиторий должен зависеть от абстракций, а не
       реализаций.
       Неправильный код:
       ```java
       public JdbcTicketRepository(BasicDataSource dataSource) {
            this.dataSource = dataSource;
       }
       ```
       Правильный код:
       ```java
       public JdbcTicketRepository(DataSource dataSource) {
            this.dataSource = dataSource;
       }
       ```
4. Название методов. Названия методов должны
   соответствовать [спецификации Spring JPA](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#appendix.query.method.subject).
   Не все проекты работают с Spring JPA, тем не менее, эта спецификация
   описывает наиболее часто используемые
   слова, которые позволяют с легкостью понять, что метод делает, без
   необходимости заглядывать в реализацию.
