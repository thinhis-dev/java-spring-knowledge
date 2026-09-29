# Quiz 05: Spring Core and Spring Boot

Level: 5 (medium). Covers: Spring lessons 01 and 02. Time: about 60 minutes.

Write your answer under each `Answer:` line. Do not open the lessons.

## Q1

Explain IoC and DI to an intern in three sentences or fewer.

Answer:

## Q2

What is the difference between Spring and Spring Boot? Name four things that Boot adds.

Answer:

## Q3

Compare constructor, setter, and field injection. Why is constructor injection the default choice?

Answer:

## Q4

What is the difference between `@Component` and `@Bean`? When do you use `@Bean`?

Answer:

## Q5

Two classes implement `NotificationSender`: `EmailSender` and `SmsSender`. `AlertService` needs one of them in its constructor. What error do you get at startup? Give three ways to fix it.

Answer:

## Q6

Name the bean scopes. What is the default? What goes wrong with a `prototype` bean injected into a singleton?

Answer:

## Q7

What three annotations does `@SpringBootApplication` combine? A new `@Service` is in `com.example.other`, and the main class is in `com.example.shop`. What happens?

Answer:

## Q8

Explain how auto-configuration decides to create a `DataSource` bean. What happens to it after you define your own `DataSource` bean?

Answer:

## Q9

Write a `@ConfigurationProperties` record for this YAML. Make `baseUrl` required.

```yaml
app:
  mail:
    base-url: https://mail.example.com
    timeout: 3s
    max-retries: 2
```

Answer:

## Q10

`server.port` is 8080 in `application.yml`, the environment has `SERVER_PORT=9000`, and the app starts with `--server.port=7000`. Which port does it use, and why?

Answer:

## Q11

What is a Spring proxy? Why does `@Transactional` not work in this code?

```java
@Service
public class ImportService {
    public void importAll(List<Row> rows) {
        for (Row r : rows) importOne(r);
    }

    @Transactional
    public void importOne(Row r) { /* ... */ }
}
```

Answer:

## Q12

Your Kotlin project uses the `kotlin-spring` (all-open) compiler plugin. Why? Does a Java project need it?

Answer:

## Q13

A needs B in its constructor, and B needs A in its constructor. What happens at startup? What is the correct fix?

Answer:

## Q14

Name three rules for the controller, service, and repository layers.

Answer:

## Q15 (interviewer view)

An intern writes `@Autowired` on every private field. Write the question that you ask, and the answer that shows understanding.

Answer:
