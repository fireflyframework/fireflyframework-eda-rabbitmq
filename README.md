# Firefly Framework - EDA - RabbitMQ

[![CI](https://github.com/fireflyframework/fireflyframework-eda-rabbitmq/actions/workflows/ci.yml/badge.svg)](https://github.com/fireflyframework/fireflyframework-eda-rabbitmq/actions/workflows/ci.yml)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Java](https://img.shields.io/badge/Java-21%2B-orange.svg)](https://openjdk.org)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.x-green.svg)](https://spring.io/projects/spring-boot)

> RabbitMQ transport adapter for the Firefly Framework EDA module, providing event publishing and consumption over RabbitMQ via Spring AMQP.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Documentation](#documentation)
- [Contributing](#contributing)
- [License](#license)

## Overview

Firefly Framework EDA RabbitMQ implements the Firefly EDA `EventPublisher` and `EventConsumer` abstractions on top of RabbitMQ using Spring AMQP. It plugs into the core `fireflyframework-eda` module: the core component-scan discovers the `@Component` publisher (`RabbitMqEventPublisher`) and consumer (`RabbitMqEventConsumer`) shipped here, while this module's auto-configurations create the broker `ConnectionFactory`, `RabbitTemplate`, and `AmqpAdmin` beans.

Configuration is hexagonal: the adapter reads EXCLUSIVELY from the `firefly.eda.publishers.rabbitmq.*` and `firefly.eda.consumer.rabbitmq.*` namespaces and never from `spring.rabbitmq.*`. An `EnvironmentPostProcessor` excludes Spring Boot's native `RabbitAutoConfiguration` unless RabbitMQ is actually used, preventing spurious connection/health failures.

**Consumer fix:** the RabbitMQ consumer is now fully functional. The `RabbitMqEventConsumer` is guarded by `@ConditionalOnBean(name = "fireflyEdaRabbitConsumerConnectionFactory")` and injects that same bean via `@Qualifier`, but no auto-configuration previously created it - so the consumer stayed dormant and was never instantiated. This adapter adds `FireflyEdaRabbitMqConsumerAutoConfiguration`, which creates the missing `fireflyEdaRabbitConsumerConnectionFactory` bean (a `CachingConnectionFactory` built from `firefly.eda.consumer.rabbitmq.default.*`), mirroring the publisher-side connection factory. The consumer now wires and runs.

## Features

- RabbitMQ `EventPublisher` implementation (`RabbitMqEventPublisher`) using Spring AMQP
- RabbitMQ `EventConsumer` implementation (`RabbitMqEventConsumer`) with dynamic queue subscription
- Publisher-side `ConnectionFactory`, `RabbitTemplate`, and `AmqpAdmin` auto-configuration
- Consumer-side `ConnectionFactory` auto-configuration (fixes the previously dormant consumer)
- Hexagonal configuration via `firefly.eda.{publishers,consumer}.rabbitmq.*` (never `spring.rabbitmq.*`)
- `EnvironmentPostProcessor` that excludes Spring Boot's `RabbitAutoConfiguration` when RabbitMQ is not in use
- Exchange / routing-key destination support and header propagation
- Health monitoring for both publisher and consumer
- Spring Boot auto-configuration registered via `AutoConfiguration.imports`

## Requirements

- Java 21+
- Spring Boot 3.x
- Maven 3.9+
- RabbitMQ broker instance

## Installation

```xml
<dependency>
    <groupId>org.fireflyframework</groupId>
    <artifactId>fireflyframework-eda-rabbitmq</artifactId>
    <version>26.05.07</version>
</dependency>
```

## Quick Start

```xml
<dependencies>
    <dependency>
        <groupId>org.fireflyframework</groupId>
        <artifactId>fireflyframework-eda</artifactId>
    </dependency>
    <dependency>
        <groupId>org.fireflyframework</groupId>
        <artifactId>fireflyframework-eda-rabbitmq</artifactId>
    </dependency>
</dependencies>
```

## Configuration

```yaml
firefly:
  eda:
    enabled: true
    publishers:
      enabled: true
      rabbitmq:
        default:
          enabled: true
          host: localhost
          port: 5672
          username: guest
          password: guest
          virtual-host: /
          default-exchange: events
    consumer:
      enabled: true
      rabbitmq:
        default:
          enabled: true
          host: localhost
          port: 5672
          username: guest
          password: guest
          virtual-host: /
          queues: events-queue
          concurrent-consumers: 1
          max-concurrent-consumers: 5
          prefetch-count: 10
```

## Documentation

No additional documentation available for this project.

## Contributing

Contributions are welcome. Please read the [CONTRIBUTING.md](CONTRIBUTING.md) guide for details on our code of conduct, development process, and how to submit pull requests.

## License

Copyright 2024-2026 Firefly Software Foundation.

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.
