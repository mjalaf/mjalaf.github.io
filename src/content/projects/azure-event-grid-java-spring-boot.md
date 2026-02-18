---
title: "Azure Event Grid Java Spring Boot Demo"
description: "A comprehensive demonstration of Azure Event Grid integration with Spring Boot, featuring publisher, consumer, topic publisher, and webhook modules."
github: "https://github.com/mjalaf/egdemojavaspringboot"
---

A comprehensive demonstration project showcasing Azure Event Grid integration with Spring Boot applications. This repository contains four modules that illustrate different Event Grid patterns and use cases.

## Project Overview

This project demonstrates how to work with Azure Event Grid using Java and Spring Boot. It includes sample applications for publishing and consuming events through both Event Grid Namespaces and Event Grid Topics, as well as handling webhook events.

---

## Modules

### 1. eventgrid-eventing-publisher

A Spring Boot application that publishes CloudEvents to Azure Event Grid Namespaces. This module demonstrates how to:
- Connect to Azure Event Grid using Azure Key Credential
- Send CloudEvents in batches
- Configure event sources and types

### 2. eventgrid-eventing-consumer

A Spring Boot application that consumes events from Azure Event Grid Namespaces. This module demonstrates how to:
- Subscribe to Event Grid Namespace topics
- Process incoming CloudEvents
- Handle event data in various formats

### 3. eventgrid-topic-publisher

A Spring Boot application that publishes events to Azure Event Grid Topics (Classic). This module demonstrates:
- Publishing events to Event Grid Topics
- Working with custom event schemas
- Batch event publishing

### 4. eventgrid-webhook

A Spring Boot web application that serves as a webhook endpoint for Event Grid. This module demonstrates:
- Handling Event Grid subscription validation
- Processing incoming webhook events
- Implementing webhook security best practices

---

## Prerequisites

- Java 17 or higher
- Maven 3.6+
- Azure subscription with Event Grid resources
- Azure Event Grid Namespace (for eventing modules)
- Azure Event Grid Topic (for topic publisher)

## Technology Stack

- **Spring Boot**: 3.3.2
- **Spring Cloud Azure**: 5.14.0
- **Azure Event Grid SDK**: 4.23.0
- **Azure Event Grid Namespaces SDK**: 1.0.0
- **Java**: 17

---

## Setup Instructions

### 1. Clone the repository

```bash
git clone https://github.com/mjalaf/egdemojavaspringboot.git
cd egdemojavaspringboot
```

### 2. Configure Azure credentials

For each module, update the configuration with your Azure Event Grid credentials:

#### eventgrid-eventing-publisher and eventgrid-eventing-consumer

Edit the Java files and replace the placeholders:

```java
public static final String ENDPOINT = "ADD-YOUR-ENDPOINT-HERE";
private static final String TOPIC_NAME = "ADD-YOUR-TOPIC-NAME-HERE";
public static final AzureKeyCredential CREDENTIAL = new AzureKeyCredential("ADD-YOUR-KEY-HERE");
```

#### eventgrid-topic-publisher

Update the configuration with your Event Grid Topic endpoint and access key.

#### eventgrid-webhook

Configure your webhook endpoint URL in Azure Event Grid subscription settings.

### 3. Build the projects

```bash
# Build all modules
mvn clean install

# Or build individual modules
cd eventgrid-eventing-publisher
mvn clean install
```

---

## Usage

### Running the Event Grid Publisher

```bash
cd eventgrid-eventing-publisher
mvn spring-boot:run
```

### Running the Event Grid Consumer

```bash
cd eventgrid-eventing-consumer
mvn spring-boot:run
```

### Running the Topic Publisher

```bash
cd eventgrid-topic-publisher
mvn spring-boot:run
```

### Running the Webhook Application

```bash
cd eventgrid-webhook
mvn spring-boot:run
```

The webhook will be available at `http://localhost:8080/webhook`

---

## Azure Event Grid Setup

1. **Create an Event Grid Namespace** (for eventing modules):
   - In the Azure Portal, create a new Event Grid Namespace
   - Create a topic within the namespace
   - Note the endpoint URL and access keys

2. **Create an Event Grid Topic** (for topic publisher):
   - Create a custom Event Grid Topic
   - Note the topic endpoint and access key

3. **Configure Event Subscriptions**:
   - For webhook testing, create an Event Grid subscription pointing to your webhook endpoint
   - Azure will send a validation request that the webhook handles automatically

---

## Resources

- [Azure Event Grid Documentation](https://docs.microsoft.com/en-us/azure/event-grid/)
- [Spring Cloud Azure Documentation](https://spring.io/projects/spring-cloud-azure)
- [Azure Event Grid Java SDK](https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/eventgrid)

---

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the Apache License 2.0 - see the [LICENSE](https://github.com/mjalaf/egdemojavaspringboot/blob/main/LICENSE) file for details.
