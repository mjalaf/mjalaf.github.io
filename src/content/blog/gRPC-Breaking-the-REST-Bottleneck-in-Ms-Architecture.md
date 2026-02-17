---
title: 'gRPC: Breaking the REST Bottleneck in Microservices Architecture'
description: 'Why the shift to Protocol Buffers and HTTP/2 is redefining inter-service communication performance.'
pubDate: 'August 22 2020'
tags:
  - english
  - architecture
  - grpc
  - microservices
  - performance
---
Since its announcement by Google in 2015, **gRPC** (Remote Procedure Call) has evolved from a niche internal tool to a cornerstone of modern cloud-native architecture. As we move deeper into 2020, the limitations of traditional REST over HTTP/1.1—text-based payloads, head-of-line blocking, and lack of strong typing—are becoming significant bottlenecks for our high-scale microservices.

gRPC isn't just a new library; it's a paradigm shift in how we think about service contracts and binary communication.

---

## The Problem: The "Text-Tax" of REST

In a distributed system with hundreds of internal calls, the overhead of REST starts to compound:
* **Payload Size:** JSON is human-readable but bulky. Sending the same field names repeatedly in every message wastes bandwidth.
* **Serialization Overhead:** Parsing and stringifying JSON is CPU-intensive at high volumes.
* **Lack of Formal Contract:** In REST, the "contract" is often a separate Swagger file that can easily get out of sync with the actual code.
* **HTTP/1.1 Limitations:** Sequential requests and the lack of multiplexing lead to increased latency.

---

## The Solution: Protocol Buffers and HTTP/2

gRPC solves these issues by combining two powerful technologies:

### 1. Protocol Buffers (Protobuf)
Instead of text, gRPC uses **Protobuf**, a binary serialization format. You define your data structure once in a `.proto` file, and gRPC generates strongly-typed code in multiple languages. It is smaller, faster, and simpler than XML or JSON.

### 2. HTTP/2 Transport
By running on HTTP/2, gRPC supports:
* **Binary Framing:** More efficient parsing.
* **Multiplexing:** Multiple requests over a single TCP connection.
* **Bidirectional Streaming:** The client and server can send a stream of messages simultaneously.

---

## Architecture Diagram

The beauty of gRPC is the "stub" system, which makes a remote call feel like a local function call.

---

## How to use it: The Workflow

As an Architect, I recommend this workflow to ensure consistency across teams:

1.  **Define the Service Contract:** Write your service and message definitions in a `.proto` file.

    ```protobuf
    syntax = "proto3";

    service CustomerService {
      rpc GetCustomer (CustomerRequest) returns (CustomerResponse);
    }

    message CustomerRequest {
      int32 id = 1;
    }

    message CustomerResponse {
      string name = 1;
      string email = 2;
    }
    ```

### Understanding Field Tags: Why name = 1?
In a JSON payload, we send the property names as strings (e.g., "name": "Martin"), which consumes unnecessary bytes in every request. In gRPC, the .proto file acts as a shared blueprint between the client and the server.

The numbers you see (like = 1, = 2) are Field Tags. These are unique identifiers used in the binary message instead of the field names.

### How it works during transmission:

Efficiency: When gRPC sends a message, it doesn't send the word "name". It sends a tiny binary header that says: "The following data belongs to Tag #1".

Space Saving: Tags from 1 to 15 take only 1 byte to encode. This is why we assign the most frequently used fields to the lowest numbers.

Backward Compatibility: Because the identification is based on the tag number and not the name, you can rename a field in your code without breaking existing clients, as long as the Tag Number stays the same.

Architectural Rule: Never reassign or change a tag number once it's in production. If a field is deprecated, mark it as reserved to prevent future reuse and ensure system stability.


2.  **Generate Code:** Use the `protoc` compiler to generate client stubs and server skeletons in your preferred language (C#, Go, Python, etc.).
3.  **Implement Logic:** The developer only needs to implement the logic on the server and call the method on the client. The "plumbing" is handled by gRPC.

---
## Architectural Insight: Where gRPC fits

While gRPC is incredibly fast, it isn't a "REST-killer" for everything. 
* **Internal Communication:** gRPC is the clear winner for inter-service communication (East-West traffic) due to its performance and strong typing.
* **Public APIs:** REST (North-South traffic) is still preferred for external consumers because of its ubiquity and browser support.

In our current **Azure** environments, we are seeing fantastic results using gRPC on **Azure Kubernetes Service (AKS)** and through **Azure App Service** (which recently added support). By reducing the serialization overhead, we are not only lowering latency but also reducing the compute costs of our clusters.

---

## Summary

The transition to gRPC represents a move toward more disciplined, contract-first engineering. By using Protocol Buffers as our source of truth, we eliminate a whole class of integration bugs and unlock a level of performance that JSON simply cannot match.

---

## Deep Dive: Reference Material & Implementation Links

To deepen your understanding of gRPC and Protocol Buffers, explore these resources:

### Documentation & Standards
* **Official gRPC Documentation:** [grpc.io](https://grpc.io/docs/)
* **Protocol Buffers Language Guide:** [developers.google.com/protocol-buffers](https://developers.google.com/protocol-buffers/docs/proto3)
* **HTTP/2 Specification:** [rfc7540](https://httpwg.org/specs/rfc7540.html)

### Azure & Microsoft Implementation
* **gRPC services with ASP.NET Core:** [Microsoft Docs](https://docs.microsoft.com/en-us/aspnet/core/grpc/aspnetcore)

* **Performance Comparison:** [Microsoft's Benchmarks](https://docs.microsoft.com/en-us/aspnet/core/grpc/comparison)