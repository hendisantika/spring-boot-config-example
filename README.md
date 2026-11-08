# spring-boot-config-example

[![Java CI with Maven](https://github.com/hendisantika/spring-boot-config-example/actions/workflows/maven.yml/badge.svg)](https://github.com/hendisantika/spring-boot-config-example/actions/workflows/maven.yml)
![Java](https://img.shields.io/badge/Java-25-orange?logo=openjdk)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.1.2-6DB33F?logo=springboot)
![Spring Cloud](https://img.shields.io/badge/Spring%20Cloud-2025.1.4-6DB33F?logo=spring)

Spring Boot Config Example: a small Spring Cloud setup with a centralized **Config Server**, a **Eureka** service
registry, and a **client service** that reads its configuration from the Config Server.

## Architecture

```
                 ┌──────────────────────┐
                 │   eureka-server      │  :8761
                 │  (service registry)  │
                 └──────────▲───────────┘
                            │ register
          ┌─────────────────┴──────────────────┐
          │                                    │
┌─────────┴────────────────┐        ┌──────────┴─────────┐
│ spring-boot-config-server│◄───────│   simple-service   │  :8080
│   (native config repo)   │ config │  (config client)   │
└──────────────────────────┘ :8888  └────────────────────┘
```

| Module                      | Description                                                                                               | Port |
|-----------------------------|-----------------------------------------------------------------------------------------------------------|------|
| `eureka-server`             | Netflix Eureka service registry                                                                           | 8761 |
| `spring-boot-config-server` | Spring Cloud Config Server using the `native` profile, serving files from `classpath:/config/simple-service/` | 8888 |
| `simple-service`            | REST client service that loads its config (`dev`, `prod` profiles) from the Config Server                 | 8080 |

## Tech Stack

- Java 25 (Temurin)
- Spring Boot 4.1.2
- Spring Cloud 2025.1.4 (Config Server/Client, Netflix Eureka, Bootstrap)
- Maven (via Maven Wrapper in every module)

## Prerequisites

- JDK 25
- No local Maven install needed: each module ships with `./mvnw`

## Build

Each module is a standalone Maven project:

```shell
cd eureka-server && ./mvnw clean verify && cd ..
cd spring-boot-config-server && ./mvnw clean verify && cd ..
cd simple-service && ./mvnw clean verify && cd ..
```

## Run

Start the applications in this order, each in its own terminal:

```shell
# 1. Service registry
cd eureka-server && ./mvnw spring-boot:run

# 2. Config server
cd spring-boot-config-server && ./mvnw spring-boot:run

# 3. Client service
cd simple-service && ./mvnw spring-boot:run
```

## Endpoints

| URL                                     | Description                                         |
|-----------------------------------------|-----------------------------------------------------|
| http://localhost:8761                   | Eureka dashboard                                    |
| http://localhost:8888/simple-service/dev  | Config for `simple-service` with the `dev` profile  |
| http://localhost:8888/simple-service/prod | Config for `simple-service` with the `prod` profile |
| http://localhost:8080/                  | `simple-service` endpoint returning the `my.prop` value |
| `POST` http://localhost:8080/refresh    | Reload configuration (`@RefreshScope`) via Actuator |

Example of refreshing configuration after editing a file under
`spring-boot-config-server/src/main/resources/config/simple-service/`:

```shell
curl -X POST http://localhost:8080/refresh
```

> **Note:** the `@Value("${my.prop}")` annotation in `SimpleServiceApplication` is currently commented out, so
> `GET /` returns `Prop value: null` until it is enabled.

## CI/CD & Dependency Management

- **GitHub Actions** ([`maven.yml`](.github/workflows/maven.yml)): builds and tests every module with JDK 25 in a
  matrix job, and submits each module's dependency graph to GitHub on pushes to `main`.
- **Dependabot** ([`dependabot.yml`](.github/dependabot.yml)):
  - Maven: daily checks for all modules, with Spring Boot/Spring Cloud updates and minor/patch updates grouped.
  - GitHub Actions: weekly grouped updates.

## License

This project is licensed under the [MIT License](LICENSE).
