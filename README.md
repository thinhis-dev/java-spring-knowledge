# Java and Spring Preparation for Intern Interviews

Created: 2026-09-29. Target: Java 17, Spring Boot 3.x. Budget: 1 hour per day for about 4 weeks.

You are a Kotlin and Spring Boot developer who will interview Java interns at the end of November. This course skips data types and variables. It focuses on OOP, Java idioms that differ from Kotlin, and REST API best practice with Spring.

## Contents

- [Setup](#setup)
- [Schedule](#schedule)
- [Course: Java](#course-java)
- [Course: Spring](#course-spring)
- [Quizzes](#quizzes)
- [How to answer a quiz](#how-to-answer-a-quiz)
- [Using this material as an interviewer](#using-this-material-as-an-interviewer)
- [Note on Spring Boot 4](#note-on-spring-boot-4)
- [References](#references)

## Setup

Java is not installed on this machine. Install a JDK 17 with SDKMAN:

```bash
curl -s "https://get.sdkman.io" | bash
sdk install java 17.0.12-tem   # run `sdk list java` to see the current 17.x patch
java -version
```

For the Java lessons, you can run a single file directly, without a build tool:

```bash
java Main.java
```

For the Spring lessons, generate a project at https://start.spring.io with Java 17, Maven or Gradle, and these dependencies: Spring Web, Validation, Spring Data JPA, H2 Database, Spring Security, Spring Boot Actuator.

## Schedule

Each lesson takes about one session of 1 hour: 30 to 40 minutes of reading, 20 to 30 minutes on the "Try it" exercise. Each quiz takes one session.

| Week                      | Sessions                                           |
| ------------------------- | -------------------------------------------------- |
| 1 (Sep 30 to Oct 6)       | Java 01, 02, 03, 04, Quiz 01                       |
| 2 (Oct 7 to Oct 13)       | Java 05, 06, 07, Quiz 02, Quiz 03, Quiz 04         |
| 3 (Oct 14 to Oct 20)      | Spring 01, 02, 03, Quiz 05, Quiz 06                |
| 4 (Oct 21 to Oct 27)      | Spring 04, 05, 06, Quiz 07, Quiz 08                |
| Buffer (Oct 28 to Nov 20) | Redo weak topics, then do a mock interview with me |

The buffer is large on purpose. If you fall behind, you still finish before the end of November.

## Course: Java

| #   | Lesson                                                                     | Focus                                                                                      |
| --- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| 01  | [Java for Kotlin developers](course/java/01-java-for-kotlin-developers.md) | Java rules that differ from Kotlin                                                         |
| 02  | [OOP core](course/java/02-oop-core.md)                                     | Classes, inheritance, interfaces, polymorphism, equals and hashCode, records, sealed types |
| 03  | [OOP design](course/java/03-oop-design.md)                                 | Composition, SOLID, common patterns                                                        |
| 04  | [Generics and collections](course/java/04-generics-and-collections.md)     | List, Set, Map, HashMap internals, wildcards                                               |
| 05  | [Lambdas, streams, Optional](course/java/05-lambdas-streams-optional.md)   | Functional style in Java                                                                   |
| 06  | [Exceptions](course/java/06-exceptions.md)                                 | Checked and unchecked, try-with-resources, custom exceptions                               |
| 07  | [Concurrency and the JVM](course/java/07-concurrency-and-jvm.md)           | Threads, executors, thread safety, memory, garbage collection                              |

## Course: Spring

| #   | Lesson                                                                     | Focus                                                 |
| --- | -------------------------------------------------------------------------- | ----------------------------------------------------- |
| 01  | [IoC, DI, and beans](course/spring/01-ioc-di-beans.md)                     | Container, injection, scopes, proxies                 |
| 02  | [Spring Boot essentials](course/spring/02-spring-boot-essentials.md)       | Auto-configuration, starters, configuration, profiles |
| 03  | [REST API best practice](course/spring/03-rest-api-best-practice.md)       | Design, DTOs, validation, errors, pagination, CORS    |
| 04  | [Data JPA and transactions](course/spring/04-data-jpa-and-transactions.md) | Entities, relations, N+1, `@Transactional`            |
| 05  | [Security](course/spring/05-security.md)                                   | Filter chain, JWT, CORS with a React client           |
| 06  | [Testing](course/spring/06-testing.md)                                     | JUnit 5, Mockito, test slices                         |

## Quizzes

The quizzes go from level 1 (basic) to level 8 (advanced). Do each quiz after the lessons it covers.

| Quiz                                                     | Level           | Covers                             |
| -------------------------------------------------------- | --------------- | ---------------------------------- |
| [01](quizzes/quiz-01-java-vs-kotlin.md)                  | Basic           | Java 01                            |
| [02](quizzes/quiz-02-oop-core.md)                        | Basic to medium | Java 02                            |
| [03](quizzes/quiz-03-oop-design-generics-collections.md) | Medium          | Java 03, 04                        |
| [04](quizzes/quiz-04-streams-exceptions-concurrency.md)  | Medium to hard  | Java 05, 06, 07                    |
| [05](quizzes/quiz-05-spring-core-boot.md)                | Medium          | Spring 01, 02                      |
| [06](quizzes/quiz-06-rest-api.md)                        | Medium to hard  | Spring 03                          |
| [07](quizzes/quiz-07-jpa-transactions.md)                | Hard            | Spring 04                          |
| [08](quizzes/quiz-08-security-testing-scenarios.md)      | Advanced        | Spring 05, 06, and mixed scenarios |

## How to answer a quiz

1. Open the quiz file.
2. Write your answer under each `Answer:` line. Code is allowed.
3. Do not look at the lessons while you answer. If you do not know, write "I do not know". That is useful data.
4. Tell me "check quiz 03". I will grade each answer, correct it, and list the topics to review.

## Using this material as an interviewer

You interview interns, so the goal is not only to know the answer. You must also judge a partial answer. For each quiz question, ask yourself three things:

1. What is the minimum answer that an intern must give to pass?
2. Which follow-up question separates a memorized answer from real understanding?
3. Which wrong answer is a red flag?

Quiz 08 contains questions of this type. For interns, quizzes 01 to 03, 05, and 06 are the realistic range. Quizzes 04, 07, and 08 are for you, so that you can follow up with confidence.

## Note on Spring Boot 4

Spring Boot 4.0 was released on 2025-11-30, and 4.1 on 2026-06-10. Open-source support for all 3.x lines ended in June 2026. Your company uses Boot 3, so this course teaches Boot 3. The concepts in this course do not change in Boot 4. The main differences are Spring Framework 7, Jackson 3, JSpecify null-safety annotations, and built-in API versioning. If your team plans a migration, an intern can join that work. Ask your team about the plan.

## References

- Java tutorials from Oracle: https://dev.java/learn/
- Spring Boot reference: https://docs.spring.io/spring-boot/
- Spring guides: https://spring.io/guides
- Spring support policy: https://spring.io/support-policy/
- Spring Boot end-of-life dates: https://endoflife.date/spring-boot
- Spring Boot 4 overview: https://www.danvega.dev/blog/spring-boot-4-is-here
- Effective Java, 3rd edition, by Joshua Bloch (the best single book for OOP in Java)
