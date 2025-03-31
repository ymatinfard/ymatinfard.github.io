---
layout: post
title:  "gRPC server-side: Building a Bidirectional messaging"
date:   2025-03-31 11:15:00 GMT+2
categories: kotlin server-side
---
![starting-image](/assets/images/post/grpc_server.png){: width="512" height="512" }

In this article, we will implement the server-side of a gRPC-based chat service using Kotlin. Previously, we built an Android client for text messaging using gRPC. Now, we will set up the server.

## Dependency Setup

Add the following dependencies in your `build.gradle.kts`:

```kotlin
implementation("net.devh:grpc-server-spring-boot-starter:2.15.0.RELEASE")
implementation("io.grpc:grpc-kotlin-stub:1.4.0")
implementation("com.google.protobuf:protobuf-kotlin:4.28.2")
```

Also, add the following plugin to generate the necessary gRPC classes:

```kotlin
id("com.google.protobuf") version "0.9.4"
```

## Configuring Protocol Buffers

Add the following to your `build.gradle.kts`:

```kotlin
protobuf {
    protoc {
        artifact = "com.google.protobuf:protoc:4.28.2"
    }
    plugins {
        create("grpc") {
            artifact = "io.grpc:protoc-gen-grpc-java:1.57.2"
        }
        create("grpckt") {
            artifact = "io.grpc:protoc-gen-grpc-kotlin:1.4.0:jdk8@jar"
        }
    }
    generateProtoTasks {
        all().configureEach {
            plugins {
                create("grpc")
                create("grpckt")
            }
        }
    }
}
```

This configuration ensures that Protocol Buffers generate the required gRPC Kotlin stubs automatically.

## Defining the gRPC Schema

Create a `chat.proto` file in `src/main/proto/`:

```proto
syntax = "proto3";

option java_package = "your_package_name";
option java_multiple_files = true;

message ChatMessage {
  string senderId = 1;
  string receiverId = 2;
  string message = 3;
  string timeStamp = 4;
}

service ChatService {
  rpc ChatStream(stream ChatMessage) returns (stream ChatMessage);
}
```

Run the following command to generate the proto files:

```sh
./gradlew generateProto
```

The generated files will be located in `build/generated/source/proto`.

## Implementing the gRPC Service

Create a Kotlin class for the gRPC server:

```kotlin
class ChatGrpcService : ChatServiceGrpcKt.ChatServiceCoroutineImplBase() {
    override fun chatStream(requests: Flow<ChatMessage>): Flow<ChatMessage> {
        return flow {
            requests.collect { request ->
                println("From client: ${request.message}")

                emit(
                    ChatMessage.newBuilder()
                        .setMessage("Echo: ${request.message}")
                        .build()
                )
            }
        }
    }
}
```

### Explanation:
- `ChatServiceGrpcKt.ChatServiceCoroutineImplBase()`: The base class for implementing a gRPC service with Kotlin coroutines.
- `chatStream`: A bidirectional streaming function that listens to client messages and echoes them back.
- `requests.collect { request -> }`: Collects incoming messages from the client.
- `emit(ChatMessage.newBuilder().setMessage("Echo: ${request.message}").build())`: Sends a response back to the client with an echo of the received message.

## Configuring the Server

Create an `application.yaml` file in the `resources` folder:

```yaml
grpc:
  server:
    port: 9090
```

This sets the gRPC server to run on port `9090`.

## Running the Server

Start your application, and your gRPC server will be ready to receive and respond to client messages.

This setup provides a fully functional gRPC server implementation in Kotlin that can handle bidirectional communication for a chat application.

To see the complete code(Spring boot + grpc + r2dbc), check the GitHub link below:
[https://github.com/ymatinfard/grpcServerSide](https://github.com/ymatinfard/grpcServerSide)


